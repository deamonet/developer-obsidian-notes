[![](media/d74341c6c18934fb1b64a5ad586e5508~100x100.jpg)](https://juejin.cn/user/4353721774642536)

[敲完代码再睡觉![lv-3](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/img/lv-3.7938ebc.png "创作等级")](https://juejin.cn/user/4353721774642536)

2020年04月16日 22:47 ·  阅读 56931

_文本内容较多（原理+实践），讲解较为详细，大约10分钟才能阅读完。_

> 本文Nginx安装和配置部分均以 Mac OS X 系统为例，win 系爱好者选择性慎入

![](media/171838101bdcb267~tplv-t2oaga2asx-zoom-in-crop-mark!4536!0!0!0.jpg)

虽然题目看起来有点像写给前端同学的一样，但是本文实际上是适合所有想学习 Nginx 但不知道如何下手的同学。笔者尽量深入浅出，讲得通俗易懂，让在座的不管是前端还是后端，阅读体验都如德芙巧克力般丝滑。

### Nginx是什么？

> “Nginx 是一款轻量级的 HTTP 服务器，采用事件驱动的异步非阻塞处理方式框架，这让其具有极好的 IO 性能，时常用于服务端的**反向代理**和**负载均衡**。”

这是大多数开发者对 Nginx 的定义。

_（什么？听不懂，反向代理、负载均衡这都什么鬼？那么请稍安勿躁，请君带着疑问往下看）_

Nginx 是一款 http 服务器 （或叫web服务器）。它是由俄罗斯人 伊戈尔·赛索耶夫为俄罗斯访问量第二的 Rambler.ru 站点开发的，并于2004年首次公开发布的。

> web服务器：负责处理和响应用户请求，一般也称为http服务器，如 Apache、IIS、Nginx
> 
> 应用服务器：存放和运行系统程序的服务器，负责处理程序中的业务逻辑，如 Tomcat、Weblogic、Jboss（现在大多数应用服务器也包含了web服务器的功能）

Nginx 是什么，总结一下就是这些：

-   一种轻量级的web服务器
    
-   设计思想是事件驱动的异步非阻塞处理（类node.js）
    
-   占用内存少、启动速度快、并发能力强
    
-   使用C语言开发
    
-   扩展性好，第三方插件非常多
    
-   在互联网项目中广泛应用
    

### 为什么要学习Nginx？

Nginx 是全球排名前三的服务器，并且近年来用户增长非常快。

有人统计，世界上约有三分之一的网址采用了Nginx。在大型网站的架构中，Nginx被普遍使用，如 百度、阿里、腾讯、京东、网易、新浪、大疆等。

Nginx 安装简单，配置简洁，作用却无可替代。Nginx 是运维和后端的必修课，也是前端进阶的必修课。

因为掌握了Nginx，能让前端站得更高，更好的设计系统架构，更好的选择问题的解决方案，更好的服务业务开发。

所以综上所述，Nginx不得不学，前端也一样不能落后。

### 如何学习Nginx？

#### 安装/卸载

推荐 Mac 电脑上内置 homebrew 工具安装。

安装 Nginx：

`brew install nginx`

卸载 Nginx：

`brew uninstall nginx`

如果不行，就在 brew 前面加上 sudo。（sudo 是什么？对 Linux 也不太熟的，可以移步 [前端工程师需要对 Linux 掌握到什么程度？](https://link.juejin.cn/?target=https%3A%2F%2Fwww.zhihu.com%2Fquestion%2F20235589%2Fanswer%2F516923465 "https://www.zhihu.com/question/20235589/answer/516923465")）

安装好了，验证一下 nginx 是否安装成功：

![](media/17182a5733a7de37~tplv-t2oaga2asx-zoom-in-crop-mark!4536!0!0!0.jpg)

homebrew 工具使起来非常称手，跟前端开发中的 npm/yarn 用法很像。

只不过 npm/yarn 高度依赖 node 环境，而 homebrew 在 mac 系统中可以随意使用。（传送门：[Homebrew介绍和使用](https://link.juejin.cn/?target=https%3A%2F%2Fp1-jj.byteimg.com%2Ftos-cn-i-t2oaga2asx%2Fgold-user-assets%2F2020%2F4%2F16%2F1718297d6079e652~tplv-t2oaga2asx-image.image "https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/4/16/1718297d6079e652~tplv-t2oaga2asx-image.image")）

#### 启动

启动 Nginx：

`sudo nginx` 或 `sudo brew services start nginx`

停止 Nginx：

`sudo nginx -s stop` 或 `sudo brew services stop nginx`

热重启 Nginx：

`sudo nginx -s reload`

强制停止 Nginx：

`sudo pkill -9 nginx`

#### 修改配置

经常要用到的几个文件路径：

1.  `/usr/local/etc/nginx/nginx.conf` （nginx配置文件路径）
2.  `/usr/local/var/www` （nginx服务器默认的根目录）
3.  `/usr/local/Cellar/nginx/1.17.9` （nginx的安装路径）
4.  `/usr/local/var/log/nginx/error.log` (nginx默认的日志路径)

nginx 默认配置文件简介：

```nginx
# 首尾配置暂时忽略
server {  
        # 当nginx接到请求后，会匹配其配置中的service模块
        # 匹配方法就是将请求携带的host和port去跟配置中的server_name和listen相匹配
        listen       8080;        
        server_name  localhost; # 定义当前虚拟主机（站点）匹配请求的主机名

        location / {
            root   html; # Nginx默认值
            # 设定Nginx服务器返回的文档名
            index  index.html index.htm; # 先找根目录下的index.html，如果没有再找index.htm
        }
}
# 首尾配置暂时忽略
复制代码
```

server{ } 其实是包含在 http{ } 内部的。每一个 server{ } 是一个虚拟主机（站点）。

上面代码块的意思是：当一个请求叫做`localhost:8080`请求nginx服务器时，该请求就会被匹配进该代码块的 server{ } 中执行。

当然 nginx 的配置非常多，用的时候可以根据文档进行配置。

> 英文文档：[nginx.org/en/docs/](https://link.juejin.cn/?target=http%3A%2F%2Fnginx.org%2Fen%2Fdocs%2F "http://nginx.org/en/docs/")
> 
> 中文文档：[www.nginx.cn/doc/](https://link.juejin.cn/?target=https%3A%2F%2Fwww.nginx.cn%2Fdoc%2F "https://www.nginx.cn/doc/")

### Nginx有哪些应用？

主要有4大应用。

#### 动静分离

![](media/171867d175eae45f~tplv-t2oaga2asx-zoom-in-crop-mark!4536!0!0!0.jpg)

如上图所示，动静分离其实就是 Nginx 服务器将接收到的请求分为**动态请求**和**静态请求**。

静态请求直接从 nginx 服务器所设定的根目录路径去取对应的资源，动态请求转发给真实的后台（前面所说的应用服务器，如图中的Tomcat）去处理。

这样做不仅能给应用服务器减轻压力，将后台api接口服务化，还能将前后端代码分开并行开发和部署。（传送门：[nginx动静分离的好处](https://link.juejin.cn/?target=https%3A%2F%2Fwww.php.cn%2Fnginx%2F424631.html "https://www.php.cn/nginx/424631.html")）

```nginx
server {  
        listen       8080;        
        server_name  localhost;

        location / {
            root   html; # Nginx默认值
            index  index.html index.htm;
        }
        
        # 静态化配置，所有静态请求都转发给 nginx 处理，存放目录为 my-project
        location ~ .*\.(html|htm|gif|jpg|jpeg|bmp|png|ico|js|css)$ {
            root /usr/local/var/www/my-project; # 静态请求所代理到的根目录
        }
        
        # 动态请求匹配到path为'node'的就转发到8002端口处理
        location /node/ {  
            proxy_pass http://localhost:8002; # 充当服务代理
        }
}
复制代码
```

访问静态资源 nginx 服务器会返回 my-project 里面的文件，如获取 index.html：

![](media/171835d8723ead00~tplv-t2oaga2asx-zoom-in-crop-mark!4536!0!0!0.jpg)

访问动态请求 nginx 服务器会将它从8002端口请求到的内容，原封不动的返回回去：

![](media/171836158ed179ef~tplv-t2oaga2asx-zoom-in-crop-mark!4536!0!0!0.jpg) ![](media/171865a653e3ce09~tplv-t2oaga2asx-zoom-in-crop-mark!4536!0!0!0.jpg)

#### 反向代理

##### 反向代理是什么？

反向代理其实就类似你去找代购帮你买东西（浏览器或其他终端向nginx请求），你不用管他去哪里买，只要他帮你买到你想要的东西就行（浏览器或其他终端最终拿到了他想要的内容，但是具体从哪儿拿到的这个过程它并不知道）。

##### 反向代理的作用

1.  保障应用服务器的安全（增加一层代理，可以屏蔽危险攻击，更方便的控制权限）
2.  实现负载均衡（稍等~下面会讲）
3.  实现跨域（号称是最简单的跨域方式）

##### 配置反向代理

配置一个简单的反向代理是很容易的，代码如下：

```nginx
server {  
        listen       8080;        
        server_name  localhost;

        location / {
            root   html; # Nginx默认值
            index  index.html index.htm;
        }
        
        proxy_pass http://localhost:8000; # 反向代理配置，请求会被转发到8000端口
}
复制代码
```

反向代理的表现很简单。那上面的代码块来说，其实就是向nginx请求`localhost:8080`跟请求 `http://localhost:8000` 是一样的效果。（跟代购的原理一样）

这是一个反向代理最简单的模型，只是为了说明反向代理的配置。但是现实中反向代理多数是用在负载均衡中。

示意图如下：

![](media/17183720f7a66978~tplv-t2oaga2asx-zoom-in-crop-mark!4536!0!0!0.jpg)

nginx 就是充当图中的 proxy。左边的3个 client 在请求时向 nginx 获取内容，是感受不到3台 server 存在的。

> 此时，proxy就充当了3个 server 的反向代理。

反向代理应用十分广泛，CDN 服务就是反向代理经典的应用场景之一。除此之外，反向代理也是实现负载均衡的基础，很多大公司的架构都应用到了反向代理。

#### 负载均衡

##### 负载均衡是什么？

随着业务的不断增长和用户的不断增多，一台服务已经满足不了系统要求了。这个时候就出现了服务器 [集群](https://link.juejin.cn/?target=https%3A%2F%2Fwww.cnblogs.com%2Fbhlsheji%2Fp%2F4026296.html "https://www.cnblogs.com/bhlsheji/p/4026296.html")。

在服务器集群中，Nginx 可以将接收到的客户端请求“均匀地”（严格讲并不一定均匀，可以通过设置权重）分配到这个集群中所有的服务器上。这个就叫做**负载均衡**。

负载均衡的示意图如下：

![](media/171862efada16376~tplv-t2oaga2asx-zoom-in-crop-mark!4536!0!0!0.jpg)

##### 负载均衡的作用

-   分摊服务器集群压力
-   保证客户端访问的稳定性

前面也提到了，负载均衡可以解决分摊服务器集群压力的问题。除此之外，Nginx还带有**健康检查**（服务器心跳检查）功能，会定期轮询向集群里的所有服务器发送健康检查请求，来检查集群中是否有服务器处于异常状态。

一旦发现某台服务器异常，那么在这以后代理进来的客户端请求都不会被发送到该服务器上（直健康检查发现该服务器已恢复正常），从而保证客户端访问的稳定性。

##### 配置负载均衡

配置一个简单的负载均衡并不复杂，代码如下：

```nginx
# 负载均衡：设置domain
upstream domain {
    server localhost:8000;
    server localhost:8001;
}
server {  
        listen       8080;        
        server_name  localhost;

        location / {
            # root   html; # Nginx默认值
            # index  index.html index.htm;
            
            proxy_pass http://domain; # 负载均衡配置，请求会被平均分配到8000和8001端口
            proxy_set_header Host $host:$server_port;
        }
}
复制代码
```

8000和8001是我本地用 Node.js 起的两个服务，负载均衡成功后可以看到访问 `localhost:8080` 有时会访问到8000端口的页面，有时会访问到8001端口的页面。

![](media/17186788e4daacc3~tplv-t2oaga2asx-zoom-in-crop-mark!4536!0!0!0.jpg)

![](media/17186790c211d628~tplv-t2oaga2asx-zoom-in-crop-mark!4536!0!0!0.jpg)

能看到这个效果，就说明你配置的负载均衡策略生效了。

实际项目中的负载均衡远比这个案例要更加复杂，但是万变不离其宗，都是根据这个理想模型衍生出来的。

受集群单台服务器内存等资源的限制，负载均衡集群的服务器也不能无限增多。但因其良好的容错机制，负载均衡成为了实现**高可用架构**中必不可少的一环。

#### 正向代理

正向代理跟反向道理正好相反。拿上文中的那个代购例子来讲，多个人找代购购买同一个商品，代购找到买这个的店后一次性给买了。这个过程中，该店主是不知道代购是帮别代买买东西的。那么代购对于多个想买商品的顾客来讲，他就充当了正向代理。

正向代理的示意图如下：

![](media/171864e773f05fe7~tplv-t2oaga2asx-zoom-in-crop-mark!4536!0!0!0.jpg)

nginx 就是充当图中的 proxy。左边的3个 client 在请求时向 nginx 获取内容，server 是感受不到3台 client 存在的。

> 此时，proxy 就充当了3个 client 的正向代理。

**正向代理**，意思是一个位于客户端和原始服务器(origin server)之间的服务器，为了从原始服务器取得内容，客户端向代理发送一个请求并指定目标(原始服务器)，然后代理向原始服务器转交请求并将获得的内容返回给客户端。客户端才能使用正向代理。当你需要把你的服务器作为代理服务器的时候，可以用Nginx来实现正向代理。

科学上网vpn（俗称`翻墙`）其实就是一个正向代理工具。

该 vpn 会将想访问墙外服务器 server 的网页请求，代理到一个可以访问该网站的代理服务器 proxy 上。这个 proxy 把墙外服务器 server 上获取的网页内容，再转发给客户。

代理服务器 proxy 就是 Nginx 搭建的。

正向代理应用远没有反向代理广泛，因为这个是 nginx 的入门篇，加上篇幅有限，这里就不给出具体的参考配置了。

本文絮絮叨叨，笔者恨不能将 nginx 的一些基础相关知识全都放在本文里面。这样也能让想学 nginx 的同学理解起来更加容易，少走弯路。

分类：

[前端](https://juejin.cn/frontend)

标签：

[Nginx](https://juejin.cn/tag/Nginx)

[安装掘金浏览器插件](https://juejin.cn/extension/?utm_source=standalone&utm_medium=post&utm_campaign=extension_promotion)

多内容聚合浏览、多引擎快捷搜索、多工具便捷提效、多模式随心畅享，你想要的，这里都有！

[前往安装](https://juejin.cn/extension/?utm_source=standalone&utm_medium=post&utm_campaign=extension_promotion)

相关小册

![「前端工程师进阶 10 日谈」封面](media/「前端工程师进阶_10_日谈」封面.jpg)

VIP

 前端工程师进阶 10 日谈

[十年踪迹![lv-5](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/img/lv-5.d08789d.png "创作等级")

![](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/07302452a7ad81cb43a173b5cd580237.svg)

](https://juejin.cn/user/712139263189303)

2432购买

¥14.95

¥29.9

首单券后价

首单券后价

![「Vue.js 组件精讲」封面](media/「Vue.js_组件精讲」封面.jpg)

VIP

 Vue.js 组件精讲

[Aresn![lv-5](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/img/lv-5.d08789d.png "创作等级")](https://juejin.cn/user/149189280930478)

6852购买

¥14.95

¥29.9

首单券后价

首单券后价

评论

![](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/58aaf1326ac763d8a1054056f3b7f2ef.svg)

看完啦，

登录

分享一下感受吧～

热门评论

[![stevekeol的头像](media/stevekeol的头像.jpg)](https://juejin.cn/user/483440848543271)

[stevekeol![lv-1](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/img/lv-1.9d93e13.png "创作等级")![掘友等级](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/9b2f31c663d17de59dd9e5fff272bb85.svg "掘友等级")](https://juejin.cn/user/483440848543271)

区块链研发工程师2年前

什么叫连前端都看得懂，你看不起前端呀大哥

6

4

[![](media/d74341c6c18934fb1b64a5ad586e5508~100x100.jpg)](https://juejin.cn/user/4353721774642536)

[敲完代码再睡觉](https://juejin.cn/user/4353721774642536)

（作者）2年前

大哥你误会了！nginx 基本是后端的必修课，而不少前端平常因为跟 nginx 接触得也比后端少很多，自然一般的 nginx 文档对前端的亲和力就不是那么够。  
这篇文章是以一个前端开发者的角度为切入点，去讲述 Nginx 的使用经验的，主要也是为了让咱们前端朋友更快的入门 Nginx！  
实不相瞒，我也是前端开发者![😊](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-assets/asset/twemoji/2.6.0/svg/1f60a.svg~tplv-t2oaga2asx-image.image)

12

回复

[![](media/918211e5cbc44bdab574403e0b3dfe6c~100x100.jpg)](https://juejin.cn/user/3755587453860926)

[ggbdpq

![](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/07302452a7ad81cb43a173b5cd580237.svg)

](https://juejin.cn/user/3755587453860926)

回复

[敲完代码再睡觉](https://juejin.cn/user/4353721774642536)

2年前

我本质翻译一下，连阿猫阿狗都看得懂的《Nginx 入门指南》，这样就好，就不单单是看不起前端了，是看不起在座的各位。其实这文章内容挺通俗易懂的，就是标题党可能跟营销号学得一半，学费没交足，故意'引人注目'。

“

大哥你误会了！nginx 基本是后端的必修课，而不少前端平常因为跟 nginx 接触得也比后端少很多，自然一般的 nginx 文档对前端的亲和力就不是那么够。  
这篇文章是以一个前端开发者的角度为切入点，去讲述 Nginx 的使用经验的，主要也是为了让咱们前端朋友更快的入门 Nginx！  
实不相瞒，我也是前端开发者![😊](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-assets/asset/twemoji/2.6.0/svg/1f60a.svg~tplv-t2oaga2asx-image.image)

”

4

回复

查看更多回复

[![小叮咚zz的头像](media/小叮咚zz的头像.jpg)](https://juejin.cn/user/4336129592008030)

[小叮咚zz![掘友等级](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/65a6a28f15d70e5a77bf881c5ec5340d.svg "掘友等级")](https://juejin.cn/user/4336129592008030)

web前端 @ 公司很大 , 你等一下1年前

盗过来大佬的一句话: 代理其实就是一个中介，A和B本来可以直连，中间插入一个C，C就是中介。刚开始的时候，代理多数是帮助内网client访问外网server用的（比如HTTP代理），从内到外 . 后来出现了反向代理，"反向"这个词在这儿的意思其实是指方向相反，即代理将来自外网client的请求forward到内网server，从外到内

11

1

[![](media/d74341c6c18934fb1b64a5ad586e5508~100x100.jpg)](https://juejin.cn/user/4353721774642536)

[敲完代码再睡觉](https://juejin.cn/user/4353721774642536)

（作者）1年前

简单易懂![👍](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-assets/asset/twemoji/2.6.0/svg/1f44d.svg~tplv-t2oaga2asx-image.image)

点赞

回复

全部评论 36

最新

最热

[![我不知道我想要的是什么的头像](media/我不知道我想要的是什么的头像.jpg)](https://juejin.cn/user/2700056290937005)

[我不知道我想要的是什么![lv-2](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/img/lv-2.99ba5b2.png "创作等级")![掘友等级](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/70f1e5e3a2fde62e0d623009ab80cb12.svg "掘友等级")](https://juejin.cn/user/2700056290937005)

3月前

通俗易懂嘞

点赞

1

[![](media/d74341c6c18934fb1b64a5ad586e5508~100x100.jpg)](https://juejin.cn/user/4353721774642536)

[敲完代码再睡觉](https://juejin.cn/user/4353721774642536)

（作者）3月前

![[嘿哈]](media/[嘿哈].png)

点赞

回复

[![user8635047349650的头像](media/user8635047349650的头像.jpg)](https://juejin.cn/user/2041167065915544)

[user8635047349650![掘友等级](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/65a6a28f15d70e5a77bf881c5ec5340d.svg "掘友等级")](https://juejin.cn/user/2041167065915544)

5月前

个人理解反向代理就好比是房产中介，商超，正向代理就好比是批发商，品牌代理商

1

1

[![](media/d74341c6c18934fb1b64a5ad586e5508~100x100.jpg)](https://juejin.cn/user/4353721774642536)

[敲完代码再睡觉](https://juejin.cn/user/4353721774642536)

（作者）4月前

对，是这么个意思![[嘿哈]](media/[嘿哈].png)

点赞

回复

[![李青不吃螺蛳粉的头像](media/李青不吃螺蛳粉的头像.jpg)](https://juejin.cn/user/4169827014674615)

[李青不吃螺蛳粉![lv-1](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/img/lv-1.9d93e13.png "创作等级")![掘友等级](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/8c6985e2aa4c06f307ae3734da4b43ac.svg "掘友等级")

![](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/07302452a7ad81cb43a173b5cd580237.svg)

](https://juejin.cn/user/4169827014674615)

前端开发 @ 无5月前

![[捂脸]](media/[捂脸].png)虽然我知道你这标题是故意吸引注意力，但是我还是不太爽

点赞

1

[![](media/d74341c6c18934fb1b64a5ad586e5508~100x100.jpg)](https://juejin.cn/user/4353721774642536)

[敲完代码再睡觉](https://juejin.cn/user/4353721774642536)

（作者）5月前

![[捂脸]](media/[捂脸].png)多担待，并无歧视之意，就像《连后端都看得懂的vue教程》这种一样。多数前端同学确实距离nginx挺遥远的，这么取题是为了占在他们角度思考，让他们更有想了解的欲望。

1

回复

[![用户9283346887137的头像](media/用户9283346887137的头像.jpg)](https://juejin.cn/user/2239074113166392)

[用户9283346887137![掘友等级](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/9b2f31c663d17de59dd9e5fff272bb85.svg "掘友等级")](https://juejin.cn/user/2239074113166392)

6月前

按照你的实列，动静分离做到了，其他两个反而不行，重启nginx，报错，反向代理那个，说配置文件的"proxy_pass"不允许写这，

![image](media/image-2.jpg)

点赞

回复

[![bin的头像](media/bin的头像.jpg)](https://juejin.cn/user/2752832844349239)

[bin![lv-3](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/img/lv-3.7938ebc.png "创作等级")![掘友等级](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/65a6a28f15d70e5a77bf881c5ec5340d.svg "掘友等级")

![](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/07302452a7ad81cb43a173b5cd580237.svg)

](https://juejin.cn/user/2752832844349239)

前端开发工程师 @ 腾讯11月前

配置反向代理  
proxy_pass 应该是配置到location里面

点赞

1

[![](media/d74341c6c18934fb1b64a5ad586e5508~100x100.jpg)](https://juejin.cn/user/4353721774642536)

[敲完代码再睡觉](https://juejin.cn/user/4353721774642536)

（作者）10月前

多谢提醒，文中有一处是失误了放到了外面。但是我刚试了也还是可以生效的，感觉这个好像没有那么敏感？

点赞

回复

[![安心zzz的头像](media/安心zzz的头像.jpg)](https://juejin.cn/user/3606861687826493)

[安心zzz![掘友等级](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/9b2f31c663d17de59dd9e5fff272bb85.svg "掘友等级")](https://juejin.cn/user/3606861687826493)

java研发工程师 @ 腾讯科技1年前

写的不错

点赞

1

[![](media/d74341c6c18934fb1b64a5ad586e5508~100x100.jpg)](https://juejin.cn/user/4353721774642536)

[敲完代码再睡觉](https://juejin.cn/user/4353721774642536)

（作者）11月前

![[愉快]](media/[愉快].png)谢谢支持

点赞

回复

[![用户1900237554723的头像](media/用户1900237554723的头像.jpg)](https://juejin.cn/user/1029575926877960)

[用户1900237554723![掘友等级](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/65a6a28f15d70e5a77bf881c5ec5340d.svg "掘友等级")](https://juejin.cn/user/1029575926877960)

1年前

太棒了，新手确实能看懂

1

1

[![](media/d74341c6c18934fb1b64a5ad586e5508~100x100.jpg)](https://juejin.cn/user/4353721774642536)

[敲完代码再睡觉](https://juejin.cn/user/4353721774642536)

（作者）1年前

![[抱拳]](media/[抱拳].png)

点赞

回复

[![小叮咚zz的头像](media/小叮咚zz的头像.jpg)](https://juejin.cn/user/4336129592008030)

[小叮咚zz![掘友等级](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/65a6a28f15d70e5a77bf881c5ec5340d.svg "掘友等级")](https://juejin.cn/user/4336129592008030)

web前端 @ 公司很大 , 你等一下1年前

盗过来大佬的一句话: 代理其实就是一个中介，A和B本来可以直连，中间插入一个C，C就是中介。刚开始的时候，代理多数是帮助内网client访问外网server用的（比如HTTP代理），从内到外 . 后来出现了反向代理，"反向"这个词在这儿的意思其实是指方向相反，即代理将来自外网client的请求forward到内网server，从外到内

11

1

[![](media/d74341c6c18934fb1b64a5ad586e5508~100x100.jpg)](https://juejin.cn/user/4353721774642536)

[敲完代码再睡觉](https://juejin.cn/user/4353721774642536)

（作者）1年前

简单易懂![👍](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-assets/asset/twemoji/2.6.0/svg/1f44d.svg~tplv-t2oaga2asx-image.image)

点赞

回复

[![用户6933840418415的头像](media/用户6933840418415的头像.jpg)](https://juejin.cn/user/2217057773957870)

[用户6933840418415![掘友等级](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/9b2f31c663d17de59dd9e5fff272bb85.svg "掘友等级")](https://juejin.cn/user/2217057773957870)

1年前

通俗易懂，能做到这个也不容易。给作者点赞

1

1

[![](media/d74341c6c18934fb1b64a5ad586e5508~100x100.jpg)](https://juejin.cn/user/4353721774642536)

[敲完代码再睡觉](https://juejin.cn/user/4353721774642536)

（作者）1年前

谢谢~

点赞

回复

[![用户3460199201733的头像](media/用户3460199201733的头像.jpg)](https://juejin.cn/user/3271781199398045)

[用户3460199201733![掘友等级](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/65a6a28f15d70e5a77bf881c5ec5340d.svg "掘友等级")](https://juejin.cn/user/3271781199398045)

2年前

有个小错误![😘](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-assets/asset/twemoji/2.6.0/svg/1f618.svg~tplv-t2oaga2asx-image.image) 配置负载均衡下面写成80001了

点赞

1

[![](media/d74341c6c18934fb1b64a5ad586e5508~100x100.jpg)](https://juejin.cn/user/4353721774642536)

[敲完代码再睡觉](https://juejin.cn/user/4353721774642536)

（作者）2年前

已修改，谢谢指正![😊](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-assets/asset/twemoji/2.6.0/svg/1f60a.svg~tplv-t2oaga2asx-image.image)

点赞

回复

[![念想_的头像](media/念想_的头像.jpg)](https://juejin.cn/user/1802854799776856)

[念想_![掘友等级](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/65a6a28f15d70e5a77bf881c5ec5340d.svg "掘友等级")](https://juejin.cn/user/1802854799776856)

前端开发2年前

通俗易懂，感谢！手动点赞！![👍](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-assets/asset/twemoji/2.6.0/svg/1f44d.svg~tplv-t2oaga2asx-image.image)

点赞

1

[![](media/d74341c6c18934fb1b64a5ad586e5508~100x100.jpg)](https://juejin.cn/user/4353721774642536)

[敲完代码再睡觉](https://juejin.cn/user/4353721774642536)

（作者）2年前

谢谢![🤝](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-assets/asset/twemoji/2.6.0/svg/1f91d.svg~tplv-t2oaga2asx-image.image)

点赞

回复

[![stevekeol的头像](media/stevekeol的头像.jpg)](https://juejin.cn/user/483440848543271)

[stevekeol![lv-1](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/img/lv-1.9d93e13.png "创作等级")![掘友等级](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/9b2f31c663d17de59dd9e5fff272bb85.svg "掘友等级")](https://juejin.cn/user/483440848543271)

区块链研发工程师2年前

什么叫连前端都看得懂，你看不起前端呀大哥

6

4

[![](media/d74341c6c18934fb1b64a5ad586e5508~100x100.jpg)](https://juejin.cn/user/4353721774642536)

[敲完代码再睡觉](https://juejin.cn/user/4353721774642536)

（作者）2年前

大哥你误会了！nginx 基本是后端的必修课，而不少前端平常因为跟 nginx 接触得也比后端少很多，自然一般的 nginx 文档对前端的亲和力就不是那么够。  
这篇文章是以一个前端开发者的角度为切入点，去讲述 Nginx 的使用经验的，主要也是为了让咱们前端朋友更快的入门 Nginx！  
实不相瞒，我也是前端开发者![😊](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-assets/asset/twemoji/2.6.0/svg/1f60a.svg~tplv-t2oaga2asx-image.image)

12

回复

[![](media/918211e5cbc44bdab574403e0b3dfe6c~100x100.jpg)](https://juejin.cn/user/3755587453860926)

[ggbdpq

![](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/07302452a7ad81cb43a173b5cd580237.svg)

](https://juejin.cn/user/3755587453860926)

回复

[敲完代码再睡觉](https://juejin.cn/user/4353721774642536)

2年前

我本质翻译一下，连阿猫阿狗都看得懂的《Nginx 入门指南》，这样就好，就不单单是看不起前端了，是看不起在座的各位。其实这文章内容挺通俗易懂的，就是标题党可能跟营销号学得一半，学费没交足，故意'引人注目'。

“

大哥你误会了！nginx 基本是后端的必修课，而不少前端平常因为跟 nginx 接触得也比后端少很多，自然一般的 nginx 文档对前端的亲和力就不是那么够。  
这篇文章是以一个前端开发者的角度为切入点，去讲述 Nginx 的使用经验的，主要也是为了让咱们前端朋友更快的入门 Nginx！  
实不相瞒，我也是前端开发者![😊](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-assets/asset/twemoji/2.6.0/svg/1f60a.svg~tplv-t2oaga2asx-image.image)

”

4

回复

查看更多回复

[![小胖说java的头像](media/小胖说java的头像.jpg)](https://juejin.cn/user/395479918322125)

[小胖说java![lv-3](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/img/lv-3.7938ebc.png "创作等级")![掘友等级](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/69eb0ad2f93abf938e832fe53b979c10.svg "掘友等级")](https://juejin.cn/user/395479918322125)

java攻城狮|wx搜索Madison龙少2年前

好棒的作者![👍](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-assets/asset/twemoji/2.6.0/svg/1f44d.svg~tplv-t2oaga2asx-image.image)nb

1

1

[![](media/d74341c6c18934fb1b64a5ad586e5508~100x100.jpg)](https://juejin.cn/user/4353721774642536)

[敲完代码再睡觉](https://juejin.cn/user/4353721774642536)

（作者）2年前

谢谢，共同学习，共同进步！

点赞

回复

[![ToKy的头像](media/ToKy的头像.jpg)](https://juejin.cn/user/1565296595315325)

[ToKy![掘友等级](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/9b2f31c663d17de59dd9e5fff272bb85.svg "掘友等级")](https://juejin.cn/user/1565296595315325)

2年前

咦，深大校友？![😉](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-assets/asset/twemoji/2.6.0/svg/1f609.svg~tplv-t2oaga2asx-image.image)

点赞

1

[![](media/d74341c6c18934fb1b64a5ad586e5508~100x100.jpg)](https://juejin.cn/user/4353721774642536)

[敲完代码再睡觉](https://juejin.cn/user/4353721774642536)

（作者）2年前

眼力过人啊哈哈，不过很遗憾我只是喜欢去深大玩

点赞

回复

[![敲完代码再睡觉的头像](media/敲完代码再睡觉的头像.jpg)](https://juejin.cn/user/4353721774642536)

[敲完代码再睡觉![lv-3](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/img/lv-3.7938ebc.png "创作等级")![掘友等级](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/65a6a28f15d70e5a77bf881c5ec5340d.svg "掘友等级")](https://juejin.cn/user/4353721774642536)

（作者）前端工程师 @ 大疆创新2年前

Nginx第二弹，欢迎关注： [![](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/3f843e8626a3844c624fb596dddd9674.svg)juejin.im](https://juejin.im/post/5e9ab2e851882573a67f62a0 "https://juejin.im/post/5e9ab2e851882573a67f62a0")

1

回复

[![zhelingwang的头像](media/zhelingwang的头像.jpg)](https://juejin.cn/user/2594503173603992)

[zhelingwang![lv-3](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/img/lv-3.7938ebc.png "创作等级")![掘友等级](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/9b2f31c663d17de59dd9e5fff272bb85.svg "掘友等级")](https://juejin.cn/user/2594503173603992)

前端2年前

非常棒

3

1

[![](media/d74341c6c18934fb1b64a5ad586e5508~100x100.jpg)](https://juejin.cn/user/4353721774642536)

[敲完代码再睡觉](https://juejin.cn/user/4353721774642536)

（作者）2年前

谢谢，互相学习！

点赞

回复

[![敲完代码再睡觉的头像](media/敲完代码再睡觉的头像.jpg)](https://juejin.cn/user/4353721774642536)

[敲完代码再睡觉![lv-3](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/img/lv-3.7938ebc.png "创作等级")![掘友等级](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/65a6a28f15d70e5a77bf881c5ec5340d.svg "掘友等级")](https://juejin.cn/user/4353721774642536)

（作者）前端工程师 @ 大疆创新2年前

本文不仅适合入门，更适合收藏了以后用的时候来查阅![😏](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-assets/asset/twemoji/2.6.0/svg/1f60f.svg~tplv-t2oaga2asx-image.image)

2

回复

[![海口椰子哥的头像](media/海口椰子哥的头像.jpg)](https://juejin.cn/user/4248168659964110)

[海口椰子哥![lv-2](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/img/lv-2.99ba5b2.png "创作等级")![掘友等级](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/8c6985e2aa4c06f307ae3734da4b43ac.svg "掘友等级")](https://juejin.cn/user/4248168659964110)

2年前

好文 简单易懂入门

2

1

[![](media/d74341c6c18934fb1b64a5ad586e5508~100x100.jpg)](https://juejin.cn/user/4353721774642536)

[敲完代码再睡觉](https://juejin.cn/user/4353721774642536)

（作者）2年前

![😄](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-assets/asset/twemoji/2.6.0/svg/1f604.svg~tplv-t2oaga2asx-image.image) 谢谢评价，后面还会出一篇续集《Nginx 助力前端开发》，如果有兴趣到时候留意一下哈