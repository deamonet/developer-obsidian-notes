发布于2018-08-09 10:53:00阅读 1K0

在开发和设计的时候，我们需要考虑编译时，运行时以及构建时这三个概念。理解这几个概念可以更好地帮助你去了解一些基本的原理。下面是初学者晋级中级水平需要知道的一些问题。

**Q.下面的代码片段中，行A和行B所标识的代码有什么区别呢？**

```javascript
public class ConstantFolding {
    static final int number1 = 5;
    static final int number2 = 6;
    static int number3 = 5;
    static int number4= 6;
    public static void main(String[ ] args) {
        int product1 = number1 * number2; //line A
        int product2 = number3 * number4; //line B
    }
}
```

复制

A.在行A的代码中，product的值是在编译期计算的，行B则是在运行时计算的。如果你使用Java反编译器（例如，jd-gui）来反编译ConstantFolding.class文件的话，那么你就会从下面的结果里得到答案。

```javascript
public class ConstantFolding{
    static final int number1 = 5;
    static final int number2 = 6;
    static int number3 = 5;
    static int number4 = 6;
    public static void main(String[ ] args)
    {
        int product1 = 30;
        int product2 = number3 * number4;
    }
}
```

复制

常量折叠是一种Java编译器使用的优化技术。由于final变量的值不会改变，因此就可以对它们优化。Java反编译器和javap命令都是查看编译后的代码（例如，字节码）的利器。

**Q.你能想出除了代码优化外，在什么情况下，查看编译过的代码是很有帮助的？**

A.Java里的泛型是在编译时构造的，可以通过查看编译后的class文件来理解泛型，也可以通过查看它来解决泛型相关的问题。

**Q.下面哪些是发生在编译时，运行时，或者两者都有？**

**1、方法重载**

这个是发生在编译时的。方法重载也被称为编译时多态，因为编译器可以根据参数的类型来选择使用哪个方法。

```javascript
public class{
    // method #1
    public static void evaluate(String param1);
    // method #2
    public static void evaluate(int param1);
}
```

复制

如果编译器要编译下面的语句的话：

evaluate(“My Test Argument passed to param1”);

它会根据传入的参数是字符串常量，生成调用#1方法的字节码

**2、方法覆盖**

这个是在运行时发生的。方法重载被称为运行时多态，因为在编译期编译器不知道并且没法知道该去调用哪个方法。JVM会在代码运行的时候做出决定。

```javascript
public class A {
    public int compute(int input) { //method #3
        return 3 * input;
    }
}
```

复制

```javascript
public class B extends A {
    @Override
    public int compute(int input) { //method #4
        return 4 * input;
    }
}
```

复制

子类B中的compute(..)方法重写了父类的compute(..)方法。如果编译器遇到下面的代码：

```javascript
public int evaluate(A reference, int arg2) {
    int result = reference.compute(arg2);
}
```

复制

编译器是没法知道传入的参数reference的类型是A还是B。因此，只能够在运行时，根据赋给输入变量“reference”的对象的类型（例如，A或者B的实例）来决定调用方法#3还是方法#4.

**3、泛型（又称类型检验）**

这个是发生在编译期的。编译器负责检查程序中类型的正确性，然后把使用了泛型的代码翻译或者重写成可以执行在当前JVM上的非泛型代码。这个技术被称为“类型擦除“。换句话来说，编译器会擦除所有在尖括号里的类型信息，来保证和版本1.4.0或者更早版本的JRE的兼容性。

```javascript
List<String> myList = new ArrayList<String>(10);
```

复制

编译后成为了：

```javascript
List myList = new ArrayList(10);
```

复制

**4、注解（Annotation）**

你可以使用运行时或者编译时的注解。

```javascript
public class B extends A {
    @Override
    public int compute(int input){ 
        return 4 * input;
    }
}
```

复制

@Override是一个简单的编译时注解，它可以用来捕获类似于在子类中把toString()写成tostring()这样的错误。在Java 5中，用户自定义的注解可以用注解处理工具(Anotation Process Tool ——APT)在编译时进行处理。到了Java 6，这个功能已经是编译器的一部分了。

```javascript
public class MyTest{
    @Test
    public void testEmptyness( ){
        org.junit.Assert.assertTrue(getList( ).isEmpty( ));
    }
    private List getList( ){
    }
}
```

复制

@Test是JUnit框架用来在运行时通过反射来决定调用测试类的哪个（些）方法的注解。

```javascript
@Test (timeout=100)
public void testTimeout( ) { 
    while(true);
}
```

复制

如果运行时间超过100ms的话，上面的测试用例就会失败。

```javascript
@Test (expected=IndexOutOfBoundsException.class)
public void testOutOfBounds( ) { 
    new ArrayList<Object>( ).get(1);
}
```

复制

如果上面的代码在运行时没有抛出IndexOutOfBoundsException或者抛出的是其他的异常的话，那么这个用例就会失败。用户自定义的注解可以在运行时通过Java反射API里新增的AnnotatedElement和”Annotation”元素接口来处理。

5**、异常（Exception）**

你可以使用运行时异常或者编译时异常。

5.1、运行时异常（RuntimeException）

