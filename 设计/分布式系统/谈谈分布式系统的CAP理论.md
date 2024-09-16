# 谈谈分布式系统的CAP理论

[![崔同学](media/崔同学.jpg)](https://www.zhihu.com/people/cui-chang-ze)

[崔同学](https://www.zhihu.com/people/cui-chang-ze)

AWS搬砖

​关注他

429 人赞同了该文章

## 动机

CAP理论作为分布式系统的基石，应该是每个入门分布式系统（包括区块链）的人都应该学习的内容，本文是我在学习本理论的一个记录，分享出来以节省大家查资料时间。

## 版权

本文转载于

[HollisChuang's Blog​www.hollischuang.com/archives/666](http://www.hollischuang.com/archives/666)

作者HollisChuang

我仅对文章的排版和部分文字进行了润色

---

> 2000年7月，加州大学伯克利分校的Eric Brewer教授在ACM PODC会议上提出CAP猜想。2年后，麻省理工学院的Seth Gilbert和Nancy Lynch从理论上证明了CAP。之后，CAP理论正式成为分布式计算领域的公认定理。

## **CAP理论概述**

**一个分布式系统最多只能同时满足一致性（Consistency）、可用性（Availability）和分区容错性（Partition tolerance）这三项中的两项**。

  

![](media/v2-2f26a48f5549c2bc4932fdf88ba4f72f_720w.webp.jpg)

  

## **CAP的定义**

## **Consistency 一致性**

一致性指“`all nodes see the same data at the same time`”，即所有节点在同一时间的数据完全一致。

一致性是因为多个数据拷贝下并发读写才有的问题，因此理解时一定要注意结合考虑多个数据拷贝下并发读写的场景。

  

对于一致性，可以分为从客户端和服务端两个不同的视角。

-   客户端

从客户端来看，一致性主要指的是多并发访问时更新过的数据如何获取的问题。

-   服务端

从服务端来看，则是更新如何分布到整个系统，以保证数据最终一致。

  

对于一致性，可以分为强/弱/最终一致性三类

从客户端角度，多进程并发访问时，更新过的数据在不同进程如何获取的不同策略，决定了不同的一致性。

-   强一致性

对于关系型数据库，要求更新过的数据能被后续的访问都能看到，这是强一致性。

-   弱一致性

如果能容忍后续的部分或者全部访问不到，则是弱一致性。

-   最终一致性

如果经过一段时间后要求能访问到更新后的数据，则是最终一致性。

## **Availability 可用性**

可用性指“`Reads and writes always succeed`”，即服务在正常响应时间内一直可用。

好的可用性主要是指系统能够很好的为用户服务，不出现用户操作失败或者访问超时等用户体验不好的情况。可用性通常情况下可用性和分布式数据冗余，负载均衡等有着很大的关联。

## **Partition Tolerance分区容错性**

分区容错性指“`the system continues to operate despite arbitrary message loss or failure of part of the system`”，即分布式系统在遇到某节点或网络分区故障的时候，仍然能够对外提供满足一致性或可用性的服务。

## **CAP的证明**

![](media/v2-fcd823804add6168202abe11994fd1d0_720w.webp.png)

上图是我们证明CAP的基本场景，网络中有两个节点N1和N2，可以简单的理解N1和N2分别是两台计算机，他们之间网络可以连通，N1中有一个应用程序A，和一个数据库V，N2也有一个应用程序B2和一个数据库V。现在，A和B是分布式系统的两个部分，V是分布式系统的数据存储的两个子数据库。

在满足一致性的时候，N1和N2中的数据是一样的，V0=V0。在满足可用性的时候，用户不管是请求N1或者N2，都会得到立即响应。在满足分区容错性的情况下，N1和N2有任何一方宕机，或者网络不通的时候，都不会影响N1和N2彼此之间的正常运作。

![](media/v2-3a4f63d521b0286b8360d1a6e8352d73_720w.webp.png)

上图是分布式系统正常运转的流程，用户向N1机器请求数据更新，程序A更新数据库Vo为V1，分布式系统将数据进行同步操作M，将V1同步的N2中V0，使得N2中的数据V0也更新为V1，N2中的数据再响应N2的请求。

这里，可以定义N1和N2的数据库V之间的数据是否一样为一致性；外部对N1和N2的请求响应为可用性；N1和N2之间的网络环境为分区容错性。

这是正常运作的场景，也是理想的场景，然而现实是残酷的，当错误发生的时候，一致性和可用性还有分区容错性，是否能同时满足，还是说要进行取舍呢？

作为一个分布式系统，它和单机系统的最大区别，就在于网络，现在假设一种极端情况，N1和N2之间的网络断开了，我们要支持这种网络异常，相当于要满足分区容错性，能不能同时满足一致性和响应性呢？还是说要对他们进行取舍。

![](media/v2-54009ad0fc14e0d4d9a45429cfbe4869_720w.webp.png)

假设在N1和N2之间网络断开的时候，有用户向N1发送数据更新请求，那N1中的数据V0将被更新为V1，由于网络是断开的，所以分布式系统同步操作M，所以N2中的数据依旧是V0；这个时候，有用户向N2发送数据读取请求，由于数据还没有进行同步，应用程序没办法立即给用户返回最新的数据V1，怎么办呢？

有二种选择，第一，牺牲数据一致性，响应旧的数据V0给用户；第二，牺牲可用性，阻塞等待，直到网络连接恢复，数据更新操作M完成之后，再给用户响应最新的数据V1。

这个过程，证明了要满足分区容错性的分布式系统，只能在一致性和可用性两者中，选择其中一个。

## **CAP权衡**

通过CAP理论，我们知道无法同时满足一致性、可用性和分区容错性这三个特性，那要舍弃哪个呢？

> CA without P：如果不要求P（不允许分区），则C（强一致性）和A（可用性）是可以保证的。但其实分区不是你想不想的问题，而是始终会存在，因此CA的系统更多的是允许分区后各子系统依然保持CA。  
> CP without A：如果不要求A（可用），相当于每个请求都需要在Server之间强一致，而P（分区）会导致同步时间无限延长，如此CP也是可以保证的。很多传统的数据库分布式事务都属于这种模式。  
> AP wihtout C：要高可用并允许分区，则需放弃一致性。一旦分区发生，节点之间可能会失去联系，为了高可用，每个节点只能用本地数据提供服务，而这样会导致全局数据的不一致性。现在众多的NoSQL都属于此类。

对于多数大型互联网应用的场景，主机众多、部署分散，而且现在的集群规模越来越大，所以节点故障、网络故障是常态，而且要保证服务可用性达到N个9，即保证P和A，舍弃C（退而求其次保证最终一致性）。虽然某些地方会影响客户体验，但没达到造成用户流程的严重程度。

对于涉及到钱财这样不能有一丝让步的场景，C必须保证。网络发生故障宁可停止服务，这是保证CA，舍弃P。貌似这几年国内银行业发生了不下10起事故，但影响面不大，报道也不多，广大群众知道的少。还有一种是保证CP，舍弃A。例如网络故障事只读不写。

孰优孰略，没有定论，只能根据场景定夺，适合的才是最好的。

编辑于 2018-04-16 20:35

[

分布式系统

](https://www.zhihu.com/topic/19570823)

[

区块链(Blockchain)

](https://www.zhihu.com/topic/19901773)

[

并发并行与分布式系统

](https://www.zhihu.com/topic/19827601)

​赞同 429​​32 条评论

​分享

​喜欢​收藏​申请转载

​

![](media/v2-c1ccd457ef0518c33a4ce0a9bf1d4fa9_l.jpg)

写下你的评论...

  

32 条评论

默认

最新

[![IT之家刺客](media/IT之家刺客.jpg)](https://www.zhihu.com/people/2737cad2a5b18d751f4252b27dd1e5fb)

[IT之家刺客](https://www.zhihu.com/people/2737cad2a5b18d751f4252b27dd1e5fb)

分区容错性就没看懂过![[飙泪笑]](media/[飙泪笑].png)![[飙泪笑]](media/[飙泪笑].png)![[飙泪笑]](media/[飙泪笑].png)

2019-09-01

​回复​15

[![not bad](media/not_bad.jpg)](https://www.zhihu.com/people/47f0b96d6c6ab88a0dfdfc5b0a44dde4)

[not bad](https://www.zhihu.com/people/47f0b96d6c6ab88a0dfdfc5b0a44dde4)

CA without P 不就是单机系统了么？

2019-09-04

​回复​11

[![殷振南](media/殷振南.jpg)](https://www.zhihu.com/people/f1d7a31f103b7afdc9423caae5c10012)

[殷振南](https://www.zhihu.com/people/f1d7a31f103b7afdc9423caae5c10012)

![](media/v2-4812630bc27d642f7cafcd6cdeca3d7a.jpg.png)

P是需要容忍的，而不是保证的。 CAP理论讨论的就是集群的问题。单节点相当于不存在P的问题，但是完全无法满足A。A的保证本身就是通过集群冗余容灾实现的。

08-24

​回复​赞

[![DsyDemo](media/DsyDemo.jpg)](https://www.zhihu.com/people/298a1814c1aba0a09cfbbc68dfbbcd93)

[DsyDemo](https://www.zhihu.com/people/298a1814c1aba0a09cfbbc68dfbbcd93)

![](media/v2-4812630bc27d642f7cafcd6cdeca3d7a.jpg.png)

[WastonYuan](https://www.zhihu.com/people/8fd90530aa1d89ecb46ae37864275c1e)

不是这么回事啊，网络就算没有故障也会有延迟

2021-12-08

​回复​赞

查看全部 7 条回复​

[![Old Driver](media/Old_Driver.jpg)](https://www.zhihu.com/people/0dd9bc1dafe2ae57fe77c4430fb63276)

[Old Driver](https://www.zhihu.com/people/0dd9bc1dafe2ae57fe77c4430fb63276)

"对于涉及到钱财这样不能有一丝让步的场景，C必须保证。网络发生故障宁可停止服务，这是保证CA，舍弃P"。这句话是有问题的，停止服务后，只是保证了C，而不是CA。

2018-07-25

​回复​11

[![大神带我来搬砖](media/大神带我来搬砖.jpg)](https://www.zhihu.com/people/8cdfd73cb465821cac650a311c9ca8e6)

[大神带我来搬砖](https://www.zhihu.com/people/8cdfd73cb465821cac650a311c9ca8e6)

[像风](https://www.zhihu.com/people/7f26e1f2a4649954f470215bf0cef6da)

我看了原文是“网络发生故障宁可停止服务，这是保证CP，舍弃A。”

2019-04-28

​回复​5

[![像风](media/像风.jpg)](https://www.zhihu.com/people/7f26e1f2a4649954f470215bf0cef6da)

[像风](https://www.zhihu.com/people/7f26e1f2a4649954f470215bf0cef6da)

对啊，我也有这个疑问。你都停止服务了，谈何可用性？

2018-09-16

​回复​2

展开其他 3 条回复​

[![达](media/达.jpg)](https://www.zhihu.com/people/6122f437644534b4321259d35ebd2c7d)

[达](https://www.zhihu.com/people/6122f437644534b4321259d35ebd2c7d)

P 是几乎绝大多数系统都要保证的，也是分布式系统强调的，所以通常分布式系统是在 AP 和 CP 之间选择，比如阿里的 Tair 支持 AP 和 CP 的用户选择。

2020-10-22

​回复​2

[![WastonYuan](media/WastonYuan.jpg)](https://www.zhihu.com/people/8fd90530aa1d89ecb46ae37864275c1e)

[WastonYuan](https://www.zhihu.com/people/8fd90530aa1d89ecb46ae37864275c1e)

read/write majority就是AP 所有节点都写完再返回就是CP 在mongodb就一个配置的问题

2020-11-01

​回复​1

[![赵老师](media/赵老师.jpg)](https://www.zhihu.com/people/ae8905e11250216264a5bc961935ac07)

[赵老师](https://www.zhihu.com/people/ae8905e11250216264a5bc961935ac07)

一致性不是并发读写带来的问题，而是数据多个拷贝带来的

2018-04-15

​回复​4

[![崔同学](media/崔同学.jpg)](https://www.zhihu.com/people/534d903a271b42dc39cc5963acf283f3)

[崔同学](https://www.zhihu.com/people/534d903a271b42dc39cc5963acf283f3)

作者

感谢您的提醒，我已经做了更新。谢谢！

2018-04-16

​回复​赞

[![李瓜皮](media/李瓜皮.jpg)](https://www.zhihu.com/people/5b4f7791f0f9ba3555d58ae9161aa13d)

[李瓜皮](https://www.zhihu.com/people/5b4f7791f0f9ba3555d58ae9161aa13d)

崔爷6啊

2018-02-25

​回复​2

[![WastonYuan](media/WastonYuan.jpg)](https://www.zhihu.com/people/8fd90530aa1d89ecb46ae37864275c1e)

[WastonYuan](https://www.zhihu.com/people/8fd90530aa1d89ecb46ae37864275c1e)

P我理解是发生 网络故障的概率 ！！！而不是出现网络故障后的容忍度，这里翻译问题会导致很多人理解错。

2020-11-01

​回复​2

[![ask-x](media/ask-x.jpg)](https://www.zhihu.com/people/7cedb0a8821cf0a49803648e97b84e09)

[ask-x](https://www.zhihu.com/people/7cedb0a8821cf0a49803648e97b84e09)

[mp.weixin.qq.com/s?](http://link.zhihu.com/?target=https%3A//mp.weixin.qq.com/s%3F__biz%3DMzUxMzcwNzc0NQ%3D%3D%26mid%3D2247483696%26idx%3D1%26sn%3D27079cd811fbfc60f76d8d432f176c68%26chksm%3Df9505abece27d3a81d0f55a2044f340749d149a702e08db46709bc86cfa8a0c5e7fc3ca37a50%26token%3D481167576%26lang%3Dzh_CN%23rd)

2019-09-14

​回复​1

[![大神带我来搬砖](media/大神带我来搬砖.jpg)](https://www.zhihu.com/people/8cdfd73cb465821cac650a311c9ca8e6)

[大神带我来搬砖](https://www.zhihu.com/people/8cdfd73cb465821cac650a311c9ca8e6)

CAP的证明有更严格的形式么？谁能解答一下啊？[jianshu.com/p/6d6ccf9f9](http://link.zhihu.com/?target=https%3A//www.jianshu.com/p/6d6ccf9f9305)

2019-04-28

​回复​1

[![WastonYuan](media/WastonYuan.jpg)](https://www.zhihu.com/people/8fd90530aa1d89ecb46ae37864275c1e)

[WastonYuan](https://www.zhihu.com/people/8fd90530aa1d89ecb46ae37864275c1e)

patition tolerance正确翻译：网络容忍度 -> 防止网络出现分区的能力(容忍度越低，就越不能让网络发生故障，越高就随便啦) -> 网络发生故障的概率！

2020-11-01

​回复​1

[![Ethan楚翔](media/Ethan楚翔.jpg)](https://www.zhihu.com/people/d75b754b2ab16e3d90b87bb53f5899e4)

[Ethan楚翔](https://www.zhihu.com/people/d75b754b2ab16e3d90b87bb53f5899e4)

![](media/v2-4812630bc27d642f7cafcd6cdeca3d7a.jpg.png)

楼主,分区容错性可以理解成集群的HA吗?

2020-05-20

​回复​赞

[![sigure](media/sigure.jpg)](https://www.zhihu.com/people/9b6f4d25bf7add17e51dcd8f69ea6089)

[sigure](https://www.zhihu.com/people/9b6f4d25bf7add17e51dcd8f69ea6089)

我也觉得钱财的那部分应该是保证CP舍弃A，其实就是分布式共识算法的思路。比如raft的实现上就是会导致请求有可能失败或者说请求没有被同步的

2020-04-14

​回复​赞

[![GoGeeker](media/GoGeeker.jpg)](https://www.zhihu.com/people/e985a689106a61881d1a90663f4a006c)

[GoGeeker](https://www.zhihu.com/people/e985a689106a61881d1a90663f4a006c)

![](media/v2-4812630bc27d642f7cafcd6cdeca3d7a.jpg.png)

简单易懂，谢谢╰(*´︶`*)╯

2019-12-19

​回复​赞

[![你好凯蒂](media/你好凯蒂.jpg)](https://www.zhihu.com/people/1d5f48a4d99e0b1d9107338a1cfb8577)

[你好凯蒂](https://www.zhihu.com/people/1d5f48a4d99e0b1d9107338a1cfb8577)

![](media/v2-4812630bc27d642f7cafcd6cdeca3d7a.jpg.png)

说得很好收藏了

2019-08-13

​回复​赞

[![我莫得感情](media/我莫得感情.jpg)](https://www.zhihu.com/people/6c80e06c9582d849f4811ab5bcfd2aaa)

[我莫得感情](https://www.zhihu.com/people/6c80e06c9582d849f4811ab5bcfd2aaa)

牛蛙牛蛙 学习了

2021-08-30

​回复​赞

[![崔同学](media/崔同学.jpg)](https://www.zhihu.com/people/534d903a271b42dc39cc5963acf283f3)

[崔同学](https://www.zhihu.com/people/534d903a271b42dc39cc5963acf283f3)

作者

666

2018-02-25

​回复​赞

[![Lyoung](media/Lyoung.jpg)](https://www.zhihu.com/people/cb2880d9ba5dbf0a3742b1c2f722d77f)

[Lyoung](https://www.zhihu.com/people/cb2880d9ba5dbf0a3742b1c2f722d77f)

这润色润的...，还是原文更通顺

2021-04-07

​回复​赞

[![忘月幽](media/忘月幽.jpg)](https://www.zhihu.com/people/fee3fd1c4874405704060cba4cfdf218)

[忘月幽](https://www.zhihu.com/people/fee3fd1c4874405704060cba4cfdf218)

讲的很清楚 感谢

2020-07-23

​回复​赞

![](media/v2-c1ccd457ef0518c33a4ce0a9bf1d4fa9_l.jpg)

写下你的评论...

  

### 文章被以下专栏收录

[

![一枚上进的程序员](media/一枚上进的程序员.jpg)

](https://www.zhihu.com/column/c_1243484041787707392)

## [

一枚上进的程序员

](https://www.zhihu.com/column/c_1243484041787707392)

致力于Java编程精品文章创作

[

![转载](media/转载.jpg)

](https://www.zhihu.com/column/c_1362795772740816898)

## [

转载

](https://www.zhihu.com/column/c_1362795772740816898)