
Original 无敌码农 无敌码农 _2019-04-15 08:00_

  

![Image](media/Image-1.jpg)

  

导读

![Image](media/Image-10.png)

今天和大家聊一下在采用Spring Cloud进行微服务架构设计时，微服务之间调用时异常处理机制应该如何设计的问题。我们知道在进行微服务架构设计时，一个微服务一般来说不可避免地会同时面向内部和外部提供相应的功能服务接口。面向外部提供的服务接口，会通过服务网关（如使用Zuul提供的apiGateway）面向公网提供服务，如给App客户端提供的用户登陆、注册等服务接口。

  

而面向内部的服务接口，则是在进行微服务拆分后由于各个微服务系统的边界划定问题所导致的功能逻辑分散，而需要微服务之间彼此提供内部调用接口，从而实现一个完整的功能逻辑，它是之前单体应用中本地代码接口调用的服务化升级拆分。例如，需要在团购系统中，从下单到完成一次支付，需要交易系统在调用订单系统完成下单后再调用支付系统，从而完成一次团购下单流程，这个时候由于交易系统、订单系统及支付系统是三个不同的微服务，所以为了完成这次用户订单，需要App调用交易系统提供的外部下单接口后，由交易系统以内部服务调用的方式再调用订单系统和支付系统，以完成整个交易流程。如下图所示：  

  

![Image](media/Image-1.jpg)

  

这里需要说明的是，在基于SpringCloud的微服务架构中，所有服务都是通过如consul或eureka这样的服务中间件来实现的服务注册与发现后来进行服务调用的，只是面向外部的服务接口会通过网关服务进行暴露，面向内部的服务接口则在服务网关进行屏蔽，避免直接暴露给公网。而内部微服务间的调用还是可以直接通过consul或eureka进行服务发现调用，这二者并不冲突，只是**外部客户端是通过调用服务网关，服务网关通过consul再具体路由到对应的微服务接口，而内部微服务则是直接通过consul或者eureka发现服务后直接进行调用**。

  

异常处理的差异

![Image](media/Image-10.png)

面向外部的服务接口，我们一般会将接口的报文形式以JSON的方式进行响应，除了正常的数据报文外，我们一般会在报文格式中冗余一个响应码和响应信息的字段，如正常的接口成功返回：  

  

```
{    "code": "0",    "msg": "success",    "data": {        "userId": "zhangsan",        "balance": 5000    }}
```

  

而如果出现异常或者错误，则会相应地返回错误码和错误信息，如：  

  

```
{    "code": "-1",    "msg": "请求参数错误",    "data": null}
```

  

在编写面向外部的服务接口时，服务端所有的异常处理我们都要进行相应地捕获，并在controller层映射成相应地错误码和错误信息，因为面向外部的是直接暴露给用户的，是需要进行比较友好的展示和提示的，即便系统出现了异常也要坚决向用户进行友好输出，千万不能输出代码级别的异常信息，否则用户会一头雾水。对于客户端而言，只需要按照约定的报文格式进行报文解析及逻辑处理即可，一般我们在开发中调用的第三方开放服务接口也都会进行类似的设计，错误码及错误信息分类得也是非常清晰！

  

而微服务间彼此的调用在异常处理方面，我们则是希望更直截了当一些，就像调用本地接口一样方便，在基于Spring Cloud的微服务体系中，微服务提供方会提供相应的客户端SDK代码，而客户端SDK代码则是通过FeignClient的方式进行服务调用，如：而微服务间彼此的调用在异常处理方面，我们则是希望更直截了当一些，就像调用本地接口一样方便，在基于Spring Cloud的微服务体系中，微服务提供方会提供相应的客户端SDK代码，而客户端SDK代码则是通过FeignClient的方式进行服务调用，如：

  

```
@FeignClient(value = "order", configuration = OrderClientConfiguration.class, fallback = OrderClientFallback.class)public interface OrderClient {    //订单(内)    @RequestMapping(value = "/order/createOrder", method = RequestMethod.POST)    OrderCostDetailVo orderCost(@RequestParam(value = "orderId") String orderId,            @RequestParam(value = "userId") long userId,            @RequestParam(value = "orderType") String orderType,            @RequestParam(value = "orderCost") int orderCost,            @RequestParam(value = "currency") String currency,            @RequestParam(value = "tradeTime") String tradeTime)}
```

  

