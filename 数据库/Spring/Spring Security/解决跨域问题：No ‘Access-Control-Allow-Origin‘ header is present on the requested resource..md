PS：如果遇到 这个问题 Request header field Content-Type is not allowed by Access-Control-Allow-Headers，解决方法见另一博文：解决：Request header field Content-Type is not allowed by Access-Control-Allow-Headers

 

1. 场景描述：

我前端是一个 vue 工程，写的是绝对 URL 请求后端工程接口，报错如题：

No 'Access-Control-Allow-Origin' header is present on the requested resource

2.解决方法，后端开放跨域：新增一个过滤器，设置头信息。

重点是这个设置：

response.setHeader("Access-Control-Allow-Origin", "*");

    package gentle.filter;
     
    import javax.servlet.*;
    import javax.servlet.annotation.WebFilter;
    import javax.servlet.http.HttpServletRequest;
    import javax.servlet.http.HttpServletResponse;
    import java.io.IOException;
     
    /**
     * 解决跨域设置
     * （可把此设置放在 nginx 中，但只能设置一处）
     *
     * @author silence
     * @date 2018/12/11 15:19
     */
     
    @WebFilter(filterName = "requestFilter", urlPatterns = {"/*"})
    public class RequestFilter implements Filter {
     
        @Override
        public void init(FilterConfig filterConfig) throws ServletException {
        }
     
        @Override
        public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
     
            HttpServletResponse response = (HttpServletResponse) servletResponse;
            HttpServletRequest request = (HttpServletRequest) servletRequest;
            
            // 此处 setHeader、addHeader 方法都可用。但 addHeader时写多个会报错：“...,but only one is allowed”
            response.setHeader("Access-Control-Allow-Origin", "*"); 
    //        response.addHeader("Access-Control-Allow-Origin", request.getHeader("origin"));
            // 解决预请求（发送2次请求），此问题也可在 nginx 中作相似设置解决。
            response.setHeader("Access-Control-Allow-Headers", "x-requested-with,Cache-Control,Pragma,Content-Type,Token, Content-Type");
            response.setHeader("Access-Control-Allow-Methods", "POST, GET, OPTIONS, DELETE");
            response.setHeader("Access-Control-Max-Age", "3600");
            response.setHeader("Access-Control-Allow-Credentials", "true");
            String method = request.getMethod();
            if (method.equalsIgnoreCase("OPTIONS")) {
                servletResponse.getOutputStream().write("Success".getBytes("utf-8"));
            } else {
                filterChain.doFilter(servletRequest, servletResponse);
            }
        }
     
        @Override
        public void destroy() {
     
        }
     
    }

 再次请求，可以了。

 

3. 前后端工程也有作反向代理。前端工程部署时使用浏览器默认端口：80 。后端工程端口为 8089 。nginx 监听端口 8082 。

前端请求后端 URL 为：http://  nginx所在服务器 IP : 8082 

前端工程请求 8082，nginx 收到请求再转发到实际服务，取得数据，并最终再返回。

（nginx 所在服务器也就是代理服务器，可以和后端服务器为同一主机）

在 nginx 配置文件中设置为：

端口占用情况如下：红框是 nginx 、黄框是前端工程、蓝框是后端工程。

PS： springboot 项目中过滤器使用方法见文章：Springboot 项目中过滤器的使用

 

---------------------------------------------------------------------

后记 ：解决报错见文章 ：springboot&ajax&has been blocked by CORS policy: No 'Access-Control-Allow-Origin

另报错：The 'Access-Control-Allow-Origin' header contains multiple values'x, *', but only one is allowed.的解决方式见文章：

解决：The 'Access-Control-Allow-Origin' header contains multiple values'x, *', but only one is allowed.

---------------------------------------------------------------------

补记：

2019.5.16

作了以上配置后出现情况：跨域问题时好时不好，最后在 nginx 代理中加了一个假性集群配置：

这样，请求后端成功。访问正常。此处的 ergouzi 只是 upstream 的名字。

事实上后端工程项目只部署在 8089 上，其实 8082 上什么也没有。

---------------------------------------------------------------------

补记：

2019.6.5

其实不用配置假性集群，之所以会出现上面时好时不好情况的原因仅是由于我队友当时的操作：

他也用我服务器，那段时间有时会重启他应用的服务，而当他重启之前会执行命令：

ps -ef | grep java | awk '{print $2}' | xargs kill -9

查出当前运行的所有java程序再一并 kill ，

这样我的服务也挂了，而我只注意到前端工程请求失败，并未去查看后端工程服务是否正常。

好吧 ，这个我也应该检讨自己太粗心 ...

 

参考：https://blog.csdn.net/qq_39403545/article/details/82116121 
文章知识点与官方知识档案匹配，可进一步学习相