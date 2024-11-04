可以用手动生成log实例变量
```java

@Service  
public interface Interface {  
  
    Logger log = LoggerFactory.getLogger(Interface.class);  
  
    static <T> void post(String url, T tr) {  
        try {  
            ObjectMapper objectMapper = new ObjectMapper();  
            String data = objectMapper.writeValueAsString(tr);  
            String response = HttpUtil.post(url, data);  
            log.info("发送成功,response={},body={}", response, data);  
        } catch (Exception exception) {  
            log.error("发送失败", exception);  
        }  
    }  
}
```