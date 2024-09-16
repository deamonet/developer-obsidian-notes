# 用户

```java
package com.solax.installerapp.utils;  
  
import com.solax.installerapp.exception.InstallAppErrorCode;  
import com.solax.installerapp.exception.InstallAppException;  
import com.solax.installerapp.service.dto.context.UserBaseDTO;  
  
/**  
 * @author Qiu  
 * @version 1.0  
 * @Date 2023/8/16  
 * @description : 存储用户信息相关数据 ***登录态接口才能在上下文使用****  
 */public class UserContext {  
  
    private static final ThreadLocal<UserBaseDTO> user = new ThreadLocal<UserBaseDTO>();  
  
    private UserContext() {  
    }  
  
    public static Long getUserId() {  
        UserBaseDTO dto = getUser();  
        return dto.getUserId();  
    }  
  
    public static Integer getUserType() {  
        UserBaseDTO dto = getUser();  
        return dto.getUserType();  
    }  
  
    public static Long getRelId() {  
        UserBaseDTO dto = getUser();  
        return dto.getRelId();  
    }  
  
    public static String getToken() {  
        UserBaseDTO dto = getUser();  
        return dto.getToken();  
    }  
  
    public static UserBaseDTO getUser() {  
        UserBaseDTO dto = user.get();  
        if (null == dto) {  
            throw new InstallAppException(InstallAppErrorCode.TOKEN_ERROR);  
        }  
        return dto;  
    }  
  
    public static void setBaseUser(UserBaseDTO dto) {  
        user.set(dto);  
    }  
  
    public static void remove() {  
        user.remove();  
    }  
}
```

# 调用链

```java
package com.solax.installerapp.utils;  
  
  
/**  
 * @author Qiu  
 * @version 1.0  
 * @Date 2023/8/16  
 * @description : 链路调用凭证  
 */  
public class TraceContext {  
  
    private static final ThreadLocal<String> TRACE = new ThreadLocal<String>();  
  
    private TraceContext() {  
    }  
  
    public static String getTraceId() {  
        return TRACE.get();  
    }  
  
    public static void setTrace(String traceId) {  
        TRACE.set(traceId);  
    }  
  
    public static void remove() {  
        TRACE.remove();  
    }  
}
```

# 共用变量

```java
package com.solax.installerapp.utils;  
  
import com.solax.installerapp.service.dto.context.CommonBaseDTO;  
import lombok.extern.slf4j.Slf4j;  
  
/**  
 * @author Qiu  
 * @version 1.0  
 * @Date 2023/9/8  
 * @description : 存储公共字段属性(语言 版本)  
 */@Slf4j  
public class CommonBaseContext {  
  
    private static final ThreadLocal<CommonBaseDTO> commonBaseDTOLocal = new ThreadLocal<CommonBaseDTO>();  
  
    private CommonBaseContext() {  
    }  
  
    public static String getLang(){  
        CommonBaseDTO baseDTO = getCommonBaseDTO();  
        return baseDTO.getLang();  
    }  
  
    public static String getVersion(){  
        CommonBaseDTO baseDTO = getCommonBaseDTO();  
        return baseDTO.getVersion();  
    }  
  
    public static CommonBaseDTO getCommonBaseDTO() {  
        CommonBaseDTO baseDTO = commonBaseDTOLocal.get();  
        if (ValidateUtil.isBlank(baseDTO.getLang())){  
            log.error("CommonBaseContext.getCommonBaseDTO() 未获取对象数据，返回默认对象");  
            return new CommonBaseDTO("en_US","1.0");  
        }  
        return baseDTO;  
  
    }  
  
    public static void setCommonBaseDTO(CommonBaseDTO baseDTO) {  
        commonBaseDTOLocal.set(baseDTO);  
    }  
  
    public static void remove() {  
        commonBaseDTOLocal.remove();  
    }  
}
```



# 初始化context

