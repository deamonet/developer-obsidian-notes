同样使用`@RestControllerAdvice`可以用来统一数据的返回结构。不过我们需要实现`ResponseBodyAdvice`接口。才能劫持方法返回的数据，然后进行重写其中的方法。

例如：

```java
@RestControllerAdvice
public class MyResponseBodyAdvice implements ResponseBodyAdvice<Object> {

    ObjectMapper mapper = new ObjectMapper();

    @Override
    public boolean supports(MethodParameter methodParameter, Class<? extends HttpMessageConverter<?>> aClass) {
        return true;
    }

    @Override
    public Object beforeBodyWrite(Object o, MethodParameter methodParameter, MediaType mediaType, Class<? extends HttpMessageConverter<?>> aClass, ServerHttpRequest serverHttpRequest, ServerHttpResponse serverHttpResponse) {
        //处理返回值是String的状况
        //处理字符串类型数据
        if(o instanceof String){
          try {
            return mapper.writeValueAsString(Result.data(o));
           } catch (JsonProcessingException e) {
              e.printStackTrace();
          }
        }

        // 处理错误
        if(o instanceof  Exception){
            return Result.getResult((Exception) o);
        }

        return o instanceof Result ? o : Result.data(o);
    }
}

```

这里还有一个坑，就是json化和String类型的冲突，即如果Controller返回的是一个String。不加这一句的话会报错：

```less
class priv.mw.utils.Result cannot be cast to class java.lang.String (priv.mw.utils.Result is in unnamed module of loader org.apache.catalina.loader.ParallelWebappClassLoader @5049b1e5; java.lang.String is in module java.base of loader 'bootstrap')
```

即没办法将`Result`类型转化为`String`。其本质原因在于由于返回的是`String`，所以会匹配到`StringHttpMessageConverter`，因为他在所有Converter的前面。但由于我们在切面中将类型改成了Result，所以就会报错。

但实际上我们需要的是`MappingJackson2HttpMessageConverter`。

所以目前的解决办法有两种：

- 在转换之前就将String转化为JSON字符串。
- 将`MappingJackson2HttpMessageConverter` 放在所有converter的首位。

第一种就是就是上面的写法。

第二种可以通过配置来实现：

- XML方式：
    

```xml
<mvc:annotation-driven>
            <mvc:message-converters>
<!--            将MappingJackson2HttpMessageConverter移到前面,处理ResponseBodyAdvice的String异常的问题-->
            <bean class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter"/>
            <bean class="org.springframework.http.converter.StringHttpMessageConverter">
                <property name="defaultCharset" value="UTF-8"/>
            </bean>
        </mvc:message-converters>
</mvc:annotation-driven>
```

- Class方式

 ```java
import org.springframework.context.annotation.Configuration;
import org.springframework.http.converter.HttpMessageConverter;
import org.springframework.http.converter.json.MappingJackson2HttpMessageConverter;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

import java.util.List;

@Configuration
public class WebConfiguration implements WebMvcConfigurer {

	@Override
	public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
		converters.add(0, new MappingJackson2HttpMessageConverter());
	}
}
        ```
        

还有一点值得注意的是：实际上错误处理是在`ResponseBodyAdvice`中，即统一结构返回中来处理。因为实际上我们需要统一返回结构，对于错误，需要判断错误类型来返回对应的数据。所以在这里处理是更加方便的。


