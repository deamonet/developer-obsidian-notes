# Nginx的日志配置参数详解

[Java软件高级开发工程师学徒](https://blog.csdn.net/pishanzhong7963 "Java软件高级开发工程师学徒")  于 2018-11-21 10:10:38 发布


文章标签： [NGINX](https://so.csdn.net/so/search/s.do?q=NGINX&t=all&o=vip&s=&l=&f=&viparticle=) [日志](https://so.csdn.net/so/search/s.do?q=%E6%97%A5%E5%BF%97&t=all&o=vip&s=&l=&f=&viparticle=)

**摘要** ：[Nginx](https://so.csdn.net/so/search?q=Nginx&spm=1001.2101.3001.7020)日志主要分为两种：访问日志和错误日志。日志开关在Nginx配置文件（/etc/nginx/nginx.conf）中设置，两种日志都可以选择性关闭，默认都是打开的。

　　Nginx日志主要分为两种：访问日志和错误日志。日志开关在Nginx配置文件（`/path/nginx/config/nginx.conf`）中设置，**两种日志都可以选择性关闭，默认都是打开的。**

> 关闭访问日志指令 `access_log off;`  
> >关闭错误日志指令 `error_log /dev/null;`

[TOC]

### 访问日志 access_log

　　访问日志主要记录客户端访问Nginx的每一个请求，格式可以自定义。通过访问日志，你可以得到用户地域来源、跳转来源、使用终端、某个URL访问量等相关信息。Nginx中访问日志相关指令主要有两条

#### log_format

`log_format`用来设置日志格式，也就是日志文件中每条日志的格式，具体如下：

`log_format name(格式名称) type(格式样式)`

```dart
log_format  main  '$server_name $remote_addr - $remote_user [$time_local] "$request"'                  '$status $uptream_status $body_bytes_sent "$http_referer"'                  '"$http_user_agent" "$http_x_forwarded_for" '                  '$ssl_protocol $ssl_cipher $upstream_addr $request_time $upstream_response_time';
```

> - `$server_name`：虚拟主机名称。
> - `$remote_addr`：远程客户端的IP地址，请求者IP。
> - -：空白，用一个“-”占位符替代，历史原因导致还存在。
> - `$remote_user`：远程客户端用户名称，用于记录浏览者进行身份验证时提供的名字，如登录百度的用户名scq2099yt，如果没有登录就是空白。
> - `[$time_local]`：访问的时间与时区，比如07/Jun/2016:08:54:27 +0800，时间信息最后的"+0800"表示服务器所处时区位于UTC之后的8小时。
> - `$request`：请求的URI和HTTP协议，这是整个PV日志记录中最有用的信息，记录服务器收到一个什么样的请求
> - `$status`：记录请求返回的http状态码，比如成功是200。
> - `$uptream_status`：upstream状态，比如成功是200.
> - `$body_bytes_sent`：发送给客户端的文件主体内容的大小，比如899，可以将日志每条记录中的这个值累加起来以粗略估计服务器吞吐量。
> - `$http_referer`：记录从哪个页面链接访问过来的。
> - `$http_user_agent`：客户端浏览器信息
> - `$http_x_forwarded_for`：客户端的真实ip，通常web服务器放在反向代理的后面，这样就不能获取到客户的IP地址了，通过$remote_add拿到的IP地址是反向代理服务器的iP地址。反向代理服务器在转发请求的http头信息中，可以增加x_forwarded_for信息，用以记录原有客户端的IP地址和原来客户端的请求的服务器地址。
> - `$ssl_protocol`：SSL协议版本，比如TLSv1。
> - `$ssl_cipher`：交换数据中的算法，比如RC4-SHA。
> - `$upstream_addr`：upstream的地址，即真正提供服务的主机地址。
> - `$request_time`：整个请求的总时间。
> - `$upstream_response_time`：请求过程中，upstream的响应时间。

访问日志中一个典型的记录如下：  
`192.168.2.36 - - [07/Jun/2016:08:54:27 +0800] "GET /1.jpg HTTP/1.1" 200 20146 "http://a.com/" "Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/50.0.2661.87 Safari/537.36"`

#### access_log

`access_log` 指令用来指定日志文件的存放路径（包含日志文件名）、格式和缓存大小。具体如下：

`access_log path(存放路径) [format(自定义日志格式名称) [buffer=size | off]]`

举例说明如下：

```cobol
access_log  logs/access.log  main;
```

如果想关闭日志，可以如下：

```cobol
access_log off;
```

> 能够使用`access_log`指令的字段包括：`http` `server`和`location`。

Tips：如果需要在`access_log`中记录post请求的参数，可以参考[这里](http://www.54chen.com/web-ral/nginx-access-log-post.html)。

#### 开启访问日志

1. 在Nginx的主配置文件`nignx.conf`的http段内定义好`log_format`，比如：
    
    ```dart
    log_format  luo  '$server_name $remote_addr - $remote_user [$time_local] "$request"'               '$status $uptream_status $body_bytes_sent "$http_referer"'               '"$http_user_agent" "$http_x_forwarded_for" '               '$ssl_protocol $ssl_cipher $upstream_addr $request_time $upstream_response_time';
    ```
    
2. 在配置文件中`http` `server`或者`location`段中开启记录。
    
    ```cobol
    access_log  logs/access.log  luo;
    ```
    

### 错误日志

　　错误日志主要记录客户端访问Nginx出错时的日志，**格式不支持自定义**。通过错误日志，我们可以得到系统某个服务或server的性能瓶颈等。因此，将日志好好利用，你可以得到很多有价值的信息。

#### error_log

错误日志由指令`error_log`来指定，具体格式如下  
`error_log path(存放路径) level(日志等级)`

path含义同access_log，level表示日志等级，具体如下：[ debug | info | notice | warn | error | crit ]

从左至右，日志详细程度逐级递减，即debug最详细，crit最少。

举例说明如下：

```cobol
error_log  logs/error.log  info;
```

> 需要注意的是：`error_log off`并不能关闭错误日志，而是会将错误日志记录到一个文件名为off的文件中。

正确的关闭错误日志记录功能的方法如下：

```cobol
error_log /dev/null;
```

上面表示将存储日志的路径设置为"空设备"