# IOC 01

一个对象依赖另一个对象，起初的时候我们会把这个东西写死，我依赖a那我在构造的时候就得把a实际的赋给我，也就是：

```java
// 背景结构
interface InterfaceB {
  xxx();
}

class ObjectB implements InterfaceB {
  xxx(){};
  
}
class ObjectB2 implements InterfaceB {
  xxx(){};
}

interface InterfaceC {
  yyy();
}

class ObjectC implements InterfaceC {
  yyy(){};
}
class ObjectC2 implements InterfaceC {
  yyy(){};
}

class ObjectA {
  InterfaceB b = new ObjectB;
  InterfaceC c = new ObjectC;
}

// 运行
public static void main(String[] args) {
  ObjectA a = new ObjectA();
  a.b.xxx();
  a.c.yyy();
}
```

这种写法，我们在ObjectA的定义中已经固定了InterfaceB\C的实现类是ObjectB\C，我们实际使用的时候直接初始化A，然后调用即可。

但这个时候，如果我们以后对于InterfaceB我们不想用ObjectB这个实现类，而想换一个实现比如ObjectB2，我们则需要修改代码，或者再写一个ObjectA，例如：

```java
// 1.修改老的代码，将b的实现类修改成ObjectB2
class ObjectA {
  private InterfaceB b = new ObjectB2;
  private InterfaceC c = new ObjectC;
}

// 2.添加一个ObjectA2，其中b的实现类时ObjectB2
class ObjectA2 {
  private InterfaceB b = new ObjectB2;
  private InterfaceC c = new ObjectC;
}
```

其中第二种方式，增加了代码的冗余度，需要写很多重复代码，之后如果有更过的实现类，就需要继续写更多的实现A这很不符合实际的使用体验。

而第一种就更不合适了，因为我的代码中可能会想要有不同的实现，那直接修改则无法满足要求了。

这个时候我们就需要IOC了。

IOC全称：Inversion of Control，控制反转，可以简单的理解为将控制权反转给别人。

将控制权反转给别人，是什么意思呢？在上面的案例里，我们的ObJectA依赖InterfaceB\C，我们要为ObjectA设置InterfaceB\C的实现类，简单说就是我们需要将InterfaceB\C的实现类赋值给ObjectA的两个变量b\c，这一步在我们上面的案例里是在类ObjectA的声明的时候就已经固定了的，**那么控制b\c变量的实际内容是什么的是ObjectA类本身**，我们在代码还没运行起来的时候我们就知道b\c实际是哪个实现类，不管谁初始化ObjectA都一样。

而控制反转就是我们要将控制b\c变量的实际内容是什么的权利反转给别人。两个问题：一、反转给谁？二、怎么反转？

1、反转给谁？

在上面我们一直在说，如果我们想要的InterfaceB\C的实现类不是ObjectB\C而是ObjectB2\C2或是别的，这里面一直有一个“我们”，也就是说实际知道想用什么的是“我们”，也就是ObjectA的调用方，那么很自然的，我们也就可以将决定InterfaceB\C的实现的权利给到“我们”就可以了。

2、怎么反转？

实际的反转其实也很简单，在上面的代码案例中，我们在声明ObjectA的时候，直接将InterfaceB\C的实现写死在了代码中，如果我们可以不写死而是把这两个变量的内容当做一种方法的入参，在实际初始化ObjectA，或是调用A的时候，才传进来的话，那不就可以实现动态的指定InterfaceB\C了吗，例如：

```java
// 1.构造函数注入
class ObjectA {
  private InterfaceB b;
  private InterfaceC c;
  
  public ObjectA() {}
  
  public ObjectA(InterfaceB b, InterfaceC c) {
    this.b = b;
    this.c = c;
  }
}

public static void main(String[] args) {
	// 初始化
  InterfaceB b = new ObjectB2();
  InterfaceC c = new ObjectC();
  ObjectA a = new ObjectA(b, c);
  // 调用
  a.b.xxx();
  a.c.yyy();
}

// 2.setter方法注入
class ObjectA {
  private InterfaceB b;
  private InterfaceC c;
  
  public void setB(InterfaceB b) {
    this.b = b;
  }
  
   public void setC(InterfaceC c) {
    this.c = c;
  }
}

public static void main(String[] args) {
	// 初始化
  InterfaceB b = new ObjectB();
  InterfaceC c = new ObjectC2();
  ObjectA a = new ObjectA();
  a.setB(b);
  a.setC(c);
  // 调用
  a.b.xxx();
  a.c.yyy();
}

// 3.接口注入
// 较复杂且实际使用并没有什么优势，已逐渐淘汰
```

从上面的两种常用反转方式中也可以看出，其实就是把变量当成方法的参数在初始化或初始化好A之后传递进来，而不是一开始就写死的区别。这就是控制反转。**就这么简单！**

到了这里也就可以理解另一个概念，**依赖注入** DI(Dependency Injection)也就是你依赖的东西需要被注入进来，而不是在一开始就写死。

总结：

其实看下来就会发现就是一个很简单的逻辑：本来在代码中需要写出所有的可能性，而现在将这些可能性抽成了一个参数，由调用者来手动填写。

举个例子：

旅行机构提供了从A地到B地的几种不同的交通方式，包括：飞机、高铁、大巴、船运

现在有一批人需要从A地到B地去，这时旅行机构需要收集人想要选择哪种交通方式，然后给他们实际的分配。有两种方式：一种是准备四种单据，单据内容是：我想要乘坐飞机去；我想要乘坐高铁去；等由用户选择其一。另一种是只打印一种卡片：卡片内容是 我想要乘坐__去 的字样，而实际内容由用户自己来填写。

这两种方式很显然第二种的方式要更加自由一些，比如如果以后我们还提供了搭火箭去，或是骑自行车去或是步行去等等的话，我们都不需要修改卡片格式，因为实际的方式是由用户来填写的，如果是第一种，我们就需要继续增发卡片样式。