示例
```java
package com.solax.installerapp.aop;  
  
import com.solax.installerapp.vo.resp.ResponseWrapper;  
import lombok.extern.slf4j.Slf4j;  
import org.springframework.core.MethodParameter;  
import org.springframework.http.MediaType;  
import org.springframework.http.converter.HttpMessageConverter;  
import org.springframework.http.server.ServerHttpRequest;  
import org.springframework.http.server.ServerHttpResponse;  
import org.springframework.web.bind.annotation.RestController;  
import org.springframework.web.bind.annotation.RestControllerAdvice;  
import org.springframework.web.servlet.mvc.method.annotation.ResponseBodyAdvice;  
  
import java.util.Objects;  
  
/**  
 * @author yedongdong  
 * @date 2023/09/26 16:32  
 * @description 统一响应处理器 在每个responseBody的响应返回之前进行处理  
 */  
  
@Slf4j  
@RestControllerAdvice(annotations = {RestController.class})  
public class ControllerReturnEncapsulateAdvice implements ResponseBodyAdvice<Object> {  
  
    @Override  
    public boolean supports(MethodParameter returnType,  
                            Class<? extends HttpMessageConverter<?>> converterType) {  
        return true;  
    }  
  
    @Override  
    public ResponseWrapper<?> beforeBodyWrite(Object body,  
                                              MethodParameter returnType,  
                                              MediaType selectedContentType,  
                                              Class<? extends HttpMessageConverter<?>> selectedConverterType,  
                                              ServerHttpRequest request,  
                                              ServerHttpResponse response) {  
        if (Objects.isNull(body)) {  
            return ResponseWrapper.ofSuccess();  
        }  
        if (body instanceof ResponseWrapper) {  
            return (ResponseWrapper<?>) body;  
        }  
        return ResponseWrapper.ofSuccess(body);  
    }  
  
}
```



# ResponseBodyAdvice cannot change the selected HttpMessageConverter

👌。  
1、The code sample of the controllerAdvice

```java
@ControllerAdvice
@Slf4j
public class ReturnObjectAdvice implements ResponseBodyAdvice<Object> {

	@Override
	public boolean supports(MethodParameter returnType,
			Class<? extends HttpMessageConverter<?>> converterType) {
		return true;
	}

	@Override
	public Object beforeBodyWrite(Object body, MethodParameter returnType, MediaType selectedContentType,
			Class<? extends HttpMessageConverter<?>> selectedConverterType, ServerHttpRequest request,
			ServerHttpResponse response) {

		ObjectWrapper objectWrapper = returnType.getMethodAnnotation(ObjectWrapper.class);
		if (null != objectWrapper) {
			try {
				Object object = objectWrapper.value().newInstance();
				Method method = objectWrapper.value().getDeclaredMethod(objectWrapper.setData(), Object.class);
				if (null != method) {
					method.setAccessible(true);
					method.invoke(object, body);
                                         // change the response type 
					return object;
				}
			} catch (Exception e) {
				log.error("", e);
			}
		}
		return body;
	}
}
```

2、The code of the controller

```java
@RestController
@RequestMapping(value = "/a")
public class A {

	@GetMapping("/test")
	@ObjectWrapper
	public String checkCode() {
		return "some String";
	}
}
```

3、The annotation

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface ObjectWrapper {
	Class<?> value() default ResponseWrapper.class;

	String setData() default "setData";

	String setCode() default "setCode";

	String setMessage() default "setMessage";
}
```

4、The response type

```java
@Getter
@Setter
public class ResponseWrapper<T> implements Serializable {
	private static final long serialVersionUID = 1L;
	private int code;
	private String message;

	private T data;

	public ResponseWrapper() {
		this.code = HttpStatus.OK.value();
		this.message = HttpStatus.OK.getReasonPhrase();
	}

	public ResponseWrapper(HttpStatus status) {
		this.code = status.value();
		this.message = status.getReasonPhrase();
	}

	public ResponseWrapper(T data) {

		if (isEmpty(data)) {
			this.code = HttpStatus.NO_CONTENT.value();
			this.message = HttpStatus.NO_CONTENT.getReasonPhrase();
		} else {
			this.code = HttpStatus.OK.value();
			this.message = HttpStatus.OK.getReasonPhrase();
			this.data = data;
		}
	}

	public ResponseWrapper(HttpStatus status, T data) {
		this.code = status.value();
		this.message = status.getReasonPhrase();
		this.data = data;
	}

	@SuppressWarnings("rawtypes")
	public boolean isEmpty(T data) {
		return data == null || data instanceof Collection && ((Collection) data).isEmpty();
	}

