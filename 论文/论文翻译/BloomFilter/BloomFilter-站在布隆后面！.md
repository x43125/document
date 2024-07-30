# BloomFilter - 站在布隆后面！

## 大纲

1. 站在布隆后面
2. 介绍
3. hash结构的 不可能三角
4. 实现
5. 错误率计算公式 -> 各个参数的选择（k,n,m）
6. 优缺点
7. 改良
8. 使用场景



## 一、站在布隆后面

![布隆](/Users/wangxiang/Downloads/布隆.png)

首先让我用一个游戏角色及他的一句对白来开始今天的分享，这是游戏《英雄联盟》中的一名英雄--布隆，他在游戏中的一句经典对白就是：“站在布隆后面”，玩家可以使用这名英雄举起他的巨大盾牌站在队友面前保护队友免受敌方的伤害。而我们今天要介绍的内容也是和他这一特点有一定相似之处的 -- 布隆过滤器

## 二、布隆过滤器

布隆过滤器是一位名叫布隆（BURTON H. BLOOM）的大兄弟于1970年发布的论文上提到的，文章中提出了两种hash结构来讨论传统的hash结构需要占用大量空间的问题。

### 2.1 Hash

说到此处，让我们先来回顾一下朝夕相处的Hash结构，hash结构通过函数映射实现了常数级时间复杂度的查询速度，即传一个值过来通过函数计算得出他在表中应该存储的位置。查询的时候类似，使用函数计算得到存储位置，去到指定位置检索即可。在这些过程中如果发生了hash冲突，也可以通过再散列法、链表法等解决冲突，是一种100%正确的结构，对于传进来的值可以绝对的确定其存在与否。

> 值 -> hashcode -> f(x) = y -> index

那这种结构不是已经很好了吗，为什么还需要新的结构？

Hash结构的缺点或者不足：

  1、范围查询支持不是很好：因为其存储的值是通过函数映射得到的，所以每一个值在表中的位置都得经过计算才能得到，因此范围查询效率不是很高（在mysql数据库引擎中对应的就有B+树的索引结构来优化这一点）；

  2、空间占用较大：因为会发生hash冲突，不同的值可能被散列到同一处，为了绝对正确的判断，需要存储实际值用于比较。对于需要查询实际值的问题这样存无可厚非，但针对那些只需要判断传入值是否存在的问题来说，占用的空间就有点大了，布隆过滤器即是对这一点做的改进。

### 2.2 布隆过滤器

布隆过滤器想必大家即使没有使用过，也不会太陌生。通常来说，我们的很多操作其实就是在空间换时间，时间换空间，或者数学公式（类似hash结构即属于这种），而布隆过滤器又加入了一种新的属性：**正确率**。布隆过滤器认为，如果可以适当的放弃一部分正确率，则可以在保留时间复杂度的基础上，极大的减少空间的使用。关于正确率的具体的内容让我们后面在慢慢讨论。

