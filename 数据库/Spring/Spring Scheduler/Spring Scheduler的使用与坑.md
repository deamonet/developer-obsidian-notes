[Spring](http://docs.spring.io/spring/docs/current/spring-framework-reference/html/)给程序猿们带来了许多便利。它包含了用于定时任务处理的[Spring Scheduler](http://docs.spring.io/spring/docs/current/spring-framework-reference/html/scheduling.html)。本文聊聊Scheduling模型和一些坑。  

## [](http://qinghua.github.io/spring-scheduler/#简介 "简介")简介

Spring Scheduler里有两个概念：任务（Task）和运行任务的框架（TaskExecutor/TaskScheduler）。TaskExecutor顾名思义，是任务的执行器，允许我们异步执行多个任务。TaskScheduler是任务调度器，来运行未来的定时任务。触发器Trigger可以决定定时任务是否该运行了，最常用的触发器是CronTrigger，具体用法会在下面章节中详细介绍。Spring内置了多种类型的TaskExecutor和TaskScheduler，方便用户根据不同业务场景选择。

本文聊的是Spring Scheduler，所以我们接下来主要介绍Scheduler的用法。

## [](http://qinghua.github.io/spring-scheduler/#运行 "运行")运行

用xml配置Spring Scheduler的话，这样就行了：  

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

<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"

       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"

       xmlns:task="http://www.springframework.org/schema/task"

       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.0.xsd

                           http://www.springframework.org/schema/task http://www.springframework.org/schema/task/spring-task-4.0.xsd">

    <task:scheduler id="myScheduler"/>

    <task:scheduled-tasks scheduler="myScheduler">

        <task:scheduled ref="doSomethingTask" method="doSomething" cron="0 * * * * *"/>

    </task:scheduled-tasks>

</beans>

当然需要能够reference到doSomethingTask这个实例了。

用注解配置Spring Scheduler的方式如下：  

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

import org.springframework.scheduling.annotation.Scheduled;

import org.springframework.stereotype.Component;

@Component

public class DoSomethingTask {

    @Scheduled(cron="0 * * * * *")

    public void doSomething() {

        System.out.println("do something");

    }

}

参考[这篇短文](http://www.baeldung.com/spring-scheduled-tasks)可以迅速了解。

## [](http://qinghua.github.io/spring-scheduler/#Cron "Cron")Cron

想了解Cron最好的方法是看[Quartz的官方文档](http://www.quartz-scheduler.org/documentation/quartz-2.2.x/tutorials/crontrigger)。本节也会大致介绍一下。

Cron表达式由6~7项组成，中间用空格分开。从左到右依次是：秒、分、时、日、月、周几、年（可省略）。值可以是数字，也可以是以下符号：  
`*`：所有值都匹配  
`?`：无所谓，不关心，通常放在“周几”里  
`,`：或者  
`/`：增量值  
`-`：区间

下面举几个例子，看了就知道了：  
`0 * * * * *`：每分钟（当秒为0的时候）  
`0 0 * * * *`：每小时（当秒和分都为0的时候）  
`*/10 * * * * *`：每10秒  
`0 5/15 * * * *`：每小时的5分、20分、35分、50分  
`0 0 9,13 * * *`：每天的9点和13点  
`0 0 8-10 * * *`：每天的8点、9点、10点  
`0 0/30 8-10 * * *`：每天的8点、8点半、9点、9点半、10点  
`0 0 9-17 * * MON-FRI`：每周一到周五的9点、10点…直到17点（含）  
`0 0 0 25 12 ?`：每年12约25日圣诞节的0点0分0秒（午夜）  
`0 30 10 * * ? 2016`：2016年每天的10点半

其中的`?`在用法上其实和`*`是相同的。但是`*`语义上表示全匹配，而`?`并不代表全匹配，而是不关心。比如对于`0 0 0 5 8 ? 2016`来说，2016年8月5日是周五，`?`表示我不关心它是周几。而`0 0 0 5 8 * 2016`中的`*`表示周一也行，周二也行……语义上和2016年8月5日冲突了，你说谁优先生效呢。

不记得也没关系，记住[Cron Maker](http://www.cronmaker.com/)也可以，它可以在线生成cron表达式。

## [](http://qinghua.github.io/spring-scheduler/#技巧 "技巧")技巧

可能会想要在不同的环境配置不同的cron，比如产品环境就是真实的cron，而测试环境希望更频繁触发，可以设置每分钟都触发。

xml配置的解决方法：  

1

2

3

4

5

6

<task:scheduler id="myScheduler"/>

<task:scheduled-tasks scheduler="myScheduler">

    <task:scheduled ref="doSomethingTask" method="doSomething" cron="${cron_expression}"/>

    <task:scheduled ref="doOtherThingTask" method="doOtherThing" cron="${cron_expression}"/>

</task:scheduled-tasks>

顺便提一句，java的properties文件支持value中带空格，所以可以放心地写`cron_expression=0 * * * * *`。

注解的解决方法：  

1

@Scheduled(cron = "${cron_expression}")

## [](http://qinghua.github.io/spring-scheduler/#坑 "坑")坑

### [](http://qinghua.github.io/spring-scheduler/#同时运行 "同时运行")同时运行

同一个task，如果前一个还没跑完后面一个就不会触发，这没有问题。但是不同的task也不能同时运行就不太合理了。不过其实是scheduler的默认线程数为1的缘故。

解决方法1：如下配置pool-size，但这样会导致同一个task前一个还没跑完后面又被触发的问题。  

1

<task:scheduler id="scheduler" pool-size="2" />

解决方法2：让任务分别运行在不同的scheduler里就好了。  

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

<task:scheduler id="myScheduler1"/>

<task:scheduler id="myScheduler2"/>

<task:scheduled-tasks scheduler="myScheduler1">

    <task:scheduled ref="doSomethingTask" method="doSomething" cron="${0 * * * * *}"/>

</task:scheduled-tasks>

<task:scheduled-tasks scheduler="myScheduler2">

    <task:scheduled ref="doOtherThingTask" method="doOtherThing" cron="${0 * * * * *}"/>

</task:scheduled-tasks>

### [](http://qinghua.github.io/spring-scheduler/#多实例 "多实例")多实例

若是scheduler与web配置在一起，在高可用的情况下，如果有多个web容器实例，scheduler会在多个实例上同时运行。

解决方法1：部署的时候，针对不同实例，使用不同的配置。比如tomcat_1打开scheduler，tomcat_2关闭。带来的问题是：

-   增加部署成本。
-   要是tomcat_1挂了，scheduler就不能运行了，高可用落空。

解决方法2：在task的基类加入一些逻辑，当开始运行时，将状态（运行机器的IP、时间等）写入数据库、缓存或者zk，运行结束时重置状态。其它实例看到有这样的数据，就直接返回。带来的问题是：

-   需要所有实例上的机器时间同步，不然一个刚结束另一个才开始，状态的维护就没有用了。
-   一定要保证结束运行后将状态重置，否则下一个运行周期，所有的task都会返回的。实在不行还得写一个task做这个事。
-   因为读写状态并非原子操作，偶尔也会发生task同时运行的事。

解决方法3：将scheduler与web分开。这样还能避免后台任务影响web端。带来的问题是：

-   增加部署成本。
-   scheduler的高可用需要重新考虑。