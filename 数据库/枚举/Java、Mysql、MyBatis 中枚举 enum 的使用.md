

[Java](https://yulaiz.com/category/aside/code/java/) · [SQL](https://yulaiz.com/category/aside/code/sql/) / 更新于 2020年3月30日 / [2 条评论](https://yulaiz.com/java-mysql-enum/#comments) / 12,998 次浏览 / [YuLaiZ](https://yulaiz.com/author/yulai/)

![](media/java-mysql-enum-banner.jpg)

**转载请注明，** 原文地址：[Java、Mysql、MyBatis 中枚举 enum 的使用](https://yulaiz.com/java-mysql-enum/)

Java 和 MySql 中都有枚举的概念，合理的使用枚举，可以让代码阅读和数据库数据查询更加直观、高效。那么我们怎么使用呢，什么时候使用，两者之间怎么进行数据关联呢？（本文使用 MyBatis 做为 Java 与 MySql 之间的关联）

文章目录 [[hide](https://yulaiz.com/java-mysql-enum/#)]

-   [1 1. 当年我们怎么定义状态](https://yulaiz.com/java-mysql-enum/#1)
    -   [1.1 1.1 数据库设计](https://yulaiz.com/java-mysql-enum/#11)
    -   [1.2 1.2 Java Bean 代码](https://yulaiz.com/java-mysql-enum/#12_Java_Bean)
    -   [1.3 1.3 MyBatis 代码](https://yulaiz.com/java-mysql-enum/#13_MyBatis)
    -   [1.4 1.4 代码效果](https://yulaiz.com/java-mysql-enum/#14)
-   [2 2. 现在我们怎么定义状态](https://yulaiz.com/java-mysql-enum/#2)
    -   [2.1 2.1 Java Bean 代码](https://yulaiz.com/java-mysql-enum/#21_Java_Bean)
    -   [2.2 2.2 数据库设计](https://yulaiz.com/java-mysql-enum/#22)
    -   [2.3 2.3 MyBatis 代码](https://yulaiz.com/java-mysql-enum/#23_MyBatis)
    -   [2.4 2.4 代码效果](https://yulaiz.com/java-mysql-enum/#24)
-   [3 3. 怎么把当年定义的状态改成使用枚举](https://yulaiz.com/java-mysql-enum/#3)
    -   [3.1 3.1 数据库设计](https://yulaiz.com/java-mysql-enum/#31)
    -   [3.2 3.2 Java Bean 代码](https://yulaiz.com/java-mysql-enum/#32_Java_Bean)
    -   [3.3 3.3 MyBatis 代码](https://yulaiz.com/java-mysql-enum/#33_MyBatis)
        -   [3.3.1 3.3.1 内置枚举转换器](https://yulaiz.com/java-mysql-enum/#331)
            -   [3.3.1.1 3.3.1.1 EnumTypeHandler](https://yulaiz.com/java-mysql-enum/#3311__EnumTypeHandler)
            -   [3.3.1.2 3.3.1.2 EnumOrdinalTypeHandler](https://yulaiz.com/java-mysql-enum/#3312__EnumOrdinalTypeHandler)
        -   [3.3.2 3.3.2 自定义枚举转换器](https://yulaiz.com/java-mysql-enum/#332)
        -   [3.3.3 3.3.3 在 Mapper.xml 中配置自定义的枚举转换器](https://yulaiz.com/java-mysql-enum/#333__Mapperxml)
            -   [3.3.3.1 方法1：修改指定xml文件，指定的Mapper生效](https://yulaiz.com/java-mysql-enum/#1xmlMapper)
            -   [3.3.3.2 方法2：全局指定枚举转换器，无需单独修改Mapper.xml](https://yulaiz.com/java-mysql-enum/#2Mapperxml)
            -   [3.3.3.3 方法3：不使用枚举转换器](https://yulaiz.com/java-mysql-enum/#3-2)
-   [4 4. 枚举到底好不好用](https://yulaiz.com/java-mysql-enum/#4)
    -   [4.1 4.1 Java 枚举的实现](https://yulaiz.com/java-mysql-enum/#41_Java)
        -   [4.1.1 4.1.1 基础声明、属性、构造函数](https://yulaiz.com/java-mysql-enum/#411)
        -   [4.1.2 4.1.2 常用方法](https://yulaiz.com/java-mysql-enum/#412)
            -   [4.1.2.1 4.1.2.1 name() 方法](https://yulaiz.com/java-mysql-enum/#4121__name)
            -   [4.1.2.2 4.1.2.2 ordinal() 方法。](https://yulaiz.com/java-mysql-enum/#4122__ordinal)
            -   [4.1.2.3 4.1.2.3 toString() 方法](https://yulaiz.com/java-mysql-enum/#4123__toString)
            -   [4.1.2.4 4.1.2.4 equals(Object other) 方法](https://yulaiz.com/java-mysql-enum/#4124__equalsObject_other)
            -   [4.1.2.5 4.1.2.5 compareTo(E o) 方法](https://yulaiz.com/java-mysql-enum/#4125__compareToE_o)
            -   [4.1.2.6 4.1.2.6 values() 方法](https://yulaiz.com/java-mysql-enum/#4126__values)
            -   [4.1.2.7 4.1.2.7 valueOf(String name) 方法](https://yulaiz.com/java-mysql-enum/#4127__valueOfString_name)
        -   [4.1.3 4.1.3 比较两个枚举值是否一致](https://yulaiz.com/java-mysql-enum/#413)
    -   [4.2 4.2 MySql 中的枚举](https://yulaiz.com/java-mysql-enum/#42_MySql)
        -   [4.2.1 4.2.1 DDL 操作效率](https://yulaiz.com/java-mysql-enum/#421_DDL)
            -   [4.2.1.1 4.2.1.1 新增一个枚举状态](https://yulaiz.com/java-mysql-enum/#4211)
            -   [4.2.1.2 4.2.1.2 修改一个枚举状态](https://yulaiz.com/java-mysql-enum/#4212)
            -   [4.2.1.3 4.2.1.3 删除一个枚举状态](https://yulaiz.com/java-mysql-enum/#4213)
        -   [4.2.2 4.2.2 查询效率](https://yulaiz.com/java-mysql-enum/#422)
-   [5 5. 枚举使用总结](https://yulaiz.com/java-mysql-enum/#5)
-   [6 6. 参考链接](https://yulaiz.com/java-mysql-enum/#6)

## 1. 当年我们怎么定义状态

我们定义状态变量时，通常需要约定几个状态，比如交易订单中，我们就有常见的创建、交易中、支付成功、支付失败等等状态，当年我们都是约定好 `0,1,2,3,4,5` 这样的字段来表示上述几个字段。

### 1.1 数据库设计

例如我们的数据库设计时，直接这样描述：

```sql
CREATE TABLE `order_test`  (
  `id` int(20) NOT NULL AUTO_INCREMENT,
  `status` int(1) NOT NULL COMMENT '0-创建，1-支付中，2-支付成功，3-支付失败，4-取消订单',
  PRIMARY KEY (`id`) USING BTREE
)
```

SQL

`0-创建，1-支付中，2-支付成功，3-支付失败，4-取消订单` 这就是我们的约定了。

当然也有恶心的设计数据库直接使用 `varchar` 数据类型，这就更恶心了，搜索都不好了。

### 1.2 Java Bean 代码

在 Java 中我们代码这样写：

```java
public class OrderInfo {
    private int id;
    //0-创建，1-支付中，2-支付成功，3-支付失败，4-取消订单
    private int status;
}
```

Java

有时候牛逼点，我们定义点常量：

```java
public interface class OrderConstants {
    //创建
    int ORDER_STATUS_CREATE = 0;
    //支付中
    int ORDER_STATUS_PAYING = 1;
    //支付成功
    int ORDER_STATUS_IN_PROGRESS = 2;
    //支付失败
    int ORDER_STATUS_FAILED = 3;
    //取消订单
    int ORDER_STATUS_REVERSED = 4;
}
```

Java

### 1.3 MyBatis 代码

然后用 MyBatis 插入查询什么的时候也简单：

```java
@Mapper
public interface OrderMapper {
    int addOrder(@Param("item") OrderInfo info);
    OrderInfo getOrderById(@Param("id") String id);
}
```

Java

```markup
<insert id="addOrder" parameterType="com.yulaiz.model.order.entity.OrderInfo">
    INSERT INTO order_test ( status )
    VALUES (
        #{item.status}
    )
</insert>
<select id="getOrderById" resultType="com.yulaiz.model.order.entity.OrderInfo">
    SELECT id, status
    FROM order_test
    WHERE id = #{id}
</select>
```

XML

### 1.4 代码效果

我们最后拿到的就是 `0,1,2,3,4` 这样的数值来表示订单状态了，当然现在的例子实在简单，实际项目中，那可不仅仅就这么 0-4，还有很多，并且一个订单的状态，可不仅仅就这些，还有其他字段来描述，那么在一些复杂的查询中，那就乱套了，一堆状态，都是数字，没办法一个个查吧，这可是真麻烦。

## 2. 现在我们怎么定义状态

### 2.1 Java Bean 代码

一般在我设计枚举字段的时候，我会先设计 Java 部分的枚举字段，因为嘛，我觉得这样粘贴复制方便一点：

```java
public enum OrderStatus {
    CREATE("创建"),
    PAYING("支付中"),
    IN_PROGRESS("支付成功"),
    FAILED("支付失败"),
    REVERSED("取消订单");

    private String value;
    OrderStatus(String value) {
        this.value = value;
    }
    public String getValue() {
        return value;
    }
}
```

Java

在使用中也很简单，首先 Java Bean 设计时直接将数据类型改为 `OrderStatus` ：

```java
public class OrderInfo {
    private int id;
    private OrderStatus status;
}
```

Java

赋值的时候使用：

```java
orderInfo.setStatus(OrderStatus.CREATE);
```

Java

### 2.2 数据库设计

在 Mysql 中这样描述 enum 字段：

```sql
CREATE TABLE `order_test`  (
  `id` int(20) NOT NULL AUTO_INCREMENT,
  `status` enum('CREATE','PAYING','IN_PROGRESS','FAILED','REVERSED') CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL DEFAULT 'CREATE' COMMENT 'CREATE-创建，PAYING-支付中，IN_PROGRESS-支付成功，FAILED-支付失败，REVERSED-取消订单',
  PRIMARY KEY (`id`) USING BTREE
)
```

SQL

### 2.3 MyBatis 代码

接着 MyBatis 中插入查询完全不用修改，还是上次那个原样就ok，是不是很简单。

### 2.4 代码效果

这样一来我们在 Java 代码中就可以直接看到这个状态是 `CREATE` 或者 `PAYING` 或者其他状态，在sql查询中也会显示 `CREATE` 或者 `PAYING` 或者其他状态，清晰明了，不需要再去一个个的查字段定义，非常方便阅读。

**其中Java代码里枚举类有一个隐藏的属性叫 `name` 就是我们定义时候使用的 `CREATE("创建"),PAYING("支付中")` 当中的 `CREATE` 和 `PAYING` ，MyBatis在插入数据库的时候，会自动使用这个 `name` 属性作为赋值，而查询的时候也是通过数据库查询的内容和 `name` 属性进行匹配。**

## 3. 怎么把当年定义的状态改成使用枚举

好了，由于本人比较懒，发现枚举的好处之后就觉得，当年写的到底是什么垃圾代码，由此也就想把之前定义的都替换成枚举。

### 3.1 数据库设计

首先，咱从底层入手，先看看数据库。

要想修改成枚举，那么我就要先新增一个枚举字段，然后根据原有的映射，来分别 update 对应数据，但是这样是很糟糕的操作：

1.  麻烦，当数据量大时，大量 update 操作效率很慢。
2.  修改后原有代码逻辑没有修改完整，导致出错。

那么数据库就先不改了，咱直接把新功能用上枚举吧，之前的代码不改动了，新功能的 Java 部分直接用上枚举，尽量让自己写代码舒服点，看着爽一些。

### 3.2 Java Bean 代码

我们仍旧想使用 `orderInfo.setStatus(OrderStatus.CREATE);` 的方式进行赋值，但是现在数据库中的数据是 `0,1,2...` 跟枚举类 OrderStatus 的 `name属性` 不能对应上了。

首先我们还是先写 Java 的枚举，由于这次要跟数据库对应上，数据结构就有点区别了：

```java
public enum OrderStatus {
    CREATE(0, "创建"),
    PAYING(1, "支付中"),
    IN_PROGRESS(2, "支付成功"),
    FAILED(3, "支付失败"),
    REVERSED(4, "取消订单");

    private int value;
    private String desc;

    OrderStatus(int value, String desc) {
        this.value = value;
        this.desc = desc;
    }

    public int getValue() {
        return value;
    }

    public String getDesc() {
        return desc;
    }
}
```

Java

当然，也可以：

```java
public enum OrderStatus {
    CREATE(0),
    PAYING(1),
    IN_PROGRESS(2),
    FAILED(3),
    REVERSED(4);

    private int value;
    OrderStatus(int value) {
        this.value = value;
    }
    public int getValue() {
        return value;
    }
}
```

Java

但我觉得还是加个中文方便，那么我就想用 `CREATE(0, "创建")` 这种方式。

### 3.3 MyBatis 代码

实际上这里的关键就是 MyBatis 了，怎么样让 MyBatis 知道我们在赋值 `orderInfo.setStatus(OrderStatus.CREATE);` 的时候用 `CREATE(0, "创建")` 中的 `0` 而不是 `CREATE` 呢。

首先，我们先看看 MyBatis 是否能够满足我们的需求。MyBatis 内置了两个枚举转换器分别是`org.apache.ibatis.type.EnumTypeHandler` 和 `org.apache.ibatis.type.EnumOrdinalTypeHandler`。

#### 3.3.1 内置枚举转换器

##### 3.3.1.1 EnumTypeHandler

这是默认的枚举转换器，转换器将枚举实例转换为实例名称的字符串，即 `name` 属性，也就是将 `OrderStatus.CREATE` 转换为 `CREATE`。就是我们在 `2.现在我们怎么定义状态` 中所使用的方式。

##### 3.3.1.2 EnumOrdinalTypeHandler

这个转换器将枚举实例的 `ordinal` 属性作为取值，这个属性可以通过 `orderInfo.getStatus().ordinal()` 来获取。

> ```
> public final int ordinal()
> ```
> 
> Returns the ordinal of this enumeration constant (its position in its enum declaration, where the initial constant is assigned an ordinal of zero). Most programmers will have no use for this method. It is designed for use by sophisticated enum-based data structures, such as `EnumSet` and `EnumMap`.
> 
> -   Returns:
>     
>     the ordinal of this enumeration constant
>     

通过官方文档并且通过自己的实验来看，这个 `ordinal` 属性其实是一个序号，这个序号从 0 开始，只跟定义枚举类时的顺序有关。

当我们按照下述方式定义枚举时：

```java
public enum OrderStatus {
    CREATE("创建"),
    PAYING("支付中"),
    IN_PROGRESS("支付成功"),
    FAILED("支付失败"),
    REVERSED("取消订单");
    //省略部分代码...
}
```

Java

其 `ordinal` 属性就是跟这个先后顺序有关，`OrderStatus.CREATE.ordinal()==0` `OrderStatus.PAYING.ordinal()==1` 刚好跟我们后来自定义的 `value` 属性同步了。

但是我们不能相信这个属性，所以 MyBatis 提供的两种枚举转换器均不适用，我们也就只好继续自定义了。

#### 3.3.2 自定义枚举转换器

MyBatis 提供了 `org.apache.ibatis.type.BaseTypeHandler` 类用于我们自己扩展类型转换器，上面的 `EnumTypeHandler` 和 `EnumOrdinalTypeHandler` 也都实现了这个接口。

```java
public class EnumOrderStatusHandler extends BaseTypeHandler<OrderStatus> {
    /**
     * 设置配置文件设置的转换类以及枚举类内容，供其他方法更便捷高效的实现
     *
     * @param type 配置文件中设置的转换类
     */
    public EnumOrderStatusHandler(Class<OrderStatus> type) {
        if (type == null)
            throw new IllegalArgumentException("Type argument cannot be null");
        this.type = type;
        this.enums = type.getEnumConstants();
        if (this.enums == null)
            throw new IllegalArgumentException(type.getSimpleName()
                    + " does not represent an enum type.");
    }

    //用于定义设置参数时，该如何把Java类型的参数转换为对应的数据库类型
    @Override
    public void setNonNullParameter(PreparedStatement ps, int i, OrderStatus parameter, JdbcType jdbcType) throws SQLException {
        // 根据数据库存储类型决定获取类型，本例子中数据库中存放int类型
        // ps.setString
        ps.setInt(i, parameter.getValue());
    }

    //用于定义通过字段名称获取字段数据时，如何把数据库类型转换为对应的Java类型
    @Override
    public OrderStatus getNullableResult(ResultSet rs, String columnName) throws SQLException {
        // 根据数据库存储类型决定获取类型，本例子中数据库中存放int类型
        // String i = rs.getString(columnName);
        int i = rs.getInt(columnName);
        if (rs.wasNull()) {
            return null;
        } else {
            // 根据数据库中的值，定位Enum子类
            return locateEnum(i);
        }
    }

    //用于定义通过字段索引获取字段数据时，如何把数据库类型转换为对应的Java类型
    @Override
    public OrderStatus getNullableResult(ResultSet rs, int columnIndex) throws SQLException {
        // 根据数据库存储类型决定获取类型，本例子中数据库中存放int类型
        // String i = rs.getString(columnIndex);
        int i = rs.getInt(columnIndex);
        if (rs.wasNull()) {
            return null;
        } else {
            // 根据数据库中的值，定位Enum子类
            return locateEnum(i);
        }
    }

    //用定义调用存储过程后，如何把数据库类型转换为对应的Java类型
    @Override
    public OrderStatus getNullableResult(CallableStatement cs, int columnIndex) throws SQLException {
        // 根据数据库存储类型决定获取类型，本例子中数据库中存放int类型
        // String i = cs.getString(columnIndex);
        int i = cs.getInt(columnIndex);
        if (cs.wasNull()) {
            return null;
        } else {
            // 根据数据库中的值，定位Enum子类
            return locateEnum(i);
        }
    }

    /**
     * 枚举类型转换
     *
     * @param value 数据库中存储的自定义属性
     * @return value对应的枚举类
     */
    private OrderStatus locateEnum(int value) {
        for (OrderStatus status : OrderStatus.values()) {
            if (status.getValue() == value) {
                return status;
            }
        }
        throw new IllegalArgumentException("未知的枚举类型：" + value);
    }
}
```

Java

类转换器就这样编写，注意的是，各个重写方法中，数据类型，用对应数据库数据的枚举属性的数据类型，本次就是 `value` 属性对应数据库中的内容，所以是 `int` 类型，再有就是在最后的方法中，给出枚举匹配的方法。

#### 3.3.3 在 Mapper.xml 中配置自定义的枚举转换器

##### 方法1：修改指定xml文件，指定的Mapper生效

我们只需要在 insert update select 等语句中配置 typeHandler , 而 Java 文件无需改动。

```markup
<insert id="addOrder" parameterType="com.yulaiz.model.order.entity.OrderInfo">
    INSERT INTO order_test ( status
    ) VALUES (
        #{item.status, typeHandler=com.yulaiz.model.order.entity.enums.mybatis.EnumOrderStatusHandler}
    )
</insert>
<resultMap id="get" type="com.yulaiz.model.order.entity.OrderInfo">
    <result column="status" property="status"
            typeHandler="com.yulaiz.model.order.entity.enums.mybatis.EnumOrderStatusHandler"/>
</resultMap>
<select id="getOrderById" id="getOrderById" resultMap="get">
    SELECT id, status
    FROM order_test
    WHERE id = #{id}
</select>
```

XML

##### 方法2：全局指定枚举转换器，无需单独修改Mapper.xml

直接在 Mybatis 的配置文件中配置自定义的类型转换，这里我使用的 Spring-Boot，直接在 application.yml 文件中配置：

```yml
mybatis:
    type-handlers-package: com.yulaiz.model.order.entity.enums.mybatis
```

YAML

##### 方法3：不使用枚举转换器

对于 insert 和 update 语句，还有一个简单的方法：

```markup
<insert id="addOrder" parameterType="com.yulaiz.model.order.entity.OrderInfo">
    INSERT INTO order_test ( status
    ) VALUES (
        #{item.status.value}
    )
</insert>
```

XML

对于 Java Bean 来说，也可以手动设置 get set 方法

```java
public class OrderInfo {
    private int id;
    private OrderStatus status;

    public int getStatus() {
        return status.getValue();
    }
    public void setStatus(int value) {
        for (OrderStatus status : OrderStatus.values()) {
            if (status.getValue() == value) {
                this.status = status;
                break;
            }
        }
    }
}
```

Java

## 4. 枚举到底好不好用

枚举这样直观方便，在大数据量的情况下到底好不好用呢，下面我们就这么一个问题进行分析。

### 4.1 Java 枚举的实现

#### 4.1.1 基础声明、属性、构造函数

java 的枚举类 以关键字 `enum` 声明，该关键字隐含着该类为 `java.lang.Enum` 的子类，Java编译器在编译枚举类时，会生成一个相关的类，这个类就是实际的枚举类，继承自 `java.lang.Enum` 。

```java
public enum OrderStatus {
    CREATE("创建"), PAYING("支付中"), IN_PROGRESS("支付成功"), FAILED("支付失败"), REVERSED("取消订单");
}
```

Java

该类有两个属性，分别是 `name` 和 `ordinal` ，在自定义的枚举类中，我们声明 `CREATE("创建"), PAYING("支付中"), IN_PROGRESS("支付成功"), FAILED("支付失败"), REVERSED("取消订单")` 的时候，前面括号外部分就是 `name` 属性，比如：`CREATE` ，而 `ordinal` 属性则是我们声明中写的顺序，从 0 开始。这两个属性在父类的构造函数中进行赋值，我们自定义的枚举类中不用去费心这两个属性的赋值。子类中如果需要，只需要在构造方法中定义其他自定义属性的赋值即可。

```java
protected Enum(String name, int ordinal) {
    this.name = name;
    this.ordinal = ordinal;
}
```

Java

而 `CREATE("创建")` 中括号内就是我们自定义的属性，也就是我们声明的 `value` 属性。

#### 4.1.2 常用方法

##### 4.1.2.1 name() 方法

返回 Enum 对象的 `name` 属性。

##### 4.1.2.2 ordinal() 方法。

返回 Enum 对象的 `ordinal` 属性。

##### 4.1.2.3 toString() 方法

返回 Enum 对象的名称，也就是 `name` 属性。

##### 4.1.2.4 equals(Object other) 方法

比较两个对象是否相等。详细用法在下方 [4.1.3 比较两个枚举值是否一致](https://yulaiz.com/java-mysql-enum/#4.1.3)

##### 4.1.2.5 compareTo(E o) 方法

比较两个枚举对象的顺序，在该对象小于、等于或大于指定对象时，分别返回负整数、零或正整数。详细用法在下方 [4.1.3 比较两个枚举值是否一致](https://yulaiz.com/java-mysql-enum/#4.1.3)

##### 4.1.2.6 values() 方法

这个方法我们查看JDK中源码没有看到，并且在API文档中也没有找到。但确实是有这么一个方法。

在 [Oracle的文档](https://docs.oracle.com/javase/tutorial/java/javaOO/enum.html) 中可以找到这么一段话

> The compiler automatically adds some special methods when it creates an enum. For example, they have a static `values` method that returns an array containing all of the values of the enum in the order they are declared. This method is commonly used in combination with the for-each construct to iterate over the values of an enum type.

大致意思就是，Java 编译器在编译枚举类的时候会自动为该类加入一些静态方法，比如说 `values()` ，这个方法返回一个包含这个枚举类所有枚举项的数组，这个数组是按照声明的顺序来排列（意味着 `ordinal` 属性可以当成这个数组的下标），这个方法通常与 for-each 语句配合在遍历枚举值时使用。

注意，这是一个 `static` 静态方法，意味着使用的时候直接通过枚举类来调用，比如 `OrderStatus.values()` 。

##### 4.1.2.7 valueOf(String name) 方法

根据传入的 `name` 名称，找到相对应 `name` 属性的枚举值。

这个方法实际上也是 Java 编译器在编译的时候加入的方法，在父类 `java.lang.Enum` 中，同样存在 valueOf 方法，不过父类的方法是有两个参数，而这个第一个参数就是用来指定是哪一个枚举类，那我们在子类的时候使用这个方法就很明确是这个类本身，所以编译器直接在编译的时候加入了一个方法，只需要传入 `name` 参数即可。

注意，这是一个 `static` 静态方法，意味着使用的时候直接通过枚举类来调用，比如： `OrderStatus.valueOf("CREATE")` 。

#### 4.1.3 比较两个枚举值是否一致

我们在实际代码中经常根据状态来进行流程向导，而枚举类大部分时间就是用来表示状态，那么我们就需要对比枚举类是否为某个值来进行判断：

-   通过 switch 语句
    
    通过 switch 语句来判断，简单方便，好看：
    
    ```java
    OrderStatus status = OrderStatus.CREATE;
    switch (status) {
      case CREATE:
          System.out.println("CREATE");
          break;
      case PAYING:
          System.out.println("PAYING");
          break;
      default:
          System.out.println("default");
    }
    ```
    
    Java
    
-   通过 equals() 方法
    
    用 String 的时候经常使用的方法：
    
    ```java
    OrderStatus status = OrderStatus.CREATE;
    boolean result = status.equals(OrderStatus.CREATE);
    ```
    
    Java
    
-   通过 == 运算符
    
    实际上效果跟 equals() 方法是一样的，我们可以点进 equals() 方法的源码查看，实际上就是使用 == 运算符来实现的：
    
    > ```java
    > /**
    >  * Returns true if the specified object is equal to this
    >  * enum constant.
    >  *
    >  * @param other the object to be compared for equality with this object.
    >  * @return  true if the specified object is equal to this
    >  *          enum constant.
    >  */
    > public final boolean equals(Object other) {
    >     return this==other;
    > }
    > ```
    > 
    > Java
    
    实际使用上就更简单了：
    
    ```java
    OrderStatus status = OrderStatus.CREATE;
    if (status == OrderStatus.CREATE) {
      System.out.println("==");
    }
    
    ```
    
    Java
    
-   通过compareTo(E o) 方法
    
    稍微麻烦一点：
    
    ```java
    OrderStatus status = OrderStatus.CREATE;
    if (status.compareTo(OrderStatus.CREATE) == 0) {
      System.out.println("==");
    }
    ```
    
    Java
    
    ​
    

### 4.2 MySql 中的枚举

首先，我们要知道对于 `select` 语句，枚举对于其展示出来的数据是非常友好的，可以直观的看到具体的含义。而枚举的优点就是固定的有限的枚举项，那么就需要我们在定义数据库的时候就定义好这个枚举可能用到的所有值，相比int类型，我们来看看对于枚举类型 enum 的枚举项的删除和修改操作的效率如何。

我在数据库中分别创建了两张表：`order_test` 、`order_test_enum` 。

```sql
CREATE TABLE `order_test`  (
  `id` int(20) NOT NULL AUTO_INCREMENT,
  `status` int(1) NOT NULL COMMENT '0-创建，1-支付中，2-支付成功，3-支付失败，4-取消订单',
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 1 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;

CREATE TABLE `order_test_enum`  (
  `id` int(20) NOT NULL AUTO_INCREMENT,
  `status` enum('CREATE','PAYING','IN_PROGRESS','FAILED','REVERSED') CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL DEFAULT 'CREATE' COMMENT 'CREATE-创建，PAYING-支付中，IN_PROGRESS-支付成功，FAILED-支付失败，REVERSED-取消订单',
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 2 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;
```

SQL

在两张表中分别插入了 1000W 条数据，其数据格式就是每个枚举项都依次循环插入，保证每个枚举项的数量尽可能平均的分布。

![javaMySqlEnumDatabaseCount](https://yulaiz-static-1254376006.file.myqcloud.com/img/article/code/java-mysql-enum/javaMySqlEnumDatabaseCount.jpg)

#### 4.2.1 DDL 操作效率

对于状态的修改，我们一般会有几种操作：

-   新增一个状态。
-   修改一个状态。
-   删除一个状态。

##### 4.2.1.1 新增一个枚举状态

对于 int 类型来说，这个直接无需操作了，要什么新状态，直接 insert 的时候直接插入就行了。

对于 enum 类型来说，这个就需要进行 DDL 操作：

```sql
ALTER TABLE `test`.`order_test_enum` 
MODIFY COLUMN `status` enum('CREATE','PAYING','IN_PROGRESS','FAILED','REVERSED','TEST') CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL DEFAULT 'CREATE' COMMENT 'CREATE-创建，PAYING-支付中，IN_PROGRESS-支付成功，FAILED-支付失败，REVERSED-取消订单，TEST-新增测试';
```

SQL

`执行时间：0.008s` 还不错的速度。

##### 4.2.1.2 修改一个枚举状态

比方说我们定义的状态，写错值了，又不想将错就错，int 类型应该不会出现这个问题吧，奈何英文水平实在太差，拼错了单词，或者单词用的不合理，我们这里假设将 `PAYING` 换成 `PAYING1` ：

```sql
ALTER TABLE `test`.`order_test_enum` 
MODIFY COLUMN `status` enum('CREATE','PAYING1','IN_PROGRESS','FAILED','REVERSED','TEST') CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL DEFAULT 'CREATE' COMMENT 'CREATE-创建，PAYING1-支付中，IN_PROGRESS-支付成功，FAILED-支付失败，REVERSED-取消订单，TEST-新增测试';
```

SQL

`1265 - Data truncated for column 'status' at row 2` 修改失败，Mysql 不允许修改已经使用的枚举项。

那么我们想到达换枚举值的情况就只能曲折一点，先新增一个，再 update 原有的值到新值，刚刚已经建好的TEST 值，那么我们现在就把 `PAYING` 修改成 `TEST` ：

```sql
UPDATE order_test_enum 
SET `status` = 'TEST' 
WHERE
    `status` = 'PAYING';
```

SQL

`执行时间: 12.456s` 这样的一个时间说得过去，因为我刚刚也在 order_test 表中做了一个差不多的操作执行时间是 `10.459s` 。

```sql
UPDATE order_test 
SET `status` = 5 
WHERE
    `status` = 1;
```

SQL

##### 4.2.1.3 删除一个枚举状态

同样 int 类型无需修改，修改 enum 类型进行 DDL 操作，假设我们删除 `IN_PROGRESS` 这项：

```sql
ALTER TABLE `test`.`order_test_enum` 
MODIFY COLUMN `status` enum('CREATE','PAYING','FAILED','REVERSED','TEST') CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL DEFAULT 'CREATE' COMMENT 'CREATE-创建，PAYING-支付中，FAILED-支付失败，REVERSED-取消订单，TEST-新增测试';
```

SQL

`1265 - Data truncated for column 'status' at row 2` 删除失败，这是由于MySql不允许删除已经被使用的枚举项。那我们删除刚刚已经把数据改为 `TEST` 的 `PAYING` 试试：

```sql
ALTER TABLE `test`.`order_test_enum` 
MODIFY COLUMN `status` enum('CREATE','IN_PROGRESS','FAILED','REVERSED','TEST') CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL DEFAULT 'CREATE' COMMENT 'CREATE-创建，IN_PROGRESS-支付成功，FAILED-支付失败，REVERSED-取消订单，TEST-新增测试';
```

SQL

`执行时间: 34.971s` 这个时间是真的不快。

#### 4.2.2 查询效率

可能有些人会说枚举对于大数据查询是非常不友好的，毕竟是字符型。

而实际上 Mysql 中的 enum 类型，还恰恰不是字符型，而是整型存储。

实际上在建立 enum 字段的时候，MySql 会根据设定的几个字段的顺序来编号，而这个编号实际上才是 MySql 真正存储的内容，这个编号在某些文章里也会称呼为索引值。

在上面的例子中，建立的枚举值分别为 `'CREATE','PAYING','IN_PROGRESS','FAILED','REVERSED'` ，那么他们的索引值就分别为：

值

索引值

NULL

NULL

”

0

‘CREATE’

1

‘PAYING’

2

‘IN_PROGRESS’

3

‘FAILED’

4

‘REVERSED’

5

所以在搜索的时候 WHERE 条件中的 `status = 'CREATE'` 和 `status = 1` 是等效的，这里也要注意就是 enum 字段同样有 NULL 值 和 ” 的情况，需要我们在定义数据类型的时候就需要注意。

```sql
SELECT
    id,
    `status` 
FROM
    order_test_enum 
WHERE
    id = 1 
    AND `status` = 1;
```

SQL

```sql
SELECT
    id,
    `status` 
FROM
    order_test_enum 
WHERE
    id = 1 
    AND `status` = 'CREATE';
```

SQL

上面两个sql查询到的内容就是一样的，那么同是整型，在查询上的差距，咱就可以忽略不计了吧。

## 5. 枚举使用总结

> 虽然 Java 中的枚举比 C 或 C++ 中的 enum 更成熟，但它仍然是一个“小”功能，Java 没有它也已经（虽然有点笨拙）存在很多年了。而本章正好说明了一个“小”功能所能带来的价值。有时恰恰因为它，你才能够优雅而干净地解决问题。正如我在本书中一再强调的那样，优雅与清晰很重要，正是它们区别了成功的解决方案与失败的解决方案。而失败的解决方案就是因为其他人无法理解它。

直接引用了《Thinking in Java》这么一段话（《Thinking in Java》第四版 19章12节），很明显，很清楚，至少在 Java 中使用枚举是值得的。

而对于数据库来说，根据实际情况来构建，至少枚举时一个特别方便的结构，如果有旧表的状态描述字段没有使用枚举，请慎重是否进行改进，需要考虑存量数据的改造，以及对接代码的改造。

即便 MySql 中依旧使用数字来表达状态，依然不妨碍在 Java 中改造成枚举，具体改造的范围需要自行斟酌，防止接口对上下游系统的不友好。

枚举是非常适合用来描述状态的结构，并且不推荐使用数字来定义，请使用字符串来表达各个状态，否则在查看代码，查看数据库数据的时候，还是一个数字，那我们使用枚举的意义何在。

总的来说，请尽量合理的使用枚举，功能虽小，但咱们很优雅。

## 6. 参考链接

[mysql enum类型存在大量数据的时候方便修改数据项么](https://segmentfault.com/q/1010000014083107)

[mysql 数据库枚举类型enum，方便添加新的枚举项吗？](https://segmentfault.com/q/1010000000245126)

[MySQL中的enum类型有什么优点？](https://segmentfault.com/q/1010000008121647?_ea=1553667)

[深入理解Java枚举类型(enum)](https://blog.csdn.net/javazejian/article/details/71333103)

[重新认识java（十） —- Enum（枚举类）](https://blog.csdn.net/qq_31655965/article/details/55049192)

[java enum(枚举)使用详解 + 总结](https://www.cnblogs.com/hyl8218/p/5088287.html)

[如何在MyBatis中优雅的使用枚举](https://segmentfault.com/a/1190000010755321)

[MYSQL中 ENUM 类型](https://www.cnblogs.com/skillCoding/archive/2012/03/14/2395404.html)

[java枚举enum类中的values()](https://blog.csdn.net/u013469218/article/details/66476182)

[MyBatis对于Java对象里的枚举类型处理](http://www.shuyangyang.com.cn/jishuliangongfang/Javabiancheng/2015-01-26/229.html)

[MyBatis对于Java对象里的枚举类型处理](https://www.cnblogs.com/jeffen/p/6380724.html)

**转载请注明：**

转载自[YuLai's Blog](https://yulaiz.com/)，原文地址：[Java、Mysql、MyBatis 中枚举 enum 的使用](https://yulaiz.com/java-mysql-enum/)

赞赏

![alipay](media/alipay.png)

![wechat](media/wechat-1.png)

如果文章对您有帮助，欢迎给作者打赏

[enum](https://yulaiz.com/tag/enum/)[java](https://yulaiz.com/tag/java/)[mybatis](https://yulaiz.com/tag/mybatis/)[mysql](https://yulaiz.com/tag/mysql/)

[](http://service.weibo.com/share/share.php?url=https%3A%2F%2Fyulaiz.com%2Fjava-mysql-enum%2F&title=Java%E3%80%81Mysql%E3%80%81MyBatis%20%E4%B8%AD%E6%9E%9A%E4%B8%BE%20enum%20%E7%9A%84%E4%BD%BF%E7%94%A8&pic=https%3A%2F%2Fyulaiz.com%2Fwp-content%2Fuploads%2F2018%2F04%2Fjava-mysql-enum-banner.jpg&appkey=)

![](media/create-qr-code.png)

[](http://sns.qzone.qq.com/cgi-bin/qzshare/cgi_qzshare_onekey?url=https%3A%2F%2Fyulaiz.com%2Fjava-mysql-enum%2F&title=Java%E3%80%81Mysql%E3%80%81MyBatis%20%E4%B8%AD%E6%9E%9A%E4%B8%BE%20enum%20%E7%9A%84%E4%BD%BF%E7%94%A8&summary=Java%20%E5%92%8C%20MySql%20%E4%B8%AD%E9%83%BD%E6%9C%89%E6%9E%9A%E4%B8%BE%E7%9A%84%E6%A6%82%E5%BF%B5%EF%BC%8C%E5%90%88%E7%90%86%E7%9A%84%E4%BD%BF%E7%94%A8%E6%9E%9A%E4%B8%BE%EF%BC%8C%E5%8F%AF%E4%BB%A5%E8%AE%A9%E4%BB%A3%E7%A0%81%E9%98%85%E8%AF%BB%E5%92%8C%E6%95%B0%E6%8D%AE%E5%BA%93%E6%95%B0%E6%8D%AE%E6%9F%A5%E8%AF%A2%E6%9B%B4%E5%8A%A0%E7%9B%B4%E8%A7%82%E3%80%81%E9%AB%98%E6%95%88%E3%80%82%E9%82%A3%E4%B9%88%E6%88%91%E4%BB%AC%E6%80%8E%E4%B9%88%E4%BD%BF%E7%94%A8%E5%91%A2%EF%BC%8C%E4%BB%80%E4%B9%88%E6%97%B6%E5%80%99%E4%BD%BF%E7%94%A8%EF%BC%8C%E4%B8%A4%E8%80%85%E4%B9%8B%E9%97%B4%E6%80%8E%E4%B9%88%E8%BF%9B%E8%A1%8C%E6%95%B0%E6%8D%AE%E5%85%B3%E8%81%94%E5%91%A2%EF%BC%9F%EF%BC%88%E6%9C%AC%E6%96%87%E4%BD%BF%E7%94%A8%20MyBatis%20%E5%81%9A%E4%B8%BA%E2%80%A6&pics=https%3A%2F%2Fyulaiz.com%2Fwp-content%2Fuploads%2F2018%2F04%2Fjava-mysql-enum-banner.jpg&site=)[](http://connect.qq.com/widget/shareqq/index.html?url=https%3A%2F%2Fyulaiz.com%2Fjava-mysql-enum%2F&title=Java%E3%80%81Mysql%E3%80%81MyBatis%20%E4%B8%AD%E6%9E%9A%E4%B8%BE%20enum%20%E7%9A%84%E4%BD%BF%E7%94%A8&summary=Java%20%E5%92%8C%20MySql%20%E4%B8%AD%E9%83%BD%E6%9C%89%E6%9E%9A%E4%B8%BE%E7%9A%84%E6%A6%82%E5%BF%B5%EF%BC%8C%E5%90%88%E7%90%86%E7%9A%84%E4%BD%BF%E7%94%A8%E6%9E%9A%E4%B8%BE%EF%BC%8C%E5%8F%AF%E4%BB%A5%E8%AE%A9%E4%BB%A3%E7%A0%81%E9%98%85%E8%AF%BB%E5%92%8C%E6%95%B0%E6%8D%AE%E5%BA%93%E6%95%B0%E6%8D%AE%E6%9F%A5%E8%AF%A2%E6%9B%B4%E5%8A%A0%E7%9B%B4%E8%A7%82%E3%80%81%E9%AB%98%E6%95%88%E3%80%82%E9%82%A3%E4%B9%88%E6%88%91%E4%BB%AC%E6%80%8E%E4%B9%88%E4%BD%BF%E7%94%A8%E5%91%A2%EF%BC%8C%E4%BB%80%E4%B9%88%E6%97%B6%E5%80%99%E4%BD%BF%E7%94%A8%EF%BC%8C%E4%B8%A4%E8%80%85%E4%B9%8B%E9%97%B4%E6%80%8E%E4%B9%88%E8%BF%9B%E8%A1%8C%E6%95%B0%E6%8D%AE%E5%85%B3%E8%81%94%E5%91%A2%EF%BC%9F%EF%BC%88%E6%9C%AC%E6%96%87%E4%BD%BF%E7%94%A8%20MyBatis%20%E5%81%9A%E4%B8%BA%E2%80%A6&pics=https%3A%2F%2Fyulaiz.com%2Fwp-content%2Fuploads%2F2018%2F04%2Fjava-mysql-enum-banner.jpg&site=)[](https://twitter.com/intent/tweet?url=https%3A%2F%2Fyulaiz.com%2Fjava-mysql-enum%2F&text=Java%E3%80%81Mysql%E3%80%81MyBatis%20%E4%B8%AD%E6%9E%9A%E4%B8%BE%20enum%20%E7%9A%84%E4%BD%BF%E7%94%A8&via=&hashtags=)[](https://plus.google.com/share?url=https%3A%2F%2Fyulaiz.com%2Fjava-mysql-enum%2F)[](http://www.evernote.com/clip.action?url=https%3A%2F%2Fyulaiz.com%2Fjava-mysql-enum%2F&title=Java%E3%80%81Mysql%E3%80%81MyBatis%20%E4%B8%AD%E6%9E%9A%E4%B8%BE%20enum%20%E7%9A%84%E4%BD%BF%E7%94%A8)[](mailto:?subject=Java%E3%80%81Mysql%E3%80%81MyBatis%20%E4%B8%AD%E6%9E%9A%E4%B8%BE%20enum%20%E7%9A%84%E4%BD%BF%E7%94%A8&body=https%3A%2F%2Fyulaiz.com%2Fjava-mysql-enum%2F)

[**AES加密出现 java.security.InvalidKeyException: Illegal key size or default parameters**](https://yulaiz.com/jdk-aes-illegal-key-size/)[**微信 Emoji 表情存储到 MySql 中乱码的问题**](https://yulaiz.com/java-mysql-emoji-save/)

## 2 条评论

![](media/avatar.jpg)

有人回复时邮件通知我

-   ![](media/75d23af433e0cea4c0e45a56dba18b30.png)**Wisdom** 
    
     [2019年12月20日](https://yulaiz.com/java-mysql-enum/#comment-1758)
    
    感谢
    
    [](https://yulaiz.com/java-mysql-enum/?replytocom=1758#respond)
    
-   ![](media/b04a981f7091515b24c033dc4735a95e.png)**ufan0** 
    
     [2019年7月22日](https://yulaiz.com/java-mysql-enum/#comment-1245)