而服务的调用方在拿到这样的SDK后就可以忽略具体的调用细节，实现像本地接口一样调用其他微服务的内部接口了，当然这个是FeignClient框架提供的功能，**它内部会集成像Ribbon和Hystrix这样的框架来实现客户端服务调用的负载均衡和服务熔断功能**（注解上会指定熔断触发后的处理代码类），由于本文的主题是讨论异常处理，这里暂时就不作展开了。  

  

现在的问题是，虽然FeignClient向服务调用方提供了类似于本地代码调用的服务对接体验，但服务调用方却是不希望调用时发生错误的，即便发生错误，如何进行错误处理也是服务调用方希望知道的事情。另一方面，我们**在设计内部接口时，又不希望将报文形式搞得类似于外部接口那样复杂**，因为大多数场景下，我们是希望服务的调用方可以直截了的获取到数据，从而直接利用FeignClient客户端的封装，将其转化为本地对象使用。  

  

```
@Data@Builderpublic class OrderCostDetailVo implements Serializable {    private String orderId;    private String userId;    private int status;   //1:欠费状态；2:扣费成功    private int orderCost;    private String currency;    private int payCost;    private int oweCost;    public OrderCostDetailVo(String orderId, String userId, int status, int orderCost, String currency, int payCost,            int oweCost) {        this.orderId = orderId;        this.userId = userId;        this.status = status;        this.orderCost = orderCost;        this.currency = currency;        this.payCost = payCost;        this.oweCost = oweCost;    }}
```

  

如我们在把返回数据就是设计成了一个正常的VO/BO对象的这种形式，而不是向外部接口那么样额外设计错误码或者错误信息之类的字段，当然，也并不是说那样的设计方式不可以，只是感觉会让内部正常的逻辑调用，变得比较啰嗦和冗余，毕竟对于内部微服务调用来说，要么对，要么错，错了就Fallback逻辑就好了。  

  

不过，话虽说如此，可毕竟**服务是不可避免的会有异常情况的**。如果内部服务在调用时发生了错误，调用方还是应该知道具体的错误信息的，只是这种错误信息的提示需要以异常的方式被集成了FeignClient的服务调用方捕获，并且不影响正常逻辑下的返回对象设计，也就是说**我不想额外在每个对象中都增加两个冗余的错误信息字段，因为这样看起来不是那么优雅！**

  

**既然如此，那么应该如何设计呢？**  

  

最佳实践设计

![Image](media/Image-10.png)

首先，无论是内部还是外部的微服务，在服务端我们都**应该设计一个全局异常处理类**，用来统一封装系统在抛出异常时面向调用方的返回信息。而实现这样一个机制，我们可以利用Spring提供的注解**@ControllerAdvice**来实现异常的全局拦截和统一处理功能。如：  

  

