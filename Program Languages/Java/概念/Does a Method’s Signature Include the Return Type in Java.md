Last updated: September 24, 2020  Written by: [Cristian Stancalau](https://www.baeldung.com/author/cristianstancalau "Posts by Cristian Stancalau")
- [Java](https://www.baeldung.com/category/java)+
- [Core Java](https://www.baeldung.com/tag/core-java)

### **Get started with Spring 5 and Spring Boot 2, through the _Learn Spring_ course:**

### **[> CHECK OUT THE COURSE](https://www.baeldung.com/ls-course-start)**

## 1. Overview[](https://www.baeldung.com/java-method-signature-return-type#overview)

## [](https://www.baeldung.com/java-method-signature-return-type#overview)

The method signature is only a subset of the entire method definition in Java. Thus, the exact anatomy of the signature may cause confusion.

In this tutorial, we’ll learn the elements of the method signature and its implications in Java programming.

## 2. Method Signature[](https://www.baeldung.com/java-method-signature-return-type#method-signature)

## [](https://www.baeldung.com/java-method-signature-return-type#method-signature)

[Methods in Java](https://www.baeldung.com/java-methods) support overloading, meaning that multiple methods with the same name can be defined in the same class or hierarchy of classes. Hence, the compiler must be able to statically bind the method the client code refers to. For this reason, the method **signature uniquely identifies each method**.

According to [Oracle](https://docs.oracle.com/javase/tutorial/java/javaOO/methods.html), the method **signature is comprised of the name and parameter types**. Therefore, all the other elements of the method’s declaration, such as modifiers, return type, parameter names, exception list, and body are not part of the signature.

Let’s take a closer look at [method overloading](https://www.baeldung.com/java-method-overload-override) and how it relates to method signatures.

## 3. Overloading Errors[](https://www.baeldung.com/java-method-signature-return-type#overloading-errors)

## [](https://www.baeldung.com/java-method-signature-return-type#overloading-errors)

Let’s consider the following code_:_

```java
public void print() {
    System.out.println("Signature is: print()");
}

public void print(int parameter) {
    System.out.println("Signature is: print(int)");
}
```

As we can see, the code compiles as the methods have different parameter type lists. In effect, the compiler can deterministically bind any call to one or the other.

Now let’s test if it’s legal to overload by adding the following method:

```java
public int print() { 
    System.out.println("Signature is: print()"); 
    return 0; 
}
```

When we compile, we get a “method is already defined in class” error. That proves the method **return type is not part of the method signature**.

Let’s try the same with modifiers:

```java
private final void print() { 
    System.out.println("Signature is: print()");
}
```

We still see the same “method is already defined in class” error. Therefore, the method **signature is not dependent on modifiers**.

Overloading by changing thrown exceptions can be tested by adding:

```java
public void print() throws IllegalStateException { 
    System.out.println("Signature is: print()");
    throw new IllegalStateException();
}
```

Again we see the “method is already defined in class” error, indicating the **throw declaration cannot be part of the signature**.

The last thing we can test is whether changing the parameter names allow overloading. Let’s add the following method:

```java
public void print(int anotherParameter) { 
    System.out.println("Signature is: print(int)");
}
```

As expected, we get the same compilation error. This means that **parameter names don’t influence the method signature**.

## 3. Generics and Type Erasure[](https://www.baeldung.com/java-method-signature-return-type#generics-and-type-erasure)

## [](https://www.baeldung.com/java-method-signature-return-type#generics-and-type-erasure)

With generic parameters**, [type erasure](https://www.baeldung.com/java-type-erasure) changes the effective signature**. In effect, it may cause a collision with another method that uses the upper bound of the generic type instead of the generic token.

Let’s consider the following code:

```java
public class OverloadingErrors<T extends Serializable> {

    public void printElement(T t) {
        System.out.println("Signature is: printElement(T)");
    }

    public void printElement(Serializable o) {
        System.out.println("Signature is: printElement(Serializable)");
    }
}
```

Even though the signatures appear different, the compiler cannot statically bind the correct method after type erasure.

We can see the compiler replacing _T_ with the upper bound, _Serializable,_ due to type erasure. Thus, it clashes with the method explicitly using _Serializable_.

We would see the same result with the base type _Object_ when the generic type has no bound.

## 4. Parameter Lists and Polymorphism[](https://www.baeldung.com/java-method-signature-return-type#parameter-lists-and-polymorphism)

## [](https://www.baeldung.com/java-method-signature-return-type#parameter-lists-and-polymorphism)

The method signature takes into account the exact types. That means we can overload a method whose parameter type is a subclass or superclass.

However, we must pay special attention as **static binding will attempt to match using polymorphism, auto-boxing, and type promotion**.

Let’s take a look at the following code:

```java
public Number sum(Integer term1, Integer term2) {
    System.out.println("Adding integers");
    return term1 + term2;
}

public Number sum(Number term1, Number term2) {
    System.out.println("Adding numbers");
    return term1.doubleValue() + term2.doubleValue();
}

public Number sum(Object term1, Object term2) {
    System.out.println("Adding objects");
    return term1.hashCode() + term2.hashCode();
}
```

The code above is perfectly legal and will compile. Confusion may arise when calling these methods as we not only need to know the exact method signature we are calling but also how Java statically binds based on the actual values.

Let’s explore a few method calls that end up bound to _sum(Integer, Integer)_:

```java
StaticBinding obj = new StaticBinding(); 
obj.sum(Integer.valueOf(2), Integer.valueOf(3)); 
obj.sum(2, 3); 
obj.sum(2, 0x1);
```

For the first call, we have the exact parameter types _Integer, Integer._ On the second call, Java will auto-box _int_ to _Integer_ for us_._ Lastly, Java will transform the byte value _0x1_ to _int_ by means of type promotion and then auto-box it to _Integer._ 

Similarly, we have the following calls that bind to _sum(Number, Number)_:

```java
obj.sum(2.0d, 3.0d);
obj.sum(Float.valueOf(2), Float.valueOf(3));
```

On the first call, we have _double_ values that get auto-boxed to _Double._ And then, by means of polymorphism, _Double_ matches _Number._ Identically, _Float_ matches _Number_ for the second call.

Let’s observe the fact that both _Float_ and _Double_ inherit from _Number_ and _Object._ However, the default binding is to _Number_. This is due to the fact that Java will automatically match to the nearest super-types that match a method signature.

Now let’s consider the following method call:

```java
obj.sum(2, "John");
```

In this example, we have an _int_ to _Integer_ auto-box for the first parameter. However, there is no _sum(Integer, String)_ overload for this method name. Consequentially, Java will run through all the parameter super-types to cast from the nearest parent to _Object_ until it finds a match. In this case, it binds to _sum(Object, Object)._ 

**To change the default binding, we can use explicit parameter casting** as follows:

```java
obj.sum((Object) 2, (Object) 3);
obj.sum((Number) 2, (Number) 3);
```

## 5. Vararg Parameters[](https://www.baeldung.com/java-method-signature-return-type#vararg-parameters)

## [](https://www.baeldung.com/java-method-signature-return-type#vararg-parameters)

Now let’s turn our attention over to how **_[varargs](https://www.baeldung.com/java-varargs)_ impact the method’s effective signature** and static binding.

Here we have an overloaded method using _varargs_:

```java
public Number sum(Object term1, Object term2) {
    System.out.println("Adding objects");
    return term1.hashCode() + term2.hashCode();
}

public Number sum(Object term1, Object... term2) {
    System.out.println("Adding variable arguments: " + term2.length);
    int result = term1.hashCode();
    for (Object o : term2) {
        result += o.hashCode();
    }
    return result;
}
```

So what are the effective signatures of the methods? We’ve already seen that _sum(Object, Object)_ is the signature for the first. Variable arguments are essentially arrays, so the effective signature for the second after compilation is _sum(Object, Object[])._ 

A tricky question is how can we choose the method binding when we have just two parameters?

Let’s consider the following calls:

```java
obj.sum(new Object(), new Object());
obj.sum(new Object(), new Object(), new Object());
obj.sum(new Object(), new Object[]{new Object()});
```

Obviously, the first call will bind to _sum(Object, Object)_ and the second to _sum(Object, Object[])._ To force Java to call the second method with two objects, we must wrap it in an array as in the third call.

The last thing to note here is that declaring the following method will clash with the vararg version:

```java
public Number sum(Object term1, Object[] term2) {
    // ...
}
```

## 6. Conclusion[](https://www.baeldung.com/java-method-signature-return-type#conclusion)

## [](https://www.baeldung.com/java-method-signature-return-type#conclusion)

In this tutorial, we learned that the method signatures are comprised of the name and the parameter types’ list. The modifiers, return type, parameter names, and exception list cannot differentiate between overloaded methods and, thus, are not part of the signature.

We’ve also looked at how type erasure and varargs hide the effective method signature and how we can override Java’s static method binding.

As usual, all the code samples shown in this article are available [over on GitHub](https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-oop-methods).