![img](https://p-blog.csdn.net/images/p_blog_csdn_net/jiaomeng/275417/o_bf1.jpg)

布隆过滤器：给定一个初始全为0的位数组，将加入的值通过多次hash散列得到多个下标，将数组中对应下标位置的值设置为1。

判断时，对传入的值依然使用这些散列函数得到多个下标，如果数组中对应的所有位置中有**任意一个元素为0**，则表示该值**肯定不存在**，如果所有位置的值都为1则表示**可能**存在该值，是否真的存在，需要进一步判断。

如下图中，x1、x2被多次散列到了如图的几个位置，由于存在hash冲突，x1,x2有部分点位共用。

![img](https://p-blog.csdn.net/images/p_blog_csdn_net/jiaomeng/275417/o_bf2.jpg)

此时判断数组中是否存在y1、y2两值，经过计算后得到如图映射，则y1的所有映射中有0值存在，则表示必然不存在y1；而y2的每一位都是1，这样只能代表他可能存在，因为他映射的几个点刚好被x2已经映射过了。

![img](https://p-blog.csdn.net/images/p_blog_csdn_net/jiaomeng/275417/o_bf3.jpg)

在这个结构里面与传统hash比较，特别的地方就是存储的值，布隆过滤器存储的是0，1代表该位是否被映射。只存储0，1那么每位只有1bit，这样比普通直接存值可以少上很多的空间。

不妨计算一下两者的存储占比，平均一个值20个字符，也就是40个字节，320位，也就是1位的**320倍**。而实际使用中存储的值往往不止20个字符，则可以节省更多的空间。

## 三、实现

知道了结构那么实现起来也就很简单了

> guava的布隆过滤器实现

```java
class BloomFilter {
    private final LockFreeBitArray bits;
    private final int numHashFunctions;
    private final Strategy strategy;

    ......
}

interface Strategy {
    /**
    * 存入一个值，经过多次散列映射到多个点位
    */
    void put(T object, Funnel<? super T> funnel, int numHashFunctions, LockFreeBitArray bits);

    /**
    * 判断传进来的值的多个点位是否均为1
    * 如果有任意位==0，则返回false，如果均==1则返回true
    */
    boolean mightContain(T object, Funnel<? super T> funnel, int numHashFunctions, LockFreeBitArray bits);
}

class Strategy01 implements Strategy {
    public <T extends @Nullable Object> boolean put(
        @ParametricNullness T object,
        Funnel<? super T> funnel,
        int numHashFunctions,
        LockFreeBitArray bits) {
        long bitSize = bits.bitSize();
        long hash64 = Hashing.murmur3_128().hashObject(object, funnel).asLong();
        int hash1 = (int) hash64;
        int hash2 = (int) (hash64 >>> 32);

        boolean bitsChanged = false;
        for (int i = 1; i <= numHashFunctions; i++) {
            int combinedHash = hash1 + (i * hash2);
            // Flip all the bits if it's negative (guaranteed positive number)
            if (combinedHash < 0) {
                combinedHash = ~combinedHash;
            }
            bitsChanged |= bits.set(combinedHash % bitSize);
        }
        return bitsChanged;
    }
  
    public <T> boolean mightContain(@ParametricNullness T object, Funnel<? super T> funnel, int numHashFunctions, LockFreeBitArray bits) {
        long bitSize = bits.bitSize();
        long hash64 = Hashing.murmur3_128().hashObject(object, funnel).asLong();
        int hash1 = (int)hash64;
        int hash2 = (int)(hash64 >>> 32);

        for(int i = 1; i <= numHashFunctions; ++i) {
            int combinedHash = hash1 + i * hash2;
            if (combinedHash < 0) {
                combinedHash = ~combinedHash;
            }

            if (!bits.get((long)combinedHash % bitSize)) {
                return false;
            }
        }

        return true;
    }
}

```

利用redis可以实现出分布式的布隆过滤器，逻辑基本想通，只是将存储值的位置挪到redis中。也可以使用redis官方推荐的RedisBloom模块，或是Redisson中的实现。总体结构大差不差。

## 四、误判率

看完了实现，现在让我们来聊一些理论上的东西 -- 误判率

误判率又叫假阳性率(false-positive)，即指实际不属于某个集合的元素中，被误认为属于该集合的比例，在布隆过滤器中就是指被布隆过滤器错误地识别为已添加到过滤器中的元素的概率。

让我们来简单的推导一下：

假设：初始化的位数组长度m，k个hash函数，n个插入值，那么：
$$
P(一次hash选中了某位，值改为1) = \frac{1}{m}
$$

$$
P(一次hash某位未被选中，值仍为0) = 1 - \frac{1}{m}
$$

$$
P(经过k次hash散列某位未被选中，值仍为0) = (1-\frac{1}{m})^k
$$

$$
P(插入n个数后某位仍未被选中，值仍为0) = (1-\frac{1}{m})^{kn}
$$

$$
P(插入n数后某位被选中，值为1) = 1 - (1-\frac{1}{m})^{kn}
$$

$$
P(查询一个新数命中某k位) = P(某k位数均为1) = (1 - (1-\frac{1}{m})^{kn})^k
$$

根据高等数学中的e的近似函数计算：
$$
\lim_{n\rightarrow+\infty}{(1-\frac{1}{x})}^{-x} \approx e
$$
所以将等式(4)可做如下变化：
$$
(1-\frac{1}{m})^{kn} = (1-\frac{1}{m})^{-m \frac{-kn}{m}} \approx e^{-\frac{kn}{m}}
$$
所以等式(6)等于：
$$
P(查询一个新数命中某k位) = P(某k位数均为1) = (1 - (1-\frac{1}{m})^{kn})^k \approx (1-e^{-\frac{kn}{m}})^k
$$
此即为布隆过滤器的误判率，由此可见与k, m, n三个参数相关，也可大致判断出：当m越大时误判率越小，当n越大时误判率越高，而k则需要特别讨论。

我们先从直觉上来判断一下：

当k越大时，括号内部的值越大，总体也就越大。也就是当hash函数过多的时候，一个值占用的位空间过多，因此容易引起冲突误判。

而当k越小趋向1的时候，多个值只通过很少的函数散列，那本身就很容易产生冲突误判，因此k存在一个最优值。

对误判率公式(9)进行求导，得到最佳的 k 使得误判率最小
$$
k = \frac{m}{n}ln2
$$
此公式表示当m、n已经确定之后，k应约等于 m/n 的 ln2倍

**最终得到：误判率公式(9),最佳k个数公式(10)**

这是一个可视化布隆过滤器各参数效果的网站：https://hur.st/bloomfilter/

![image-20240731012707923](/Users/wangxiang/Library/Application Support/typora-user-images/image-20240731012707923.png)

## 五、优缺点

### 5.1 优缺点



### 5.2 改良



## 六、使用场景