```
@Slf4j@RestController@ControllerAdvicepublic class GlobalExceptionHandler {    @Resource    MessageSource messageSource;    @ExceptionHandler({org.springframework.web.bind.MissingServletRequestParameterException.class})    @ResponseBody    public APIResponse processRequestParameterException(HttpServletRequest request,            HttpServletResponse response,            MissingServletRequestParameterException e) {        response.setStatus(HttpStatus.FORBIDDEN.value());        response.setContentType("application/json;charset=UTF-8");        APIResponse result = new APIResponse();        result.setCode(ApiResultStatus.BAD_REQUEST.getApiResultStatus());        result.setMessage(                messageSource.getMessage(ApiResultStatus.BAD_REQUEST.getMessageResourceName(),                        null, LocaleContextHolder.getLocale()) + e.getParameterName());        return result;    }    @ExceptionHandler(Exception.class)    @ResponseBody    public APIResponse processDefaultException(HttpServletResponse response,            Exception e) {        //log.error("Server exception", e);        response.setStatus(HttpStatus.INTERNAL_SERVER_ERROR.value());        response.setContentType("application/json;charset=UTF-8");        APIResponse result = new APIResponse();        result.setCode(ApiResultStatus.INTERNAL_SERVER_ERROR.getApiResultStatus());        result.setMessage(messageSource.getMessage(ApiResultStatus.INTERNAL_SERVER_ERROR.getMessageResourceName(), null,                LocaleContextHolder.getLocale()));        return result;    }    @ExceptionHandler(ApiException.class)    @ResponseBody    public APIResponse processApiException(HttpServletResponse response,            ApiException e) {        APIResponse result = new APIResponse();        response.setStatus(e.getApiResultStatus().getHttpStatus());        response.setContentType("application/json;charset=UTF-8");        result.setCode(e.getApiResultStatus().getApiResultStatus());        String message = messageSource.getMessage(e.getApiResultStatus().getMessageResourceName(),                null, LocaleContextHolder.getLocale());        result.setMessage(message);        //log.error("Knowned exception", e.getMessage(), e);        return result;    }    /**     * 内部微服务异常统一处理方法     */    @ExceptionHandler(InternalApiException.class)    @ResponseBody    public APIResponse processMicroServiceException(HttpServletResponse response,            InternalApiException e) {        response.setStatus(HttpStatus.OK.value());        response.setContentType("application/json;charset=UTF-8");        APIResponse result = new APIResponse();        result.setCode(e.getCode());        result.setMessage(e.getMessage());        return result;    }}
```

  

如上述代码，我们在全局异常中针对内部统一异常及外部统一异常分别作了全局处理，这样只要服务接口抛出了这样的异常就会被全局处理类进行拦截并统一处理错误的返回信息。  

  

理论上我们可以在这个全局异常处理类中，捕获处理服务接口业务层抛出的所有异常并统一响应，只是**那样会让全局异常处理类变得非常臃肿**，所以从最佳实践上考虑，我们一般**会为内部和外部接口分别设计一个统一面向调用方的异常对象，**如外部统一接口异常我们叫ApiException，而内部统一接口异常叫InternalApiException。这样，我们就需要在面向外部的服务接口controller层中，将所有的业务异常转换为ApiException；而在面向内部服务的controller层中将所有的业务异常转化为InternalApiException。如：

  

```
@RequestMapping(value = "/creatOrder", method = RequestMethod.POST)public OrderCostDetailVo orderCost(         @RequestParam(value = "orderId") String orderId,         @RequestParam(value = "userId") long userId,         @RequestParam(value = "orderType") String orderType,         @RequestParam(value = "orderCost") int orderCost,         @RequestParam(value = "currency") String currency,         @RequestParam(value = "tradeTime") String tradeTime)throws InternalApiException {         OrderCostVo costVo = OrderCostVo.builder().orderId(orderId).userId(userId).busiId(busiId).orderType(orderType)                .duration(duration).bikeType(bikeType).bikeNo(bikeNo).cityId(cityId).orderCost(orderCost)                .currency(currency).strategyId(strategyId).tradeTime(tradeTime).countryName(countryName)                .build();        OrderCostDetailVo orderCostDetailVo;        try {            orderCostDetailVo = orderCostServiceImpl.orderCost(costVo);            return orderCostDetailVo;        } catch (VerifyDataException e) {            log.error(e.toString());            throw new InternalApiException(e.getCode(), e.getMessage());        } catch (RepeatDeductException e) {            log.error(e.toString());            throw new InternalApiException(e.getCode(), e.getMessage());        } }
```

  

如上面的内部服务接口的controller层中将所有的业务异常类型都统一转换成了内部服务统一异常对象InternalApiException了。这样全局异常处理类，就可以针对这个异常进行统一响应处理了。

  

对于外部服务调用方的处理就不多说了。而对于内部服务调用方而言，为了能够更加优雅和方便地实现异常处理，我们也需要在基于FeignClient的SDK代码中抛出统一内部服务异常对象，如：  

  

