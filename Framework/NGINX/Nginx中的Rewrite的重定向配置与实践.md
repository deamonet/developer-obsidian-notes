Nginx中的Rewrite的重定向配置与实践

 2019-05-01 00:03  龙恩0707  阅读(120410)  评论(0)  编辑  收藏  举报

阅读目录

    一：理解地址重写 与 地址转发的含义。
    二：理解 Rewrite指令 使用
    三：理解if指令
    四：理解防盗链及nginx配置

  简介：Rewrite是Nginx服务器提供的一个重要的功能，它可以实现URL重定向功能。
回到顶部

一：理解地址重写 与 地址转发的含义。

地址重写与地址转发是两个不同的概念。

地址重写 是为了实现地址的标准化，比如我们可以在地址栏中中输入 www.baidu.com. 我们也可以输入 www.baidu.cn. 最后都会被重写到 www.baidu.com 上。浏览器的地址栏也会显示www.baidu.com。

地址转发：它是指在网络数据传输过程中数据分组到达路由器或桥接器后，该设备通过检查分组地址并将数据转发到最近的局域网的过程。

因此地址重写和地址转发有以下不同点：

1. 地址重写会改变浏览器中的地址，使之变成重写成浏览器最新的地址。而地址转发他是不会改变浏览器的地址的。
2. 地址重写会产生两次请求，而地址转发只会有一次请求。
3. 地址转发一般发生在同一站点项目内部，而地址重写且不受限制。
4. 地址转发的速度比地址重定向快。
回到顶部

二：理解 Rewrite指令 使用

该指令是通过正则表达式的使用来改变URI。可以同时存在一个或多个指令。需要按照顺序依次对URL进行匹配和处理。

该指令可以在server块或location块中配置，其基本语法结构如下：

rewrite regex replacement [flag];

rewrite的含义：该指令是实现URL重写的指令。
regex的含义：用于匹配URI的正则表达式。
replacement：将regex正则匹配到的内容替换成 replacement。
flag: flag标记。

flag有如下值：

last: 本条规则匹配完成后，继续向下匹配新的location URI 规则。(不常用)
break: 本条规则匹配完成即终止，不再匹配后面的任何规则(不常用)。
redirect: 返回302临时重定向，浏览器地址会显示跳转新的URL地址。
permanent: 返回301永久重定向。浏览器地址会显示跳转新的URL地址。

比如如下列子：

rewrite ^/(.*) http://www.baidu.com/$1 permanent;

说明：
rewrite 为固定关键字，表示开始进行rewrite匹配规则。
regex 为 ^/(.*)。 这是一个正则表达式，匹配完整的域名和后面的路径地址。
replacement就是 http://www.baidu.com/$1 这块了，其中$1是取regex部分()里面的内容。如果匹配成功后跳转到的URL。
flag 就是 permanent，代表永久重定向的含义，即跳转到 http://www.baidu.com/$1 地址上。

下面我们来做个简单的demo来模拟下：

1. 在我们的测试项目下有个app.js. 代码如下：
复制代码

const Koa = require('koa');
const app = new Koa();

const router = require('koa-router')();

// 添加路由
router.get('/', ctx => {
  ctx.body = '<h1>欢迎光临index page 页面</h1>';
});

router.get('/home', ctx => {
  ctx.body = '<h1>欢迎光临home页面</h1>';
});

router.get('/404', ctx => {
  ctx.body = '<h1>404...</h1>'
});

// 加载路由中间件
app.use(router.routes());

app.listen(3001, () => {  
  console.log('server is running at http://localhost:3001');
});

复制代码

然后在命令行中 运行 node app.js 后，运行，我们就可以在浏览器中 访问 http://localhost:3001 就可以访问到我们对应的页面了。但是现在我想把该node项目
部署到我本地的nginx服务器上。nginx安装请看我这篇文章 然后我想使用域名来访问我们的项目，因此我们需要在我们的nginx.conf中配置一下：

cd /usr/local/etc/nginx

然后使用命令：sudo open /usr/local/etc/nginx/nginx.conf -a 'sublime text' 命令打开 nginx.conf 配置如下：
复制代码


```nginx
worker_processes  1;

events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile        on;
    #tcp_nopush     on;
    
    #keepalive_timeout  0;
    keepalive_timeout  65;
    
    #gzip  on;
    
    server {
      listen       8081;
      server_name  localhost;
      location / {
        root   html;
        index  index.html index.htm;  
      }
      error_page   500 502 503 504  /50x.html;
      location = /50x.html {
        root   html;
      }
    }
    server {
      listen 8088;
      server_name xxx.abc.com;
      location / {
        proxy_pass http://127.0.0.1:3001;
        rewrite ^/(.*) http://www.baidu.com permanent;
      }
    }
}
```

