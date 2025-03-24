2022-03-30   分类: JavaScript/前端   470浏览

什么是跨域 ？

跨域这个问题大家并不陌生，这也是面试的高频问题，很多人都背过，什么因为同源策略啊，CORS 啊等等，跨域的标致就是浏览器控制台出现 Access to XMLHttpRequest at 'https://xxx.xxx.com' from origin 'https://xxx.xxx.com' has been blocked by CORS policy: No '

Access-Control-Allow-Origin' header is present on the requested resource.。

首先，只有 Web 浏览器才会产生跨域，这是因为浏览器的同源策略在限制。同源策略就是 [域名（又称为主机 host），端口（port），协议（protocol）] 要统一，否则就会被浏览器判定为跨域，然后拒绝请求。但是严格的说，浏览器并不是拒绝所有的跨域请求，实际上拒绝的是跨域的读操作。

同源策略并不是不好，它在一定程度上保证了浏览器的安全，只是无法适应现代的潮流，在工程服务化后，不同职责的服务分散在不同的工程中，往往这些工程的域名是不同的，但一个需求可能需要对应到多个服务，这时便需要调用不同服务的接口。

No 'Access-Control-Allow-Origin' header跨域问题「踩坑记」
哪些情况下会产生跨域 ？

No 'Access-Control-Allow-Origin' header跨域问题「踩坑记」
什么是 CORS ？

    跨域资源共享(CORS) 是一种机制，它使用额外的 HTTP 头来告诉浏览器 让运行在一个 origin (domain) 上的 Web 应用被准许访问来自不同源服务器上的指定的资源。当一个资源从与该资源本身所在的服务器不同的域、协议或端口请求一个资源时，资源会发起一个跨域 HTTP 请求。 比如，站点 http://domain-a.com 的某 HTML 页面通过 <img> 的 src 请求

    http://domain-b.com/image.jpg。网络上的许多页面都会加载来自不同域的 CSS 样式表，图像和脚本等资源。 出于安全原因，浏览器限制从脚本内发起的跨源 HTTP 请求。 例如，XMLHttpRequest 和 Fetch API 遵循同源策略。 这意味着使用这些 API 的 Web 应用程序只能从加载应用程序的同一个域请求 HTTP 资源，除非响应报文包含了正确 CORS 响应头。


No 'Access-Control-Allow-Origin' header跨域问题「踩坑记」

    跨域资源共享（ CORS ）机制允许 Web 应用服务器进行跨域访问控制，从而使跨域数据传输得以安全进行。现代浏览器支持在 API 容器中（例如 XMLHttpRequest 或 Fetch ）使用 CORS，以降低跨域 HTTP 请求所带来的风险。

不过呢，并不一定是浏览器限制了发起跨站请求，也可能是跨站请求可以正常发起，但是返回结果被浏览器拦截了。

在“上古时代”，解决跨域有很多黑科技，什么 JSONP 啊，window.name 啊，document.domain 啊等等都用上了，但现代用的基本都是CORS跨域，所以上述方法在这里就不讨论了。

由上述可知，实现 CORS 通信的关键是服务器。只要服务器实现了 CORS 接口，就可以跨源通信。

虽然工作重心在后端，但是作为前端，也要了解这方面的知识，否则就会出现204预请求，然后后端没处理，就甩锅给前端的情况。

下面介绍一些 CORS 过程中常见的 HTTP 响应头（Response Headers）。
什么是 Access-Control-Allow-Origin ？

这是 HTTP 响应首部中的一个字段，

具体格式是：

Access-Control-Allow-Origin: <origin> | *。

其中，origin 参数的值指定了允许访问该资源的外域 URI。对于不需要携带身份凭证的请求，服务器可以指定该字段的值为通配符，表示允许来自所有域的请求。

如果服务端指定了具体的域名而非*，那么响应首部中的 Vary 字段的值必须包含 Origin。这将告诉客户端：服务器对不同的源站返回不同的内容。例如：

// 只响应来自 http://mozilla.com 的请求

Access-Control-Allow-Origin: http://mozilla.com

什么是 Access-Control-Allow-Methods ？

该字段必需，它的值是逗号分隔的一个字符串，表明服务器支持的所有跨域请求的方法。注意，返回的是所有支持的方法，而不单是浏览器请求的那个方法。这是为了避免多次"预检"请求。

具体格式是：

Access-Control-Allow-Methods: <method>[, <method>]*
什么是 Access-Control-Allow-Headers ?

