# 常用功能

1. 前端服务器（性能强大）
2. 虚拟服务器（同一主机可作为多个网站服务器，充分利用IP等资源）
3. 反向代理（隐藏后端服务地址）
4. 负载均衡（upstream）

# 安装

1. 下载源码编译安装make install
2. YUM REPO (CENTOS) 需要额外添加软件源
3. APT INSTALL (UBUNTU)

 用软件源的安装方式好处就是不需要自己配置环境变量

# 启动

两种启动方式

1. 直接执行nginx可执行二进制文件 ./nginx，不过这里要注意当前用户的权限是否足够。
2. 利用系统的启动守护进程，这也分成两种init.d和systemd。可以用前者存放启动nginx脚本，可以用service命令管理脚本，后者也可以撰写对应systemd unit配置实现开机自启动nginx进程，也包括使用systemctl管理nginx unit。如果要用的话，更推荐后者。后者属于新一代linux系统守护进程管理器。

其他常用的nginx参数：

1. 不加参数 直接启动
2. -t 检查配置
3. -s stop 停止进程
4. -s reload 加载更新后的配置
5. -s reopen 重启

# 配置

一般情况，配置文件路径是 /etc/nginx/nginx.conf。也可以用include指定额外的配置文件，通常每个server都可以写成一个单独的配置文件。此外需要删掉nginx默认的配置文件才能使自定义的配置文件生效。

风格：

1. 层次分明，各级配置用大括号括起来
2. 设置项与取值之间用空格隔开
3. 用on/off，而不是yes/no和true/false
4. 每一行配置后必须加分号
5. 可以用 nginx -t 命令检查配置是否合规。

组成：全局、EVENTS、HTTP、SERVER、LOCATION。每一个部分可能存在通用的参数名，不过当出现在不同的配置块时，作用和作用域都不同。

## 全局

| 参数             | 含义           | 值                            | 解释                                                         |
| ---------------- | -------------- | ----------------------------- | :----------------------------------------------------------- |
| user             | 用户权限       | 默认是www-data                | NGINX master进程由当前用户执行，worker进程有这里指定的用户执行，设置不好，可能存在权限问题，造成资源不能访问。 |
| worker_processes | worker进程数量 | 可以写auto，通常写CPU核心数。 | Nginx能够做到如此高性能的特点就是靠异步多进程，避免了频繁的CPU上下文切换；而比多线程又减少了维护这么多线程所消耗的资源。 |

## EVENTS

## HTTP

## SERVER

### server_name

虚拟主机域名（或者IP）（不同的域名经过DNS解析到相同的IP地址，经由NGINX处理）

支持正则表达式和星号通配符，支持多个用空格分隔的域名

用_ __ @等等无效的域名可以用来匹配任意域名

不写server_name视作虚拟主机域名是空字符串

### listen

监听IP和端口。一个server可以用多个listen参数，监听不同的端口，包括ssl加密。端口前加上IP可以监听指定IP地址的请求。

同时listen后还可以添加default_server参数，用来接受不与其他server匹配的请求。如果不加该参数，配置文件中的第一个server会被当成默认。

```nginx
server {
    listen 80 default_server;
    server_name _;
    return 403; # 403 forbidden
}
```

例如

```nginx
server {
    listen 80 default_server;
    listen 443 ssl default_server;
    server_name _;
    return 301 $scheme://www.domain.com;
}
```

### upstream

转发后端服务，负载均衡，例如：

```nginx
upstream clusterApi {
    ip_hash # 一个IP每次都链接同一个server，如果不用ip hash，默认是轮询的方式
    server solax84:8031 max_fails=3 fail_timeout=30s;  # 30秒内如果3次都失败，则下一个30秒不再连接该服务器 
    server solax52:8031 max_fails=3 fail_timeout=30s weight=3; # weight更大，表示更高权重。
    server solax82:8031 max_fails=3 fail_timeout=30s backup; #备用
}
```

### rewrite

直接修改URI，支持正则表达式，支持临时重定向和永久重定向。修改URI之后，会根据修改之后的URI继续LOCATION的匹配。

语法： rewrite regex replacement [flag];

其中flag可选 last  break redirect permanent

break会停止location匹配，last会继续location匹配。**由于location当中也可以写rewrite**，所以会继续这一循环。但不能超过十次。这两个不会改变用户浏览器中的地址。

两个重定向的返回HTTP_CODE不同（redirect 302 permanent 301），浏览器表现有些许不同（浏览器会记住永久重定向，第二次访问不经过NGINX的重定向）

### 其他与location块共用的参数

- root表示当前虚拟主机所有location文件路径的共用路径前缀
- proxy_pass可以直接转发一整个虚拟主机的请求

## LOCATION

Location表示用户请求的URL最终会指向本地文件 。Location分成五种路径匹配规则：空格、=、^~、~、~*。

~、~*是基于正则表达式匹配规则。前者区分大小写，后者不区分。

### space

```nginx
location /a {
    root /xxx/xxx;
}
```

普通前缀匹配，当其他的匹配不满足时会匹配这一个。

```nginx
location / {
    root /xxx/xxx;
    index index.html index.htm;
}
```

而如果不加前缀，就是优先级最低的默认匹配。

### =

```nginx
location = /error {
   root /xxx/xxx;
}
```

严格匹配，只有URL路径完全等于/error时，才会匹配到该规则。优先级最高。

### ^~

```nginx
location ^~ {
    root /xxx/xxx;
}
```

最长前缀匹配，第二优先级。当存在多个^~的location可以与URI匹配时，选择其中最长的那个，与配置文件中的先后顺序无关。

### ~