	@Override
	public String toString() {
		try {
			return ReflectionToStringBuilder.toString(this, ToStringStyle.MULTI_LINE_STYLE);
		} catch (Exception e) {
			return super.toString();
		}
	}
}
```




# [ControllerAdvice ResponseBodyAdvice failed to enclose a String response](https://stackoverflow.com/questions/44121648/controlleradvice-responsebodyadvice-failed-to-enclose-a-string-response)

Asked 6 years, 4 months ago

Modified [2 years ago](https://stackoverflow.com/questions/44121648/controlleradvice-responsebodyadvice-failed-to-enclose-a-string-response?lastactivity "2021-09-18 17:34:34Z")

Viewed 6k times

3

[](https://stackoverflow.com/posts/44121648/timeline)

Not sure why, but when an endpoint return a String, the ResponseBodyAdvice always throws a Casting exception. Any other data type such as Long, List worked as desired, only the String data got exception. Any suggestion on how to fix this issue?

```
@RestController
public class AppController {
    private static Logger logger = LogManager.getLogger(AppController.class);

    @RequestMapping(path = "/hello",
                    method = GET)
    public String hello() {
        logger.info("... AppController.hello()");

        return "hello, world!";
    }

    @RequestMapping(path = "/timestamp",
                    method = GET)
    public long timestamp() {
        logger.info("... AppController.timestamp()");
        return System.currentTimeMillis();
    }
}
```

here is the adviser:

```
@ControllerAdvice
public class ResponseAdviser implements ResponseBodyAdvice<Object> {
    private static Logger logger = LogManager.getLogger(ResponseAdviser.class);

    @Override
    public boolean supports(MethodParameter returnType,
                            Class<? extends HttpMessageConverter<?>> converterType) {
        logger.trace("... ResponseAdviser.supports()");

        return true;
    }

    @Override
    public Object beforeBodyWrite(Object body,
                                  MethodParameter returnType,
                                  MediaType selectedContentType,
                                  Class<? extends HttpMessageConverter<?>> selectedConverterType,
                                  ServerHttpRequest request,
                                  ServerHttpResponse response) {
        logger.info("... ResponseAdviser.beforeBodyWrite()");

        Response resp = new Response();
        resp.setData(body);

        return resp;
    }
}
```

And the Reponse class is basically, adding a timestamp and enclose endpoint returned data

```
@Data
public class Response {
    private Date   timestamp;
    private Object data;

    public Response() {
        this.timestamp = new Date();
    }
}
```

Here is the stack trace:

```
Servlet.service() for servlet [dispatcherServlet] in context with path [] threw exception [Request processing failed; nested exception is java.lang.ClassCastException: org.phan.message.Response cannot be cast to java.lang.String] with root cause

