
```java
@Configuration  
@Slf4j  
public class FeignConfiguration {  
  
    private static final String ROUTE_SERVICE_VERSION = "version";  
    private static final String SERVICE_NAME_REPLACE_ALL = "-version";  
    @Resource  
    private GrayService grayService;  
  
    /**  
     * 设置超时时间  
     *  
     * @return  
     */  
    @Bean  
    Request.Options feignOptions() {  
        return new Request.Options(5 * 1000, 5 * 1000);  
    }  
  
  
    /**  
     * 自定义重试机制  
     *  
     * @return  
     */  
    @Bean  
    public Retryer feignRetryer() {  
        // 最大请求次数为3，初始间隔时间为100ms，下次间隔时间1.5倍递增，重试间最大间隔时间为1s，  
        return new Retryer.Default(100L, 1, 3);  
    }  
  
    /**  
     * RequestInterceptor:在请求发出之前拦截,可以修改请求。  
     * 请求增加Head信息  
     *  
     * @return  
     */  
    @Bean  
    public RequestInterceptor requestInterceptor() {  
        return template -> {  
            Target<?> target = template.feignTarget();  
            if (target.name().contains(ROUTE_SERVICE_VERSION)) {  
                String querySerViceName = target.name().replaceAll(SERVICE_NAME_REPLACE_ALL, "");  
                String version = grayService.getServiceNameVersion(querySerViceName);  
                String serviceName = querySerViceName + "-" + version;  
                Class<?> type = target.type();  
                String realUrl = target.url().replaceAll(ROUTE_SERVICE_VERSION, version);  
                Target.HardCodedTarget<?> hardCodedTarget = new Target.HardCodedTarget<>(type, serviceName, realUrl);  
                template.feignTarget(hardCodedTarget);  
                template.target(realUrl);  
            }  
            template.header(Constant.TRANCE_KEY, TraceContext.getTraceId());  
            ReqHeadThreadLocalDTO headThreadLocalDTO = HeadContext.getReqHeadDTO();  
            template.header(Constant.USER_ID, UserInfoContext.getUserInfoDTONotError() == null ? "-1" : String.valueOf(UserInfoContext.getUserInfoDTO().getUserId()));  
            template.header(Constant.Head.HEAD_VERSION, headThreadLocalDTO.getGrayVersion());  
            template.header(Constant.Head.HEAD_LANG, headThreadLocalDTO.getLang());  
            template.header(Constant.Head.HEAD_SOURCE, headThreadLocalDTO.getSource());  
            template.header(Constant.Head.WEB_SITE_TYPE, headThreadLocalDTO.getWebSiteType());  
            template.header(Constant.Head.APP_VER, headThreadLocalDTO.getVersion());  
            template.header(Constant.Head.PLATFORM, headThreadLocalDTO.getPlatform());  
            // 内部服务调用直接取token  
            template.header(Constant.RPC_TOKEN, RequestSourceEnum.getUserCacheToken());  
            log.info("通过feign请求服务【{}】，调用Url=>{}", template.feignTarget().name(), template.url());  
        };  
    }  
}
```