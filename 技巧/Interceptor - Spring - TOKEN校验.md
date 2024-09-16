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