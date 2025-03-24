2018年07月31日 16:20 ·  阅读 4429

原文链接： [click.aliyun.com](https://link.juejin.cn?target=http%3A%2F%2Fclick.aliyun.com%2Fm%2F1000010364%2F)

## 目标

1.  对于几种常见的入参方式，了解如何进行校验以及该如何处理错误消息；
2.  了解springboot 内置的参数异常类型，并能利用拦截器实现自定义处理；
3.  能实现简单的自定义校验规则

## 一、PathVariable 校验

在定义 Restful 风格的接口时，通常会采用 PathVariable 指定关键业务参数，如下：

```less
@GetMapping("/path/{group:[a-zA-Z0-9_]+}/{userid}")
@ResponseBody
public String path(@PathVariable("group") String group, @PathVariable("userid") Integer userid) {
    return group + ":" + userid;
}复制代码
```

**{group:[a-zA-Z0-9_]+}** 这样的表达式指定了 group 必须是以大小写字母、数字或下划线组成的字符串。  
我们试着访问一个错误的路径：

```vbnet
GET /path/testIllegal.get/10000复制代码
```

此时会得到 **404**的响应，因此对于PathVariable 仅由正则表达式可达到校验的目的

## 二、方法参数校验

类似前面的例子，大多数情况下，我们都会直接将HTTP请求参数映射到方法参数上。

```less
@GetMapping("/param")
@ResponseBody
public String param(@RequestParam("group")@Email String group, 
                    @RequestParam("userid") Integer userid) {
   return group + ":" + userid;
}复制代码
```

上面的代码中，@RequestParam 声明了映射，此外我们还为 group 定义了一个规则(复合Email格式)  
这段代码是否能直接使用呢？答案是否定的，为了启用方法参数的校验能力，还需要完成以下步骤：

-   声明 **MethodValidationPostProcessor**

```typescript
@Bean
public MethodValidationPostProcessor methodValidationPostProcessor() {
     return new MethodValidationPostProcessor();
}复制代码
```

-   Controller指定**@Validated**注解

```less
@Controller
@RequestMapping("/validate")
@Validated
public class ValidateController {复制代码
```

如此之后，方法上的@Email规则才能生效。

**校验异常**  
如果此时我们尝试通过非法参数进行访问时，比如提供非Email格式的 group  
会得到以下错误：

```sql
GET /validate/param?group=simple&userid=10000
====>
{
    "timestamp": 1530955093583,
    "status": 500,
    "error": "Internal Server Error",
    "exception": "javax.validation.ConstraintViolationException",
    "message": "No message available",
    "path": "/validate/param"
}复制代码
```

而如果参数类型错误，比如提供非整数的 userid，会得到：

```sql
GET /validate/param?group=simple&userid=1f
====>
{
    "timestamp": 1530954430720,
    "status": 400,
    "error": "Bad Request",
    "exception": "org.springframework.web.method.annotation.MethodArgumentTypeMismatchException",
    "message": "Failed to convert value of type 'java.lang.String' to required type 'java.lang.Integer'; nested exception is java.lang.NumberFormatException: For input string: \"1f\"",
    "path": "/validate/param"
}复制代码
```

当存在参数缺失时，由于定义的@RequestParam注解中，属性 required=true，也将会导致失败：

```bash
GET /validate/param?userid=10000
====>
{
    "timestamp": 1530954345877,
    "status": 400,
    "error": "Bad Request",
    "exception": "org.springframework.web.bind.MissingServletRequestParameterException",
    "message": "Required String parameter 'group' is not present",
    "path": "/validate/param"
}复制代码
```

## 三、表单对象校验

页面的表单通常比较复杂，此时可以将请求参数封装到表单对象中，  
并指定一系列对应的规则，参考[JSR-303](https://link.juejin.cn?target=https%3A%2F%2Fwww.ibm.com%2Fdeveloperworks%2Fcn%2Fjava%2Fj-lo-jsr303%2Findex.html "https://www.ibm.com/developerworks/cn/java/j-lo-jsr303/index.html")

```less
public static class FormRequest {

    @NotEmpty
    @Email
    private String email;

    @Pattern(regexp = "[a-zA-Z0-9_]{6,30}")
    private String name;

    @Min(5)
    @Max(199)
    private int age;复制代码
```

上面定义的属性中：

-   email必须非空、符合Email格式规则；
-   name必须为大小写字母、数字及下划线组成，长度在6-30个；
-   age必须在5-199范围内

Controller方法中的定义：

```less
@PostMapping("/form")
@ResponseBody
public FormRequest form(@Validated FormRequest form) {
    return form;
}复制代码
```

**@Validated**指定了参数对象需要执行一系列校验。

**校验异常**  
此时我们尝试构造一些违反规则的输入，会得到以下的结果：

```sql
{
    "timestamp": 1530955713166,
    "status": 400,
    "error": "Bad Request",
    "exception": "org.springframework.validation.BindException",
    "errors": [
        {
            "codes": [
                "Email.formRequest.email",
                "Email.email",
                "Email.java.lang.String",
                "Email"
            ],
            "arguments": [
                {
                    "codes": [
                        "formRequest.email",
                        "email"
                    ],
                    "arguments": null,
                    "defaultMessage": "email",
                    "code": "email"
                },
                [],
                {
                    "arguments": null,
                    "codes": [
                        ".*"
                    ],
                    "defaultMessage": ".*"
                }
            ],
            "defaultMessage": "不是一个合法的电子邮件地址",
            "objectName": "formRequest",
            "field": "email",
            "rejectedValue": "tecom",
            "bindingFailure": false,
            "code": "Email"
        },
        {
            "codes": [
                "Pattern.formRequest.name",
                "Pattern.name",
                "Pattern.java.lang.String",
                "Pattern"
            ],
            "arguments": [
                {
                    "codes": [
                        "formRequest.name",
                        "name"
                    ],
                    "arguments": null,
                    "defaultMessage": "name",
                    "code": "name"
                },
                [],
                {
                    "arguments": null,
                    "codes": [
                        "[a-zA-Z0-9_]{6,30}"
                    ],
                    "defaultMessage": "[a-zA-Z0-9_]{6,30}"
                }
            ],
            "defaultMessage": "需要匹配正则表达式\"[a-zA-Z0-9_]{6,30}\"",
            "objectName": "formRequest",
            "field": "name",
            "rejectedValue": "fefe",
            "bindingFailure": false,
            "code": "Pattern"
        },
        {
            "codes": [
                "Min.formRequest.age",
                "Min.age",
                "Min.int",
                "Min"
            ],
            "arguments": [
                {
                    "codes": [
                        "formRequest.age",
                        "age"
                    ],
                    "arguments": null,
                    "defaultMessage": "age",
                    "code": "age"
                },
                5
            ],
            "defaultMessage": "最小不能小于5",
            "objectName": "formRequest",
            "field": "age",
            "rejectedValue": 2,
            "bindingFailure": false,
            "code": "Min"
        }
    ],
    "message": "Validation failed for object='formRequest'. Error count: 3",
    "path": "/validate/form"
}
复制代码
```

如果是参数类型不匹配，会得到：

```sql
{
    "timestamp": 1530955359265,
    "status": 400,
    "error": "Bad Request",
    "exception": "org.springframework.validation.BindException",
    "errors": [
        {
            "codes": [
                "typeMismatch.formRequest.age",
                "typeMismatch.age",
                "typeMismatch.int",
                "typeMismatch"
            ],
            "arguments": [
                {
                    "codes": [
                        "formRequest.age",
                        "age"
                    ],
                    "arguments": null,
                    "defaultMessage": "age",
                    "code": "age"
                }
            ],
            "defaultMessage": "Failed to convert property value of type 'java.lang.String' 
to required type 'int' for property 'age'; nested exception is java.lang.NumberFormatException: 
For input string: \"\"",
            "objectName": "formRequest",
            "field": "age",
            "rejectedValue": "",
            "bindingFailure": true,
            "code": "typeMismatch"
        }
    ],
    "message": "Validation failed for object='formRequest'. Error count: 1",
    "path": "/validate/form"
}复制代码
```

> Form表单参数上，使用@Valid注解可达到同样目的，而关于两者的区别则是：

@Valid 基于JSR303，即 Bean Validation 1.0，由Hibernate Validator实现；  
@Validated 基于JSR349，是Bean Validation 1.1，由Spring框架扩展实现；

后者做了一些增强扩展，如支持分组校验，有兴趣可[参考这里](https://link.juejin.cn?target=https%3A%2F%2Fmy.oschina.net%2Fzzuqiang%2Fblog%2F761862%3Futm_medium%3Dreferral "https://my.oschina.net/zzuqiang/blog/761862?utm_medium=referral")。

## 四、RequestBody 校验

对于直接Json消息体输入，同样可以定义校验规则：

```less
@PostMapping("/json")
@ResponseBody
public JsonRequest json(@Validated @RequestBody JsonRequest request) {

    return request;
}

...
public static class JsonRequest {

    @NotEmpty
    @Email
    private String email;

    @Pattern(regexp = "[a-zA-Z0-9_]{6,30}")
    private String name;

    @Min(5)
    @Max(199)
    private int age;复制代码
```

**校验异常**  
构造一个违反规则的Json请求体进行输入，会得到：

```json
{
    "timestamp": 1530956161314,
    "status": 400,
    "error": "Bad Request",
    "exception": "org.springframework.web.bind.MethodArgumentNotValidException",
    "errors": [
        {
            "codes": [
                "Min.jsonRequest.age",
                "Min.age",
                "Min.int",
                "Min"
            ],
            "arguments": [
                {
                    "codes": [
                        "jsonRequest.age",
                        "age"
                    ],
                    "arguments": null,
                    "defaultMessage": "age",
                    "code": "age"
                },
                5
            ],
            "defaultMessage": "最小不能小于5",
            "objectName": "jsonRequest",
            "field": "age",
            "rejectedValue": 1,
            "bindingFailure": false,
            "code": "Min"
        }
    ],
    "message": "Validation failed for object='jsonRequest'. Error count: 1",
    "path": "/validate/json"
}
复制代码
```

此时与**FormBinding**的情况不同，我们得到了一个**MethodArgumentNotValidException**异常。  
而如果发生参数类型不匹配，比如输入**age=1f**，会产生以下结果：

```swift
{
    "timestamp": 1530956206264,
    "status": 400,
    "error": "Bad Request",
    "exception": "org.springframework.http.converter.HttpMessageNotReadableException",
    "message": "Could not read document: Can not deserialize value of type int from String \"ff\": not a valid Integer value\n at [Source: java.io.PushbackInputStream@68dc9800; line: 2, column: 8](through reference chain: org.zales.dmo.boot.controllers.ValidateController$JsonRequest[\"age\"]); nested exception is com.fasterxml.jackson.databind.exc.InvalidFormatException: Can not deserialize value of type int from String \"ff\": not a valid Integer value\n at [Source: java.io.PushbackInputStream@68dc9800; line: 2, column: 8](through reference chain: org.zales.dmo.boot.controllers.ValidateController$JsonRequest[\"age\"])",
    "path": "/validate/json"
}复制代码
```

这表明在JSON转换过程中已经失败！

## 五、自定义校验规则

框架内预置的校验规则可以满足大多数场景使用，  
但某些特殊情况下，你需要制作自己的校验规则，这需要用到**ContraintValidator**接口。

我们以一个密码校验的场景作为示例，比如一个注册表单上，  
我们需要检查 **密码输入** 与 **密码确认** 是一致的。

**首先定义 PasswordEquals 注解

```less
@Documented
@Constraint(validatedBy = { PasswordEqualsValidator.class })
@Target({ ElementType.METHOD, ElementType.FIELD, ElementType.TYPE })
@Retention(RetentionPolicy.RUNTIME)
public @interface PasswordEquals {

    String message() default "Password is not the same";

    Class<?>[] groups() default {};

    Class<? extends Payload>[] payload() default {};
}复制代码
```

**在表单上声明@PasswordEquals 注解**

```less
@PasswordEquals
public class RegisterForm {
   
    @NotEmpty
    @Length(min=5,max=30)
    private String username;
    
    @NotEmpty
    private String password;
    
    @NotEmpty
    private String passwordConfirm;复制代码
```

**针对@PasswordEquals实现校验逻辑**

```typescript
public class PasswordEqualsValidator implements ConstraintValidator<PasswordEquals, RegisterForm> {

    @Override
    public void initialize(PasswordEquals anno) {
    }

    @Override
    public boolean isValid(RegisterForm form, ConstraintValidatorContext context) {
        String passwordConfirm = form.getPasswordConfirm();
        String password = form.getPassword();

        boolean match = passwordConfirm != null ? passwordConfirm.equals(password) : false;
        if (match) {
            return true;
        }

        String messageTemplate = context.getDefaultConstraintMessageTemplate();
        
        // disable default violation rule
        context.disableDefaultConstraintViolation();

        // assign error on password Confirm field
        context.buildConstraintViolationWithTemplate(messageTemplate).addPropertyNode("passwordConfirm")
                .addConstraintViolation();
        return false;

    }
}
复制代码
```

如此，我们已经完成了自定义的校验工作。

## 六、异常拦截器

SpringBoot 框架中可通过 @ControllerAdvice 实现Controller方法的拦截操作。  
可以利用拦截能力实现一些公共的功能，比如权限检查、页面数据填充，以及全局的异常处理等等。

在前面的篇幅中，我们提及了各种校验失败所产生的异常，整理如下表：

异常类型

描述

ConstraintViolationException

违反约束，javax扩展定义

BindException

绑定失败，如表单对象参数违反约束

MethodArgumentNotValidException

参数无效，如JSON请求参数违反约束

MissingServletRequestParameterException

参数缺失

TypeMismatchException

参数类型不匹配

如果希望对这些异常实现统一的捕获，并返回自定义的消息，  
可以参考以下的代码片段：

```typescript
@ControllerAdvice
public static class CustomExceptionHandler extends ResponseEntityExceptionHandler {

    @ExceptionHandler(value = { ConstraintViolationException.class })
    public ResponseEntity<String> handle(ConstraintViolationException e) {
        Set<ConstraintViolation<?>> violations = e.getConstraintViolations();
        StringBuilder strBuilder = new StringBuilder();
        for (ConstraintViolation<?> violation : violations) {
            strBuilder.append(violation.getInvalidValue() + " " + violation.getMessage() + "\n");
        }
        String result = strBuilder.toString();
        return new ResponseEntity<String>("ConstraintViolation:" + result, HttpStatus.BAD_REQUEST);
    }

    @Override
    protected ResponseEntity<Object> handleBindException(BindException ex, HttpHeaders headers, HttpStatus status,
            WebRequest request) {
        return new ResponseEntity<Object>("BindException:" + buildMessages(ex.getBindingResult()),
                HttpStatus.BAD_REQUEST);
    }

    @Override
    protected ResponseEntity<Object> handleMethodArgumentNotValid(MethodArgumentNotValidException ex,
            HttpHeaders headers, HttpStatus status, WebRequest request) {
        return new ResponseEntity<Object>("MethodArgumentNotValid:" + buildMessages(ex.getBindingResult()),
                HttpStatus.BAD_REQUEST);
    }

    @Override
    public ResponseEntity<Object> handleMissingServletRequestParameter(MissingServletRequestParameterException ex,
            HttpHeaders headers, HttpStatus status, WebRequest request) {
        return new ResponseEntity<Object>("ParamMissing:" + ex.getMessage(), HttpStatus.BAD_REQUEST);
    }

    @Override
    protected ResponseEntity<Object> handleTypeMismatch(TypeMismatchException ex, HttpHeaders headers,
            HttpStatus status, WebRequest request) {
        return new ResponseEntity<Object>("TypeMissMatch:" + ex.getMessage(), HttpStatus.BAD_REQUEST);
    }

    private String buildMessages(BindingResult result) {
        StringBuilder resultBuilder = new StringBuilder();

        List<ObjectError> errors = result.getAllErrors();
        if (errors != null && errors.size() > 0) {
            for (ObjectError error : errors) {
                if (error instanceof FieldError) {
                    FieldError fieldError = (FieldError) error;
                    String fieldName = fieldError.getField();
                    String fieldErrMsg = fieldError.getDefaultMessage();
                    resultBuilder.append(fieldName).append(" ").append(fieldErrMsg).append(";");
                }
            }
        }
        return resultBuilder.toString();
    }
}复制代码
```

默认情况下，对于非法的参数输入，框架会产生 **HTTP_BAD_REQUEST(status=400)** 错误码，  
并输出友好的提示消息，这对于一般情况来说已经足够。

更多的输入校验及提示功能应该通过客户端去完成(服务端仅做同步检查)，  
客户端校验的用户体验更好，而这也符合**富客户端(rich client)**的发展趋势。

[refer document](https://link.juejin.cn?target=https%3A%2F%2Fwww.cnblogs.com%2Flittleatp%2Fp%2F9391856.html "https://www.cnblogs.com/littleatp/p/9391856.html")

## 参考文档

[springmvc-validation样例](https://link.juejin.cn?target=http%3A%2F%2Fwww.baeldung.com%2Fspring-mvc-custom-validator "http://www.baeldung.com/spring-mvc-custom-validator")  
[使用validation api进行操作](https://link.juejin.cn?target=https%3A%2F%2Fdocs.spring.io%2Fspring%2Fdocs%2F4.1.x%2Fspring-framework-reference%2Fhtml%2Fvalidation.html "https://docs.spring.io/spring/docs/4.1.x/spring-framework-reference/html/validation.html")  
[hibernate-validation官方文档](https://link.juejin.cn?target=https%3A%2F%2Fdocs.jboss.org%2Fhibernate%2Fvalidator%2F5.0%2Freference%2Fen-US%2Fhtml%2Fvalidator-customconstraints.html%23 "https://docs.jboss.org/hibernate/validator/5.0/reference/en-US/html/validator-customconstraints.html#")  
[Bean-Validation规范](https://link.juejin.cn?target=https%3A%2F%2Fwww.ibm.com%2Fdeveloperworks%2Fcn%2Fjava%2Fj-lo-beanvalid%2F "https://www.ibm.com/developerworks/cn/java/j-lo-beanvalid/")

欢迎继续关注"美码师的补习系列-springboot篇" ，期待更多精彩内容^-^