java.lang.ClassCastException: org.phan.message.Response cannot be cast to java.lang.String
    at org.springframework.http.converter.StringHttpMessageConverter.getContentLength(StringHttpMessageConverter.java:41) ~[spring-web-4.3.8.RELEASE.jar:4.3.8.RELEASE]
    at org.springframework.http.converter.AbstractHttpMessageConverter.addDefaultHeaders(AbstractHttpMessageConverter.java:260) ~[spring-web-4.3.8.RELEASE.jar:4.3.8.RELEASE]
    at org.springframework.http.converter.AbstractHttpMessageConverter.write(AbstractHttpMessageConverter.java:205) ~[spring-web-4.3.8.RELEASE.jar:4.3.8.RELEASE]
    at org.springframework.web.servlet.mvc.method.annotation.AbstractMessageConverterMethodProcessor.writeWithMessageConverters(AbstractMessageConverterMethodProcessor.java:247) ~[spring-webmvc-4.3.8.RELEASE.jar:4.3.8.RELEASE]
    at org.springframework.web.servlet.mvc.method.annotation.RequestResponseBodyMethodProcessor.handleReturnValue(RequestResponseBodyMethodProcessor.java:174) ~[spring-webmvc-4.3.8.RELEASE.jar:4.3.8.RELEASE]
    at org.springframework.web.method.support.HandlerMethodReturnValueHandlerComposite.handleReturnValue(HandlerMethodReturnValueHandlerComposite.java:81) ~[spring-web-4.3.8.RELEASE.jar:4.3.8.RELEASE]
    at org.springframework.web.servlet.mvc.method.annotation.ServletInvocableHandlerMethod.invokeAndHandle(ServletInvocableHandlerMethod.java:113) ~[spring-webmvc-4.3.8.RELEASE.jar:4.3.8.RELEASE]
    at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.invokeHandlerMethod(RequestMappingHandlerAdapter.java:827) ~[spring-webmvc-4.3.8.RELEASE.jar:4.3.8.RELEASE]
    at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.handleInternal(RequestMappingHandlerAdapter.java:738) ~[spring-webmvc-4.3.8.RELEASE.jar:4.3.8.RELEASE]
    at org.springframework.web.servlet.mvc.method.AbstractHandlerMethodAdapter.handle(AbstractHandlerMethodAdapter.java:85) ~[spring-webmvc-4.3.8.RELEASE.jar:4.3.8.RELEASE]
    at org.springframework.web.servlet.DispatcherServlet.doDispatch(DispatcherServlet.java:963) ~[spring-webmvc-4.3.8.RELEASE.jar:4.3.8.RELEASE]
    at org.springframework.web.servlet.DispatcherServlet.doService(DispatcherServlet.java:897) ~[spring-webmvc-4.3.8.RELEASE.jar:4.3.8.RELEASE]
    at org.springframework.web.servlet.FrameworkServlet.processRequest(FrameworkServlet.java:970) ~[spring-webmvc-4.3.8.RELEASE.jar:4.3.8.RELEASE]
    at org.springframework.web.servlet.FrameworkServlet.doGet(FrameworkServlet.java:861) ~[spring-webmvc-4.3.8.RELEASE.jar:4.3.8.RELEASE]
    at javax.servlet.http.HttpServlet.service(HttpServlet.java:635) ~[tomcat-embed-core-8.5.14.jar:8.5.14]
    at org.springframework.web.servlet.FrameworkServlet.service(FrameworkServlet.java:846) ~[spring-webmvc-4.3.8.RELEASE.jar:4.3.8.RELEASE]
    at javax.servlet.http.HttpServlet.service(HttpServlet.java:742) ~[tomcat-embed-core-8.5.14.jar:8.5.14]
    at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:231) ~[tomcat-embed-core-8.5.14.jar:8.5.14]
    at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166) ~[tomcat-embed-core-8.5.14.jar:8.5.14]
    at org.apache.tomcat.websocket.server.WsFilter.doFilter(WsFilter.java:52) ~[tomcat-embed-websocket-8.5.14.jar:8.5.14]
    at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193) ~[tomcat-embed-core-8.5.14.jar:8.5.14]
    at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166) ~[tomcat-embed-core-8.5.14.jar:8.5.14]
    at org.springframework.boot.web.filter.ApplicationContextHeaderFilter.doFilterInternal(ApplicationContextHeaderFilter.java:55) ~[spring-boot-1.5.3.RELEASE.jar:1.5.3.RELEASE]
    at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:107) ~[spring-web-4.3.8.RELEASE.jar:4.3.8.RELEASE]
    at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193) ~[tomcat-embed-core-8.5.14.jar:8.5.14]
    at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166) ~[tomcat-embed-core-8.5.14.jar:8.5.14]
    at org.springframework.boot.actuate.trace.WebRequestTraceFilter.doFilterInternal(WebRequestTraceFilter.java:110) ~[spring-boot-actuator-1.5.3.RELEASE.jar:1.5.3.RELEASE]
    at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:107) ~[spring-web-4.3.8.RELEASE.jar:4.3.8.RELEASE]
    at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193) ~[tomcat-embed-core-8.5.14.jar:8.5.14]
    at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166) ~[tomcat-embed-core-8.5.14.jar:8.5.14]
    at org.springframework.web.filter.CharacterEncodingFilter.doFilterInternal(CharacterEncodingFilter.java:197) ~[spring-web-4.3.8.RELEASE.jar:4.3.8.RELEASE]
    at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:107) ~[spring-web-4.3.8.RELEASE.jar:4.3.8.RELEASE]
    at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193) ~[tomcat-embed-core-8.5.14.jar:8.5.14]
    at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166) ~[tomcat-embed-core-8.5.14.jar:8.5.14]
    at org.springframework.boot.actuate.autoconfigure.MetricsFilter.doFilterInternal(MetricsFilter.java:106) ~[spring-boot-actuator-1.5.3.RELEASE.jar:1.5.3.RELEASE]
    at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:107) ~[spring-web-4.3.8.RELEASE.jar:4.3.8.RELEASE]
    at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193) ~[tomcat-embed-core-8.5.14.jar:8.5.14]
    at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166) ~[tomcat-embed-core-8.5.14.jar:8.5.14]
    at org.apache.catalina.core.StandardWrapperValve.invoke(StandardWrapperValve.java:198) ~[tomcat-embed-core-8.5.14.jar:8.5.14]
    at org.apache.catalina.core.StandardContextValve.invoke(StandardContextValve.java:96) [tomcat-embed-core-8.5.14.jar:8.5.14]
    at org.apache.catalina.authenticator.AuthenticatorBase.invoke(AuthenticatorBase.java:478) [tomcat-embed-core-8.5.14.jar:8.5.14]
    at org.apache.catalina.core.StandardHostValve.invoke(StandardHostValve.java:140) [tomcat-embed-core-8.5.14.jar:8.5.14]
    at org.apache.catalina.valves.ErrorReportValve.invoke(ErrorReportValve.java:80) [tomcat-embed-core-8.5.14.jar:8.5.14]
    at org.apache.catalina.core.StandardEngineValve.invoke(StandardEngineValve.java:87) [tomcat-embed-core-8.5.14.jar:8.5.14]
    at org.apache.catalina.connector.CoyoteAdapter.service(CoyoteAdapter.java:342) [tomcat-embed-core-8.5.14.jar:8.5.14]
    at org.apache.coyote.http11.Http11Processor.service(Http11Processor.java:799) [tomcat-embed-core-8.5.14.jar:8.5.14]
    at org.apache.coyote.AbstractProcessorLight.process(AbstractProcessorLight.java:66) [tomcat-embed-core-8.5.14.jar:8.5.14]
    at org.apache.coyote.AbstractProtocol$ConnectionHandler.process(AbstractProtocol.java:861) [tomcat-embed-core-8.5.14.jar:8.5.14]
    at org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.doRun(NioEndpoint.java:1455) [tomcat-embed-core-8.5.14.jar:8.5.14]
    at org.apache.tomcat.util.net.SocketProcessorBase.run(SocketProcessorBase.java:49) [tomcat-embed-core-8.5.14.jar:8.5.14]
    at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142) [na:1.8.0_121]
    at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617) [na:1.8.0_121]
    at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61) [tomcat-embed-core-8.5.14.jar:8.5.14]
    at java.lang.Thread.run(Thread.java:745) [na:1.8.0_121]
