

```java
package com.solax.installerapp.aop;  
  
  
import com.solax.enums.ModuleType;  
import com.solax.installerapp.exception.InstallAppErrorCode;  
import com.solax.installerapp.exception.InstallAppException;  
import com.solax.installerapp.vo.resp.ResponseWrapper;  
import lombok.extern.slf4j.Slf4j;  
import org.springframework.web.bind.annotation.ExceptionHandler;  
import org.springframework.web.bind.annotation.RestControllerAdvice;  
  
import javax.validation.ConstraintViolationException;  
  
@RestControllerAdvice  
@Slf4j  
public class ControllerExceptionAdvice {  
  
  
    @ExceptionHandler(value = InstallAppException.class)  
    public ResponseWrapper<?> business(InstallAppException ex) {  
        log.error("business exception", ex);  
        return new ResponseWrapper<Void>(ex.getErrorCode(), ex.getErrorMsg());  
    }  
  
    @ExceptionHandler(value = IllegalArgumentException.class)  
    public ResponseWrapper<?> business(IllegalArgumentException ex) {  
        log.error("business exception", ex);  
        return new ResponseWrapper<Void>(InstallAppErrorCode.PARAMETER_ERROR.getErrorCode(), ex.getMessage());  
    }  
  
    /**  
     * 校验 http get 方法中的参数  
     * @param exception  
     * @return  
     */  
    @ExceptionHandler(value = ConstraintViolationException.class)  
    public ResponseWrapper<?> requestParameterValidation(Exception exception){  
        log.error("business exception", exception);  
        // 这个 exception 返回的 message 会加上方法名和参数,中间用点隔开  
        String[] message = exception.getMessage().split("\\.");  
        return new ResponseWrapper<Void>(InstallAppErrorCode.PARAMETER_ERROR.getErrorCode(), message[message.length-1]);  
    }  
  
    @ExceptionHandler(value = ModuleType.NoSuchDeviceException .class)  
    public ResponseWrapper<?> noSuchDeviceException(Exception exception){  
        log.error("arguments error", exception);  
        return ResponseWrapper.ofFail(InstallAppErrorCode.DEVICE_NOT_EXIST);  
    }  
  
    @ExceptionHandler(value = Exception.class)  
    public ResponseWrapper<?> exception(Exception ex) {  
        log.error("system error", ex);  
        return ResponseWrapper.ofFail(InstallAppErrorCode.SYSTEM_ERROR);  
    }  
  
    @ExceptionHandler(value = Throwable.class)  
    public ResponseWrapper<?> throwable(Exception ex) {  
        log.error("throwable error", ex);  
        return ResponseWrapper.ofFail(InstallAppErrorCode.SYSTEM_ERROR);  
    }  
}
```