可支持的请求首部名字。请求头会列出所有支持的首部列表，用逗号隔开。

如果浏览器请求包括

Access-Control-Request-Headers字段，则

Access-Control-Allow-Headers字段是必需的。它也是一个逗号分隔的字符串，表明服务器支持的所有头信息字段，不限于浏览器在"预检"中请求的字段。

具体格式是：

Access-Control-Allow-Headers: <header-name>[, <header-name>]*
什么是 Access-Control-Allow-Credentials ？

该字段可选。它的值是一个布尔值，表示是否允许发送 Cookie。默认情况下，Cookie 不包括在 CORS 请求之中。设为 true，即表示服务器明确许可，Cookie 可以包含在请求中，一起发给服务器。
浏览器的正常请求和回应

一旦服务器通过了"预检"请求，以后每次浏览器正常的 CORS 请求，就都跟简单请求一样，会有一个 Origin 头信息字段。服务器的回应，也都会有一个

Access-Control-Allow-Origin 头信息字段。

好了，铺垫完成，接下来要说说我踩的坑了。

No 'Access-Control-Allow-Origin' header跨域问题「踩坑记」
问题实记
项目背景

公司因历史遗留，同时存在3种后端语言：Java，PHP，Node.js，因为后端同事都不懂 Node.js，所以 Node 项目一直是前端维护。老项目用的是 Koa 框架，之前我在上面写逻辑，上传图片是没有问题的，直到我用了 Nest 框架重构，将后台管理系统的逻辑拆分解耦出来。
踩坑过程

新项目一切请求都正常，本地跑的时候也正常，但是到了线上，只有上传图片不正常。每次上传都会预请求一次 204 OPTIONS，这一步没问题，但接下来的 POST 请求就有问题了：

No 'Access-Control-Allow-Origin' header跨域问题「踩坑记」

我一看控制台的信息，结合多年的开发经验（其实并没有），这不就是跨域嘛，于是看代码，main.ts 中已经有了 app.enableCors()，百思不得其解，于是 Google，发现也有人遇到过类似的问题，说是 Nest 某些版本在线上环境无法正确使用 CORS，然后就形成了下列代码：

No 'Access-Control-Allow-Origin' header跨域问题「踩坑记」

但是，并没有什么卵用，调试了好一会儿才注意到响应头是 nginx 返回的，然后就像发现新大陆一样屁颠屁颠的找运维老大：代码里面已经设置了 CORS 跨域了，是不是被 nginx 拦截了？（公司的服务器通过 nginx 做负载均衡，然后才到 node）

然后运维老大很配合的帮我配置了 nginx 的跨域，然后就悲剧了，连 OPTIONS 都过不去：

No 'Access-Control-Allow-Origin' header跨域问题「踩坑记」

控制台的大致意思是 CORS 规则冲突了，只能使用一种。

No 'Access-Control-Allow-Origin' header跨域问题「踩坑记」

所以这个问题其实和 nginx 没有什么关系，运维大佬看了访问日志，说是 OPTIONS 请求是响应了的，但是到 POST 请求的时候就断开连接了，然后让我试试直接访问端口，于是控制台又出现了如下信息：

No 'Access-Control-Allow-Origin' header跨域问题「踩坑记」


唔，大致意思是，源头是 https 协议的话，就不能请求 http 协议的资源。

于是我把网站上的 https 换成 http，就出现了如下信息：


No 'Access-Control-Allow-Origin' header跨域问题「踩坑记」

我就觉得很奇怪，因为本地开发的时候，是能正常上传图片的。唯一不同的地方就在于，线上使用的是 pm2 进程管理工具，而我本地用的是自带的 nodemon。为了验证我的猜测，于是自己的电脑上也装了 pm2，然后跑起来，然后就。。。Bug 果然复现了。

于是让运维大佬不用 pm2 直接用 nodemon 启动试试，然后折腾我很久的跨域问题就“解决了”。为什么打引号呢，因为运维大佬并不是很想用 nodemon 来管理进程，主要是如果服务器宕机，不能自动拉起，战役还远没有结束。

然后我就围绕着 pm2 继续探索，把官方文档看了个遍，也没找到关键点，郁闷之下，只好检查的 pm2 启动配置，然后注意到这个：

No 'Access-Control-Allow-Origin' header跨域问题「踩坑记」

