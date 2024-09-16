```java
package com.solax.cloud.utils.context;  
  
import lombok.extern.slf4j.Slf4j;  
import org.springframework.web.context.request.RequestContextHolder;  
import org.springframework.web.context.request.ServletRequestAttributes;  
import javax.servlet.http.HttpServletRequest;  
  
@Slf4j  
public class RequestSourceUtil {  
  
    /**  
     * 获取请求来源  
     * @return  
     */  
    public static String  getSource(){  
        HttpServletRequest request = ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes()).getRequest();  
        return request.getHeader("x-request-source");  
    }  
}
```