```

- [rest](https://stackoverflow.com/questions/tagged/rest "show questions tagged 'rest'")
- [spring-boot](https://stackoverflow.com/questions/tagged/spring-boot "show questions tagged 'spring-boot'")

[Share](https://stackoverflow.com/q/44121648 "Short permalink to this question")

Follow

[edited May 23, 2017 at 14:24](https://stackoverflow.com/posts/44121648/revisions "show all edits to this post")

asked May 22, 2017 at 20:30

[

![TX T's user avatar](https://www.gravatar.com/avatar/240e5706567a0f36fbb5d5d1139650de?s=64&d=identicon&r=PG)

](https://stackoverflow.com/users/1313917/tx-t)

[TX T](https://stackoverflow.com/users/1313917/tx-t)

81722 gold badges1616 silver badges2525 bronze badges

- what are you trying to do ? . May be there is a easier way which we can suggest.
    
    – [pvpkiran](https://stackoverflow.com/users/6565093/pvpkiran "25,672 reputation")
    
    [May 23, 2017 at 7:54](https://stackoverflow.com/questions/44121648/controlleradvice-responsebodyadvice-failed-to-enclose-a-string-response#comment75276273_44121648)
    
- Basically, I like to return my Response container that includes a few pieces of information along with the return from the endpoint. i.e timestamp, and endpoint returned data. In this case, "hello, world!"
    
    – [TX T](https://stackoverflow.com/users/1313917/tx-t "817 reputation")
    
    [May 23, 2017 at 13:13](https://stackoverflow.com/questions/44121648/controlleradvice-responsebodyadvice-failed-to-enclose-a-string-response#comment75290182_44121648)
    

- can you put full stack trace
    
    – [pvpkiran](https://stackoverflow.com/users/6565093/pvpkiran "25,672 reputation")
    
    [May 23, 2017 at 13:30](https://stackoverflow.com/questions/44121648/controlleradvice-responsebodyadvice-failed-to-enclose-a-string-response#comment75290998_44121648)
    
- just added the stack trace
    
    – [TX T](https://stackoverflow.com/users/1313917/tx-t "817 reputation")
    
    [May 23, 2017 at 14:24](https://stackoverflow.com/questions/44121648/controlleradvice-responsebodyadvice-failed-to-enclose-a-string-response#comment75293558_44121648)
    

[Add a comment](https://stackoverflow.com/questions/44121648/controlleradvice-responsebodyadvice-failed-to-enclose-a-string-response# "Use comments to ask for more information or suggest improvements. Avoid answering questions in comments.")

## 6 Answers

Sorted by:

7

[](https://stackoverflow.com/posts/48093969/timeline)

You can try this code below:

```
 @Configuration
 public class WebConfig  extends WebMvcConfigurerAdapter {

     @Override
     public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {

        super.configureMessageConverters(converters);
        converters.add(0, new MappingJackson2HttpMessageConverter());
     }
 }
