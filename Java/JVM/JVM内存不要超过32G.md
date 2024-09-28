[_](https://www.cnblogs.com/Chary) 

- [管理](https://i.cnblogs.com/)

随笔 - 1125  文章 - 1036  评论 - 73  阅读 - 176万

# [JVM内存不要超过32G](https://www.cnblogs.com/Chary/p/15891674.html "发布于 2022-02-14 11:00")

> 不要超过32G

  
事实上jvm在内存小于32G的时候会采用一个内存对象指针压缩技术。  
   
在java中，所有的对象都分配在堆上，然后有一个指针引用它。指向这些对象的指针大小通常是CPU的字长的大小，不是32bit就是64bit，这取决于你的处理器，指针指向了你的值的精确位置。  
   
对于32位系统，你的内存最大可使用4G。对于64系统可以使用更大的内存。但是64位的指针意味着更大的浪费，因为你的指针本身大了。浪费内存不算，更糟糕的是，更大的指针在主内存和缓存器（例如LLC, L1等）之间移动数据的时候，会占用更多的带宽。  
   
Java 使用一个叫内存指针压缩的技术来解决这个问题。它的指针不再表示对象在内存中的精确位置，而是表示偏移量。这意味着32位的指针可以引用40亿个对象，而不是40亿个字节。最终，也就是说堆内存长到32G的物理内存，也可以用32bit的指针表示。  
   
一旦你越过那个神奇的30-32G的边界，指针就会切回普通对象的指针，每个对象的指针都变长了，就会使用更多的CPU内存带宽，也就是说你实际上失去了更多的内存。事实上当内存到达40-50GB的时候，有效内存才相当于使用内存对象指针压缩技术时候的32G内存。  
   
这段描述的意思就是说：即便你有足够的内存，也尽量不要超过32G，因为它浪费了内存，降低了CPU的性能，还要让GC应对大内存。

[JVM内存不要超过32G_zsj777的专栏-CSDN博客_jvm内存超过32g](https://blog.csdn.net/zsj777/article/details/80352962)

## 1、将Java Heap Size设置的大于32G会对性能有什么影响？

开门见山的说，结果有几点（这几点其实也是内部关联）：

- 触发JVM的临界值，优化策略Compressed OOPS失效（之前Heap Size在[4G~32G]区间内采用此优化）
    
- 由于优化策略失效，同时堆内存>32G，所以JVM被迫使用8字节（64位）来对Java对象寻址（之前4字节（32位）就够了）
    
- 通常64位JVM消耗的内存会比32位的大1.5倍，这是因为对象指针在64位架构下，长度会翻倍（事实上当内存到达40-50GB的时候，有效内存才相当于使用Compressed OOPS技术时候的32G内存）
    
- 更大的指针在主内存和缓存器（例如LLC, L1等）之间移动数据的时候，会占用更多的带宽
    
- 让JVM的GC面临更大压力的指针对象（在实际应用中构建大于12-16G的堆时，若无很好的性能调优与测评，你很容易就会引起一个耗时数分钟的完全GC）
    

### 1.1 JVM的OOPS

OOP = “ordinary object pointer” 普通对象指针

启用CompressOops后，会压缩的对象：

- 每个class的属性指针（静态成员变量）
    
- 每个对象的属性指针
    
- 普通对象数组的每个元素指针
    

### 1.2 JVM的优化策略Compressed OOPS

从JDK 1.6 update14开始，64 bit JVM正式支持了 -XX:+UseCompressedOops ，这个可以压缩指针，起到节约内存占用的新参数。

Compressed OOPS，即大雾的对象压缩技术，压缩引用到32位，以降低堆的占用空间。其伪代码原理就不贴了，

在堆大小在[4G~32G]的时候，这项技术会被触发，在JVM执行时加入编/解码指令，即

> JVM在将对象存入堆时编码,在堆中读取对象时解码

内存地址确定公式类似于

|   |   |
|---|---|
|1|`<narrow-oop-base(64bits)> +(<narrow-oop(32bits)><< 3) +<field-offset>`|

Zero Based Compressed OOPS（零基压缩优化）则进一步将基地址置为0(并不一定是内存空间地址为0,只是JVM相对的逻辑地址为0,如可用CPU的寄存器相对寻址) 这样转换公式变为：

|   |   |
|---|---|
|1|`(<narrow-oop << 3) +<field-offset>`|

从而进一步提高了压解压效率。

> 使用Zero Based Compressed OOPS后，它的指针不再表示对象在内存中的精确位置，而是表示偏移量。这意味着32位的指针可以引用40亿个对象，而不是40亿个字节。

### 1.3 Zero Based Compressed OOPS的多种策略

它可以针对不同的堆大小使用多种策略，具体可以 ps + grep查看：

- 堆小于4G，无需编/解码操作，JVM会使用低虚拟地址空间（low virutal address space，64位下模拟32位）
    
- 小于32G而大于4G，使用Zero Based Compressed OOPS
    
- 大于32G，不使用Compressed OOPS
    

## 2、结论

- Compressed OOPS，可以让跑在64位平台下的JVM，不需要因为更宽的寻址，而付出Heap容量损失的代价
    
- 它的实现方式是在机器码中植入压缩与解压指令，可能会给JVM增加额外的开销
    

 [为什么JVM开启指针压缩后支持的最大堆内存是32G? - 知乎](https://www.zhihu.com/question/365436606)

# 为什么JVM开启指针压缩后支持的最大堆内存是32G?

1.在64位平台的HotSpot中使用32位指针，内存使用会多出1.5倍左右，使用较大指针在主内存和缓存之间移动数据，占用较大宽带，同时GC也会承受较大压力

2.为了减少64位平台下内存的消耗，启用指针压缩功能

3.在jvm中，32位地址表示4G个对象的指针，在4G-32G堆内存范围内，可以通过编码、解码方式进行优化，使得jvm可以支持更大的内存配置

4.堆内存小于4G时，不需要启用指针压缩，jvm会直接去除高32位地址，即使用低虚拟地址空间

5.堆内存大于32G时，压缩指针会失效，会强制使用64位(即8字节)来对java对象寻址，这就会出现1的问题，所以堆内存不要大于32G为好

先放出结论：

如果配置最大堆内存超过 32 GB（当 JVM 是 8 字节对齐），那么压缩指针会失效。 但是，这个 32 GB 是和字节对齐大小相关的，也就是-XX:ObjectAlignmentInBytes配置的大小(默认为8字节，也就是 Java 默认是 8 字节对齐)。-XX:ObjectAlignmentInBytes可以设置为 8 的整数倍，最大 128。也就是如果配置-XX:ObjectAlignmentInBytes为 24，那么配置最大堆内存超过 96 GB 压缩指针才会失效。  
  
压缩指针这个属性默认是打开的，可以通过`-XX:-UseCompressedOops`关闭。

首先说一下为何需要压缩指针呢？32 位的存储，可以描述多大的内存呢？假设每一个1代表1字节，那么可以描述 0~2^32-1 这 2^32 字节也就是 4 GB 的内存。

![](https://pic3.zhimg.com/80/v2-78b3df077014f49f77f5c0e3899c4e3b_720w.jpg?source=1940ef5c)

但是呢，Java 默认是 8 字节对齐的内存，也就是一个对象占用的空间，必须是 8 字节的整数倍，不足的话会填充到 8 字节的整数倍。也就是其实描述内存的时候，不用从 0 开始描述到 8（就是根本不需要定位到之间的1，2，3，4，5，6，7）因为对象起止肯定都是 8 的整数倍。所以，2^32 字节如果一个1代表8字节的话，那么最多可以描述 2^32 * 8 字节也就是 32 GB 的内存。

![](https://pica.zhimg.com/80/v2-7baa8a6335e30d29249b9b074badead0_720w.jpg?source=1940ef5c)

这就是压缩指针的原理。如果配置最大堆内存超过 32 GB（当 JVM 是 8 字节对齐），那么压缩指针会失效。 但是，这个 32 GB 是和字节对齐大小相关的，也就是`-XX:ObjectAlignmentInBytes`配置的大小(默认为8字节，也就是 Java 默认是 8 字节对齐)。`-XX:ObjectAlignmentInBytes`可以设置为 8 的整数倍，最大 128。也就是如果配置`-XX:ObjectAlignmentInBytes`为 24，那么配置最大堆内存超过 96 GB 压缩指针才会失效。

编写程序测试下：

```text
A a = new A();
System.out.println("------After Initialization------\n" + ClassLayout.parseInstance(a).toPrintable());
```

首先，以启动参数：`-XX:ObjectAlignmentInBytes=8 -Xmx16g`执行：

```text
------After Initialization------
com.hashjang.jdk.TestObjectAlign$A object internals:
 OFFSET  SIZE   TYPE DESCRIPTION                               VALUE
      0     4        (object header)                           05 00 00 00 (00000101 00000000 00000000 00000000) (5)
      4     4        (object header)                           00 00 00 00 (00000000 00000000 00000000 00000000) (0)
      8     4        (object header)                           48 72 06 00 (01001000 01110010 00000110 00000000) (422472)
     12     4        (alignment/padding gap)                  
     16     8   long A.d                                       0
Instance size: 24 bytes
Space losses: 4 bytes internal + 0 bytes external = 4 bytes total
```

可以看到类型字大小为 4 字节`48 72 06 00 (01001000 01110010 00000110 00000000) (422472)`，压缩指针生效。

首先，以启动参数：`-XX:ObjectAlignmentInBytes=8 -Xmx32g`执行：

```text
------After Initialization------
com.hashjang.jdk.TestObjectAlign$A object internals:
 OFFSET  SIZE   TYPE DESCRIPTION                               VALUE
      0     4        (object header)                           05 00 00 00 (00000101 00000000 00000000 00000000) (5)
      4     4        (object header)                           00 00 00 00 (00000000 00000000 00000000 00000000) (0)
      8     4        (object header)                           a0 5b c6 00 (10100000 01011011 11000110 00000000) (12999584)
     12     4        (object header)                           b4 02 00 00 (10110100 00000010 00000000 00000000) (692)
     16     8   long A.d                                       0
Instance size: 24 bytes
Space losses: 0 bytes internal + 0 bytes external = 0 bytes total
```

可以看到类型字大小为 8 字节，压缩指针失效:

```text
a0 5b c6 00 (10100000 01011011 11000110 00000000) (12999584)
b4 02 00 00 (10110100 00000010 00000000 00000000) (692)
```

修改对齐大小为 16 字节，也就是以`-XX:ObjectAlignmentInBytes=16 -Xmx32g`执行：

```text
------After Initialization------
com.hashjang.jdk.TestObjectAlign$A object internals:
 OFFSET  SIZE   TYPE DESCRIPTION                               VALUE
      0     4        (object header)                           05 00 00 00 (00000101 00000000 00000000 00000000) (5)
      4     4        (object header)                           00 00 00 00 (00000000 00000000 00000000 00000000) (0)
      8     4        (object header)                           48 72 06 00 (01001000 01110010 00000110 00000000) (422472)
     12     4        (alignment/padding gap)                  
     16     8   long A.d                                       0
     24     8        (loss due to the next object alignment)
Instance size: 32 bytes
Space losses: 4 bytes internal + 8 bytes external = 12 bytes total
```

可以看到类型字大小为 4 字节`48 72 06 00 (01001000 01110010 00000110 00000000) (422472)`，压缩指针生效。

开启-XX:+UseCompressedOops后对象计算有三种模式

![](https://pic2.zhimg.com/80/v2-c34fcc70011fd461dc68c11f6fa0c104_720w.jpg?source=1940ef5c)

(1) 如果堆的高位地址小于32G，说明不需要基址（base）就能定位堆中任意对象，这种模式也叫做Zero-based Compressed Oops Mode

(2) 如果堆高位大于等于32G，说明需要基地址，这时如果堆大小小于4G，说明基址+偏移能定位堆中任意对象

(3) 如果堆处于4G到32G的范围，这时只能通过基址+偏移x缩放（scale）才能定位堆中任意对象

如果有shift的存在，对象地址还必须8字节对齐8，如果不幸堆大于32G，那么无法使用压缩对象指针。

## 为什么要引入压缩指针(明白的跳过)

**先要明白：  
32位操作系统可以寻址到多大内存 答：4g 因为 2^32=4 * 1024 * 1024=4g  
64位呢？答：近似无穷大**

**为什么要用64位操作系统 答：因为连你家电脑的内存都不止4g了吧，你用8g的内存在32位电脑上是只有4g有效的，而4g满足不了我们的需求**

**可是用64位有些那些问题？  
答：64位过长，给我们寻址带宽和对象内引用造成了负担**

**什么负担？往下看！**

**同一个对象存在堆里会花费更多的空间！！！！**

**口说无凭，首先我们计算下同一个对象在不同操作系统的堆中存放的大小**

## 下面的东西是一个对象占用的字节数，

**对象头**  
32位系统，占用 8 字节(markWord4字节+kclass4字节)  
64位系统，开启 UseCompressedOops(压缩指针)时，占用 12 字节，否则是16字节(markWord8字节+kclass8字节，开启时markWord8字节+kclass4字节)  
**实例数据**  
boolean 1  
byte 1  
short 2  
char 2  
int 4  
float 4  
long 8  
double 8  
**引用类型**  
32位系统占4字节 (因为此引用类型要去方法区中找类信息,所以地址为32位即4字节同理64位是8字节)  
64位系统，开启 UseCompressedOops时，占用4字节，否则是8字节  
**对齐填充**  
如果对象头+实例数据的值不是8的倍数，那么会补上一些，补够8的倍数

## **好了开始举例**

假设有一个对象

```java
class A{
	int a;//基本类型
	B b;//引用类型
}
```

32位操作系统 花费的内存空间为  
**对象头-8字节 + 实例数据 int类型-4字节 + 引用类型-4字节+补充0字节(16是8的倍数) 16个字节**

64位操作系统  
**对象头-16字节 + 实例数据 int类型-4字节 + 引用类型-8字节+补充4字节(28不是8的倍数补充4字节到达32字节) 32个字节**

**同样的对象需要将近两倍的容量,(实际平均1.5倍),所以需要开启压缩指针：**

64位开启压缩指针 **对象头-12字节 + 实例数据 int类型-4字节 + 引用类型-4字节+补充4字节=24个字节**  
开启后可以减缓堆空间的压力(同样的内存更不容易发生oom)

## 压缩指针是怎么实现的

**JVM的实现方式是**  
不再保存所有引用，而是每隔8个字节保存一个引用。例如，原来保存每个引用0、1、2…，现在只保存0、8、16…。因此，指针压缩后，并不是所有引用都保存在堆中，而是以8个字节为间隔保存引用。  
在实现上，堆中的引用其实还是按照0x0、0x1、0x2…进行存储。只不过当引用被存入64位的寄存器时，JVM将其左移3位（相当于末尾添加3个0），例如0x0、0x1、0x2…分别被转换为0x0、0x8、0x10。而当从寄存器读出时，JVM又可以右移3位，丢弃末尾的0。（oop在堆中是32位，在寄存器中是35位，2的35次方=32G。也就是说，使用32位，来达到35位oop所能引用的堆内存空间）  
**仔细看图~ 仔细看图 ~仔细看图**  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200504182039156.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpb25jYXRjaA==,size_16,color_FFFFFF,t_70)

**哪些信息会被压缩？**  
1.对象的全局静态变量(即类属性)  
2.对象头信息:64位平台下，原生对象头大小为16字节，压缩后为12字节  
3.对象的引用类型:64位平台下，引用类型本身大小为8字节，压缩后为4字节  
4.对象数组类型:64位平台下，数组类型本身大小为24字节，压缩后16字节

**哪些信息不会被压缩？**  
1.指向非Heap的对象指针  
2.局部变量、传参、返回值、NULL指针

总结:

在JVM中（不管是32位还是64位），对象已经按8字节边界对齐了。对于大部分处理器，这种对齐方案都是最优的。所以，使用压缩的oop并不会带来什么损失，反而提升了性能。

## 压缩指针32g指针失效问题

讲到这应该很明了了，因为寄存器中3的32次方只能寻址到32g左右(不是准确的32g，有可能在31g就发生指压缩失效)，所以当你的内存超过32g时，jvm就默认停用压缩指针，用64位寻址来操作，这样可以保证能寻址到你的所有内存，但这样所有的对象都会变大，实际上未开启开启后的比较，40g的对象存储个数比不上30g的存储个数

[jvm 对象的指针压缩 32G内存指针压缩失效 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/385904799)

学习于：[jvm压缩指针原理以及32g内存压缩指针失效详解_超负荷运转-CSDN博客_压缩指针](https://link.zhihu.com/?target=https%3A//blog.csdn.net/lioncatch/article/details/105919666)

[为什么压缩指针超过32G失效](https://link.zhihu.com/?target=https%3A//mp.weixin.qq.com/s/oHHNuq2AuHsP2HnVvO0Idg)

首先是最基本的：什么东西被压缩了？

**哪些信息会被压缩？**

1.对象的全局静态变量(即类属性)

2.对象头信息:64位平台下，原生对象头大小为16字节，压缩后为12字节

3.对象的引用类型:64位平台下，引用类型本身大小为8字节，压缩后为4字节

4.对象数组类型:64位平台下，数组类型本身大小为24字节，压缩后16字节

**主要是对象头里面的kclass指针，即指向方法区的类信息的指针，由8字节变为4字节。 还有就是引用类型指针也由8字节变为4字节**。

压缩后的好处：

[jvm指针压缩的简单理解_HaiQ~~的博客-CSDN博客](https://link.zhihu.com/?target=https%3A//blog.csdn.net/weixin_44187446/article/details/114768967)

1。减缓GC的压力，即每个对象的大小都变小了，就不需要那么频繁的GC了。

2。降低CPU缓存的命中率。即CPU缓存本身的大小就小的多，如果采用八字节，CPU能缓存的oop（普通对象指针）肯定比四字节少，从而降低命中率。

指针压缩的具体底层实现：

一种理解：

当开启指针压缩后，KlassPointer的寻址极限是4 byte × 8 bit=32 bit，即KlassPointer可以存放2^32（=4G）个内存单元地址。

因为**每个对象的长度一定是8的整数倍，所以KlassPointer每一数位存放的一定是8的整数倍的地址**，即0/8/16/24/32/40/48/64……，也就是4G × 8 = 32G。当分配给JVM的内存空间大于32G时，KlassPointer将无法寻找大于32G的内存地址，因此设置的压缩指针将失效。

第二种理解：

JVM的实现方式是

不再保存所有引用，而是每隔8个字节保存一个引用。例如，原来保存每个引用0、1、2…，现在只保存0、8、16…。因此，指针压缩后，并不是所有引用都保存在堆中，而是以8个字节为间隔保存引用。

在实现上，堆中的引用其实还是按照0x0、0x1、0x2…进行存储。只不过**当引用被存入64位的寄存器时，JVM将其左移3位（相当于末尾添加3个0），例如0x0、0x1、0x2…分别被转换为0x0、0x8、0x10。而当从寄存器读出时，JVM又可以右移3位，丢弃末尾的0。（oop在堆中是32位，在寄存器中是35位，2的35次方=32G。也就是说，使用32位，来达到35位oop所能引用的堆内存空间**）

我感觉**主要就是2的32次方等于4G，然后8个bit的长度，即2的三次方，2的35次方就是32G了**。

所以啥，**对象已经按8字节边界对齐这个因素还是很关键**的。

[JVM - 剖析Java对象头Object Header之指针压缩](https://link.zhihu.com/?target=https%3A//blog.csdn.net/yangshangwei/article/details/106963927)

同时在64位平台的HotSpot中使用32位指针(实际存储用64位)，内存使用会多出1.5倍左右，使用较大指针在主内存和缓存之间移动数据，占用较大宽带。

- 当堆内存小于4G时，不需要启用指针压缩，jvm会直接去除高32位地址，即使用低虚拟地址空间
- 当堆内存大于32G时，压缩指针会失效，会强制使用64位(即8字节)来对java对象寻址， 那这样的话内存占用较大，GC压力等等

摘抄自网络，便于检索查找。

好文要顶 关注我 收藏该文 微信分享

[![](https://pic.cnblogs.com/face/572188/20180529163926.png)](https://home.cnblogs.com/u/Chary/)

[CharyGao](https://home.cnblogs.com/u/Chary/)   
[粉丝 - 71](https://home.cnblogs.com/u/Chary/followers/) [关注 - 301](https://home.cnblogs.com/u/Chary/followees/)  

+加关注

0

0

[升级成为会员](https://cnblogs.vip/)

[«](https://www.cnblogs.com/Chary/p/15846360.html) 上一篇：  [日志框架，jdk-logging（jdk自带的logging），原生Logger的logging.properties配置文件简单分析](https://www.cnblogs.com/Chary/p/15846360.html "发布于 2022-01-26 14:17")   
[»](https://www.cnblogs.com/Chary/p/15954166.html) 下一篇：  [[计算机网络]IP、UDP、TCP和HTTP报文格式总结](https://www.cnblogs.com/Chary/p/15954166.html "发布于 2022-03-02 11:09")

posted @ 2022-02-14 11:00  [CharyGao](https://www.cnblogs.com/Chary)  阅读(2919)  评论(0)  [编辑](https://i.cnblogs.com/EditPosts.aspx?postid=15891674)  收藏  举报

[会员力量，点亮园子希望](https://cnblogs.vip/)

[刷新页面](https://www.cnblogs.com/Chary/p/15891674.html#)[返回顶部](https://www.cnblogs.com/Chary/p/15891674.html#top)

登录后才能查看或发表评论，立即 登录 或者 [逛逛](https://www.cnblogs.com/) 博客园首页

[【推荐】秋天希望的田野，九月最后的救园：终身会员计划](https://www.cnblogs.com/cmt/p/18401581)  
[【推荐】轻量又高性能的 SSH 工具 IShell：AI 加持，快人一步](http://ishell.cc/)  
[【推荐】100%开源！大型工业跨平台软件C++源码提供，建模，组态！](http://www.uccpsoft.com/index.htm)  
[【推荐】2024阿里云超值优品季，精心为您准备的上云首选必备产品](https://click.aliyun.com/m/1000396435/)  

[![](https://img2024.cnblogs.com/blog/35695/202409/35695-20240914114555421-1162088530.jpg)](https://www.cnblogs.com/cmt/p/18302049)

**编辑推荐：**   
· [Linux服务器磁盘空间占用情况分析与清理指南](https://www.cnblogs.com/zengzuo613/p/18434300)   
· [redisson 内存泄漏问题排查](https://www.cnblogs.com/jtea/p/18428499)   
· [使用.NET并行任务库(TPL)与并行Linq(PLINQ)充分利用多核性能](https://www.cnblogs.com/GuZhenYin/p/18429430)   
· [Redis 内存突增时，如何定量分析其内存使用情况](https://www.cnblogs.com/ivictor/p/18426260)   
· [记一次 RabbitMQ 消费者莫名消失问题的排查](https://www.cnblogs.com/youzhibing/p/18426017)   

**阅读排行：**   
·  [风雨过后见彩虹：救园倒计时，最后2天](https://www.cnblogs.com/cmt/p/18432948)   
·  [《HelloGitHub》第 102 期](https://www.cnblogs.com/xueweihan/p/18434827)   
·  [.Net Web项目中，实现轻量级本地事件总线 框架](https://www.cnblogs.com/kong-ming/p/18422632)   
·  [Linux服务器磁盘空间占用情况分析与清理指南](https://www.cnblogs.com/zengzuo613/p/18434300)   
·  [.NET 开源高性能 MQTT 类库](https://www.cnblogs.com/1312mn/p/18412658)   

Copyright © 2024 CharyGao   
Powered by .NET 8.0 on Kubernetes