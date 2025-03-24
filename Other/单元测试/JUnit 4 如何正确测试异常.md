2012-06-08 | 阅读(24,037)

本篇讲述如何在 JUnit 4 下正确测试异常，我会从 try..catch 的方式谈起，然后说到 @Test(expected=Exception.class), 最后论及 @Rules public ExpectedException 的实现方式，最终基本可确定用 @Rules 是最方便的。

我们在用 JUnit 测试方法异常的时候，最容易想到的办法就是用 try...catch 去捕获异常，需要断言以下几个条件：

1. 确实抛出的异常  
2. 抛出异常的 Class 类型  
3. 抛出异常的具体类型，一般检查异常的 message 属性中包含的字符串的断定

所以常用的代码你可能会这么写：

1

2

3

4

5

6

7

8

9

10

@Test

public void passwordLengthLessThan6LettersThrowsException(){

    try{

        Password.validate("123");

        fail("No exception thrown.");

    }catch(Exception ex){

        assertTrue(ex instanceof InvalidPasswordException);

        assertTrue(ex.getMessage().contains("contains at least 6"));

    }

}

这里被测试的方法是 Password.validate() 方法是否抛出了相应的异常，注意这里别漏 try 中的

fail("No Exception thrown.") 代码行，不然如果被测试的方法如果没有抛出异常的话，这个用例是通过的，而你预期的是要抛出异常的。

上面的土办法对于哪个 JUnit 版本是是适合，可是我们早已步入了 JUnit 4 的时候，大可不必如此这般的去测试方法异常。虽然这样也能测定出是否执行出预期的异常来，但它仍有弊端，接下来会一对比就知道了，try...catch 的方法，JUnit 无法为你提示出详细的断言失败原因。

那么来看看自从 JUnit 4 后可以怎么去测试异常呢？用 @Test(execpted=Exception.class) 注解就行，参考如下代码：

1

2

3

4

@Test(expected = NullPointerException.class)

public void passwordIsNullThrowsException() throws InvalidPasswordException {

    Password.validate(null);

}

如果被测试的方法有抛出 NullPointerException 类型便是断言成功，对了 @Test(expected = NullPointerException.class) 只能判断出异常的类型，并无相应的注解能断言出异常的更具体的信息，即无法判定抛出异常的  message 属性。

那么，有时候我们会在一个方法中多次抛出一种类型的异常，但原因不同，即异常的 message 信息不同，比如出现 InvalidPasswordException 时会有以下两种异常：

new InvalidPasswordException("Password must contains at least 6 letters.")  
new InvalidPasswordException("Password length less than 15 letters")

这就要有办法去断言异常的 message 了，针对于此，自 JUnit 4.7 之后又给了我们更完美的选择，就是下面的代码：

1

2

3

4

5

6

7

8

9

@Rule

public ExpectedException expectedEx = ExpectedException.none();

@Test

public void passwordIsEmptyThrowsException() throws InvalidPasswordException {

    expectedEx.expect(InvalidPasswordException.class);

    expectedEx.expectMessage("required");

    Password.validate("");

}

上面代码需重点关注几个：

1. @Rule 注解的  ExpectedException 变量声明，它必须为  public  
2. @Test 处，不能写成 @Test(expected=InvalidPasswordException.class)，否则不能正确测试，也就是              
          @Test(expected=InvalidPasswordException.class) 和测试方法中的 expectedEx.expectXxx() 方法是不能同时并存的  
3. expectedEx.expectMessage() 中的参数是 Matcher 或  subString，就是说可用正则表达式判定，或判断是否包含某个子字符串  
4. 再就是有一点很重，把被测试方法写在 expectedEx.expectXxx() 方法后面，不然也不能正确测试的异常  
5. 最后一个是，只要测试方法直接抛出被测试方法的异常即可，并不影响你所关心的异常

前面说到用 try...catch 的办法也能正确测试到异常，@Test(expected=...) 或 @Rule 与 try...catch 的方法对比有什么好处呢，显然用 JUnit 4 推荐的方法简洁明了。再来看测试失败时 JUnit 会为你提示什么呢？

try...catch 测试异常失败时，得到的提示: 无异常时：

java.lang.AssertionError: No exception thrown.  
    at org.junit.Assert.fail(Assert.java:91)  
    at cc.unmi.PasswordTest.passwordLengthLessThan6LettersThrowsException(PasswordTest.java:20)

