[![](media/3114521287~100x100.awebp.webp)](https://juejin.cn/user/4248168663103565)

[CharlesYao ![lv-2](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ad1d5b8ec0974b0bbc14446acdd7c20d~tplv-k3u1fbpfcp-no-mark:0:0:0:0.awebp "创作等级")](https://juejin.cn/user/4248168663103565) 

2020年05月25日 14:46 ·  阅读 5468

@ControllerAdvice，是Spring3.2提供的新注解,它是一个Controller增强器,可对controller中被 @RequestMapping注解的方法加一些逻辑处理。主要作用有一下三种

-   通过@ControllerAdvice注解可以将对于控制器的全局配置放在同一个位置。
-   注解了@ControllerAdvice的类的方法可以使用@ExceptionHandler、@InitBinder、@ModelAttribute注解到方法上。
    -   @ExceptionHandler：用于全局处理控制器里的异常，进行全局异常处理
    -   @InitBinder：用来设置WebDataBinder，用于自动绑定前台请求参数到Model中，全局数据预处理。
    -   @ModelAttribute：本来作用是绑定键值对到Model中，此处让全局的@RequestMapping都能获得在此处设置的键值对 ，全局数据绑定。
-   @ControllerAdvice注解将作用在所有注解了@RequestMapping的控制器的方法上。

全局异常处理：

需要配合@ExceptionHandler使用。 当将异常抛到controller时,可以对异常进行统一处理,规定返回的json格式或是跳转到一个错误页面

```less
/**
 * @ClassName:CustomExceptionHandler
 * @Description: 全局异常捕获
 * @Author: 
 * @Date: 2020/5/25、13:38
 */
@Slf4j
@ControllerAdvice(annotations = {Controller.class, RestController.class})
public class WebControllerAdvice {
    @ResponseBody
    @ExceptionHandler
    public Map errorHandler(Exception ex) {
        Map errorMap = new HashMap();
        errorMap.put("code", 400);
        //判断异常的类型,返回不一样的返回值
        if (ex instanceof MissingServletRequestParameterException) {
            errorMap.put("msg", "缺少必需参数：" + ((MissingServletRequestParameterException) ex).getParameterName());
        } else if (ex instanceof MyException) {
            errorMap.put("msg", "这是自定义异常");
        }
        return errorMap;
    }
复制代码
```

自定义异常

```scala
/**
 * @ClassName:MyException
 * @Description: 定义异常
 * @Author: 
 * @Date: 2020/5/25、13:44
 */
public class MyException extends RuntimeException {
    private long code;
    private String msg;

    public MyException(Long code, String msg) {
        super(msg);
        this.code = code;
        this.msg = msg;
    }
    public MyException(String msg) {
        super(msg);
        this.msg = msg;
    }
}
复制代码
```

测试Controller

```java
@RestController
public class TestController {
    @RequestMapping("testException")
    public String testException() throws Exception{
        throw new MissingServletRequestParameterException("name","String");
    }

    @RequestMapping("testMyException")
    public String testMyException() throws MyException{
        throw new MyException("i am a myException");
    }
复制代码
```

测试结果：

```json
{"msg":"缺少必需参数：name","code":400}
{"msg":"这是自定义异常","code":400}
复制代码
```

全局数据绑定

全局数据绑定功能可以用来做一些初始化的数据操作，我们可以将一些公共的数据定义在添加了 @ControllerAdvice 注解的类中，这样，在每一个 Controller 的接口中，就都能够访问导致这些数据。使用步骤，首先定义全局数据，如下：

```typescript
/**
 * @ClassName:MyGlobalDataHandler
 * @Description: 全局数据
 * @Author: 
 * @Date: 2020/5/25、14:01
 */
@ControllerAdvice
public class MyGlobalDataHandler {
    @ModelAttribute(name = "md")
    public Map<String,Object> getGlobalData(){
        HashMap<String, Object> map = new HashMap<>();
        map.put("age", 99);
        map.put("gender", "男");
        return map;
    }
复制代码
```

使用 @ModelAttribute 注解标记该方法的返回数据是一个全局数据，默认情况下，这个全局数据的 key 就是返回的变量名，value 就是方法返回值，当然开发者可以通过 @ModelAttribute 注解的 name 属性去重新指定 key。定义完成后，在任何一个Controller 的接口中，都可以获取到这里定义的数据：

```typescript
    @GetMapping("/hello")
    public String hello(Model model) {
        Map<String, Object> map = model.asMap();
        System.out.println(map);
        int i = 1 / 0;
        return "hello controller advice";
    }
复制代码
```

运行结果

```ini
{md={gender=男, age=99}}
2020-05-25 14:04:44.388 - [WARN ] - [org.springframework.web.servlet.handler.AbstractHandlerExceptionResolver:logException:197] - Resolved [java.lang.ArithmeticException: / by zero] 
复制代码
```

全局数据预处理

考虑我有两个实体类，Book 和 Author，分别定义如下：

```scala
@Getter
@Setter
@AllArgsConstructor
@NoArgsConstructor
public class Book extends BaseEntity {
    private String name;
    private Long price;
}
@Getter
@Setter
@AllArgsConstructor
@NoArgsConstructor
public class Author  extends BaseEntity {
    private String name;
    private Long price;
}
复制代码
```

如果我定义一个数据添加接口，如下：

```csharp
@PostMapping("/book")
public void addBook(Book book, Author author) {
    System.out.println(book);
    System.out.println(author);
}
复制代码
```

这个时候，添加操作就会有问题，因为两个实体类都有一个 name 属性，从前端传递时 ，无法区分。此时，通过 @ControllerAdvice 的全局数据预处理可以解决这个问题

解决步骤如下: 1.给接口中的变量取别名:

```less
@PostMapping("/book")
public void addBook(@ModelAttribute("b") Book book, @ModelAttribute("a") Author author) {
    System.out.println(book);
    System.out.println(author);
}
复制代码
```

2.进行请求数据预处理 在 @ControllerAdvice 标记的类中添加如下代码:

```typescript
@InitBinder("b")
public void b(WebDataBinder binder) {
    binder.setFieldDefaultPrefix("b.");
}
@InitBinder("a")
public void a(WebDataBinder binder) {
    binder.setFieldDefaultPrefix("a.");
}
复制代码
```

@InitBinder("b") 注解表示该方法用来处理和Book和相关的参数,在方法中,给参数添加一个 b 前缀,即请求参数要有b前缀.

3.发送请求

请求发送时,通过给不同对象的参数添加不同的前缀,可以实现参数的区分.

![](media/1724a9482336422a~tplv-t2oaga2asx-zoom-in-crop-mark!4536!0!0!0.image.png)

这是我得微信公众号：**程序猿微刊** 更多文章请关注微信公众号

![](media/1724a94cd07c16c3~tplv-t2oaga2asx-zoom-in-crop-mark!4536!0!0!0.image.png)

分类：

[后端](https://juejin.cn/backend)

标签：

[Java](https://juejin.cn/tag/Java)