在学习和开发过程中，我们经常会讨论 short ，int 和 long 这些基本数据类型的取值范围，但是对于 String 类型我们好像很少注意它的“取值范围”。那么对于 String 类型，它到底有没有长度限制呢？

其实 String 类型的对象，他们是有长度限制的， String 对象并不能“存储”无限长度的字符串。关于 String 的长度限制要从**编译时限制**和**运行时限制**两方面考虑。

## 编译期限制[#](https://www.cnblogs.com/54chensongxia/p/13640352.html#%E7%BC%96%E8%AF%91%E6%9C%9F%E9%99%90%E5%88%B6)

有JVM虚拟机相关知识的同学肯定知道，下面定义的字符串常量“自由之路”会被放入方法区的常量池中。

```java
String s = "自由之路";
System.out.println(s);
```

Stirng 长度之所以会受限制，是因JVM规范对常量池有所限制。常量池中的每一种数据项都有自己的类型。Java中的UTF-8编码的Unicode字符串在常量池中以CONSTANT_Utf8类型表示。

CONSTANT_Utf8的数据结构如下：

```perl
CONSTANT_Utf8_info {
    u1 tag;
    u2 length;
    u1 bytes[length];
}
```

我们重点关注下长度为 length 的那个bytes数组，这个数组就是真正存储常量数据的地方，而 length 就是数组可以存储的最大字节数。length 的类型是u2，u2是无符号的16位整数，因此理论上允许的的最大长度是2^16-1=65535。所以上面byte数组的最大长度可以是65535。

```java
//65535个d，编译报错
String s = "dd..dd";

//65534个d，编译通过
String s1 = "dd..d";
```

上面的列子中长度为65535的字符串s还是编译失败了，但是长度为65534的字符串 s1 编译是成功的。这个好像和我们刚刚的结论不符合。

其实，这是Javac编译器的额外限制。在Javac的源代码中可以找到以下代码：

```java
private void checkStringConstant(DiagnosticPosition var1, Object var2) {
    if (this.nerrs == 0 && var2 != null && var2 instanceof String &&   ((String)var2).length() >= 65535) {
        this.log.error(var1, "limit.string", new Object[0]);
        ++this.nerrs;
    }
}
```

代码中可以看出，当参数类型为String，并且长度大于等于65535的时候，就会导致编译失败。

这里需要重点强调下的是：String 的限制还有一个部分，那就是对字符串底层存储的字节数的限制。**也就是说：在编译时，一个字符串的长度大于等于65535或者底层存储占用的字节数大于65535时就会报错**。这句话可能比较抽象，下面举个列子就清楚了。

Java中的字符常量都是使用UTF8编码的，UTF8编码使用1~4个字节来表示具体的Unicode字符。所以有的字符占用一个字节，而我们平时所用的大部分中文都需要3个字节来存储。

```java
//65534个字母，编译通过
String s1 = "dd..d";

//21845个中文”自“,编译通过
String s2 = "自自...自";

//一个英文字母d加上21845个中文”自“，编译失败
String s3 = "d自自...自";
```

对于s1，一个字母d的UTF8编码占用一个字节，65534字母占用65534个字节，长度是65534，长度和存储都没超过限制，所以可以编译通过。

对于s2，一个中文占用3个字节，21845个正好占用65535个字节，而且字符串长度是21845，长度和存储也都没超过限制，所以可以编译通过。

对于s3，一个英文字母d加上21845个中文”自“占用65536个字节，超过了存储最大限制，编译失败。

## 运行时限制[#](https://www.cnblogs.com/54chensongxia/p/13640352.html#%E8%BF%90%E8%A1%8C%E6%97%B6%E9%99%90%E5%88%B6)

String 运行时的限制主要体现在 String 的构造函数上。下面是 String 的一个构造函数：

```java
public String(char value[], int offset, int count) {
    ...
}
```

上面的count值就是字符串的最大长度。在Java中，int的最大长度是2^31-1。所以在运行时，String 的最大长度是2^31-1。

但是这个也是理论上的长度，实际的长度还要看你JVM的内存。我们来看下，最大的字符串会占用多大的内存。

```markdown
(2^31-1)*2*16/8/1024/1024/1024 = 4GB
```

所以在最坏的情况下，一个最大的字符串要占用4GB的内存。如果你的虚拟机不能分配这么多内存的话，会直接报错的。

JDK9以后对String的存储进行了优化。底层不再使用char数组存储字符串，而是使用byte数组。对于LATIN1字符的字符串可以节省一倍的内存空间。

## 简单总结[#](https://www.cnblogs.com/54chensongxia/p/13640352.html#%E7%AE%80%E5%8D%95%E6%80%BB%E7%BB%93)

String 的长度是有限制的。

-   编译期的限制：字符串的UTF8编码值的字节数不能超过65535，字符串的长度不能超过65534；
-   运行时限制：字符串的长度不能超过2^31-1，占用的内存数不能超过虚拟机能够提供的最大值。

## 公众号推荐[#](https://www.cnblogs.com/54chensongxia/p/13640352.html#%E5%85%AC%E4%BC%97%E5%8F%B7%E6%8E%A8%E8%8D%90)

欢迎大家关注我的微信公众号「程序员自由之路」

[![](media/1775037-20200505091245079-544605853.jpg)](https://img2020.cnblogs.com/blog/1775037/202005/1775037-20200505091245079-544605853.jpg)

作者：程序员自由之路

出处：[https://www.cnblogs.com/54chensongxia/p/13640352.html](https://www.cnblogs.com/54chensongxia/p/13640352.html)

版权：本作品采用「[署名-非商业性使用-相同方式共享 4.0 国际](https://creativecommons.org/licenses/by-nc-sa/4.0/)」许可协议进行许可。

 分类: [01-Java基本功](https://www.cnblogs.com/54chensongxia/category/1532231.html)

 标签: [String](https://www.cnblogs.com/54chensongxia/tag/String/), [Java](https://www.cnblogs.com/54chensongxia/tag/Java/)