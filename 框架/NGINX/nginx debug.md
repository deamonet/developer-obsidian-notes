# 调试日志

​
要开启调试日志，需要在编译 Nignx 时增加如下配置：

./configure --with-debug ...

之后应该使用

指令设置调试级别：

error_log /path/to/log debug;

要验证 nginx 是否已经配置为支持调试功能，请运行 `nginx -V` 命令：

configure arguments: --with-debug ...

预构建

包为 nginx-debug 二进制文件的调试日志提供了开箱即用的支持，可以使用命令运行。

service nginx stop

service nginx-debug start

之后设置 `debug` 级别。Windows 的 nginx 在编译时就已经支持调试日志，因此只需设置 `debug` 级别即可。

请注意，重新定义日志而不指定 `debug` 级别将禁止调试日志。在下面的示例中，重新定义 `server` 上的日志级别，nginx 将不会在此服务器上做日志调试。

error_log /path/to/log debug;

​

http {

server {

error_log /path/to/log;

...

为了避免这种情况，重新定义日志级别应该被注释掉，或者明确指定日志为 `debug` 级别。

error_log /path/to/log debug;

​

http {

server {

error_log /path/to/log debug;

...

为指定客户端做调试日志[](https://docshome.gitbook.io/nginx-docs/readme/tiao-shi-ri-zhi#wei-zhi-ding-ke-hu-duan-zuo-tiao-shi-ri-zhi)

也可以仅为选定的客户端地址启用调试日志：

error_log /path/to/log;

​

events {

debug_connection 192.168.1.1;

debug_connection 192.168.10.0/24;

}

记录日志到循环内存缓冲区[](https://docshome.gitbook.io/nginx-docs/readme/tiao-shi-ri-zhi#ji-lu-ri-zhi-dao-xun-huan-nei-cun-huan-chong-qu)

调试日志可以被写入到循环内存缓冲区中：

error_log memory:32m debug;

在 `debug` 级别将日志写入到内存缓冲区中，即使在高负载情况下也不会对性能产生重大的影响。在这种情况下，可以使用如下 `gdb` 脚本来提取日志：

set $log = ngx_cycle->log

​

while $log->writer != ngx_log_memory_writer

set $log = $log->next

end

​

set $buf = (ngx_log_memory_buf_t *) $log->wdata

dump binary memory debug_log.txt $buf->start $buf->end

原文档[](https://docshome.gitbook.io/nginx-docs/readme/tiao-shi-ri-zhi#yuan-wen-dang)

​​http://nginx.org/en/docs/debugging_log.html