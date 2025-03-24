开发者训练营官方  发布于 2021-3-30 10:39 4175浏览 0收藏

# **Java中为何要分别定义强引用、软引用、弱引用、虚引用四种引用类型？** 
我们对于对象的定义不能只局限于“引用”和“未被引用”两种状态，因为我们还存在几种引用类型能描述这样一类对象：当内存空间还足够时，仍保留在内存之中，如果内存空间在进行垃圾收集后仍然非常紧张，那就可以抛弃这些对象（这类对象应用在很多系统的缓存功能上）。

在JDK 1.2版之后，Java对引用的概念进行了扩充，将引用分为强引用（Strongly Re-ference）、软引用（Soft Reference）、弱引用（WeakReference）和虚引用（Phantom Reference）4种，这4种引用强度依次逐渐减弱。

## **定义这四种引用类型的目的：**  

1.让程序员通过代码的方式决定某些对象的生命周期；  
2.有利于JVM进行垃圾回收。

## **强引用：必不可少的，不会被垃圾回收器进行回收的。**  
当内存空间不足的时候，程序宁愿抛出java.lang.OutOfMemoryError错误，使程序异常终止，也不回收这种对象。

```markup
 public static void main(String[] args) {
        Object[] objects=new Object[Integer.MAX_VALUE];  
    }
```

复制

```markup
"C:\Program Files\Java\jdk1.8.0_192\bin\java.exe" "-javaagent:C:\Program Files\JetBrains\IntelliJ IDEA 2020.1.3\lib\idea_rt.jar=6744:C:\Program Files\JetBrains\IntelliJ IDEA 2020.1.3\bin" ...Day27.TestDemo
Exception in thread "main" java.lang.OutOfMemoryError: Requested array size exceeds VM limit
 at Day27.TestDemo.main(TestDemo.java:5)

Process finished with exit code 1
```

复制

  
如果想要中断强引用和某个对象之间的联系的时候，可以显示的将引用赋值为null，这样,JVM在合适的时间就会回收该对象。

**软引用：有用但不是必需的对象。**  
SoftReference类  
只被软引用关联着的对象，在系统将要发生内存溢出异常前，会把这些对象列进回收范围之中进行第二次回收，如果这次回收还没有足够的内存，才会抛出内存溢出异常。只有在内存空间不足的时候，Java虚拟机才会回收该对象，如一些网页缓存，图片缓存就使用的软引用。

```java
public static void main(String[] args) throws InterruptedException {
        //创建方式
        Object obj = new Object();
        SoftReference<Object> sf = new SoftReference<Object>(obj);
        //使用方式
        Object obj1 = sf.get();
        //说明没有被回收
        if(obj1 != null) obj1.wait();//使用对象

    }
```

复制

  
**弱引用：非必需的对象。**  
WeakReference类  
被弱引用关联的对象只能生存到下一次垃圾收集发生为止，等下一次垃圾回收回收的时候，无论JVM内存空间是否充足，都会回收被弱引用关联的对象。

**弱引用和软引用的区别：**  
生存周期不同：弱引用的对象拥有更短暂的生命周期，在垃圾回收器扫描它所管辖的内存区域的过程中，一旦发现了具有弱引用的对象，不管当前内存是否足够，都会进行回收它。垃圾回收器是一个优先级很低的线程，因此不一定会发现那些只具有弱引用的对象，所以被软引用关联的对象只有在内存空间不足的时候才会被回收，而被弱引用关联的对象在JVM进行垃圾回收时总会被回收。

**虚引用：最弱的一种引用，如同虚设一样**  
一个对象是否有虚引用的存在，完全不会对其生存时间构成影响，也无法通过虚引用来取得一个对象实例。为一个对象设置虚引用关联的唯一目的只是为了能在这个对象被收集器回收时收到一个系统通知。  
PhantomReference类

**总结：**

引用类型

 被垃圾回收的时间

 用途

 生存时间

强引用

 从来不会

对象的一般状态

JVM停止运行时终止

软引用

在内存不足时

 对象缓存

内存不足时终止

弱引用

在垃圾回收时

对象缓存 

GC运行后终止

虚引用

未知

让这个对象被收集器回收时收到一个系统通知

未知

  
**强引用：**  
不会被回收的对象。  
**软引用：**  
如果弱引用对象回收完之后，内存还是报警，继续回收软引用对象  
**弱引用：**  
如果虚引用对象回收完之后，内存还是报警，继续回收弱引用对象  
**虚引用：**  
虚拟机的内存不够使用，开始报警，这时候垃圾回收机制开始执行System.gc(); 如果没有对象回收了，就回收没虚引用的对象  
————————————————  
版权声明：本文为博主「AI小艾」的原创文章

分类

[框架语言](https://ost.51cto.com/category/49/53)