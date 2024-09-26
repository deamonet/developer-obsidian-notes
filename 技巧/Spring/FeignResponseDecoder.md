```java
@Configuration  
@Slf4j  
public class FeignResponseDecoder extends SpringDecoder {  
  
    public FeignResponseDecoder(ObjectFactory<HttpMessageConverters> messageConverters) {  
        super(messageConverters);  
    }  
  
    @Override  
    public Object decode(final Response response, Type type) throws IOException, FeignException {  
        if (response.headers().getOrDefault("content-type", Collections.emptyList()).stream().noneMatch("text/html;charset=utf-8"::equals)) {  
            return super.decode(response, type);  
        }  
        InputStream stream = response.body().asInputStream();  
        InputStreamReader reader = new InputStreamReader(stream);  
        BufferedReader bufferedReader = new BufferedReader(reader);  
        String body = bufferedReader.lines().collect(Collectors.joining(System.lineSeparator()));  
        log.error("微服务调度异常,headers={},body={}", response.headers(), body);  
        if ("404".equals(body)) {  
            throw new Exception()
        } else {  
            throw new Exception()
        }  
    }  
}
```