# mybatis使用记录

- `_parameter`: 表示传入的参数，如果传入的参数只有一个，那`_parameter`就指这个参数，如果传入多个参数，则参数会被封装成一个map，`_parameter`指代这个map，使用get(0)等来获取参数

```xml
<select id="selectByExample" parameterType="com.macro.mall.model.PmsProductExample" resultMap="BaseResultMap">
    select
    <if test="distinct">
        distinct
    </if>
    <include refid="Base_Column_List" />
    	from pms_product
    <if test="_parameter != null">
        <include refid="Example_Where_Clause" />
    </if>
    <if test="orderByClause != null">
        order by ${orderByClause}
    </if>
</select>
```

其中的`<if test="_parameter != null">`指代传过来的PmsProductExample，此句意：**为若PmsProductExample不为空则执行其下语句**

- 分析一段mapper

```xml
<sql id="Example_Where_Clause">
    <where>
        <foreach collection="oredCriteria" item="criteria" separator="or">
            <if test="criteria.valid">
                <trim prefix="(" prefixOverrides="and" suffix=")">
                    <foreach collection="criteria.criteria" item="criterion">
                        <choose>
                            <when test="criterion.noValue">
                                and ${criterion.condition}
                            </when>
                            <when test="criterion.singleValue">
                                and ${criterion.condition} #{criterion.value}
                            </when>
                            <when test="criterion.betweenValue">
                                and ${criterion.condition} #{criterion.value} and #{criterion.secondValue}
                            </when>
                            <when test="criterion.listValue">
                                and ${criterion.condition}
                                <foreach close=")" collection="criterion.value" item="listItem" open="(" separator=",">
                                    #{listItem}
                                </foreach>
                            </when>
                        </choose>
                    </foreach>
                </trim>
            </if>
        </foreach>
    </where>
</sql>
```

1. **sql**

	- 一段可以复用的sql语句
	- 由唯一id指定，复用时使用\<include>标志，加上唯一id。 e.g.：`<include refid="Example_Where_Clause" />`

2. **if**

3. **where**

	- 其下往往会有许多判断类语句比如\<if ...>，如果if不成立则会导致语句不符合语法，比如：

	```xml
	<select id="findActiveBlogLike" resultType="Blog">
	    SELECT * FROM BLOG 
	    WHERE 
	    <if test="state != null">
	        state = #{state}
	    </if> 
	    <if test="title != null">
	        AND title like #{title}
	    </if>
	    <if test="author != null and author.name != null">
	        AND author_name like #{author.name}
	    </if>
	</select>
	```

	如果以上if判断没有一个成立，那么sql最终会蜕变成：

	```sql
	select * from blog
	where
	```

	如果以上只有第二个if成立，则会变成：

	```sql
	select * from blog
	where
	and title like #{title}
	```

	这些很明显都不符合sql的语法，为了解决这个问题mybatis提出了一个方法：where元素

	- where 元素当只有在一个以上的if条件有值的情况下才会去插入“WHERE”子句
	- 并且，若最后句子中剩下的内容以`and或or`开头，where 元素会智能的去除掉它
	- 最后，还提供了一个`trim`元素来定制where元素的功能

4. 