2022-09-05526阅读3分钟

# 一.关于RestTemplate的三种创建方式

最早接触RestTemplate是学到微服务间服务的通信使用http的rest风格在应用层进行通信，不过视频讲的很少，我就决定自己做一个总结，不过由于没有接触过restTemplate请求中发送cookie或者jwt等，所以业务方面可能写的简单点. RestTemplate的主要三种配置方式有

- 使用RestTemplateBuilder来配置超时信息
- 使用RestTemplateCustomizer来自定义化配置
- 使用RestTemplateCustomizer和RestTemplateBuilder结合配置

### 1.使用RestTemplateBuilder来进行配置

下图是springApi 2.7.3关于RestTemplateBuilder的解释

![图片.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ddd4887d3db2462689798df0e82a5fd2~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?) 大致意思：**restTemplateBuilder是一个生成restTemplate的类，可以很方便的注册errorHandler，MessageConverts，URL模板，超时时间的设置等，同时这个类会被自动注册为bean，你可以在需要的地方直接注入。**

下面是一个简单的使用：

java

代码解读

复制代码

`@Configuration public class BaseConfiguration {     @Autowired     LoadBalancerClient loadBalancerClient;     @Bean     public RestTemplate restTemplate(RestTemplateBuilder restTemplateBuilder){         return restTemplateBuilder                 .additionalInterceptors(new LoadBalancerInterceptor(loadBalancerClient))                 .setConnectTimeout(Duration.ofSeconds(5))                 .build();     }`

### 2.使用RestTemplateCustomizer来自定义

Customizer的意思本身就是定制器，我们查看springAPi的解释：

![图片.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/34805a20040e4f3094506424ccfbc30d~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

> 值得一提的是，RestTemplatecustomizer分别在两个包中都有定义，分别是： 1.org.springframework.boot.web.client.RestTemplateCustomizer; 2.org.springframework.cloud.client.loadbalancer.RestTemplateCustomizer 按照boot api的定义，我认为是通过RestTemplateCustomizer来定义的，两个接口的内部方法差不多，第二个的官方文档没有找到，决定使用第一个

官方api的意思翻译过来就是：通过实现这个接口的customize(RestTemplate restTemplate)这个回调函数，可以给RestTemplate进行定制。 下面是简单的使用： 一个简单的拦截器：

java

代码解读

复制代码

`// 自定义一个简单的拦截器 package com.interceptor; import org.slf4j.Logger; import org.slf4j.LoggerFactory; import org.springframework.http.HttpRequest; import org.springframework.http.client.ClientHttpRequestExecution; import org.springframework.http.client.ClientHttpRequestInterceptor; import org.springframework.http.client.ClientHttpResponse; import java.io.IOException; public class CustomRestTemplateCustomizerInterceptor implements ClientHttpRequestInterceptor {     private static final Logger logger = LoggerFactory.getLogger(CustomRestTemplateCustomizerInterceptor.class);     @Override     public ClientHttpResponse intercept(HttpRequest request, byte[] body, ClientHttpRequestExecution execution) throws IOException {         logDetails(request);         return execution.execute(request, body);     }     public void logDetails(HttpRequest request){         logger.info("url:{}",request.getURI());         logger.info("method:{}",request.getMethod());         logger.info("header:{}",request.getHeaders());     } }`

回调函数中添加进去这个拦截器

java

代码解读

复制代码

`package com.bean; import com.interceptor.CustomRestTemplateCustomizerInterceptor; import org.springframework.boot.web.client.RestTemplateCustomizer; import org.springframework.web.client.RestTemplate; public class CustomRestTemplateCustomizer implements RestTemplateCustomizer {     @Override     public void customize(RestTemplate restTemplate) {         // 自定制interceptor         restTemplate.getInterceptors().add(new CustomRestTemplateCustomizerInterceptor());     } }`

将我们写的定制器注册为bean

typescript

代码解读

复制代码

`@Bean public CustomRestTemplateCustomizer customRestTemplateCustomizer(){     return new CustomRestTemplateCustomizer(); }`

运行试试:

![图片.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dc4f0aed205140f082f4d8d237c17428~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?) 可以看到我们的通过定制器自定义的拦截器起了作用。

### 3.使用RestTemplateCustomizer和RestTemplateBuilder结合配置

这是一个破坏性最大，也是定制性最高的方式,至于破坏性，我们在后面说，先来说一说这种的如何配置

less

代码解读

复制代码

`@Bean @DependsOn({"customRestTemplateCustomizer"}) public RestTemplateBuilder restTemplateBuilder(CustomRestTemplateCustomizer customRestTemplateCustomizer){     RestTemplateBuilder restTemplateBuilder = new RestTemplateBuilder(customRestTemplateCustomizer);     System.out.println("添加了定制器的"+restTemplateBuilder);     return restTemplateBuilder; }`

这种配置是将定制器作为参数传入到RestTemplateCustomizer的构造方法中去。

**疑问1：我们构造为bean的RestTemplateCustomizer会不会和自动构造的（前文有提到）RestTemplateCustomizer产生冲突还是覆盖？** 实验1: 代码：

typescript

代码解读

复制代码

`@Configuration public class BaseConfiguration {     @Autowired     LoadBalancerClient loadBalancerClient;     @Bean     public RestTemplate restTemplate(RestTemplateBuilder restTemplateBuilder){         // mark1         System.out.println("这是第一个方法里面的"+restTemplateBuilder);         return restTemplateBuilder                 .additionalInterceptors(new LoadBalancerInterceptor(loadBalancerClient))                 .setConnectTimeout(Duration.ofSeconds(5))                 .build();     }     @Bean     public CustomRestTemplateCustomizer customRestTemplateCustomizer(){         return new CustomRestTemplateCustomizer();     }     @Bean     @DependsOn({"customRestTemplateCustomizer"})     public RestTemplateBuilder restTemplateBuilder(CustomRestTemplateCustomizer customRestTemplateCustomizer){         RestTemplateBuilder restTemplateBuilder = new RestTemplateBuilder(customRestTemplateCustomizer);         // mark2         System.out.println("添加了定制器的"+restTemplateBuilder);         return restTemplateBuilder;     } }`

我们看到写了两个mark标记，查看这两个是不是同一个RestTemplateBuilder。 运行查看输出：

![图片.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/98f59774a85c4822886a614aeef726a2~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?) 我们可以看到这里我们自己构造的RestTemplateBuilder完全覆盖了框架提供的RestTemplateBuilder。

**疑问2:为什么说这种方式的破坏性极大** 实验2：我们首先不定制，打断点之后查看框架提供的RestTemplateBuilder有哪些属性：

![图片.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b456c5d8cc744a7cb35426e820e29ab1~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

![图片.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/37e58e02d2cb4b849235cdd8d028f8f4~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?) 这是不自定义得到的框架提供的RestTemplateBuilder，我们可以看到有许多的消息转换器

我们通过刚刚的代码定制覆盖之后查看缺少了那些属性：

![图片.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7172ae254b414efc82587a5da0f9dc6b~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?) 我们可以看到缺少了消息转换器以及定制器的数量也发生了变化。

以上就是三种不同的生成RestTemplate的方式。

RestTemplate的原理以及代码解析：暂时写作中