```java
package com.solax.cloud.user.filter;  
  
  
import lombok.extern.slf4j.Slf4j;  
import org.slf4j.MDC;  
import org.springframework.core.annotation.Order;  
import org.springframework.stereotype.Component;  
  
import javax.servlet.*;  
import javax.servlet.http.HttpServletRequest;  
import java.io.IOException;  
import java.util.UUID;  
  
/**  
 * @author Qiu  
 * @version 1.0  
 * @Date 2023/8/25  
 * @description : 调用链  
 */  
@Component  
@Order(-1)  
@Slf4j  
public class TraceFilter implements Filter {  
    public static final String TRANCE_KEY = "x-transaction-id";  
  
    @Override  
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {  
        try {  
            HttpServletRequest request = (HttpServletRequest) servletRequest;  
            String traceId = request.getHeader(TRANCE_KEY);  
            if (!StringUtils.hasText(traceId)) {  
                traceId = UUID.randomUUID().toString();  
            }  
            MDC.put(TRANCE_KEY, traceId);
            filterChain.doFilter(servletRequest, servletResponse);  
        } finally {  
            MDC.remove(TRANCE_KEY);  
        }  
    }  
}
```