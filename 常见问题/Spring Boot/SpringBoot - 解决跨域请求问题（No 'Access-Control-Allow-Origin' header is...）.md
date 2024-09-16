| 发布：hangge 阅读：19298                                                                                                                                                 |  |
|--------------------------------------------------------------------------------------------------------------------------------------------------------------------|--|
|     在开发前后端分离的项目时，常常会碰到跨域请求的问题。即由于浏览器的安全性限制，不允许 AJAX 访问协议不同、域名不同、端口号不同的数据接口，否则会出报 No 'Access-Control-Allow-Origin' header is present on the requested resource. 错误。 |
| 原文:SpringBoot - 解决跨域请求问题（No 'Access-Control-Allow-Origin' header is...）                                                                                            |
|                                                                                                                                                                    |
|     Spring Boot 支持通过设置 CORS（跨源资源共享）来解决跨域请求问题。具体如下两个地方可以进行配置，我们选择一种即可。                                                                                              |
|                                                                                                                                                                    |
| 1，在请求方法上配置                                                                                                                                                         |
| （1）我们可以直接在相应的请求方法上添加 @CrossOrigin 注解，那么该方法则支持跨域。                                                                                                                   |
| 1                                                                                                                                                                  |
| 2                                                                                                                                                                  |
| 3                                                                                                                                                                  |
| 4                                                                                                                                                                  |
| 5                                                                                                                                                                  |
| 6                                                                                                                                                                  |
| 7                                                                                                                                                                  |
| 8                                                                                                                                                                  |
| 9                                                                                                                                                                  |
| 10                                                                                                                                                                 |
| 11                                                                                                                                                                 |
|                                                                                                                                                                    |  |
| @RestController                                                                                                                                                    |
| public class WebAPIController {                                                                                                                                    |
|     @Autowired                                                                                                                                                     |
|     DeviceDataManager deviceDataManager;                                                                                                                           |
|                                                                                                                                                                    |
|     @GetMapping("/getDeviceDatas")                                                                                                                                 |
|     @CrossOrigin                                                                                                                                                   |
|     public List<DeviceData> getDeviceDatas() {                                                                                                                     |
|         return deviceDataManager.getDatas();                                                                                                                       |
|     }                                                                                                                                                              |
| }                                                                                                                                                                  |
|                                                                                                                                                                    |
| （2）我们也可以在控制器上添加 @CrossOrigin 注解，那么该控制器下的所有方法都支持跨域。                                                                                                                 |
| 1                                                                                                                                                                  |
| 2                                                                                                                                                                  |
| 3                                                                                                                                                                  |
| 4                                                                                                                                                                  |
| 5                                                                                                                                                                  |
| 6                                                                                                                                                                  |
| 7                                                                                                                                                                  |
| 8                                                                                                                                                                  |
| 9                                                                                                                                                                  |
| 10                                                                                                                                                                 |
| 11                                                                                                                                                                 |
|                                                                                                                                                                    |  |
| @RestController                                                                                                                                                    |
| @CrossOrigin                                                                                                                                                       |
| public class WebAPIController {                                                                                                                                    |
|     @Autowired                                                                                                                                                     |
|     DeviceDataManager deviceDataManager;                                                                                                                           |
|                                                                                                                                                                    |
|     @GetMapping("/getDeviceDatas")                                                                                                                                 |
|     public List<DeviceData> getDeviceDatas() {                                                                                                                     |
|         return deviceDataManager.getDatas();                                                                                                                       |
|     }                                                                                                                                                              |
| }                                                                                                                                                                  |
|                                                                                                                                                                    |
| （3）@CrossOrigin 注解还支持更加丰富的参数配置：                                                                                                                                    |
|                                                                                                                                                                    |
|     value：表示支持的域。这里表示来自 http://localhost:8081 域的请求是支持跨域的。默认为 *，表示所有域都可以。                                                                                           |
|     maxAge：表示探测请求的有效期（先进性判断是否有效）。探测请求不用每次都发送，可以配置一个有效期，有效期过了之后才会发送探测请求。默认为 1800 秒，即 30 分钟。                                                                         |
|     allowedHeaders：表示允许的请求头。默认为 *，表示该域中的所有的请求都被允许。                                                                                                                 |
|                                                                                                                                                                    |
| 1                                                                                                                                                                  |
| 2                                                                                                                                                                  |
| 3                                                                                                                                                                  |
| 4                                                                                                                                                                  |
| 5                                                                                                                                                                  |
| 6                                                                                                                                                                  |
| 7                                                                                                                                                                  |
| 8                                                                                                                                                                  |
| 9                                                                                                                                                                  |
| 10                                                                                                                                                                 |
| 11                                                                                                                                                                 |
|                                                                                                                                                                    |  |
| @RestController                                                                                                                                                    |
| public class WebAPIController {                                                                                                                                    |
|     @Autowired                                                                                                                                                     |
|     DeviceDataManager deviceDataManager;                                                                                                                           |
|                                                                                                                                                                    |
|     @GetMapping("/getDeviceDatas")                                                                                                                                 |
|     @CrossOrigin(value = "http://localhost:8081", maxAge = 1800, allowedHeaders ="*")                                                                              |
|     public List<DeviceData> getDeviceDatas() {                                                                                                                     |
|         return deviceDataManager.getDatas();                                                                                                                       |
|     }                                                                                                                                                              |
| }                                                                                                                                                                  |
|                                                                                                                                                                    |
| 2，采用全局配置                                                                                                                                                           |
| （1）全局配置需要添加自定义类实现 WebMvcConfigurer 接口，然后实现接口中的 addCorsMappings 方法。下面是一个简单的样例代码：                                                                                    |
|                                                                                                                                                                    |
|     addMapping：表示对哪种格式的请求路径进行跨域处理。                                                                                                                                 |
|     allowedHeaders：表示允许的请求头，默认允许所有的请求头信息。                                                                                                                          |
|     allowedMethods：表示允许的请求方法，默认是 GET、POST 和 HEAD。这里配置为 * 表示支持所有的请求方法。                                                                                              |
|     maxAge：表示探测请求的有效期                                                                                                                                              |
|     allowedOrigins 表示支持的域                                                                                                                                          |
|                                                                                                                                                                    |
| 1                                                                                                                                                                  |
| 2                                                                                                                                                                  |
| 3                                                                                                                                                                  |
| 4                                                                                                                                                                  |
| 5                                                                                                                                                                  |
| 6                                                                                                                                                                  |
| 7                                                                                                                                                                  |
| 8                                                                                                                                                                  |
| 9                                                                                                                                                                  |
| 10                                                                                                                                                                 |
| 11                                                                                                                                                                 |
| 12                                                                                                                                                                 |
|                                                                                                                                                                    |  |
| @Configuration                                                                                                                                                     |
| public class MyWebMvcConfig implements WebMvcConfigurer {                                                                                                          |
|                                                                                                                                                                    |
|     @Override                                                                                                                                                      |
|     public void addCorsMappings(CorsRegistry registry) {                                                                                                           |
|         registry.addMapping("/device/**")                                                                                                                          |
|                 .allowedHeaders("*")                                                                                                                               |
|                 .allowedMethods("*")                                                                                                                               |
|                 .maxAge(1800)                                                                                                                                      |
|                 .allowedOrigins("http://localhost:8081");                                                                                                          |
|     }                                                                                                                                                              |
| }                                                                                                                                                                  |
|                                                                                                                                                                    |
| （2）我们也可以采用如下配置，直接让所有请求、所有域都支持跨域：                                                                                                                                   |
| 1                                                                                                                                                                  |
| 2                                                                                                                                                                  |
| 3                                                                                                                                                                  |
| 4                                                                                                                                                                  |
| 5                                                                                                                                                                  |
| 6                                                                                                                                                                  |
| 7                                                                                                                                                                  |
| 8                                                                                                                                                                  |
| 9                                                                                                                                                                  |
| 10                                                                                                                                                                 |
| 11                                                                                                                                                                 |
| 12                                                                                                                                                                 |
|                                                                                                                                                                    |  |
| @Configuration                                                                                                                                                     |
| public class MyWebMvcConfig implements WebMvcConfigurer {                                                                                                          |
|                                                                                                                                                                    |
|     @Override                                                                                                                                                      |
|     public void addCorsMappings(CorsRegistry registry) {                                                                                                           |
|         registry.addMapping("/**")                                                                                                                                 |
|                 .allowedHeaders("*")                                                                                                                               |
|                 .allowedMethods("*")                                                                                                                               |
|                 .maxAge(1800)                                                                                                                                      |
|                 .allowedOrigins("*");                                                                                                                              |
|     }                                                                                                                                                              |
| }                                                                                                                                                                  |
|                                                                                                                                                                    |
| 原文出自：www.hangge.com  转载请保留原文链接：https://www.hangge.com/blog/cache/detail_2470.html                                                                                  |