异常类型不对或异常的 message 不对时：

java.lang.AssertionError:  
    at org.junit.Assert.fail(Assert.java:91)    at org.junit.Assert.assertTrue(Assert.java:43)    at org.junit.Assert.assertTrue(Assert.java:54)    at cc.unmi.PasswordTest.passwordLengthLessThan6LettersThrowsException(PasswordTest.java:22)

上面能提供给我们的定位错误的帮助不是特别大

再看 @Test(expected=InvalidPasswordException.class) 时测试失败时的提示：

java.lang.AssertionError: Expected exception: cc.unmi.InvalidPasswordException  
    at org.junit.internal.runners.statements.ExpectException.evaluate(ExpectException.java:32)  
    at org.junit.rules.ExpectedException$ExpectedExceptionStatement.evaluate(ExpectedException.java:110)

用 @Rules ExpectedException方式来测试异常，失败时的提示：

java.lang.AssertionError: Expected: (exception with message a string containing "YES. required" and an instance of java.lang.NullPointerException)      got: <cc.unmi.InvalidPasswordException: Password is required.>    at org.junit.Assert.assertThat(Assert.java:778)    at org.junit.Assert.assertThat(Assert.java:736)    at org.junit.rules.ExpectedException$ExpectedExceptionStatement.evaluate(ExpectedException.java:114)

特别是 @Rules  ExpectedException 方法时为何测试失败提示的清清楚楚。期望什么异常，异常 message 中含何字符串，实际上确得到什么类型的异常，异常中 message 是什么。有了这，你一看到就知道怎么去修补你的程序。

所以到了这里，我们真的没理由不用 @Test(expected=...) 或  @Rule ExpectedException 的写法了。

末了，还是补上一段代码，开篇的 try...catch 测试异常的代码是说明别忽略了 catch 块中的 fail() 代码行, 如果没有 fail() 那么就可以写成下面那样子的：

1

2

3

4

5

6

7

8

9

10

11

12

13

14

@Test

public void passwordLengthMoreThan15LettersThrowsException(){

    Throwable t = null;

    try{

        Password.validate("1234567890123456");

    }catch(Exception ex){

        t = ex;

    }

    assertNotNull(t);

    assertTrue(t instanceof InvalidPasswordException);

    assertTrue(t.getMessage().contains("less than 15"));

}

不过，总之呢，有了 JUnit 4 呢，不提 try...catch 的实现方式也罢。

最后附上两个我为这篇博文写的测试代码 [Password.java](http://yanbin.blog/wp-content/downloads/Password.java "Password.java") 和 [PasswordTest.java](http://yanbin.blog/wp-content/downloads/PasswordTest.java "PasswordTest.java")，点击这两文件名即可下载。这里面有种写法的测试用例，文中的代码也这里面也有，可供参考。

参考：1. [JUnit 4.7的新特性：Rule](http://www.infoq.com/cn/news/2009/07/junit-4.7-rules "JUnit 4.7 rules")    
            2. [ExpectedException (JUnit API)](http://kentbeck.github.com/junit/javadoc/latest/org/junit/rules/ExpectedException.html "JUnit ExpectedException")  
            3. [JUnit 4 抢先看  
](http://www.ibm.com/developerworks/cn/java/j-junit4.html "JUnit 4")            4. [JUnit: Custom ExpectedException rules…rule!  
](http://alexruiz.developerblogs.com/?p=1530 "Custom ExpectedException Rules")            5. [Expected Exception Testing With JUnit](http://www.agilefinance.co.uk/2010/10/07/expected-exception-testing-junit/)

本文链接 [https://yanbin.blog/junit-4-how-to-test-exceptions/](https://yanbin.blog/junit-4-how-to-test-exceptions/), 来自 [隔叶黄莺 Yanbin Blog](https://yanbin.blog/)

[版权声明] [![Creative Commons License](media/Creative_Commons_License.png)](https://creativecommons.org/licenses/by-nc-sa/4.0/deed.zh) 本文采用 [署名-非商业性使用-相同方式共享 4.0 国际 (CC BY-NC-SA 4.0)](https://creativecommons.org/licenses/by-nc-sa/4.0/deed.zh) 进行许可。