```java
package com.solax.installerapp.filter;  
  
  
import com.solax.installerapp.service.dto.context.CommonBaseDTO;  
import com.solax.installerapp.utils.*;  
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
  
    private static final String HEAD_LANG = "lang";  
  
    private static final String HEAD_VERSION = "version";  
  
  
    @Override  
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {  
        try {  
            HttpServletRequest request = (HttpServletRequest) servletRequest;  
            String traceId = request.getHeader(TRANCE_KEY);  
            if (ValidateUtil.isEmpty(traceId)) {  
                traceId = UUID.randomUUID().toString();  
            }  
            TraceContext.setTrace(traceId);  
            MDC.put(TRANCE_KEY, traceId);  
  
            // 设置公共属性 ThreadLocal            
            CommonBaseDTO baseDTO = new CommonBaseDTO();  
            baseDTO.setLang(request.getHeader(HEAD_LANG));  
            baseDTO.setVersion(request.getHeader(HEAD_VERSION));  
            CommonBaseContext.setCommonBaseDTO(baseDTO);  
            filterChain.doFilter(servletRequest, servletResponse);  
        } finally {  
            MDC.remove(TRANCE_KEY);  
            CommonBaseContext.remove();  
            TraceContext.remove();  
        }  
    }  
}
```


```java

package com.solax.installerapp.interceptors;  
  
import com.alibaba.fastjson.JSON;  
import com.solax.entity.phoebus.SysUser;  
import com.solax.installerapp.annotations.Anonymous;  
import com.solax.installerapp.exception.InstallAppErrorCode;  
import com.solax.installerapp.exception.InstallAppException;  
import com.solax.installerapp.service.user.UserInfoService;  
import com.solax.installerapp.service.dto.context.UserBaseDTO;  
import com.solax.installerapp.utils.UserContext;  
import com.solax.installerapp.utils.ValidateUtil;  
import lombok.extern.slf4j.Slf4j;  
import org.springframework.core.annotation.AnnotationUtils;  
import org.springframework.stereotype.Component;  
import org.springframework.web.method.HandlerMethod;  
import org.springframework.web.servlet.HandlerInterceptor;  
  
import javax.annotation.Resource;  
import javax.servlet.http.HttpServletRequest;  
import javax.servlet.http.HttpServletResponse;  
  
/**  
 * @author Qiu  
 * @version 1.0  
 * @Date 2023/8/14  
 * @description : token校验  
 */  
@Slf4j  
@Component  
public class TokenAuthInterceptor implements HandlerInterceptor {  
  
    private static final String HEAD_TOKEN = "token";  
  
    @Resource  
    private UserInfoService userInfoService;  
  
    @Override  
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {  
        if (handler instanceof HandlerMethod) {  
            HandlerMethod handlerMethod = (HandlerMethod) handler;  
            // 匿名登录 不校验token  
            Anonymous anonymous = AnnotationUtils.findAnnotation(handlerMethod.getMethod(), Anonymous.class);  
            if (anonymous != null) {  
                return Boolean.TRUE;  
            }  
            //从header中获取token  
            String token = request.getHeader(HEAD_TOKEN);  
            if (ValidateUtil.isEmpty(token)) {  
                throw new InstallAppException(InstallAppErrorCode.TOKEN_EMPTY);  
            }  
            SysUser sysUserCache = userInfoService.getSysUserByToken(token);  
            // 判断token是否过期 进行续存处理  
            userInfoService.refreshToken(token);  
            // 设置ThreadLocal  
            UserBaseDTO dto = new UserBaseDTO();  
            dto.setToken(token);  
            dto.setLoginName(sysUserCache.getLoginName());  
            dto.setUserId(sysUserCache.getId());  
            dto.setRelId(sysUserCache.getRelId());  
            dto.setUserType(sysUserCache.getUserType());  
            if (log.isInfoEnabled()) {  
                log.info("TokenAuthInterceptor.Token:{}，校验成功，userBaseDTO=>{}", token, JSON.toJSONString(dto));  
            }  
            UserContext.setBaseUser(dto);  
        }  
        return true;  
    }  
  
    @Override  
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {  
        UserContext.remove();  
    }  
}
```