区分大小写的正则匹配。第三优先级，如果存在多个都可以匹配的正则location，选择在配置文件中靠前的那个

### ~*

不区分大小写的正则匹配。第三优先级，如果存在多个都可以匹配的正则location，选择在配置文件中靠前的那个

### index

配置index.html的路径

### root 和 alias

都用来指向实际文件的路径，区别在于：root后添加uri就是文件路径；alias则是替换文件的路径。

针对这样一个uri:   /static/test.jpg

```nginx
location /static/ {
    root /var/www/app;
    autoindex off;
}
```

以上配置得到实际文件路径是：/var/www/app/static/test.jpg

```nginx
location /static/ {
    alias /var/www/app/static/;
    autoindex off;
}
```

以上配置得到实际文件路径是：/var/www/app/static/test.jpg

### proxy_pass

转发给后端服务，也即反向代理。可以写真实的后端服务域名，也可以写upstream的名字。

location跟proxy_pass最后是否加上斜线，会造成很不一样的结果。

简单来说，如果proxy_pass最后加上斜线，server会替换URI中与location匹配的部分；如果不加server直接拼接请求的URI。类似alias和root。

### 反向代理的请求头配置

**注意以下指令(directive)都可以定义在http`, `server`, `location这三个配置段中。**

#### proxy_pass_header

控制 Nginx 代理服务器向客户端发送的响应头(Permits passing otherwise disabled header fields from a proxied server to a client. )
这个指令可以接受一个参数，表示要发送的响应头的名称。例如，如果你想让 Nginx 代理服务器向客户端发送服务器的名称，可以使用这个指令：

```nginx
proxy_pass_header server;  
```

#### proxy_set_header

NGINX发起的HTTP请求不会主动转发用户的HTTP请求HEADER中的数据，所以需要手动设置HTTP HEADER各项参数。

```nginx
proxy_set_header Host $host
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Real-PORT $remote_port;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
```

### 其他协议的反向代理

如果后端服务采用的不是HTTP协议，例如FastCGI和Python Django所使用的uwsgi协议，也可以分别用fastcgi_pass和uwsgi_pass。甚至包括gRPC协议的grpc_pass。

### return

用来返回HTTP状态代码以及指定的application/type或者重定向URL，例如简单的文本/JSON等。需要配合default_type参数使用。典型配置如下：

```nginx
return (1xx | 2xx | 4xx | 5xx) ["text"];
```

```nginx
location ~ /xxx{
    default_type text/plain;
    return 200 '/xxx';
}
```

例如强制加密

```nginx
server {
    listen 80;
    server_name www.domain.com;
    return 301 https://www.domain.com$request_uri;
}
```

# 日志

分成两个：访问日志和错误日志(access_log & error_log)

通常情况下，默认配置文件已经开启。配置是：

```nginx
access_log /var/log/nginx/access.log;
error_log /var/log/nginx/error.log;
```

把路径换成off即可关闭日志，路径参数设置为linux下的空设备/dev/null一样可以关闭日志。

## 访问日志格式

访问日志的实质是一张表格，其默认的字段分别是

```nginx
$remote_addr $remote_user [$time_local] $request $status $body_bytes_sent $http_referer $http_user_agent
```

这种格式与combined参数等价，combined参数可以接在日志配置的后面，如下：

```nginx
access_log /var/log/nginx/access.log combined;
```

用log_format参数可以自定义日志的格式，并当作参数替换上面的combined，比如添加更多的nginx全局变量。例如：

```nginx
log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                  '$status $body_bytes_sent "$http_referer" '
                  '"$http_user_agent" "$http_x_forwarded_for"';
```

以下是一个combined格式的一行访问记录。注意分隔符是短横（ hyphen）。

172.31.128.1 - - [29/Jun/2023:18:51:54 +0800] "GET /a/b HTTP/1.1" 200 4 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:109.0) Gecko/20100101 Firefox/114.0"

## 日志记录规则

例如指定某些IP的访问不记录，或者指定某些类型的错误不记录，有助于节省磁盘空间。

# 正则表达式

举例给出常用的正则表达式

- /.*(jpg|png|css|js) 匹配各种静态文件，括号加竖线表示任选其一
- /(?<name>.+)\.html 匹配任意html文件，并且可以取出文件名作为变量name
- ^/page(.+)/$ 匹配/开头和/结尾（URI规范中以斜线结尾通常表示目录）

# 参考资料

1.Module ngx_http_log_module. https://nginx.org/en/docs/http/ngx_http_log_module.html.

2.Module ngx_http_proxy_module. https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_pass.

3.Module ngx_http_proxy_module. https://nginx.org/en/docs/http/ngx_http_proxy_module.html.

4.Module ngx_http_rewrite_module. http://nginx.org/en/docs/http/ngx_http_rewrite_module.html.

5.Nginx -- static file serving confusion with root & alias - Stack Overflow. https://stackoverflow.com/questions/10631933/nginx-static-file-serving-confusion-with-root-alias.

6.Nginx location 和 proxy_pass路径配置详解 - 自由早晚乱余生 - 博客园. [https://www.cnblogs.com/operationhome/p/15212801.html#%E4%B8%80nginx-location-%E5%9F%BA%E6%9C%AC%E9%85%8D%E7%BD%AE](https://www.cnblogs.com/operationhome/p/15212801.html#一nginx-location-基本配置).

7.proxy_pass_header server-掘金. [https://juejin.cn/s/proxy_pass_header%20server](https://juejin.cn/s/proxy_pass_header server).

8.Server names. http://nginx.org/en/docs/http/server_names.html.

9.Server names. http://nginx.org/en/docs/http/server_names.html#regex_names.