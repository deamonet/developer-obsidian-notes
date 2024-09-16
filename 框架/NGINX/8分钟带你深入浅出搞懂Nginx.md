[![地球的外星人君](media/地球的外星人君.jpg)](https://www.zhihu.com/people/diqiuyo)

[地球的外星人君](https://www.zhihu.com/people/diqiuyo)

Linux云计算和Python推动市场提升的学习研究者。

​关注他

2,800 人赞同了该文章

> **文章开始前给需要学习的同学提供一份nginx官方中文文档，希望能帮到大家，下方卡片直达领取。**

[粉丝福利，自学资料限时领取！​magedu2018.mikecrm.com/I9zX5EO](http://magedu2018.mikecrm.com/I9zX5EO)

![](media/v2-477580fe30829ef66f8258669386fb0a_1440w.jpg)

  

---

**开始正文！**

分享一篇来自简书的文章，对Nginx的讲解非常到位，文章链接：[深入浅出Nginx](https://www.jianshu.com/p/5eab0f83e3b4)，作者主页：[张丰哲 - 简书](https://www.jianshu.com/u/cb569cce501b)。  

Nginx是一款轻量级的Web服务器、反向代理服务器，由于它的内存占用少，启动极快，高并发能力强，在互联网项目中广泛应用。

![](media/v2-e1826bab1d07df8e97d61aa809b94a10_1440w.jpg)

架构图

上图基本上说明了当下流行的技术架构，其中Nginx有点入口网关的味道。

## 反向代理服务器？

经常听人说到一些术语，如反向代理，那么什么是反向代理，什么又是正向代理呢？

**正向代理：**

![](media/v2-c8ac111c267ae0745f984e326ef0c47f_1440w.jpg)

正向代理示意图

**反向代理：**

![](media/v2-4787a512240b238ebf928cd0651e1d99_1440w.jpg)

反向代理示意图

由于防火墙的原因，我们并不能直接访问谷歌，那么我们可以借助VPN来实现，这就是一个简单的正向代理的例子。这里你能够发现，正向代理“代理”的是客户端，而且客户端是知道目标的，而目标是不知道客户端是通过VPN访问的。

当我们在外网访问百度的时候，其实会进行一个转发，代理到内网去，这就是所谓的反向代理，即反向代理“代理”的是服务器端，而且这一个过程对于客户端而言是透明的。

## Nginx的Master-Worker模式

![](media/v2-0951372e22a6314b1e9b520b3cd6b3b6_1440w.jpg)

nginx进程

启动Nginx后，其实就是在80端口启动了Socket服务进行监听，如图所示，Nginx涉及Master进程和Worker进程。

![](media/v2-b24eb2b29b48f59883232a58392ddae3_1440w.jpg)

Master-Worker模式

![](media/v2-d21393745de9c470934575ef76cefd29_1440w.jpg)

nginx.conf

Master进程的作用是？

**读取并验证配置文件nginx.conf；管理worker进程；**

Worker进程的作用是？

**每一个Worker进程都维护一个线程（避免线程切换），处理连接和请求；注意Worker进程的个数由配置文件决定，一般和CPU个数相关（有利于进程切换），配置几个就有几个Worker进程。**

## 思考：Nginx如何做到热部署？

所谓热部署，就是配置文件nginx.conf修改后，不需要stop Nginx，不需要中断请求，就能让配置文件生效！（nginx -s reload 重新加载/nginx -t检查配置/nginx -s stop）

通过上文我们已经知道worker进程负责处理具体的请求，那么如果想达到热部署的效果，可以想象：

方案一：

修改配置文件nginx.conf后，主进程master负责推送给woker进程更新配置信息，woker进程收到信息后，更新进程内部的线程信息。（有点valatile的味道）

方案二：

修改配置文件nginx.conf后，重新生成新的worker进程，当然会以新的配置进行处理请求，而且新的请求必须都交给新的worker进程，至于老的worker进程，等把那些以前的请求处理完毕后，kill掉即可。

Nginx采用的就是方案二来达到热部署的！

## 思考：Nginx如何做到高并发下的高效处理？

上文已经提及Nginx的worker进程个数与CPU绑定、worker进程内部包含一个线程高效回环处理请求，这的确有助于效率，但这是不够的。

**作为专业的程序员，我们可以开一下脑洞：BIO/NIO/AIO、异步/同步、阻塞/非阻塞...**

要同时处理那么多的请求，要知道，有的请求需要发生IO，可能需要很长时间，如果等着它，就会拖慢worker的处理速度。

**Nginx采用了Linux的epoll模型，epoll模型基于事件驱动机制，它可以监控多个事件是否准备完毕，如果OK，那么放入epoll队列中，这个过程是异步的。worker只需要从epoll队列循环处理即可。**

## 思考：Nginx挂了怎么办？

Nginx既然作为入口网关，很重要，如果出现单点问题，显然是不可接受的。

答案是：**Keepalived+Nginx实现高可用**。

Keepalived是一个高可用解决方案，主要是用来防止服务器单点发生故障，可以通过和Nginx配合来实现Web服务的高可用。（其实，Keepalived不仅仅可以和Nginx配合，还可以和很多其他服务配合）

Keepalived+Nginx实现高可用的思路：

第一：请求不要直接打到Nginx上，应该先通过Keepalived（这就是所谓虚拟IP，VIP）

第二：Keepalived应该能监控Nginx的生命状态（提供一个用户自定义的脚本，定期检查Nginx进程状态，进行权重变化,，从而实现Nginx故障切换）

![](media/v2-ec3208d1ea659d126fe2a008ec5ae927_1440w.jpg)

Keepalived+Nginx

## 我们的主战场：nginx.conf

很多时候，在开发、测试环境下，我们都得自己去配置Nginx，就是去配置nginx.conf。

nginx.conf是典型的分段配置文件，下面我们来分析下。

## 虚拟主机

![](media/v2-b418e69a42a65f033cfdf3b80b988d83_1440w.jpg)

http的server段

![](media/v2-bec9b433b145d892b4eddfaf5b2aee1e_1440w.jpg)

访问结果

其实这是把Nginx作为web server来处理静态资源。

第一：location可以进行正则匹配，应该注意正则的几种形式以及优先级。（这里不展开）

第二：Nginx能够提高速度的其中一个特性就是：动静分离，就是把静态资源放到Nginx上，由Nginx管理，动态请求转发给后端。

**第三：我们可以在Nginx下把静态资源、日志文件归属到不同域名下（也即是目录），这样方便管理维护。**

**第四：Nginx可以进行IP访问控制，有些电商平台，就可以在Nginx这一层，做一下处理，内置一个黑名单模块，那么就不必等请求通过Nginx达到后端在进行拦截，而是直接在Nginx这一层就处理掉。**

## 反向代理【proxy_pass】

所谓反向代理，很简单，其实就是在location这一段配置中的root替换成**proxy_pass**即可。root说明是静态资源，可以由Nginx进行返回；而proxy_pass说明是动态请求，需要进行转发，比如代理到Tomcat上。

反向代理，上面已经说了，过程是透明的，比如说request -> Nginx -> Tomcat，那么对于Tomcat而言，请求的IP地址就是Nginx的地址，而非真实的request地址，这一点需要注意。不过好在Nginx不仅仅可以反向代理请求，还可以由用户**自定义设置HTTP HEADER**。

## 负载均衡【upstream】

上面的反向代理中，我们通过proxy_pass来指定Tomcat的地址，很显然我们只能指定一台Tomcat地址，那么我们如果想指定多台来达到负载均衡呢？

第一，通过**upstream**来定义一组Tomcat，并指定负载策略（IPHASH、加权论调、最少连接），健康检查策略（Nginx可以监控这一组Tomcat的状态）等。

第二，将proxy_pass替换成upstream指定的值即可。

**负载均衡可能带来的问题？**

负载均衡所带来的明显的问题是，一个请求，可以到A server，也可以到B server，这完全不受我们的控制，当然这也不是什么问题，只是我们得注意的是：**用户状态的保存问题，如Session会话信息，不能在保存到服务器上。**

## 缓存

缓存，是Nginx提供的，可以加快访问速度的机制，说白了，在配置上就是一个开启，同时指定目录，让缓存可以存储到磁盘上。具体配置，大家可以参考Nginx官方文档，这里就不在展开了。

**最近看到一些相关文章，比较详细。想深入了解可以看看。**

[Nginx详解](https://mp.weixin.qq.com/s/XoqGvYBabW8YBl9xEeNYZw)

[一文详解负载均衡和反向代理的真实区别](https://mp.weixin.qq.com/s/TYM83F2O-keMvn4ZYa5nqw)

---

你想更深入了解学习Linux知识体系，你可以看一下我们花费了一个多月整理了上百小时的几百个知识点体系内容：

[2020Linux高薪实战学习大合集​apprhkaai3v6603.h5.xiaoeknow.com/v1/course/text/i_5f040f3ce4b036f1c0cf1f90?type=2&pro_id=p_5fe2fb75e4b04db7c0969ffb![](media/v2-a34bb8085933d0d6d02bd267ab885bc8_180x120.jpg.png)](https://apprhkaai3v6603.h5.xiaoeknow.com/v1/course/text/i_5f040f3ce4b036f1c0cf1f90?type=2&pro_id=p_5fe2fb75e4b04db7c0969ffb)

往期精彩文章，欢迎点赞收藏！

[一份超全的Python学习资料汇总！](https://zhuanlan.zhihu.com/p/225866345)

[这是我见过最全的《MySQL笔记》，涵盖MySQL所有高级知识点！](https://zhuanlan.zhihu.com/p/339009072)

[一份超全的Linux自学资源整理合集！](https://zhuanlan.zhihu.com/p/338448408)

[不用怀疑！Kubernetes决定弃用Docker](https://zhuanlan.zhihu.com/p/328172459)

[一键申请多个证书 shell 脚本](https://zhuanlan.zhihu.com/p/326406890)

编辑于 2021-12-16 14:02

[

Linux

](https://www.zhihu.com/topic/19554300)

[

Linux 运维

](https://www.zhihu.com/topic/19648078)

[

Nginx

](https://www.zhihu.com/topic/19574050)

​赞同 2800​​78 条评论

​分享

​喜欢​收藏​申请转载

​

![](media/v2-c1ccd457ef0518c33a4ce0a9bf1d4fa9_l-1.jpg)

评论千万条，友善第一条

  

78 条评论

默认

最新

[![运维朱工](media/运维朱工.jpg)](https://www.zhihu.com/people/5a8cdfb5a41cdc1614d9a8e6404062ac)

[运维朱工](https://www.zhihu.com/people/5a8cdfb5a41cdc1614d9a8e6404062ac)

很多地方没有展开讲哦，比如为什么配置了缓存，速度就会快呢？缓存数据同样是在服务器的磁盘上，直接返回不是一样的？这个需要给小白同学说清楚原因才好。

2019-07-18

​回复​35

[![魏 同学](media/魏_同学.jpg)](https://www.zhihu.com/people/19096866fa5d4c649b53a69ae71c6101)

[魏 同学](https://www.zhihu.com/people/19096866fa5d4c649b53a69ae71c6101)

果然是运维

2022-10-04

​回复​赞

[![李黄河](media/李黄河.jpg)](https://www.zhihu.com/people/e089b382df11697caaf46c3713d19da5)

[李黄河](https://www.zhihu.com/people/e089b382df11697caaf46c3713d19da5)

![](media/v2-4812630bc27d642f7cafcd6cdeca3d7a.jpg.png)

够入门了

2018-03-26

​回复​18

[![代码咚咚锵](media/代码咚咚锵.jpg)](https://www.zhihu.com/people/9ca9fe9db80f516dafe58e1b3379a688)

[代码咚咚锵](https://www.zhihu.com/people/9ca9fe9db80f516dafe58e1b3379a688)

[juejin.im/post/5e982d4b](https://juejin.im/post/5e982d4b51882573b0474c07) 看看这个，或许你能有些相见恨晚哈哈

2020-04-19

​回复​22

[![雨时亦晴](media/雨时亦晴.jpg)](https://www.zhihu.com/people/94ff42576dbe19c10c2d970293f89ae3)

[雨时亦晴](https://www.zhihu.com/people/94ff42576dbe19c10c2d970293f89ae3)

[代码咚咚锵](https://www.zhihu.com/people/9ca9fe9db80f516dafe58e1b3379a688)

十分感谢,这个讲的清楚

2021-08-24

​回复​赞

[![Colin](media/Colin.jpg)](https://www.zhihu.com/people/069f00ff3072493a3aa1b1fabbd04b20)

[Colin](https://www.zhihu.com/people/069f00ff3072493a3aa1b1fabbd04b20)

没入门

2021-04-19

​回复​8

[![且听风吟](media/且听风吟.jpg)](https://www.zhihu.com/people/a0dacdb14fa53649cc932efb9c3f2eb2)

[且听风吟](https://www.zhihu.com/people/a0dacdb14fa53649cc932efb9c3f2eb2)

有点豁然开朗的意思，感谢😊

2018-03-27

​回复​6

[![雲散水枯](media/雲散水枯.jpg)](https://www.zhihu.com/people/087c79b41a6ce0b3d38da3c2102aeb6f)

[雲散水枯](https://www.zhihu.com/people/087c79b41a6ce0b3d38da3c2102aeb6f)

有实例就更好了。

2018-03-29

​回复​3

[![杨亚洲(MongoDB)](media/杨亚洲(MongoDB).jpg)](https://www.zhihu.com/people/05e88cd731f8e0118f4ab3fd469a6357)

[杨亚洲(MongoDB)](https://www.zhihu.com/people/05e88cd731f8e0118f4ab3fd469a6357)

nginx多进程模型最大的优势是做高性能代理，之前把nginx多进程、高性能、低延时、高并发等特性应用于redis、memcache代理中间件，每秒峰值流量可以轻松过百万，详见下面文章，希望对你有帮助：

  

高性能 -Nginx 多进程高并发、低时延、高可靠机制在百万级缓存 (redis、memcache) 代理中间件中的应用

[xie.infoq.cn/article/2e](https://xie.infoq.cn/article/2ee961483c66a146709e7e861)

2020-12-08

​回复​3

[![爱吃香蕉](media/爱吃香蕉.jpg)](https://www.zhihu.com/people/c8179a1edf6bd58f4e018f3b8515c3eb)

[爱吃香蕉](https://www.zhihu.com/people/c8179a1edf6bd58f4e018f3b8515c3eb)

[杨亚洲(MongoDB)](https://www.zhihu.com/people/05e88cd731f8e0118f4ab3fd469a6357)

大佬厉害

2021-11-25

​回复​赞

展开其他 1 条回复​

[![Pine](media/Pine.jpg)](https://www.zhihu.com/people/659159e7359b7d9523151c335db704ed)

[Pine](https://www.zhihu.com/people/659159e7359b7d9523151c335db704ed)

学弱狗提问:目前只学过计算机网络的话，想要从事cdn安全方面的方向，学习路线是什么![[拜托]](media/[拜托].png)

2019-07-09

​回复​3

[![谭九鼎](media/谭九鼎.jpg)](https://www.zhihu.com/people/3c8f92da29fb45edc45b0c00c851bd73)

[谭九鼎](https://www.zhihu.com/people/3c8f92da29fb45edc45b0c00c851bd73)

typo: valatile

2019-03-01

​回复​2

[![且待风尽](media/且待风尽.jpg)](https://www.zhihu.com/people/f8e4317a2a4c876351f300ac94765a8b)

[且待风尽](https://www.zhihu.com/people/f8e4317a2a4c876351f300ac94765a8b)

为什么nginx总是做反向代理，很少做正向代理？

2019-07-01

​回复​2

[![谭九鼎](media/谭九鼎.jpg)](https://www.zhihu.com/people/3c8f92da29fb45edc45b0c00c851bd73)

[谭九鼎](https://www.zhihu.com/people/3c8f92da29fb45edc45b0c00c851bd73)

[谭九鼎](https://www.zhihu.com/people/3c8f92da29fb45edc45b0c00c851bd73)

唉呀，我之前怎么会这么想![[捂脸]](media/[捂脸]-1.png)  
答案是正代需要客户端进行一些操作，而不需要服务端操作，反代刚好相反。对于一般的网页服务来说正代没有意义

2021-10-28

​回复​3

[![谭九鼎](media/谭九鼎.jpg)](https://www.zhihu.com/people/3c8f92da29fb45edc45b0c00c851bd73)

[谭九鼎](https://www.zhihu.com/people/3c8f92da29fb45edc45b0c00c851bd73)

因为正代要模块支持

2020-05-09

​回复​2

展开其他 2 条回复​

[![代码咚咚锵](media/代码咚咚锵.jpg)](https://www.zhihu.com/people/9ca9fe9db80f516dafe58e1b3379a688)

[代码咚咚锵](https://www.zhihu.com/people/9ca9fe9db80f516dafe58e1b3379a688)

我整理了一篇适合前端同学入门 Nginx 的文章，借楼广告，欢迎各位大佬移步点评 [juejin.im/post/5e982d4b](https://juejin.im/post/5e982d4b51882573b0474c07)

2020-06-23

​回复​2

[![代码咚咚锵](media/代码咚咚锵.jpg)](https://www.zhihu.com/people/9ca9fe9db80f516dafe58e1b3379a688)

[代码咚咚锵](https://www.zhihu.com/people/9ca9fe9db80f516dafe58e1b3379a688)

连前端都看得懂的《Nginx 入门指南》 - 掘金 [juejin.im/post/5e982d4b](https://juejin.im/post/5e982d4b51882573b0474c07) 这篇文章对于初学者来说更容易接受，讲得很仔细，墙裂推荐

2020-04-19

​回复​1

[![404NOT FOUND](media/404NOT_FOUND.jpg)](https://www.zhihu.com/people/efdb010435432a05a64e118cbda35e89)

[404NOT FOUND](https://www.zhihu.com/people/efdb010435432a05a64e118cbda35e89)

[juejin.im/post/5e941ec4](https://juejin.im/post/5e941ec4e51d45471263ef32)

2020-06-06

​回复​赞

[![zoyua](media/zoyua.jpg)](https://www.zhihu.com/people/3c7e5e1d018b71ba00771e0a7cfc69b8)

[zoyua](https://www.zhihu.com/people/3c7e5e1d018b71ba00771e0a7cfc69b8)

是volatile吧

2020-03-27

​回复​1

[![颜海镜](media/颜海镜.jpg)](https://www.zhihu.com/people/c4c42e9b96c03894a273d27b33e6d519)

[颜海镜](https://www.zhihu.com/people/c4c42e9b96c03894a273d27b33e6d519)

非常棒

2018-06-01

​回复​1

[![小白不白](media/小白不白.jpg)](https://www.zhihu.com/people/aa67e9b586c2e945dc6ef9d0a1dabde6)

[小白不白](https://www.zhihu.com/people/aa67e9b586c2e945dc6ef9d0a1dabde6)

踩踩前端大佬

![](media/v2-db92f653a2ec17ea3ff309d6d56e8507.gif)

2020-10-18

​回复​1

[![游鱼](media/游鱼.jpg)](https://www.zhihu.com/people/3935e23945229a8cbaae82931222910b)

[游鱼](https://www.zhihu.com/people/3935e23945229a8cbaae82931222910b)

精确分析到位，点赞

2018-04-13

​回复​1

[![ERIC](media/ERIC.jpg)](https://www.zhihu.com/people/f9f98f007ed18922fdf0e6127c36356f)

[ERIC](https://www.zhihu.com/people/f9f98f007ed18922fdf0e6127c36356f)

Ngnix缓存技术咋样，我看网上有squid apache varnish 等代理，哪个比较好 我是小白

2022-10-31

​回复​赞

[![billy chen](media/billy_chen.jpg)](https://www.zhihu.com/people/0b1de60c07a1831f73481c9d8b42d053)

[billy chen](https://www.zhihu.com/people/0b1de60c07a1831f73481c9d8b42d053)

kanbudong

2020-03-22

​回复​赞

[![老瓜甭子](media/老瓜甭子.jpg)](https://www.zhihu.com/people/2ef59b5cb9b859049718a9cc4a70aa79)

[老瓜甭子](https://www.zhihu.com/people/2ef59b5cb9b859049718a9cc4a70aa79)

爱了~

2020-03-22

​回复​赞

[![OOP](media/OOP.png)](https://www.zhihu.com/people/fb7ce8e08b2a810342ed52b2327ba202)

[OOP](https://www.zhihu.com/people/fb7ce8e08b2a810342ed52b2327ba202)

![](media/v2-878d130f7db8314bf8eac78484d68fb3.gif)

2020-02-29

​回复​赞

[![ithewei](media/ithewei.jpg)](https://www.zhihu.com/people/46ad0238fb35d4ba57b8232bb1497d1c)

[ithewei](https://www.zhihu.com/people/46ad0238fb35d4ba57b8232bb1497d1c)

分享一个性能媲美nginx的异步IO事件库libhv  
[hewei.blog.csdn.net/art](https://hewei.blog.csdn.net/article/details/103903123)

2020-01-16

​回复​赞

[![路行](media/路行.jpg)](https://www.zhihu.com/people/caeb47fed62287cd9306d7bb9b83bd04)

[路行](https://www.zhihu.com/people/caeb47fed62287cd9306d7bb9b83bd04)

通俗易懂，赞

2019-11-27

​回复​赞

点击查看全部评论

![](media/v2-c1ccd457ef0518c33a4ce0a9bf1d4fa9_l-1.jpg)

评论千万条，友善第一条

  

### 文章被以下专栏收录

[

![Linux高薪集训营](media/Linux高薪集训营.jpg)

](https://www.zhihu.com/column/c_128843875)

## [

Linux高薪集训营

](https://www.zhihu.com/column/c_128843875)

专注分享Linux技术进阶文章

[

![Linux高薪集训营](media/Linux高薪集训营.jpg)

](https://www.zhihu.com/column/c_1286332758722842624)

## [

Linux高薪集训营

](https://www.zhihu.com/column/c_1286332758722842624)

专注分享Linux技术进阶文章