这是当初我为了输出日志的时候，更新了文件，防止被 pm2 监听到，导致服务一直重启所做的措施。然后就想到了，我上传图片的时候，是先在硬盘保存，然后读取 Buffer 流，然后再上传到 oss，最后删掉硬盘的图片，核心代码如下：

No 'Access-Control-Allow-Origin' header跨域问题「踩坑记」

No 'Access-Control-Allow-Origin' header跨域问题「踩坑记」

所以我就在想，是不是因为上传的时候，存本地的图片触发了 pm2 的监听，导致服务重启，所以就会报 net::ERR_CONNECTION_RESET，于是我改成了临时目录 const uploadCachePath = '/tmp/assets/uploads'; (Mac OS、Linux 都有这个目录)，然后 pm2 启动，上传，成功。

No 'Access-Control-Allow-Origin' header跨域问题「踩坑记」

至此，折磨了我近一周的 Bug 终于修复，结果和跨域没有半毛钱关系。

No 'Access-Control-Allow-Origin' header跨域问题「踩坑记」
插曲

有读者可能注意到 /tmp/assets/uploads 路径，要是同事用的是 Windows 系统开发咋办？这个我自然也想到了，于是改了 pm2 的启动项：

No 'Access-Control-Allow-Origin' header跨域问题「踩坑记」

可是无论怎样改，依然会触发上述 Bug，因为服务器同时跑着 2 个 Node 项目，另一个项目也有自己的 pm2 启动项，所以感觉这个配置被另一个覆盖了，pm2 似乎是全局的。如果有其他大神深入了解过 pm2 的可以指点一下。

所以和运维大佬讨论了一下，给我开了权限，就暂时用这个临时目录，待以后找到更好的解决方案再优化，反正目前这个项目也只有我一人在维护。
遇到的其他场景
1. 服务器宕机

就在我刚找到解决方案的时候，我带的小弟跑过来问我是不是动了配置文件，老项目怎么都跨域了。我去看他的控制台，确实有 Access to ... has been blocked by CORS policy: No '

Access-Control-Allow-Origin' header is present on the requested resource. 的信息，由于刚踩的坑，何况我也没动过配置文件，所以觉得这肯定不是跨域问题。然后我看了服务器的日志，发现一直在重启，因为有个模块他没同步上去，只同步了路由文件，导致路由找不到对应的函数，服务就一直在报错重启，根本就没处理请求。于是让他把代码重新上传，问题解决。

所以个人猜测（因为没有权限看服务器的配置），这次跨域是 nginx 代理的时候，因为访问不到 Node 服务，所以自然而然地就读不到 CORS 配置，然后报跨域错误。
2. Axios 的自行检查

Axios 创建实例时，有个字段需要注意：

No 'Access-Control-Allow-Origin' header跨域问题「踩坑记」

如果不设为 false，则会得到下面报错：

No 'Access-Control-Allow-Origin' header跨域问题「踩坑记」

原因就是前面提到的

Access-Control-Allow-Credentials 字段，CORS 请求默认不发送 Cookie 和 HTTP 认证信息。如果要把 Cookie 发到服务器，一方面要服务器同意:

    Access-Control-Allow-Credentials: true

另一方面，开发者必须在 AJAX 请求中打开 withCredentials 属性，也就是 Axios 默认打开的 withCredentials: true。

否则，即使服务器同意发送 Cookie，浏览器也不会发送。或者，服务器要求设置 Cookie，浏览器也不会处理。

但是，如果省略withCredentials设置，有的浏览器还是会一起发送 Cookie。这时，需要显示关闭 withCredentials: false。

需要注意的是，如果要发送 Cookie，

Access-Control-Allow-Origin 就不能设为星号，必须指定明确的、与请求网页一致的域名。同时，Cookie 依然遵循同源政策，只有用服务器域名设置的 Cookie 才会上传，其他域名的 Cookie 并不会上传，且（跨源）原网页代码中的 document.cookie 也无法读取服务器域名下的 Cookie。
总结

由上述可以总结，在后端配置了 CORS 的情况下，还会造成 Access to ... has been blocked by CORS policy ... 的情况大致有：

    服务器突然重启，导致代理服务器转发中断；
    服务器宕机，导致代理服务器读不到 CORS 配置；
    Axios 等请求插件设置了withCredentials: true，导致先行验证并拦截了请求；

有些时候，浏览器控制台给出的错误信息，不一定能真正地指出问题的所在，做为前端，还需要多去了解一些更本质的东西。

No 'Access-Control-Allow-Origin' header跨域问题「踩坑记」