[![](media/16dbf1821d122e70~tplv-t2oaga2asx-no-mark!100!100!100!100.awebp.webp)](https://juejin.cn/user/3368559359568696)

[程序新视界 ![lv-5](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2fa3f08a7107485f81157b296fd9d41f~tplv-k3u1fbpfcp-no-mark:0:0:0:0.awebp "创作等级")](https://juejin.cn/user/3368559359568696) 

2021年08月05日 07:38 ·  阅读 13635

几乎在所有的项目中，定时任务的使用都是不可或缺的，如果使用不当甚至会造成资损。还记得多年前在做金融系统时，出款业务是通过定时任务对外打款，当时由于银行接口处理能力有限，外加定时任务使用不当，导致发出大量重复出款请求。还好在后面环节将交易卡在了系统内部，未发生资损。

所以，系统的学习一下定时任务，是非常有必要的。这篇文章就带大家整体梳理学习一下Java领域中常见的几种定时任务实现。

## 线程等待实现

先从最原始最简单的方式来讲解。可以先创建一个thread，然后让它在while循环里一直运行着，通过sleep方法来达到定时任务的效果。

```java
public class Task {

    public static void main(String[] args) {
        // run in a second
        final long timeInterval = 1000;
        Runnable runnable = new Runnable() {
            @Override
            public void run() {
                while (true) {
                    System.out.println("Hello !!");
                    try {
                        Thread.sleep(timeInterval);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        };
        Thread thread = new Thread(runnable);
        thread.start();
    }
}
复制代码
```

这种方式简单直接，但是能够实现的功能有限，而且需要自己来实现。

## JDK自带Timer实现

目前来看，JDK自带的Timer API算是最古老的定时任务实现方式了。Timer是一种定时器工具，用来在一个后台线程计划执行指定任务。它可以安排任务“执行一次”或者定期“执行多次”。

在实际的开发当中，经常需要一些周期性的操作，比如每5分钟执行某一操作等。对于这样的操作最方便、高效的实现方式就是使用java.util.Timer工具类。

### 核心方法

Timer类的核心方法如下：

```scss
// 在指定延迟时间后执行指定的任务
schedule(TimerTask task,long delay);

// 在指定时间执行指定的任务。（只执行一次）
schedule(TimerTask task, Date time);

// 延迟指定时间（delay）之后，开始以指定的间隔（period）重复执行指定的任务
schedule(TimerTask task,long delay,long period);

// 在指定的时间开始按照指定的间隔（period）重复执行指定的任务
schedule(TimerTask task, Date firstTime , long period);

// 在指定的时间开始进行重复的固定速率执行任务
scheduleAtFixedRate(TimerTask task,Date firstTime,long period);

// 在指定的延迟后开始进行重复的固定速率执行任务
scheduleAtFixedRate(TimerTask task,long delay,long period);

// 终止此计时器，丢弃所有当前已安排的任务。
cancal()；

// 从此计时器的任务队列中移除所有已取消的任务。
purge()；
复制代码
```

### 使用示例

下面用几个示例演示一下核心方法的使用。首先定义一个通用的TimerTask类，用于定义用执行的任务。

```scala
public class DoSomethingTimerTask extends TimerTask {

    private String taskName;

    public DoSomethingTimerTask(String taskName) {
        this.taskName = taskName;
    }

    @Override
    public void run() {
        System.out.println(new Date() + " : 任务「" + taskName + "」被执行。");
    }
}
复制代码
```

#### 指定延迟执行一次

在指定延迟时间后执行一次，这类是比较常见的场景，比如：当系统初始化某个组件之后，延迟几秒中，然后进行定时任务的执行。

```typescript
public class DelayOneDemo {

    public static void main(String[] args) {
        Timer timer = new Timer();
        timer.schedule(new DoSomethingTimerTask("DelayOneDemo"),1000L);
    }
}
复制代码
```

执行上述代码，延迟一秒之后执行定时任务，并打印结果。其中第二个参数单位为毫秒。

#### 固定间隔执行

在指定的延迟时间开始执行定时任务，定时任务按照固定的间隔进行执行。比如：延迟2秒执行，固定执行间隔为1秒。

```typescript
public class PeriodDemo {

    public static void main(String[] args) {
        Timer timer = new Timer();
        timer.schedule(new DoSomethingTimerTask("PeriodDemo"),2000L,1000L);
    }
}
复制代码
```

执行程序，会发现2秒之后开始每隔1秒执行一次。

#### 固定速率执行

在指定的延迟时间开始执行定时任务，定时任务按照固定的速率进行执行。比如：延迟2秒执行，固定速率为1秒。

```typescript
public class FixedRateDemo {

    public static void main(String[] args) {
        Timer timer = new Timer();
        timer.scheduleAtFixedRate(new DoSomethingTimerTask("FixedRateDemo"),2000L,1000L);
    }
}
复制代码
```

执行程序，会发现2秒之后开始每隔1秒执行一次。

此时，你是否疑惑schedule与scheduleAtFixedRate效果一样，为什么提供两个方法，它们有什么区别？

### schedule与scheduleAtFixedRate区别

在了解schedule与scheduleAtFixedRate方法的区别之前，先看看它们的相同点：

-   任务执行未超时，下次执行时间 = 上次执行开始时间 + period；
-   任务执行超时，下次执行时间 = 上次执行结束时间；

在任务执行未超时时，它们都是上次执行时间加上间隔时间，来执行下一次任务。而执行超时时，都是立马执行。

它们的不同点在于侧重点不同，schedule方法侧重保持间隔时间的稳定，而scheduleAtFixedRate方法更加侧重于保持执行频率的稳定。

#### schedule侧重保持间隔时间的稳定

schedule方法会因为前一个任务的延迟而导致其后面的定时任务延时。计算公式为scheduledExecutionTime(第n+1次) = realExecutionTime(第n次) + periodTime。

也就是说如果第n次执行task时，由于某种原因这次执行时间过长，执行完后的systemCurrentTime>= scheduledExecutionTime(第n+1次)，则此时不做时隔等待，立即执行第n+1次task。

而接下来的第n+2次task的scheduledExecutionTime(第n+2次)就随着变成了realExecutionTime(第n+1次)+periodTime。这个方法更注重保持间隔时间的稳定。

#### scheduleAtFixedRate保持执行频率的稳定

scheduleAtFixedRate在反复执行一个task的计划时，每一次执行这个task的计划执行时间在最初就被定下来了，也就是scheduledExecutionTime(第n次)=firstExecuteTime +n*periodTime。

如果第n次执行task时，由于某种原因这次执行时间过长，执行完后的systemCurrentTime>= scheduledExecutionTime(第n+1次)，则此时不做period间隔等待，立即执行第n+1次task。

接下来的第n+2次的task的scheduledExecutionTime(第n+2次)依然还是firstExecuteTime+（n+2)*periodTime这在第一次执行task就定下来了。说白了，这个方法更注重保持执行频率的稳定。

如果用一句话来描述任务执行超时之后schedule和scheduleAtFixedRate的区别就是：schedule的策略是错过了就错过了，后续按照新的节奏来走；scheduleAtFixedRate的策略是如果错过了，就努力追上原来的节奏（制定好的节奏）。

### Timer的缺陷

Timer计时器可以定时（指定时间执行任务）、延迟（延迟5秒执行任务）、周期性地执行任务（每隔个1秒执行任务）。但是，Timer存在一些缺陷。首先Timer对调度的支持是基于绝对时间的，而不是相对时间，所以它对系统时间的改变非常敏感。

其次Timer线程是不会捕获异常的，如果TimerTask抛出的了未检查异常则会导致Timer线程终止，同时Timer也不会重新恢复线程的执行，它会错误的认为整个Timer线程都会取消。同时，已经被安排单尚未执行的TimerTask也不会再执行了，新的任务也不能被调度。故如果TimerTask抛出未检查的异常，Timer将会产生无法预料的行为。

## JDK自带ScheduledExecutorService

ScheduledExecutorService是JAVA 1.5后新增的定时任务接口，它是基于线程池设计的定时任务类，每个调度任务都会分配到线程池中的一个线程去执行。也就是说，任务是并发执行，互不影响。

需要注意：只有当执行调度任务时，ScheduledExecutorService才会真正启动一个线程，其余时间ScheduledExecutorService都是出于轮询任务的状态。

ScheduledExecutorService主要有以下4个方法：

```arduino
ScheduledFuture<?> schedule(Runnable command,long delay, TimeUnit unit);
<V> ScheduledFuture<V> schedule(Callable<V> callable,long delay, TimeUnit unit);
ScheduledFuture<?> scheduleAtFixedRate(Runnable command,long initialDelay,long period,TimeUnitunit);
ScheduledFuture<?> scheduleWithFixedDelay(Runnable command,long initialDelay,long delay,TimeUnitunit);
复制代码
```

其中scheduleAtFixedRate和scheduleWithFixedDelay在实现定时程序时比较方便，运用的也比较多。

ScheduledExecutorService中定义的这四个接口方法和Timer中对应的方法几乎一样，只不过Timer的scheduled方法需要在外部传入一个TimerTask的抽象任务。 而ScheduledExecutorService封装的更加细致了，传Runnable或Callable内部都会做一层封装，封装一个类似TimerTask的抽象任务类（ScheduledFutureTask）。然后传入线程池，启动线程去执行该任务。

### scheduleAtFixedRate方法

scheduleAtFixedRate方法，按指定频率周期执行某个任务。定义及参数说明：

```arduino
public ScheduledFuture<?> scheduleAtFixedRate(Runnable command,
				long initialDelay,
				long period,
				TimeUnit unit);
复制代码
```

参数对应含义：command为被执行的线程；initialDelay为初始化后延时执行时间；period为两次开始执行最小间隔时间；unit为计时单位。

使用实例：

```typescript
public class ScheduleAtFixedRateDemo implements Runnable{

    public static void main(String[] args) {
        ScheduledExecutorService executor = Executors.newScheduledThreadPool(1);
        executor.scheduleAtFixedRate(
                new ScheduleAtFixedRateDemo(),
                0,
                1000,
                TimeUnit.MILLISECONDS);
    }

    @Override
    public void run() {
        System.out.println(new Date() + " : 任务「ScheduleAtFixedRateDemo」被执行。");
        try {
            Thread.sleep(2000L);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
复制代码
```

上面是scheduleAtFixedRate方法的基本使用方式，但当执行程序时会发现它并不是间隔1秒执行的，而是间隔2秒执行。

这是因为，scheduleAtFixedRate是以period为间隔来执行任务的，如果任务执行时间小于period，则上次任务执行完成后会间隔period后再去执行下一次任务；但如果任务执行时间大于period，则上次任务执行完毕后会不间隔的立即开始下次任务。

### scheduleWithFixedDelay方法

scheduleWithFixedDelay方法，按指定频率间隔执行某个任务。定义及参数说明：

```arduino
public ScheduledFuture<?> scheduleWithFixedDelay(Runnable command,
				long initialDelay,
				long delay,
				TimeUnit unit);
复制代码
```

参数对应含义：command为被执行的线程；initialDelay为初始化后延时执行时间；period为前一次执行结束到下一次执行开始的间隔时间（间隔执行延迟时间）；unit为计时单位。

使用实例：

```typescript
public class ScheduleAtFixedRateDemo implements Runnable{

    public static void main(String[] args) {
        ScheduledExecutorService executor = Executors.newScheduledThreadPool(1);
        executor.scheduleWithFixedDelay(
                new ScheduleAtFixedRateDemo(),
                0,
                1000,
                TimeUnit.MILLISECONDS);
    }

    @Override
    public void run() {
        System.out.println(new Date() + " : 任务「ScheduleAtFixedRateDemo」被执行。");
        try {
            Thread.sleep(2000L);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
复制代码
```

上面是scheduleWithFixedDelay方法的基本使用方式，但当执行程序时会发现它并不是间隔1秒执行的，而是间隔3秒。

这是因为scheduleWithFixedDelay是不管任务执行多久，都会等上一次任务执行完毕后再延迟delay后去执行下次任务。

## Quartz框架实现

除了JDK自带的API之外，我们还可以使用开源的框架来实现，比如Quartz。

Quartz是Job scheduling（作业调度）领域的一个开源项目，Quartz既可以单独使用也可以跟spring框架整合使用，在实际开发中一般会使用后者。使用Quartz可以开发一个或者多个定时任务，每个定时任务可以单独指定执行的时间，例如每隔1小时执行一次、每个月第一天上午10点执行一次、每个月最后一天下午5点执行一次等。

Quartz通常有三部分组成：调度器（Scheduler）、任务（JobDetail）、触发器（Trigger，包括SimpleTrigger和CronTrigger）。下面以具体的实例进行说明。

### Quartz集成

要使用Quartz，首先需要在项目的pom文件中引入相应的依赖：

```xml
<dependency>
    <groupId>org.quartz-scheduler</groupId>
    <artifactId>quartz</artifactId>
    <version>2.3.2</version>
</dependency>
<dependency>
    <groupId>org.quartz-scheduler</groupId>
    <artifactId>quartz-jobs</artifactId>
    <version>2.3.2</version>
</dependency>
复制代码
```

定义执行任务的Job，这里要实现Quartz提供的Job接口：

```java
public class PrintJob implements Job {
    @Override
    public void execute(JobExecutionContext jobExecutionContext) throws JobExecutionException {
        System.out.println(new Date() + " : 任务「PrintJob」被执行。");
    }
}
复制代码
```

创建Scheduler和Trigger，并执行定时任务：

```scss
public class MyScheduler {

    public static void main(String[] args) throws SchedulerException {
        // 1、创建调度器Scheduler
        SchedulerFactory schedulerFactory = new StdSchedulerFactory();
        Scheduler scheduler = schedulerFactory.getScheduler();
        // 2、创建JobDetail实例，并与PrintJob类绑定(Job执行内容)
        JobDetail jobDetail = JobBuilder.newJob(PrintJob.class)
                .withIdentity("job", "group").build();
        // 3、构建Trigger实例，每隔1s执行一次
        Trigger trigger = TriggerBuilder.newTrigger().withIdentity("trigger", "triggerGroup")
                .startNow()//立即生效
                .withSchedule(SimpleScheduleBuilder.simpleSchedule()
                        .withIntervalInSeconds(1)//每隔1s执行一次
                        .repeatForever()).build();//一直执行

        //4、Scheduler绑定Job和Trigger，并执行
        scheduler.scheduleJob(jobDetail, trigger);
        System.out.println("--------scheduler start ! ------------");
        scheduler.start();
    }
}
复制代码
```

执行程序，可以看到每1秒执行一次定时任务。

在上述代码中，其中Job为Quartz的接口，业务逻辑的实现通过实现该接口来实现。

JobDetail绑定指定的Job，每次Scheduler调度执行一个Job的时候，首先会拿到对应的Job，然后创建该Job实例，再去执行Job中的execute()的内容，任务执行结束后，关联的Job对象实例会被释放，且会被JVM GC清除。

Trigger是Quartz的触发器，用于通知Scheduler何时去执行对应Job。SimpleTrigger可以实现在一个指定时间段内执行一次作业任务或一个时间段内多次执行作业任务。

CronTrigger功能非常强大，是基于日历的作业调度，而SimpleTrigger是精准指定间隔，所以相比SimpleTrigger，CroTrigger更加常用。CroTrigger是基于Cron表达式的。

常见的Cron表达式示例如下：

![cron](media/cron.webp)

可以看出，基于Quartz的CronTrigger可以实现非常丰富的定时任务场景。

## Spring Task

从Spring 3开始，Spring自带了一套定时任务工具Spring-Task，可以把它看成是一个轻量级的Quartz，使用起来十分简单，除Spring相关的包外不需要额外的包，支持注解和配置文件两种形式。通常情况下在Spring体系内，针对简单的定时任务，可直接使用Spring提供的功能。

基于XML配置文件的形式就不再介绍了，直接看基于注解形式的实现。使用起来非常简单，直接上代码：

```typescript
@Component("taskJob")
public class TaskJob {

    @Scheduled(cron = "0 0 3 * * ?")
    public void job1() {
        System.out.println("通过cron定义的定时任务");
    }

    @Scheduled(fixedDelay = 1000L)
    public void job2() {
        System.out.println("通过fixedDelay定义的定时任务");
    }

    @Scheduled(fixedRate = 1000L)
    public void job3() {
        System.out.println("通过fixedRate定义的定时任务");
    }
}
复制代码
```

如果是在Spring Boot项目中，需要在启动类上添加@EnableScheduling来开启定时任务。

上述代码中，@Component用于实例化类，这个与定时任务无关。@Scheduled指定该方法是基于定时任务进行执行，具体执行的频次是由cron指定的表达式所决定。关于cron表达式上面CronTrigger所使用的表达式一致。与cron对照的，Spring还提供了fixedDelay和fixedRate两种形式的定时任务执行。

### fixedDelay和fixedRate的区别

fixedDelay和fixedRate的区别于Timer中的区别很相似。

fixedRate有一个时刻表的概念，在任务启动时，T1、T2、T3就已经排好了执行的时刻，比如1分、2分、3分，当T1的执行时间大于1分钟时，就会造成T2晚点，当T1执行完时T2立即执行。

fixedDelay比较简单，表示上个任务结束，到下个任务开始的时间间隔。无论任务执行花费多少时间，两个任务间的间隔始终是一致的。

### Spring Task的缺点

Spring Task 本身不支持持久化，也没有推出官方的分布式集群模式，只能靠开发者在业务应用中自己手动扩展实现，无法满足可视化，易配置的需求。

## 分布式任务调度

以上定时任务方案都是针对单机的，只能在单个JVM进程中使用。而现在基本上都是分布式场景，需要一套在分布式环境下高性能、高可用、可扩展的分布式任务调度框架。

### Quartz分布式

首先，Quartz是可以用于分布式场景的，但需要基于数据库锁的形式。简单来说，quartz的分布式调度策略是以数据库为边界的一种异步策略。各个调度器都遵守一个基于数据库锁的操作规则从而保证了操作的唯一性，同时多个节点的异步运行保证了服务的可靠。

因此，Quartz的分布式方案只解决了任务高可用（减少单点故障）的问题，处理能力瓶颈在数据库，而且没有执行层面的任务分片，无法最大化效率，只能依靠shedulex调度层面做分片，但是调度层做并行分片难以结合实际的运行资源情况做最优的分片。

### 轻量级神器XXL-Job

XXL-JOB是一个轻量级分布式任务调度平台。特点是平台化，易部署，开发迅速、学习简单、轻量级、易扩展。由调度中心和执行器功能完成定时任务的执行。调度中心负责统一调度，执行器负责接收调度并执行。

针对于中小型项目，此框架运用的比较多。

### 其他框架

除此之外，还有Elastic-Job、Saturn、SIA-TASK等。

Elastic-Job具有高可用的特性，是一个分布式调度解决方案。

Saturn是唯品会开源的一个分布式任务调度平台，在Elastic Job的基础上进行了改造。

SIA-TASK是宜信开源的分布式任务调度平台。

## 小结

通过本文梳理了6种定时任务的实现，就实践场景的运用来说，目前大多数系统已经脱离了单机模式。对于并发量并不是太高的系统，xxl-job或许是一个不错的选择。

源码地址：[github.com/secbr/java-…](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsecbr%2Fjava-schedule "https://github.com/secbr/java-schedule")

> 博主简介：《SpringBoot技术内幕》技术图书作者，酷爱钻研技术，写技术干货文章。
> 
> 公众号：「程序新视界」，博主的公众号，欢迎关注~
> 
> 技术交流：请联系博主微信号：zhuan2quan