复制代码

如上代码，我监听端口号是8088，然后server_name 配置设置为 xxx.abc.com， 然后当我们访问 http://xxx.abc.com:8088/的时候，会先反向代理到我们的http://127.0.0.1:3001下的node对应的页面上来，反向代理完成后，会使用 rewrite 重定向百度页面去了。如上配置完成后，我们需要重启下nginx服务器；使用命令：

nginx -s reload

然后当我们在浏览器访问 http://xxx.abc.com:8088/ 的时候，会执行如下图所示，它会先对 http://xxx.abc.com:8088/ 进行永久重定向(301), 然后会访问百度(307),临时重定向到百度页面来，最终加载百度页面的地址；如下演示所示：

![Alt text](561794-20190430235454708-1591276065.png)

![Alt text](561794-20190430235501506-202996736.png)
    
![Alt text](561794-20190430235508060-673751730.png)

但是如果我把 permanent 改成 redirect 的话，比如nginx配置：rewrite ^/(.*) http://www.baidu.com redirect; 后，它就会变成302临时重定向了。如下所示：

![Alt text](561794-20190430235529531-149691559-1.png)


三：理解if指令

 该指令用来支持条件判断的，并且根据条件判断结果来选择不同的nginx的配置，我们可以在server块或location块中配置该指令，它的语法结构为：

if (condition) {
  // ....
}

condition 是布尔值 true/false的含义。

Rewrite 指令可用的全局变量如下：

1. $args: 该变量中存放了请求URL中的请求指令。比如 http://127.0.0.1:3001?arg1=value1&arg2=value2 中的
"arg1=value1&arg2=value2"。
2. $content_length: 该变量中存放了请求头中的Content-length字段。
3. $content_type: 该变量中存放了请求头中的 Content-type字段。
4. $document_root: 该变量中存放了针对当前请求的根路径。
5. $document_uri: 该变量中存放了请求的当前URI, 但是不包括请求指令。比如 http://xxx.abc.com/home/1?arg1=value1&
arg2=value2; 中的 "/home/1"
6. $host: 变量中存放了请求的URL中的主机部分字段，比如http://xxx.abc.com:8080/home中的 xxx.abc.com.
7. `$http_host: 该变量与$host`唯一区别带有端口号：比如上面的是 xxx.abc.com:8080
8. $http_user_agent: 变量中存放客户端的代理信息。
9. $http_cookie, 该变量中存放客户端的cookie信息。
10. $remote_addr 该变量中存放客户端的地址。
11. $remote_port 该变量中存放了客户端与服务器建立连接的端口号。
12. $remote_user 变量中存放客户端的用户名。
13. $request_body_file 变量中存放了发给后端服务器的本地文件资源的名称
14. $request_method 变量中存放了客户端的请求方式，比如 'GET'、'POST'等。
15. $request_filename 变量中存放了当前请求的资源文件的路径名。
16. $request_uri 变量中存放了当前请求的URI，并且带请求指令。
17. $query_string 和变量$args含义一样。
18. $scheme 变量中存放了客户端请求使用的协议，比如 'http', 'https'等。
19. $server_protocol 变量中存放了客户端请求协议的版本, 比如 'HTTP/1.0'、'HTTP/1.1' 等。
..... 等等

正则表达式的基本语法：

1. 对变量进行匹配

'~' 表示匹配过程中对大小写敏感。
'~*' 表示匹配过程中对大小写不敏感。
'!~' 如果 '~' 匹配失败时，那么该条件就为true。
'!~*' 如果 '~*' 匹配失败时，那么该条件就为true。

比如如下：

if ($http_user_agent ~ MSIE) {
  // 代码的含义：$http_user_agent值中是否含有 MSIE 字符串，如果包含为true，否则为false
}

2. 判断请求的文件是否存在

'-f' 如果请求的文件存在，那么该条件为true。
'!-f' 如果该文件的目录存在，该文件不存在，那么返回true。如果该文件和目录都不存在，则为false。
如果请求的目录不存在，请求的文件存在，也为false。
复制代码

```nginx
if (-f $request_filename) {
  // 判断请求的文件是否存在
}

if (!-f $request_filename) {
  // 判断请求的文件是否不存在
}
```

复制代码

3. 判断请求的目录是否存在使用 '-d' 和 '!-d'

使用 '-d'，如果请求的目录存在，则返回true。否则返回false。
使用 '!-d', 如果请求的目录不存在，但是该请求的上级目录存在，则返回true。如果该上级目录不存在，则返回false.... 等等其他一些语法，不多介绍。

现在我们使用if指令来对nginx加一些判断；比如说我们访问http://xxx.abc.com:8080/home时候，如果$host = 'xxx.abc.com' 的时候，就做重定向跳转，nginx配置代码如下：
复制代码

