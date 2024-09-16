Last modified: October 29, 2019

Written by: [baeldung](https://www.baeldung.com/author/baeldung "Posts by baeldung")

-   [Java](https://www.baeldung.com/category/java)+

-   [Exception](https://www.baeldung.com/tag/exception)
-   [Java Streams](https://www.baeldung.com/tag/java-streams)

### **Get started with Spring 5 and Spring Boot 2, through the _Learn Spring_ course:**

### **[> CHECK OUT THE COURSE](https://www.baeldung.com/ls-course-start)**

## **1. Overview**[](https://www.baeldung.com/java-stream-operated-upon-or-closed-exception#overview)

## [](https://www.baeldung.com/java-stream-operated-upon-or-closed-exception#overview)

In this brief article, we're going to discuss a common _Exception_ that we may encounter when working with the _Stream_ class in Java 8:

```plaintext
IllegalStateException: stream has already been operated upon or closed.
```

We'll discover the scenarios when this exception occurs, and the possible ways of avoiding it, all along with practical examples.

## **2. The Cause**[](https://www.baeldung.com/java-stream-operated-upon-or-closed-exception#cause)

## [](https://www.baeldung.com/java-stream-operated-upon-or-closed-exception#cause)

In Java 8, each _Stream_ class represents a single-use sequence of data and supports several I/O operations.

> A _Stream_ should be operated on (invoking an intermediate or terminal stream operation) only once. A Stream implementation may throw _IllegalStateException_ if it detects that the _Stream_ is being reused.

Whenever a terminal operation is called on a _Stream_ object, the instance gets consumed and closed.

Therefore, **we're only allowed to perform a single operation that consumes a** _**Stream**,_ otherwise, we'll get an exception that states that the _Stream_ has already been operated upon or closed.

Let's see how this can be translated to a practical example:

```java
Stream<String> stringStream = Stream.of("A", "B", "C", "D");
Optional<String> result1 = stringStream.findAny(); 
System.out.println(result1.get()); 
Optional<String> result2 = stringStream.findFirst();
```

As a result:

```java
A
Exception in thread "main" java.lang.IllegalStateException: 
  stream has already been operated upon or closed
```

After the _#findAny()_ method is invoked, the _stringStream_ is closed, therefore, any further operation on the _Stream_ will throw the _IllegalStateException_, and that's what happened after invoking the _#findFirst()_ method.

## **3. The Solution**[](https://www.baeldung.com/java-stream-operated-upon-or-closed-exception#solution)

## [](https://www.baeldung.com/java-stream-operated-upon-or-closed-exception#solution)

Simply put, the solution consists of creating a new _Stream_ each time we need one.

We can, of course, do that manually, but that's where the _Supplier_ functional interface becomes really handy:

```java
Supplier<Stream<String>> streamSupplier 
  = () -> Stream.of("A", "B", "C", "D");
Optional<String> result1 = streamSupplier.get().findAny();
System.out.println(result1.get());
Optional<String> result2 = streamSupplier.get().findFirst();
System.out.println(result2.get());
```

As a result:

```plaintext
A
A
```

We've defined the _streamSupplier_ object with the type _Stream<String>_, which is exactly the same type which the _#get()_ method returns. The _Supplier_ is based on a lambda expression that takes no input and returns a new _Stream_.

Invoking the functional method _get()_ on the _Supplier_ returns a freshly created _Stream_ object, on which we can safely perform another _Stream_ operation.

## **5. Conclusion**[](https://www.baeldung.com/java-stream-operated-upon-or-closed-exception#conclusion)

## [](https://www.baeldung.com/java-stream-operated-upon-or-closed-exception#conclusion)

In this quick tutorial, we've seen how to perform terminal operations on a _Stream_ multiple times, while avoiding the famous _IllegalStateException_ that is thrown when the _Stream_ is already closed or operated upon.

You can find the complete source code and all code snippets for this article [over on GitHub](https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-streams).