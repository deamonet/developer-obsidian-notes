
[赤蓝紫](https://juejin.cn/user/1416592459577320) 2022年06月17日 14:14 ·  阅读 912


> 本文并不是讲解Cookie在实际项目中的应用,而只是**简单**地实践一下,自动保存Cookie,然后后续请求自动携带Cookie，主要是通过使用刚学到的fetch API和差不多快忘记的express来实践。

Cookie 用于在客户端存储会话信息。它通过服务器响应请求时，响应头的`Set-Cookie`字段来设置 Cookie。**Cookie 是服务端生成，保存在客户端**

![image-20220529091708813](media/image-20220529091708813.webp)

这个 HTTP 响应会设置一个名为`name`,值为`value`的 Cookie。**名和值在发送时都会经过 URL 编码**。 浏览器会存储这些会话信息，并且**之后的每个请求都会通过请求头的`Cookie`字段再将它们发回服务器**。

```ini
GET /index.jsl HTTP/1.1
Cookie: name=value
Other-header: other-header-value
复制代码
```

发回给服务器的`Cookie`字段可用于唯一标识发送请求的客户端。

Cookie 有大小限制，一般 4K 左右。

## Cookie 的构成

-   **名称**(`name`=value)：Cookie 的名称。不区分大小写，必须经过 URL 编码。
-   **值**(name=`value`)：Cookie 的值。必须经过 URL 编码
-   **域**(`Domain`=clzczh.top)：Cookie 有效的域。发送到该域名的所有请求都会包含对应的 Cookie。如果不明确设置，则默认为设置 Cookie 的域。
-   **路径**(`Path`=/)：请求 URL 中包含此路径才会携带 Cookie 发送请求。
-   **过期时间**(`Expires`=Date)：删除 Cookie 的时间戳，用于设置删除 Cookie 的时间，这个值是 GMT 格式（Wdy, DD-Mon-YYYY HH:MM:SS GMT）。当到达该时间后，就会删除 Cookie；没到达该时间时，即使关闭浏览器，Cookie 还会保留。**把过期时间设置为过去的时间会立即删除 Cookie**。默认只在浏览器关闭前有效
-   **安全标志**(`Secure`)：只在 HTTPS 安全连接时才可以发送 Cookie
-   **禁止 JS 读取 Cookie**(`HttpOnly`)：通过 JS 脚本无法获取 Cookie，可以有效地防止`XSS攻击`。

Cookie 中实际发送给服务器的只有名/值对，其他部分只是告诉浏览器什么时候应该在请求中携带 Cookie 等。

## Cookie 的简单实践

简单地说一下下面的代码：

1.  express 实现的后端服务
2.  通过`app.post`开启 post 接口
3.  `res.cookie`设置 Cookie，第一个参数是 Cookie 名，第二个参数是 Cookie 值，第三个参数是 Cookie 的限制对象(如过期时间`expires`)

```js
const express = require("express");
const cors = require("cors");

const app = express();

app.use(cors());

app.post("/token", function (req, res) {
  // 设置Cookie
  res.cookie("token", "123456", {
    httpOnly: true,
    expires: new Date(2030, 10, 10),
  });

  res.status(200).json({
    msg: "获取token成功",
  });
});

app.get("/getInfo", function (req, res) {
  res.json({
    msg: "成功",
  });
});

app.listen(8088, () => {
  console.log("http://localhost:8088");
});
复制代码
```

前端试一下，能不能接收到`Cookie`。(使用 Fetch API，免装`axios`，实际使用和`axios`差不多，简单使用可查看之前的文章)

```html
<body>
  <button id="btn">获取token</button>
  <button id="test-btn">测试自动携带Cookie</button>
  <script>
    const btn = document.getElementById("btn");
    btn.addEventListener("click", fetchData);

    const testBtn = document.getElementById("test-btn");
    testBtn.addEventListener("click", getInfo);

    function fetchData() {
      fetch("http://localhost:8088/token", {
        method: "post",
      }).then((res) => {
        // 获取响应的数据
        res.json().then((data) => {
          console.log(data);
        });
      });
    }

    function getInfo() {
      fetch("http://localhost:8088/getInfo").then((res) => {
        console.log(res);
      });
    }
  </script>
</body>
复制代码
```

![image-20220529093428201](media/image-20220529093428201.webp)

看似万事大吉了，实际上，还是有问题的：

![image-20220529093530601](media/image-20220529093530601.webp)

Cookie压根没存到客户端。

### 解决方案1

1.  使用`fetch`发送请求时，设置`credentials`为`include`(`axios`则是设置`withCredentials`为`true`)，这样子跨域请求时夜会发送Cookie(也可以用来保存跨域请求响应的Cookie)
    
    ```js
    fetch('http://localhost:8088/token', {
        method: 'post',
        credentials: 'include'
    })
    ```
    
    ![image-20220529100902883](media/image-20220529100902883.webp)
    
2.  当我们设置`credentials`为`include`时，
    
    -   我们解决跨域时的`Access-Control-Allow-Origin`不应该还是通配符，而应该是具体的地址，所以**后端express**应该调整一下不再使用`cors`中间件，而是自己设置响应头
    -   `Access-Control-Allow-Credentials`也应该设置为`true`
    
    ```js
    // 使用cors中间件部分换成下面的形式
    app.use(function (req, res, next) {
      res.header('Access-Control-Allow-Origin', 'http://127.0.0.1:5501')
      res.header('Access-Control-Allow-Credentials', 'true')
      next()
    })
    复制代码
    ```
    
    ![image-20220529103925082](media/image-20220529103925082.webp)
    
3.  上面已经的警告已经说了：`Cookie`有一个`SameSite`属性，它默认是`Lax`，要求响应是对顶层导航的响应(**这个顶层导航并不是很懂，有懂得小伙伴欢迎评论**)。先按她的提示，设置Cookie的`SameSite`属性为`none`(安全性会下降)。**有`SameSite`属性的话，也必须要有`Secure`属性**
    
    ```js
    // 设置Cookie
    res.cookie("token", "123456", {
      httpOnly: true,
      expires: new Date(2030, 10, 10),
      secure: true,
      sameSite: 'none'
    });
    复制代码
    ```
    
    ![image-20220529105604140](media/image-20220529105604140.webp)
    
    ![image-20220529105657457](media/image-20220529105657457.png)
    
    最终代码：
    
    express：
    
    ```js
    const express = require("express");
    const cors = require("cors");
    
    const app = express();
    
    app.use(function (req, res, next) {
      res.header('Access-Control-Allow-Origin', 'http://127.0.0.1:5501')
      res.header('Access-Control-Allow-Credentials', 'true')
      next()
    })
    
    app.post("/token", function (req, res) {
    
      // 设置Cookie
      res.cookie("token", "123456", {
        httpOnly: true,
        expires: new Date(2030, 10, 10),
        secure: true,
        sameSite: 'none'
      });
    
      res.status(200).json({
        msg: "获取token成功",
      });
    });
    
    app.get("/getInfo", function (req, res) {
      res.json({
        msg: "成功",
      });
    });
    
    app.listen(8088, () => {
      console.log("http://localhost:8088");
    });
    复制代码
    ```
    
    html
    
    ```html
    <body>
        <button id="btn">获取token</button>
        <button id="test-btn">测试自动携带Cookie</button>
        <script>
            const btn = document.getElementById('btn')
            btn.addEventListener('click', fetchData)
    
            const testBtn = document.getElementById('test-btn')
            testBtn.addEventListener('click', getInfo)
    
            function fetchData() {
                fetch('http://localhost:8088/token', {
                    method: 'post',
                    credentials: 'include'
                })
            }
    
            function getInfo() {
                fetch('http://localhost:8088/getInfo', {
                    credentials: 'include'
                })
            }
        </script>
    </body>
    复制代码
    ```
    

### 解决方案2

上面的解决方案1,非常的麻烦,还把Cookie的`SameSite`属性改成`None`了,安全性也会下降一点

实际上呢,我们有一个更简单的解决方案,只需要把他们变成不跨域就行了。

用`express`来测试的话,就是把之前的html代码放到`express`下的`public`文件夹里,

然后通过`app.use(express.static(__dirname + '/public'))`将静态文件目录设置为`项目根目录+/public`

![image-20220529110925343](media/image-20220529110925343.webp)

然后,访问`http://localhost:8088`,就是我们写的html,不跨域,自然就没有解决方案1中出现的问题了.

当然,只看上面的例子的话,好像是用解决方案2的话,前后端就不能很好的分离了.其实并不是,我们可以通过`nginx`的代理来解决前后端的跨域问题.

可以使用`Vue`来简单实践代理能否解决这个保存携带Cookie问题.

首先呢?我们需要修改配置文件,实现代理.

```js
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [vue()],
  server: {
    // 实现其他设备能访问本机开启的服务
    host: '0.0.0.0',

    proxy: {
      '/api': {
        target: 'http://localhost:8088',
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api/, '')
      }
    }
  }
})
复制代码
```

fetch API的请求地址就不再需要去到后端的那个接口地址了,而是变成`/api`即可,这样子代理就会把这个请求转发给真实服务器.

```html
<template>
  <button @click="fetchData">获取token</button>
  <button @click="getInfo">测试自动携带Cookie</button>
</template>

<script setup>
function fetchData() {
  fetch("/api/token", {
    method: "post",
  }).then((res) => {
    // 获取响应的数据
    res.json().then((data) => {
      console.log(data);
    });
  });
}

function getInfo() {
  fetch("/api/getInfo").then((res) => {
    console.log(res);
  });
}
</script>
复制代码
```

![image-20220529113512641](media/image-20220529113512641.webp)

![image-20220529113535623](media/image-20220529113535623.png)

分类：

[前端](https://juejin.cn/frontend)

标签：

[JavaScript](https://juejin.cn/tag/JavaScript)[Express](https://juejin.cn/tag/Express)

文章被收录于专栏：

![cover](media/cover-1.webp)

JavaScript

JavaScript

[安装掘金浏览器插件](https://juejin.cn/extension/?utm_source=standalone&utm_medium=post&utm_campaign=extension_promotion)

多内容聚合浏览、多引擎快捷搜索、多工具便捷提效、多模式随心畅享，你想要的，这里都有！

[前往安装](https://juejin.cn/extension/?utm_source=standalone&utm_medium=post&utm_campaign=extension_promotion)

相关小册

![「Vue.js 组件精讲」封面](media/「Vue.js_组件精讲」封面.webp)

VIP

Vue.js 组件精讲

[Aresn ![lv-5](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2fa3f08a7107485f81157b296fd9d41f~tplv-k3u1fbpfcp-no-mark:0:0:0:0.awebp "创作等级")](https://juejin.cn/user/149189280930478) 

6942购买

¥14.95

¥29.9

首单券后价

首单券后价

![「基于 Node 的 DevOps 实战」封面](media/「基于_Node_的_DevOps_实战」封面.webp)

VIP

基于 Node 的 DevOps 实战

[CookieBoty ![lv-5](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2fa3f08a7107485f81157b296fd9d41f~tplv-k3u1fbpfcp-no-mark:0:0:0:0.awebp "创作等级")

![](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/162b40efbd71af9a806dd2b54c4580ef.svg)](https://juejin.cn/user/2717648473821736) 

2195购买

¥19.95

¥39.9

首单券后价

首单券后价

评论

![](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/58aaf1326ac763d8a1054056f3b7f2ef.svg)

看完啦，

登录

分享一下感受吧～

相关推荐

-   [
    
    帅的成为负担
    
    ](https://juejin.cn/user/1319853267107704)
    
    1年前
    
    [前端](https://juejin.cn/tag/%E5%89%8D%E7%AB%AF)
    
    [Fetch二次封装](https://juejin.cn/post/6971279513099599886 "Fetch二次封装")
    

-   3986

-   11

-   -   1
    
-   [
    
    sanhuamao
    
    ](https://juejin.cn/user/1706045728898247)
    
    2年前
    
    [JavaScript](https://juejin.cn/tag/JavaScript)
    
    [Fetch API介绍及使用Fetch发送请求](https://juejin.cn/post/6868138631714848775 "Fetch API介绍及使用Fetch发送请求")
    
-   8876

-   13

-   -   评论
    
-   [
    
    XMJ
    
    ](https://juejin.cn/user/1372611399917357)
    
    1年前
    
    [前端](https://juejin.cn/tag/%E5%89%8D%E7%AB%AF)
    
    [为什么有了 XMLHttpRequest，还要设计一套 fetch API?](https://juejin.cn/post/6978132730718617637 "为什么有了 XMLHttpRequest，还要设计一套 fetch API?")
    
-   521

-   4

-   -   评论
    
-   [
    
    碎碎酱
    
    ](https://juejin.cn/user/817692379458967)
    
    4年前
    
    [GitHub](https://juejin.cn/tag/GitHub) [Ajax](https://juejin.cn/tag/Ajax) [axios](https://juejin.cn/tag/axios)
    
    [Cookie的设置、读取以及是否自动携带问题](https://juejin.cn/post/6844903648384778247 "Cookie的设置、读取以及是否自动携带问题")
    
-   1.7w

-   80

-   -   9
    
-   [
    
    SDK.cn
    
    ](https://juejin.cn/user/2875978145860263)
    
    7年前
    
    [API](https://juejin.cn/tag/API) [后端](https://juejin.cn/tag/%E5%90%8E%E7%AB%AF)
    
    [页面抓取神器 WrapAPI: 简单四步，把任何网站转换成参数化的JSON API](https://juejin.cn/post/6844903430008340487 "页面抓取神器 WrapAPI: 简单四步，把任何网站转换成参数化的JSON API")
    
-   2840

-   108

-   -   5
    
-   [
    
    闲D阿强
    
    ](https://juejin.cn/user/3835563370095262)
    
    1年前
    
    [React.js](https://juejin.cn/tag/React.js) [前端](https://juejin.cn/tag/%E5%89%8D%E7%AB%AF)
    
    [写一个简单的JSON Schema，为了应对后台接口细节的口口相传](https://juejin.cn/post/6976239698729517087 "写一个简单的JSON Schema，为了应对后台接口细节的口口相传")
    
-   1041

-   11

-   -   评论
    
-   [
    
    YimWu
    
    ](https://juejin.cn/user/729731453951550)
    
    9月前
    
    [前端](https://juejin.cn/tag/%E5%89%8D%E7%AB%AF) [面试](https://juejin.cn/tag/%E9%9D%A2%E8%AF%95)
    
    [面试官：说说 Cookie 和 Token 的区别？](https://juejin.cn/post/7111349594625146887 "面试官：说说 Cookie 和 Token 的区别？")
    
-   3.8w

-   394

-   -   128
    
-   [
    
    iFangcy_
    
    ](https://juejin.cn/user/4195392100501208)
    
    4年前
    
    [Ajax](https://juejin.cn/tag/Ajax)
    
    [认识 Fetch API](https://juejin.cn/post/6844903783491698701 "认识 Fetch API")
    
-   944

-   点赞

-   -   评论
    
-   [
    
    Tong
    
    ](https://juejin.cn/user/4037062426365053)
    
    2年前
    
    [JavaScript](https://juejin.cn/tag/JavaScript)
    
    [为什么有了 XMLHttpRequest，还要设计一套 fetch API?](https://juejin.cn/post/6847009771170562062 "为什么有了 XMLHttpRequest，还要设计一套 fetch API?")
    
-   3485

-   40

-   -   5
    
-   [
    
    阿离王
    
    ](https://juejin.cn/user/1072724804906152)
    
    1年前
    
    [JavaScript](https://juejin.cn/tag/JavaScript) [前端](https://juejin.cn/tag/%E5%89%8D%E7%AB%AF)
    
    [fetch拦截器和封装使用](https://juejin.cn/post/7021444877778944037 "fetch拦截器和封装使用")
    
-   4369

-   17

-   -   5
    
-   [
    
    编程三昧
    
    ](https://juejin.cn/user/2893570333750744)
    
    1年前
    
    [前端](https://juejin.cn/tag/%E5%89%8D%E7%AB%AF) [JavaScript](https://juejin.cn/tag/JavaScript)
    
    [有同学问我：Fetch 和 Ajax 有什么区别？](https://juejin.cn/post/6997784981816737800 "有同学问我：Fetch 和 Ajax 有什么区别？")
    
-   1.4w

-   276

-   -   60
    
-   [
    
    Ethan01
    
    ](https://juejin.cn/user/4441682706170525)
    
    1年前
    
    [前端](https://juejin.cn/tag/%E5%89%8D%E7%AB%AF) [后端](https://juejin.cn/tag/%E5%90%8E%E7%AB%AF)
    
    [面试题 -- 跨域请求如何携带cookie?](https://juejin.cn/post/7066420545327218725 "面试题 -- 跨域请求如何携带cookie?")
    
-   6.6w

-   877

-   -   100
    
-   [
    
    前端先锋
    
    ](https://juejin.cn/user/641770489918542)
    
    3年前
    
    [JavaScript](https://juejin.cn/tag/JavaScript)
    
    [如何取消 Fetch 请求](https://juejin.cn/post/6844904113130438663 "如何取消 Fetch  请求")
    
-   6610

-   30

-   -   评论
    
-   [
    
    zenghongtu
    
    ](https://juejin.cn/user/800100193676968)
    
    4年前
    
    [面试](https://juejin.cn/tag/%E9%9D%A2%E8%AF%95)
    
    [23行代码实现一个带并发数限制的fetch请求函数](https://juejin.cn/post/6844903796506624014 "23行代码实现一个带并发数限制的fetch请求函数")
    
-   1.2w

-   110

-   -   83
    
-   [
    
    随风而逝_风逝
    
    ](https://juejin.cn/user/4054654613718350)
    
    3年前
    
    [Node.js](https://juejin.cn/tag/Node.js) [前端](https://juejin.cn/tag/%E5%89%8D%E7%AB%AF)
    
    [一个开箱即用，功能完善的 Express 项目](https://juejin.cn/post/6844904022080651277 "一个开箱即用，功能完善的 Express 项目")
    
-   3.1w

-   608

-   -   77
    
-   [
    
    ximikang
    
    ](https://juejin.cn/user/4327294241087096)
    
    2年前
    
    [前端框架](https://juejin.cn/tag/%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6)
    
    [fetch请求中的跨域和携带Cookies问题](https://juejin.cn/post/6939170306799960100 "fetch请求中的跨域和携带Cookies问题")
    
-   3221

-   3

-   -   评论
    
-   [
    
    小弟调调™
    
    ](https://juejin.cn/user/8451821675006)
    
    6年前
    
    [JavaScript](https://juejin.cn/tag/JavaScript) [前端](https://juejin.cn/tag/%E5%89%8D%E7%AB%AF)
    
    [本地存储 cookie 的封装，提供简单的 API](https://juejin.cn/post/6844903444088619015 "本地存储 cookie 的封装，提供简单的 API")
    
-   1904

-   91

-   -   4
    
-   [
    
    前端敌法师
    
    ](https://juejin.cn/user/4406498337251463)
    
    2年前
    
    [前端](https://juejin.cn/tag/%E5%89%8D%E7%AB%AF)
    
    [fetch API新特性: Stremaing Request](https://juejin.cn/post/6871869373787668493 "fetch API新特性: Stremaing Request")
    
-   950

-   4

-   -   1
    
-   [
    
    猫克杯
    
    ](https://juejin.cn/user/1063982988803421)
    
    2年前
    
    [SwiftUI](https://juejin.cn/tag/SwiftUI)
    
    [[SwiftUI 100天] 在 SwiftUI 中动态过滤 @FetchRequest](https://juejin.cn/post/6844904147171409928 "[SwiftUI 100天] 在 SwiftUI 中动态过滤 @FetchRequest")
    
-   1072

-   2

-   -   4
    
-   [
    
    Pober_Wong
    
    ](https://juejin.cn/user/3051900006314984)
    
    7年前
    
    [JavaScript](https://juejin.cn/tag/JavaScript) [API](https://juejin.cn/tag/API) [前端](https://juejin.cn/tag/%E5%89%8D%E7%AB%AF)
    
    [【翻译】这个 API 很 “迷人”- 新的 Fetch API](https://juejin.cn/post/6844903427575644174 "【翻译】这个 API 很 “迷人”- 新的 Fetch API")
    
-   2509

-   106

-   -   2
    

友情链接：

-   [jsp大马密码](https://frontend.devrank.cn/traffic-aggregation/69555 "jsp大马密码")

[![](media/6a593328474afc5a282fee62e727b008~100x100.awebp.webp)

赤蓝紫 ![lv-3](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a3f5a3e7550645a08184e5c4247cc3d4~tplv-k3u1fbpfcp-no-mark:0:0:0:0.awebp "创作等级")

![](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/162b40efbd71af9a806dd2b54c4580ef.svg) 

公众号: 赤蓝紫



](https://juejin.cn/user/1416592459577320)

[

私信

](https://juejin.cn/notification/im?participantId=1416592459577320)

获得点赞  135

文章被阅读  23,368

-   [](https://juejin.cn/post/7218068880224550972?utm_source=DFK&utm_medium=banner&utm_campaign=gengwen202304)

[](https://juejin.cn/user/center/signin?from=item)

相关文章

[

浅谈 Fetch

258点赞

 · 

38评论



](https://juejin.cn/post/6844903518000644109 "浅谈 Fetch")[

傻傻分不清之 Cookie、Session、Token、JWT

3069点赞

 · 

185评论



](https://juejin.cn/post/6844904034181070861 "傻傻分不清之 Cookie、Session、Token、JWT")[

Fetch 入门

82点赞

 · 

11评论



](https://juejin.cn/post/6844903741057925128 "Fetch 入门")[

JavaScript冷门知识

3点赞

 · 

13评论



](https://juejin.cn/post/7087054474383998984 "JavaScript冷门知识")[

面试题 -- 跨域请求如何携带cookie?

877点赞

 · 

100评论



](https://juejin.cn/post/7066420545327218725 "面试题 -- 跨域请求如何携带cookie?")

目录

-   [Express+FetchAPI 简单实践Cookie](https://juejin.cn/post/7110088238991147022#heading-0 "Express+FetchAPI 简单实践Cookie")
    
    -   [Cookie 的构成](https://juejin.cn/post/7110088238991147022#heading-1 "Cookie 的构成")
        
    -   [Cookie 的简单实践](https://juejin.cn/post/7110088238991147022#heading-2 "Cookie 的简单实践")
        
        -   [解决方案1](https://juejin.cn/post/7110088238991147022#heading-3 "解决方案1")
            
        -   [解决方案2](https://juejin.cn/post/7110088238991147022#heading-4 "解决方案2")
            