```nginx
server {
  listen 8088;
  server_name xxx.abc.com;
  location / {
    proxy_pass http://127.0.0.1:3001;
    if ($host = 'xxx.abc.com') {
      rewrite ^/(.*) http://www.cnblogs.com redirect;
    }
  }
}
```

复制代码

nginx 如上配置，如果我们访问 http://xxx.abc.com:8088 的时候，它就会重定向到 http://www.cnblogs.com 来了。
![Alt text](561794-20190430235855016-1969076509.png)

四：理解防盗链及nginx配置

什么是防盗链？盗链可以理解盗图链接，也就是说把别人的图片偷过来用在自己的服务器上，那么防盗链可以理解为防止其他人把我的图片盗取过去。

防盗链的实现原理：客户端向服务器端请求资源时，为了减少网络带宽，提高响应时间，服务器一般不会一次将所有资源完整地传回客户端。比如请求一个网页时，首先会传回该网页的文本内容，当客户端浏览器在解析文本的过程中发现有图片存在时，会再次向服务器发起对该图片资源的请求，服务器将存储的图片资源再发送给客户端。但是如果这个图片是链接到其他站点的服务器上去了呢，比如在我项目中，我引用了的是淘宝中的一张图片的话，那么当我们网站重新加载的时候，就会请求淘宝的服务器，那么这就很有可能造成淘宝服务器负担。因此这个就是盗链行为。因此我们要实现防盗链。

实现防盗链：使用http协议中请求头部的Referer头域来判断当前访问的网页或文件的源地址。通过该头域的值，我们可以检测访问目标资源的源地址。如果目标源地址不是我们自己站内的URL的话，那么这种情况下，我们采取阻止措施，实现防盗链。但是注意的是：Referer头域中的值是可以被更改的。因此该方法也不能完全安全阻止防盗链。

使用Nginx服务器的Rewrite功能实现防盗链。

Nginx中有一个指令 valid_referers. 该指令可以用来获取 Referer 头域中的值，并且根据该值的情况给 Nginx全局变量 $invalid_referer 赋值。如果Referer头域中没有符合 valid_referers指令的值的话，$invalid_referer变量将会赋值为1. valid_referers 指令基本语法如下：

`valid_referers  none | blocked | server_names | string`

none: 检测Referer头域不存在的情况。
blocked： 检测Referer头域的值被防火墙或者代理服务器删除或伪装的情况。那么在这种情况下，该头域的值不以"http://" 或 "https://" 开头。

server_names: 设置一个或多个URL，检测Referer头域的值是否是URL中的某个。

因此我们有了 valid_referers指令和$invalid_referer变量的话，我们就可以通过 Rewrite功能来实现防盗链。
下面我们介绍两种方案：第一：根据请求资源的类型。第二：根据请求目录。

1. 根据请求文件类型实现防盗链配置实列如下：
复制代码

```nginx
server {
  listen 8080;
  server_name xxx.abc.com
  location ~* ^.+\.(gif|jpg|png|swf|flv|rar|zip)$ {
    valid_referers none blocked www.xxx.com www.yyy.com *.baidu.com  *.tabobao.com;
    if ($invalid_referer) {
      rewrite ^/ http://www.xxx.com/images/forbidden.png;
    }
  }
}
```

复制代码

如上基本配置，当有网络连接对以 gif、jpg、png为后缀的图片资源时候、当有以swf、flv为后缀的媒体资源时、或以 rar、zip为后缀的压缩资源发起请求时，如果检测到Referer头域中没有符合 valid_referers指令的话，那么说明不是本站的资源请求。

`location ~* ^.+\.(gif|jpg|png|swf|flv|rar|zip)$` 该配置的含义是 设置防盗链的文件类型。

`valid_referers none blocked www.xxx.com www.yyy.com *.baidu.com *.tabobao.com; `可以理解为白名单，允许文件链出的域名白名单，如果请求的资源文件不是以这些域名开头的话，就说明请求的资源文件不是该域下的请求，因此可以判断它是盗链。因此如果不是该域下的请求，就会使用 Rewrite进行重定向到 http://www.xxx.com/images/forbidden.png 这个图片，比如这张图片是一个x或其他的标识，然后其他的网站就访问不了你这个图片哦。

2. 根据请求目录实现防盗链的配置实列如下：
复制代码

```nginx
server {
  listen 8080;
  server_name xxx.abc.com
  location /file/ {
    root /server/file/;
    valid_referers none blocked www.xxx.com www.yyy.com *.baidu.com  *.tabobao.com;
    if ($invalid_referer) {
      rewrite ^/ http://www.xxx.com/images/forbidden.png;
    }
  }
}
```

复制代码