```

[Share](https://stackoverflow.com/a/48093969 "Short permalink to this answer")

Follow

[edited Jan 4, 2018 at 14:50](https://stackoverflow.com/posts/48093969/revisions "show all edits to this post")

[

![pringi's user avatar](https://i.stack.imgur.com/lO2Iq.png?s=64&g=1)

](https://stackoverflow.com/users/496038/pringi)

[pringi](https://stackoverflow.com/users/496038/pringi)

4,05055 gold badges3535 silver badges4545 bronze badges

answered Jan 4, 2018 at 11:10

[

![gushen's user avatar](https://www.gravatar.com/avatar/cda5c5bb48aa4f112f620c4b73a5782d?s=64&d=identicon&r=PG&f=y&so-version=2)

](https://stackoverflow.com/users/9144466/gushen)

[gushen](https://stackoverflow.com/users/9144466/gushen)

8611 silver badge44 bronze badges

- Thanks! That helped me too
    
    – [NikichXP](https://stackoverflow.com/users/4605754/nikichxp "147 reputation")
    
    [Aug 11, 2018 at 15:20](https://stackoverflow.com/questions/44121648/controlleradvice-responsebodyadvice-failed-to-enclose-a-string-response#comment90558521_48093969)
    

[Add a comment](https://stackoverflow.com/questions/44121648/controlleradvice-responsebodyadvice-failed-to-enclose-a-string-response# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

5

[](https://stackoverflow.com/posts/65015720/timeline)

**Reason:**

The root cause of `ClassCastException` thrown is that you convert the response type from `String` to `Response` in the method `ResponseAdviser.beforeBodyWrite`.

When your return type of a controller request method is `String`, the `HttpMessageConverter` instance used to convert the return value is `StringHttpMessageConverter`.

The sequence of converters to use when processing your return value of a controller method is like this: [![enter image description here](https://i.stack.imgur.com/gM4Hg.png)](https://i.stack.imgur.com/gM4Hg.png)

`ByteArrayHttpMessageConverter` cannot process `String` return type.

As we know, before write and flush the response, `ContentLength` should be calculated(when add http headers). The implements of `StringHttpMessageConverter.getContentLength` is like this: [![enter image description here](https://i.stack.imgur.com/wOseV.png)](https://i.stack.imgur.com/wOseV.png)

We can see, the type of input parameter str is `String`, but actually, the response type has already been modified to `Response`. So the `ClassCastException` thrown.

**Solution:**

Tell spring framework, don't use `StringHttpMessageConverter` to process `String` return type in controller method. You just need to add a `MappingJackson2HttpMessageConverter` in front of `StringHttpMessageConverter`. sample code:

```
@Configuration
public class WebMvcConfiguration implements WebMvcConfigurer {