```
@FeignClient(value = "order", configuration = OrderClientConfiguration.class, fallback = OrderClientFallback.class)public interface OrderClient {    //订单(内)    @RequestMapping(value = "/order/createOrder", method = RequestMethod.POST)    OrderCostDetailVo orderCost(@RequestParam(value = "orderId") String orderId,            @RequestParam(value = "userId") long userId,            @RequestParam(value = "orderType") String orderType,            @RequestParam(value = "orderCost") int orderCost,            @RequestParam(value = "currency") String currency,            @RequestParam(value = "tradeTime") String tradeTime)throws InternalApiException};
```

  

这样在调用方进行调用时，就会强制要求调用方捕获这个异常，在正常情况下调用方不需要理会这个异常，像本地调用一样处理返回对象数据就可以了。在异常情况下，则会捕获到这个异常的信息，而这个异常信息则一般在服务端全局处理类中会被设计成一个带有错误码和错误信息的json数据，为了避免客户端额外编写这样的解析代码，**FeignClient为我们提供了异常解码机制**。如：  

  

```
@Slf4j@Configurationpublic class FeignClientErrorDecoder implements feign.codec.ErrorDecoder {    private static final Gson gson = new Gson();    @Override    public Exception decode(String methodKey, Response response) {        if (response.status() != HttpStatus.OK.value()) {            if (response.status() == HttpStatus.SERVICE_UNAVAILABLE.value()) {                String errorContent;                try {                    errorContent = Util.toString(response.body().asReader());                    InternalApiException internalApiException = gson.fromJson(errorContent, InternalApiException.class);                    return internalApiException;                } catch (IOException e) {                    log.error("handle error exception");                    return new InternalApiException(500, "unknown error");                }            }        }        return new InternalApiException(500, "unknown error");    }}
```

  

我们只需要在**服务调用方增加这样一个FeignClient解码器，就可以在解码器中完成错误消息的转换**。这样，我们在通过FeignClient调用微服务时就可以直接捕获到异常对象，从而**实现向本地一样处理远程服务返回的异常对象了**。

  

以上就是在利用Spring Cloud进行微服务拆分后关于异常处理机制的一点分享了，因为最近发现公司项目在使用Spring Cloud的微服务拆分过程中，这方面的处理比较混乱，所以写一篇文章和大家一起探讨下，如有更好的方式，也欢迎大家给我留言！  

  

推荐阅读：

[Spring Cloud微服务中网关服务是如何实现的?（Zuul篇）](http://mp.weixin.qq.com/s?__biz=MzU3NDY4NzQwNQ==&mid=2247484696&idx=1&sn=3f5822b1a6220264ea8d6ef491e6e208&chksm=fd2fd4daca585dccd4717e53644c471997a1122ddb638e02215880650e1273cc35ee35fdaf55&scene=21#wechat_redirect)

[Spring Cloud是怎么运行的？](http://mp.weixin.qq.com/s?__biz=MzU3NDY4NzQwNQ==&mid=2247484660&idx=1&sn=79196a6da3cd88c64f1e5af6804b51b1&chksm=fd2fd536ca585c20898d8381ca0f64ad2ac15fdd38173ee3c527c4df29ea59b94fa9c3d18abf&scene=21#wechat_redirect)

[Spring Boot到底是怎么运行的，你知道吗？](http://mp.weixin.qq.com/s?__biz=MzU3NDY4NzQwNQ==&mid=2247484593&idx=2&sn=3e112e9c5359dc3fb36aeabc14e8de56&chksm=fd2fd573ca585c652f17429362461953f2c5ecd7e1e078dab3196c81eae608f64fb462b0e05e&scene=21#wechat_redirect)

[基于SpringCloud的微服务架构演变史？](http://mp.weixin.qq.com/s?__biz=MzU3NDY4NzQwNQ==&mid=2247484134&idx=1&sn=7e988f2064f8b174e23a6e3bde4b0c1e&chksm=fd2fd324ca585a324741b38fb01fd20b015e7201839d195dbfcf5b3ee9753b49e5381cd883cf&scene=21#wechat_redirect)  

  

![Image](media/Image.gif)

  

—————END—————

  

![Image](media/Image.jpg)  

识别图片二维码，关注“无敌码农”获取精彩内容

People who liked this content also liked

![](media/qrcode.bmp)

Scan to Follow