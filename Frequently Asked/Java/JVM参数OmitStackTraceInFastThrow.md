
# 异常栈信息不见了之JVM参数OmitStackTraceInFastThrow

## 问题描述

某天收到生产环境error日志告警（对error.log监控，超过一定大小就会给开发人员发送告警短信）。但是tail查看最新的异常信息只有这些，好忧伤：

```erlang
... ...java.lang.NullPointerExceptionjava.lang.NullPointerExceptionjava.lang.NullPointerExceptionjava.lang.NullPointerExceptionjava.lang.NullPointerExceptionjava.lang.NullPointerException... ...
```

后来有个同事从error.log前面开始看起，能看到完整的异常栈信息，大致如下，这样的信息就能够准确的定位问题：

```cobol
... ...java.lang.NullPointerException    at com.afei.juc.WithNPESimulate.run(NPEMain.java:21)    at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)    at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)    at java.lang.Thread.run(Thread.java:745)java.lang.NullPointerException    at com.afei.juc.WithNPESimulate.run(NPEMain.java:21)    at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)    at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)    at java.lang.Thread.run(Thread.java:745)... ...
```

那么前面那段只有简单的**java.lang.NullPointerException**，没有详细异常栈信息的原因是什么呢？这需要从一个[JVM](https://so.csdn.net/so/search?q=JVM&spm=1001.2101.3001.7020)参数说起。

## 原因分析

JVM中有个参数：**OmitStackTraceInFastThrow**，字面意思是**省略异常栈信息从而快速抛出**，那么JVM是如何做到快速抛出的呢？JVM对一些特定的异常类型做了**Fast Throw**优化，如果检测到在代码里某个位置连续多次抛出同一类型异常的话，C2会决定用**Fast Throw**方式来抛出异常，而异常Trace即详细的异常栈信息会被清空。这种异常抛出速度非常快，因为不需要在堆里分配内存，也不需要构造完整的异常栈信息。相关的源码的JVM源码的**graphKit.cpp**文件中，相关源码如下：

```rust
//------------------------------builtin_throw----------------------------------void GraphKit::builtin_throw(Deoptimization::DeoptReason reason, Node* arg) {  bool must_throw = true;   ... ...  // 首先判断条件是否满足  // If this particular condition has not yet happened at this  // bytecode, then use the uncommon trap mechanism, and allow for  // a future recompilation if several traps occur here.  // If the throw is hot（表示在代码某个位置重复抛出异常）, try to use a more complicated inline mechanism  // which keeps execution inside the compiled code.  bool treat_throw_as_hot = false;   if (ProfileTraps) {    if (too_many_traps(reason)) {      treat_throw_as_hot = true;    }    // (If there is no MDO at all, assume it is early in    // execution, and that any deopts are part of the    // startup transient, and don't need to be remembered.)     // Also, if there is a local exception handler, treat all throws    // as hot if there has been at least one in this method.    if (C->trap_count(reason) != 0        && method()->method_data()->trap_count(reason) != 0        && has_ex_handler()) {        treat_throw_as_hot = true;    }  }   // If this throw happens frequently, an uncommon trap might cause  // a performance pothole.  If there is a local exception handler,  // and if this particular bytecode appears to be deoptimizing often,  // let us handle the throw inline, with a preconstructed instance.  // Note:   If the deopt count has blown up, the uncommon trap  // runtime is going to flush this nmethod, not matter what.  // 这里要满足两个条件：1.检测到频繁抛出异常，2. OmitStackTraceInFastThrow为true，或StackTraceInThrowable为false  if (treat_throw_as_hot      && (!StackTraceInThrowable || OmitStackTraceInFastThrow)) {    // If the throw is local, we use a pre-existing instance and    // punt on the backtrace.  This would lead to a missing backtrace    // (a repeat of 4292742) if the backtrace object is ever asked    // for its backtrace.    // Fixing this remaining case of 4292742 requires some flavor of    // escape analysis.  Leave that for the future.    ciInstance* ex_obj = NULL;    switch (reason) {    case Deoptimization::Reason_null_check:      ex_obj = env()->NullPointerException_instance();      break;    case Deoptimization::Reason_div0_check:      ex_obj = env()->ArithmeticException_instance();      break;    case Deoptimization::Reason_range_check:      ex_obj = env()->ArrayIndexOutOfBoundsException_instance();      break;    case Deoptimization::Reason_class_check:      if (java_bc() == Bytecodes::_aastore) {        ex_obj = env()->ArrayStoreException_instance();      } else {        ex_obj = env()->ClassCastException_instance();      }      break;    }    ... ...}
```

> 说明：**OmitStackTraceInFastThrow**和**StackTraceInThrowable**都默认为true，所以条件`(!StackTraceInThrowable || OmitStackTraceInFastThrow)`为true，即JVM默认开启了**Fast Throw**优化。如果想关闭这个优化，很简单，配置`-XX:-OmitStackTraceInFastThrow`，**StackTraceInThrowable**保持默认配置`-XX:+OmitStackTraceInFastThrow`即可。

另外，根据这段源码的`switch .. case ..`部分可知，JVM只对几个特定类型异常开启了**Fast Throw**优化，这些异常包括：

- **NullPointerException**
- **ArithmeticException**
- **ArrayIndexOutOfBoundsException**
- **ArrayStoreException**
- **ClassCastException**

## 问题验证

为了验证这个问题，笔者写了下面这段代码：

```cobol
/** * @author afei * @version 1.0.0 * @since 2018年06月08日 */class WithNPE extends Thread{     private static int count = 0;     @Override    public void run() {        try{            System.out.println(this.getClass().getSimpleName()+"--"+(++count));            String str = null;            // 制造空指针NPE            System.out.println(str.length());        }catch (Throwable e){            e.printStackTrace();        }    }} public class FastThrowMain {    public static void main(String[] args) throws InterruptedException {        WithNPE withNPE = new WithNPE();        ExecutorService executorService = Executors.newFixedThreadPool(20);        for (int i=0; i<Integer.MAX_VALUE; i++) {            executorService.execute(withNPE);            // 稍微sleep一下, 是为了不要让异常抛出太快, 导致控制台输出太快, 把有异常栈信息冲掉, 只留下fast throw方式抛出的异常            Thread.sleep(2);        }    }}
```

运行部分日志如下：

```cobol
WithNPE--6686... ...java.lang.NullPointerException    at com.afei.juc.WithNPE.run(FastThrowMain.java:21)    at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)    at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)    at java.lang.Thread.run(Thread.java:745)java.lang.NullPointerException    at com.afei.juc.WithNPE.run(FastThrowMain.java:21)    at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)    at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)    at java.lang.Thread.run(Thread.java:745)java.lang.NullPointerException    at com.afei.juc.WithNPE.run(FastThrowMain.java:21)    at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)    at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)    at java.lang.Thread.run(Thread.java:745)WithNPE--6687WithNPE--6688WithNPE--6689java.lang.NullPointerExceptionjava.lang.NullPointerExceptionjava.lang.NullPointerExceptionWithNPE--6690WithNPE--6691WithNPE--6692java.lang.NullPointerExceptionjava.lang.NullPointerExceptionjava.lang.NullPointerException... ...
```

> 从这段日志可知，抛出了几千次带有详细异常栈信息的异常后，只会抛出**java.lang.NullPointerException**这种没有详细异常栈信息只有异常类型的异常信息。这就是**Fast Throw**优化后抛出的异常。如果我们配置了`-XX:-OmitStackTraceInFastThrow`，再次运行，就不会看到**Fast Throw**优化后抛出的异常，全是包含了详细异常栈的异常信息。

配置JVM参数关闭**Fast Throw**后，即使抛出了2w+次异常，依然全是包含了详细异常栈的异常信息，日志如下：

```cobol
WithNPE--20719WithNPE--20720java.lang.NullPointerException    at com.afei.juc.WithNPE.run(FastThrowMain.java:21)    at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)    at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)    at java.lang.Thread.run(Thread.java:745)java.lang.NullPointerException    at com.afei.juc.WithNPE.run(FastThrowMain.java:21)    at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)    at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)    at java.lang.Thread.run(Thread.java:745
```


# [When does JVM start to omit stack traces?](https://stackoverflow.com/questions/58696093/when-does-jvm-start-to-omit-stack-traces)

- 1
    
    Performance. Will happen if it's been generating the same stack trace over and over.
    
    – [Michael](https://stackoverflow.com/users/1898563/michael "43,830 reputation")
    
    [Commented Nov 4, 2019 at 15:09](https://stackoverflow.com/questions/58696093/when-does-jvm-start-to-omit-stack-traces#comment103688736_58696093)
    

- Re: the extreme case. I think if the JVM is generating an out of memory error, for example, it may not even have enough memory to construct the stack trace, so it has to be empty.
    
    – [Michael](https://stackoverflow.com/users/1898563/michael "43,830 reputation")
    
    [Commented Nov 4, 2019 at 15:15](https://stackoverflow.com/questions/58696093/when-does-jvm-start-to-omit-stack-traces#comment103688946_58696093)
    
- If the memory consumption is that critical, it is better off to throw OOM than "silently" suppressing the stack trace, right?
    
    – [Mohamed Anees A](https://stackoverflow.com/users/6140314/mohamed-anees-a "4,501 reputation")
    
    [Commented Nov 4, 2019 at 15:19](https://stackoverflow.com/questions/58696093/when-does-jvm-start-to-omit-stack-traces#comment103689071_58696093)
    
- 2
    
    "_Isn't it wise always to do that in Production machines?_" No. Our prod machines have this feature enabled. We care about the performance gain more than we care about getting the full stack trace every time. If it is being omitted, it means that it's been generated already. To get the full stack trace, just go to the first occurrence in your logs.
    
    – [Michael](https://stackoverflow.com/users/1898563/michael "43,830 reputation")
    
    [Commented Nov 4, 2019 at 15:21](https://stackoverflow.com/questions/58696093/when-does-jvm-start-to-omit-stack-traces#comment103689119_58696093)
    
- I don't know what you mean. OOM exception is always thrown. It's just whether the OOM that is thrown includes the stack trace or not. If there is no memory to generate it, there is no option but to throw it without.
    
    – [Michael](https://stackoverflow.com/users/1898563/michael "43,830 reputation")
    
    [Commented Nov 4, 2019 at 15:24](https://stackoverflow.com/questions/58696093/when-does-jvm-start-to-omit-stack-traces#comment103689217_58696093)
    

[Show **1** more comment](https://stackoverflow.com/questions/58696093/when-does-jvm-start-to-omit-stack-traces# "Expand to show all comments on this post")

## 1 Answer

Sorted by:

29

[](https://stackoverflow.com/posts/58700744/timeline)

1. `OmitStackTraceInFastThrow` is an optimization in **C2 compiled** code to throw certain implicit exceptions without stack traces.
2. The optimization applies only to **implicit** exceptions, i.e. exceptions thrown by JVM itself, not by user code. These implicit exceptions are:
    - NullPointerException
    - ArithmeticException
    - ArrayIndexOutOfBoundsException
    - ArrayStoreException
    - ClassCastException
3. Stack traces are omitted, only if JVM knows the exception has happened earlier at this particular place.

So, in HotSpot JVM an exception won't have a stack trace, if all of the following conditions are met: 1) a throwing method is hot, i.e. compiled by C2; 2) it is an implicit exception, e.g. NPE thrown by `obj.method()`, but not by `throw new NullPointerException()`; 3) an exception is thrown at least for the second time.

The purpose of the optimization is to eliminate performance impact of those (arguably rare) cases when an implicit exception is repeatedly thrown on the fast path. In my opinion, this isn't a normal situation. Exceptions, especially implicit, typically denote an error condition which needs to be fixed. In this sense, it's OK to disable `-XX:-OmitStackTraceInFastThrow`. E.g. in our production environment we always disable it, and it saved us much debugging time. However, I admit that if such an optimization exists, there were cases where it helped to solve performance issues.

**TL;DR** Adding `-XX:-OmitStackTraceInFastThrow` option can be indeed a good idea, unless there are many implicit exceptions on a hot path in your application.