也称作未检测的异常（unchecked exception），这表示这种异常不需要编译器来检测。RuntimeException是所有可以在运行时抛出的异常的父类。一个方法除要捕获异常外，如果它执行的时候可能会抛出RuntimeException的子类，那么它就不需要用throw语句来声明抛出的异常。

例如：NullPointerException，ArrayIndexOutOfBoundsException，等等

5.2、受检查异常（checked exception）

都是编译器在编译时进行校验的，通过throws语句或者try{}cathch{} 语句块来处理检测异常。编译器会分析哪些异常会在执行一个方法或者构造函数的时候抛出。

6**、面向切面的编程（Aspect Oriented Programming-AOP）**

切面可以在编译时，运行时或，加载时或者运行时织入。

6.1、编译期

编译期织入是最简单的方式。如果你拥有应用的代码，你可以使用AOP编译器（例如，ajc – AspectJ编译器）对源码进行编译，然后输出织入完成的class文件。AOP编译的过程包含了waver的调用。切面的形式可以是源码的形式也可以是二进制的形式。如果切面需要针对受影响的类进行编译，那么你就需要在编译期织入了。

6.2、编译后

这种方式有时候也被称为二进制织入，它被用来织入已有的class文件和jar文件。和编译时织入方式相同，用来织入的切面可以是源码也可以是二进制的形式，并且它们自己也可以被织入切面。

6.3、装载期

这种织入是一种二进制织入，它被延迟到JVM加载class文件和定义类的时候。为了支持这种织入方式，需要显式地由运行时环境或者通过一种“织入代理（weaving agent）“来提供一个或者多个“织入类加载器（weaving class loader）”。

6.4、运行时

对已经加载到JVM里的类进行织入

7**、其他分类**

继承 – 发生在编译时，因为它是静态的

代理或者组合 – 发生在运行时，因为它更加具有动态性和灵活性。

**Q.你有没有听说过“组合优于继承”这样的说法呢？如果听说过的话，那么你是怎么理解的呢？**

A.继承是一种多态工具，而不是一种代码复用工具。有些开发者喜欢用继承的方式来实现代码复用，即使是在没有多态关系的情况下。是否使用继承的规则是继承只能用在类之间有“父子”关系的情况下。

1.不要仅仅为了代码复用而继承。当你使用组合来实现代码复用的时候，是不会产生继承关系的。过度使用继承（通过“extends”关键字）的话，如果修改了父类，会损坏所有的子类。这是因为子类和父类的紧耦合关系是在编译期产生的。

2.不要仅仅为了多态而继承。如果你的类之间没有继承关系，并且你想要实现多态，那么你可以通过接口和组合的方式来实现，这样不仅可以实现代码重用，同时也可以实现运行时的灵活性。

这就是为什么（Gang of Four）的设计模式里更倾向于使用组合而不是继承的原因。面试者会在你的答案里着重关注这几个词语——“耦合”，“静态还是动态”，以及“发生在编译期还是运行时”。运行时的灵活性可以通过组合来实现，因为类可以在运行时动态地根据一个结果有条件或者无条件地进行组合。但是继承却是静态的。

**Q.你能够通过实例来区别编译期继承和运行时继承，以及指出Java支持哪种吗？**

A.“继承”表示动作和属性从一个对象传递到另外一个对象的场景。Java语言本身只支持编译期继承，它是通过“extends”关键字来产生子类的方式实现的，如下所示：

```javascript
public class Parent {
    public String saySomething( ) { return “Parent is called”;
    }
}
public class Child extends Parent {
    @Override
    public String saySomething( ) { 
    return super.saySomething( ) + “, Child is called”;
    }
}
```

复制

“Child”类的saySomething()方法的调用会返回“Parent is called，Child is Called”，因为，子类的调用继承了父类的“Parenet is called”。关键字“super”是用来调用“Parent”类的方法。运行时继承表示在运行时构建父/子类关系。Java语言本身不支持运行时继承，但是有一种替代的方案叫做“代理”或者“组合”，它表示在运行时组件一个层次对象的子类。这样可以模拟运行时继承的实现。在Java里，代理的典型实现方式如下：

```javascript
public class Parent {
    public String saySomething( ) { 
        return “Parent is called”;
    }
}
public class Child {
    public String saySomething( ) { 
        return new Parent( ).saySomething( ) + “, Child is called”;
    }
}
```

复制

子类代理了父类的调用。组合可以按照下面的方式来实现：

```javascript
public class Child {
    private Parent parent = null;
    public Child( ){ this.parent = new Parent( );
    }
    public String saySomething( ) { 
        return this.parent.saySomething( ) + “, Child is called”;
    }
}
```

复制

**<End>**

版权声明：“Java后端技术”所推送文章，为本人原创、网上收集或其他作者投稿，对于网上收集部分除非确实无法确认，我们都会注明作者和来源。部分文章推送时未能与原作者取得联系。若涉及版权问题，烦请原作者联系我们，我们会在24小时内删除处理，谢谢！^_^ QQ:1573876303

文章分享自微信公众号：

![](media/code-2.jpg)

Java后端技术

复制公众号名称

本文参与 [腾讯云自媒体分享计划](https://cloud.tencent.com/developer/support-plan) ，欢迎热爱写作的你一起参与！

作者：爱Java

原始发表时间：2016-10-12

如有侵权，请联系 cloudcommunity@tencent.com 删除。