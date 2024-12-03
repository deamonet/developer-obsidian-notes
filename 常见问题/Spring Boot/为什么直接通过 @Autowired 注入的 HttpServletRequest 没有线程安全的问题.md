 梦康  2023-07-07 14:45:58  541

我们在各个地方注入依赖时，大多数情况下被注入的对象都是单例 bean。而在 web 开发中，每次请求的 `HttpServletRequest` 都是新创建的。

那为什么还可以直接 `@Autowired` 注入 `HttpServletRequest`呢？而且没有线程安全的问题呢？带着这个问题我做了如下笔记。

## 核心秘密

我们知道线程安全的做法一般是通过 `ThreadLocal` 来实现，这里也不例外。`Spring`设计了`RequestContextHolder`来管理存放所有的请求的信息。

```
public abstract class RequestContextHolder  {

    private static final ThreadLocal<RequestAttributes> requestAttributesHolder =
            new NamedThreadLocal<RequestAttributes>("Request attributes");

    private static final ThreadLocal<RequestAttributes> inheritableRequestAttributesHolder =
            new NamedInheritableThreadLocal<RequestAttributes>("Request context");
}
```

看完上面的代码想必你已经理解90%了。

为什么有两个`ThreadLocal`?

> `inheritableRequestAttributesHolder` 使用 `InheritableThreadLocal` 存储 `RequestAttributes` 对象。这意味着子线程可以继承父线程中存储的 RequestAttributes 对象，从而在多线程并发操作时更加方便。  
> **默认都是使用`requestAttributesHolder`**

新建一个 demo 工程，增加一个`TestController`，注入`HttpServletRequest`。

```
@RestController
public class TestController {

    @Autowired
    HttpServletRequest request;

    @RequestMapping("/")
    public String index() {
        return request.getQueryString();
    }
}
```

下面的分析以这块代码为例。

## 来龙去脉

### 应用启动

在 Spring Boot 应用启动阶段，当 `ApplicationContext` 初始化的时候，会注册很多依赖的处理方法，你会发现`DefaultListableBeanFactory.registerResolvableDependency()`方法将 `RequestObjectFactory` 注册为 `ServletRequest` 的依赖处理方法。具体代码如下：

```
public abstract class WebApplicationContextUtils {
    
    public static void registerWebApplicationScopes(ConfigurableListableBeanFactory beanFactory, ServletContext sc) {
        // ...    
        beanFactory.registerResolvableDependency(ServletRequest.class, new RequestObjectFactory());
    }

    private static class RequestObjectFactory implements ObjectFactory<ServletRequest>, Serializable {

        @Override
        public ServletRequest getObject() {
            return currentRequestAttributes().getRequest();
        }

        @Override
        public String toString() {
            return "Current HttpServletRequest";
        }
    }

    private static ServletRequestAttributes currentRequestAttributes() {
        return (ServletRequestAttributes) RequestContextHolder.currentRequestAttributes();
    }
}
```

**暂时可以粗略的认为**，当在生产`A` bean 的时候，如果它依赖了一个成员`ServletRequest`类型的变量，在使用`ServletRequest`时，就会通过 `RequestObjectFactory` 来获取。

### Bean 初始化

Spring Boot 上下文更新完毕之后，会初始化 `beanFactory` ，然后对单例 bean 进行前置初始化。单例 bean `TestController`在创建完毕之后，会执行 `postProcessProperties`的回调。

![postProcessProperties](https://mengkang.net/uploads/20230707/df8827aef55a497bc0b8c4a7d8b984a3.png "postProcessProperties")

![resolveFieldValue](https://mengkang.net/uploads/20230707/22d6d1e800f4406842823f184f7aa4aa.png "resolveFieldValue")

最后`AutowiredAnnotationBeanPostProcessor.AutowiredFieldElement#resolveFieldValue`通过  
`DefaultListableBeanFactory#resolveDependency`来处理`@Autowired`依赖注入的对象。如上面的截图，最后将`Proxy.newProxyInstance`生成的代理类，作为`request`的值赋值给了 `TestController` 单例 bean。

这里代码太多就不层层剖析了，有兴趣的同学按照上面的关键字打个断点即可查看。

### 请求初始化

`RequestContextFilter` 将请求的 `HttpServletRequest` 等对象绑定到当前线程上下文中，也就是最开始说的`ThreadLocal`中。同时在请求完毕之后也从`ThreadLocal`中移除，防止内存泄漏。

```
public class RequestContextFilter extends OncePerRequestFilter {

    private boolean threadContextInheritable = false;

    // ...

    @Override
    protected void doFilterInternal(
        HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
    throws ServletException, IOException {

        ServletRequestAttributes attributes = new ServletRequestAttributes(request, response);
        initContextHolders(request, attributes);

        try {
            filterChain.doFilter(request, response);
        } finally {
            resetContextHolders();
            attributes.requestCompleted();
        }
    }

    private void initContextHolders(HttpServletRequest request, ServletRequestAttributes requestAttributes) {
        LocaleContextHolder.setLocale(request.getLocale(), this.threadContextInheritable);
        RequestContextHolder.setRequestAttributes(requestAttributes, this.threadContextInheritable);
    }

    private void resetContextHolders() {
        LocaleContextHolder.resetLocaleContext();
        RequestContextHolder.resetRequestAttributes();
    }

}
```

### 请求执行

假如项目地址为 [http://localhost:8080/?a=1](http://localhost:8080/?a=1) 访问该地址，运行到`request.getQueryString()`时，实际这里用的 request 仍然是代理对象。

![请求执行](https://mengkang.net/uploads/20230708/82516d88f28ae810f66e26cd237a8fc7.png "请求执行")

注意其代理的是`WebApplicationContextUtils.RequestObjectFactory`，最终执行依赖动态代理的`invoke`机制，这里补充下简化了的 `ObjectFactoryDelegatingInvocationHandler` 的代码。

```
abstract class AutowireUtils {
    private static class ObjectFactoryDelegatingInvocationHandler implements InvocationHandler, Serializable {

        private final ObjectFactory<?> objectFactory;

        ObjectFactoryDelegatingInvocationHandler(ObjectFactory<?> objectFactory) {
            this.objectFactory = objectFactory;
        }

        @Override
        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
            return method.invoke(this.objectFactory.getObject(), args);
        };
    }
}
```

到这里，最后可以通过 `WebApplicationContextUtils.RequestObjectFactory` 查询到当前线程中的 `ServletRequest`。

### 补课篇 - 动态代理

```
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

public interface Hello {
    void sayHello(String name);
}

public class HelloImpl implements Hello {
    @Override
    public void sayHello(String name) {
        System.out.println("Hello, " + name + "!");
    }
}

public class HelloProxy implements InvocationHandler {
    private Object target;

    public HelloProxy(Object target) {
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("Before invoking " + method.getName());
        Object result = method.invoke(target, args);
        System.out.println("After invoking " + method.getName());
        return result;
    }
}

public class Main {
    public static void main(String[] args) {
        Hello hello = new HelloImpl();
        HelloProxy helloProxy = new HelloProxy(hello);
        Hello proxy = (Hello) Proxy.newProxyInstance(
            Main.class.getClassLoader(),
            new Class[]{Hello.class},
            helloProxy
        );
        proxy.sayHello("World");
    }
}
```