    @Override
    public void configureMessageConverters(List<HttpMessageConverter<?>> 
        converters) {

        WebMvcConfigurer.super.configureMessageConverters(converters);
        converters.add(0, new MappingJackson2HttpMessageConverter());
    }

}
```

Why `MappingJackson2HttpMessageConverter` won't throw the `ClassCastException`? Just take a look at the implements(in its super class `AbstractJackson2HttpMessageConverter`): [![enter image description here](https://i.stack.imgur.com/BbsMH.png)](https://i.stack.imgur.com/BbsMH.png)

The type of input parameter object is `Object`, so it can process both `String` and `Response`.

[Share](https://stackoverflow.com/a/65015720 "Short permalink to this answer")

Follow

answered Nov 26, 2020 at 3:29

[

![Aaren Shar's user avatar](https://i.stack.imgur.com/DOjMV.png?s=64&g=1)

](https://stackoverflow.com/users/1573200/aaren-shar)

[Aaren Shar](https://stackoverflow.com/users/1573200/aaren-shar)

50855 silver badges99 bronze badges

- Very Nicely explained . This should be the accepted answer
    
    – [animo3991](https://stackoverflow.com/users/6306636/animo3991 "181 reputation")
    
    [Sep 2, 2021 at 20:03](https://stackoverflow.com/questions/44121648/controlleradvice-responsebodyadvice-failed-to-enclose-a-string-response#comment122012102_65015720)
    

[Add a comment](https://stackoverflow.com/questions/44121648/controlleradvice-responsebodyadvice-failed-to-enclose-a-string-response# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

2

[](https://stackoverflow.com/posts/69231168/timeline)

In the `ResponseBodyAdvice` class, add the `beforeBodyWrite` method:

```
@Override
    public Object beforeBodyWrite(Object body, MethodParameter returnType,
                                  MediaType selectedContentType, Class<? extends HttpMessageConverter<?>> selectedConverterType,
                                  ServerHttpRequest request, ServerHttpResponse response) {
      ...

        Response resp = new Response();
        resp.setData(body);

        //返回值body为String类型时
        if (body != null && body instanceof String) {
            try {
                response.getHeaders().setContentType(MediaType.APPLICATION_JSON_UTF8);
                return objectMapper.writeValueAsString(resp);
            } catch (JsonProcessingException e) {
                log.error(e.getMessage(), e);
                throw new RuntimeException(e.getMessage(), e);
            }
        }

        ...
    }
```

[Share](https://stackoverflow.com/a/69231168 "Short permalink to this answer")

Follow

[edited Sep 18, 2021 at 17:34](https://stackoverflow.com/posts/69231168/revisions "show all edits to this post")

[

![double-beep's user avatar](https://www.gravatar.com/avatar/e478f2600848a3dc5238b7ffb2e148f6?s=64&d=identicon&r=PG)

](https://stackoverflow.com/users/10607772/double-beep)

[double-beep](https://stackoverflow.com/users/10607772/double-beep)

5,0381717 gold badges3333 silver badges4141 bronze badges

answered Sep 18, 2021 at 2:39

[

![chen kqing's user avatar](https://www.gravatar.com/avatar/47ce5bc5501f9ac108669248a45d5222?s=64&d=identicon&r=PG&f=y&so-version=2)

](https://stackoverflow.com/users/5983602/chen-kqing)

[chen kqing](https://stackoverflow.com/users/5983602/chen-kqing)

2955 bronze badges

[Add a comment](https://stackoverflow.com/questions/44121648/controlleradvice-responsebodyadvice-failed-to-enclose-a-string-response# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

0

[](https://stackoverflow.com/posts/44131855/timeline)

Since you are using the `@RestController` annotation to identify your rest endpoint, if you would like to attach an `Advice` to it you have to use the `@RestControllerAdvice` annotation designed for rest endpoints.

[Share](https://stackoverflow.com/a/44131855 "Short permalink to this answer")

Follow