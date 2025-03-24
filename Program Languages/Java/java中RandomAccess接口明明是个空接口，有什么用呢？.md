[

![头像](media/头像-4.jpg)

**cafebabe**
2019-10-10
https://segmentfault.com/a/1190000020640043/

## RandomAccess接口

### 一、官方描述

  首先我们先看一下这个接口在java中是怎么描述的。

> 位置：rt.jar中java.util.RandomAccess

/**
 * Marker interface used by <tt>List</tt> implementations to indicate that
 * they support fast (generally constant time) random access.  The primary
 * purpose of this interface is to allow generic algorithms to alter their
 * behavior to provide good performance when applied to either random or
 * sequential access lists.
 *
 * <p>The best algorithms for manipulating random access lists (such as
 * <tt>ArrayList</tt>) can produce quadratic behavior when applied to
 * sequential access lists (such as <tt>LinkedList</tt>).  Generic list
 * algorithms are encouraged to check whether the given list is an
 * <tt>instanceof</tt> this interface before applying an algorithm that would
 * provide poor performance if it were applied to a sequential access list,
 * and to alter their behavior if necessary to guarantee acceptable
 * performance.
 *
 * <p>It is recognized that the distinction between random and sequential
 * access is often fuzzy.  For example, some <tt>List</tt> implementations
 * provide asymptotically linear access times if they get huge, but constant
 * access times in practice.  Such a <tt>List</tt> implementation
 * should generally implement this interface.  As a rule of thumb, a
 * <tt>List</tt> implementation should implement this interface if,
 * for typical instances of the class, this loop:
 * <pre>
 *     for (int i=0, n=list.size(); i &lt; n; i++)
 *         list.get(i);
 * </pre>
 * runs faster than this loop:
 * <pre>
 *     for (Iterator i=list.iterator(); i.hasNext(); )
 *         i.next();
 * </pre>

> 大致的意思是说：这是一个用于List的实现类的中使用的标记接口，能够支持快速随机访问（一定时间内）。此接口的主要目的是允许通用算法在应用于随机或顺序访问列表时改变其行为以提供良好的性能。  
> 只在List的实现类ArrayList可以for + get（index）使用该算法执行，在LinkedList类中不行。

### 二、查看ArrayList和LinkedList

> 位置：rt.jar中java.util.ArrayList和java.util.LinkedList

#### 2.1 ArrayList

public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    private static final long serialVersionUID = 8683452581122892189L;
    ...
}

#### 2.2 LinkedList

public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable
{
    transient int size = 0;
    ... 
}

> 由源码可以看出：ArrayList是一个实现了List、RandomAccess、Cloneable、 java.io.Serializable等接口的类。实现可以随机访问、克隆及序列化的数组集合。  
> LinkedList则是实现了List、Deque、Cloneable、java.io.Serializable等接口的类，所以它是一个克隆、序列化的双端连表。

### 三、验证RandomAccess实践，当然选择ArrayList。

> 代码如下：

public class Application {

    public static void main(String[] args) {

        System.out.println("开始准备数据");
        List<Integer> data = new ArrayList<Integer>();

        int length = 50000000;
        while (length > 0){
            data.add(length);
            length--;
        }
        System.out.println("准备数据完成");



        System.out.println("开始使用get(index)");
        Long timeIndex = System.currentTimeMillis();
        for (int i = 0; i < data.size(); i++){
            data.get(i);
        }

        System.out.println("结束使用get(index) :" + String.valueOf(System.currentTimeMillis() - timeIndex));


        System.out.println("开始使用Iterable");
        Long timeIterable = System.currentTimeMillis();

        Iterator<Integer> iterator = data.iterator();
        while (iterator.hasNext()){
            iterator.next();
        }
        System.out.println("结束使用Iterable :" + String.valueOf(System.currentTimeMillis() - timeIterable));

    }

}

运行结果：

开始准备数据
准备数据完成
开始使用get(index)
结束使用get(index) :4
开始使用Iterable
结束使用Iterable :8

### 四、总结

   可以看出当集合的数量达到一定值的时候，效果很明显，但是当length小的时候结果不是很明显。实际使用中看自己的场景使用即可。本身对于程序性能影响并不大。