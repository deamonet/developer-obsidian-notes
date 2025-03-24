更新时间：2024/07/08

[产品信息](https://ecloud.10086.cn/portal/product/redis)

Lettuce由于其优异的性能、易维护、线程安全等特性，在spring boot 2.0开始取代jedis，作为默认的客户端集成到spring boot中。但是，很多客户在业务上云后会出现io.lettuce.core.RedisCommandTimeoutException: Command timed out after XX second(s) 命令超时报错。移动云数据库工程师持续跟进并分析解决问题，通过分析Lettuce源码，发现Lettuce 客户端底层使用netty进行网络管理，在超时情况下不会重新建立连接，因此旧连接在网络波动、防火墙超时等原因断连的情况下，可能未产生RST到客户端，会存在超时问题，移动云有安全可靠的网络架构，从业务到数据库会经过多层防火墙和网关，对于Lettuce客户端的问题就表现更为突出，需要注意的是，即使业务在非云环境，Lettuce超时问题仍旧存在。

## Lettuce频繁超时问题解决方式

由于通过配置 TCP 的 KeepAlive 保活等方式都无法解决自身缺陷产生的问题，为了规避此问题，业界通常建议是：用Jedis客户端替换Lettuce客户端。2023年11月15日，lettuce社区针对此问题发布了新版本通过新增TCP_USER_TIMEOUT参数解决这个问题。

Lettuce 6.3.0版本发布和问题跟进如下：

[https://github.com/lettuce-io/lettuce-core/releases/tag/6.3.0.RELEASE](https://github.com/lettuce-io/lettuce-core/releases/tag/6.3.0.RELEASE)

[https://github.com/lettuce-io/lettuce-core/issues/2082](https://github.com/lettuce-io/lettuce-core/issues/2082)

移动云结合客户业务实际情况，给出如下解决方案：

## 方案一：为 Lettuce 添加心跳机制

由于大部分客户的spring boot客户端版本还比较低，升级到最新的lettuce 6.3.0后需要升级spring boot版本或者对依赖进行升级修改，会一定程度上影响业务代码，需要进行修改，对于spring boot低版本的客户，不想有太多修改的，可使用Lettuce 添加心跳机制方式。

**步骤一：**保证spring-boot-starter-parent版本在2.1.0.RELEASE以上，2.1.0.RELEASE以上版本提供了NettyCustomizer重写机制。

<groupId>org.springframework.boot</groupId>  
<artifactId>spring-boot-starter-parent</artifactId>  
<version>2.1.0.RELEASE</version>

**步骤二：**客户端增加LettuceConfig配置类，出现上述问题主要是因为跨多网关网络链路出现波动等异常导致lettuce未收回包而没能触发ChanelInactived事件，因此如果需要规避这个问题就需要添加心跳机制，在应用代码中新建Netty配置，缩短重连间隔时间，重新打包部署Lettuce客户端应用程序，此方式会对客户端有一定的消耗。

```
import io.lettuce.core.resource.ClientResources;import io.lettuce.core.resource.NettyCustomizer;import io.netty.channel.Channel;import io.netty.channel.ChannelDuplexHandler;import io.netty.channel.ChannelHandlerContext;import io.netty.handler.timeout.IdleStateEvent;import io.netty.handler.timeout.IdleStateHandler;import org.springframework.context.annotation.Bean;import org.springframework.context.annotation.Configuration;@Configurationpublic class LettuceConfig {  @Bean  public ClientResources clientResources(){    NettyCustomizer nettyCustomizer = new NettyCustomizer() {    @Override            public void afterChannelInitialized(Channel channel) {                channel.pipeline().addLast(                        //第一个参数readerIdleTimeSeconds设置为小于超时时间timeout，单位为秒，                        //每隔readerIdleTimeSeconds会进行重连,在超时之前重连就能避免命令超时报错。                        new IdleStateHandler(30, 0, 0));                channel.pipeline().addLast(new ChannelDuplexHandler() {                    @Override                    public void userEventTriggered(ChannelHandlerContext ctx, Object isEvt) throws Exception {                        if (isEvt instanceof IdleStateEvent) {                            ctx.disconnect();                        }                    }                });            }        };    return ClientResources.builder().nettyCustomizer(nettyCustomizer).build();    }}
```

**注意**

如果配置文件重写了，注意将clientResources进行覆盖，避免未生效。

```
<groupId>org.springframework.boot</groupId>LettuceClientConfiguration clientConfiguration = LettucePoolingClientConfiguration.builder()  .clientResources(clientResources).build();        return new LettuceConnectionFactory(redisStandaloneConfiguration,clientConfiguration);
```

## 方案二：更新Lettuce版本，增加TCP_USER_TIMEOUT参数

将Lettuce客户端版本升级到6.3.0.RELEASE及以上，同时netty版本升级到4.1.100.Final以上，对于spring boot版本较低的，可能需要进行相关依赖更新。

**步骤一：**更新pom依赖中lettuce和netty的版本。

```
<dependencies>
```

**步骤二：**增加lettuce初始化配置，打开并配置重连时间TCP_USER_TIMEOUT，解决Lettuce超时的问题。

**说明**

TCP_USER_TIMEOUT定义了当TCP连接处于非活动状态时，内核等待应用程序读取数据或写入数据的最大时间。如果在这个时间内没有发生任何数据交换，并且应用程序也没有关闭连接，那么内核会主动终止这个连接。

@Configuration  
public class LettuceConfig {  
/**  
    *  TCP_KEEPALIVE打开，配置三个参数  
    */  
   private static final int TCP_KEEPALIVE_IDLE = 30;  
   private static final int TCP_KEEPINTVL = 10;  
 private static final int TCP_KEEPCNT = 3;  
**  
    * TCP_USER_TIMEOUT打开并配置重连时间，解决Lettuce超时的问题，需要配置时间小于超时时间。  
    */  
   private static final int TCP_USER_TIMEOUT = 30;  
@Bean  
   LettuceConnectionFactory redisConnectionFactory() {  
       RedisStandaloneConfiguration config = new RedisStandaloneConfiguration();  
       config.setHostName("127.0.0.1");  
       config.setPort(6379);  
SocketOptions socketOptions = SocketOptions.builder()  
               .keepAlive(SocketOptions.KeepAliveOptions.builder()  
                       .enable()  
                       .idle(Duration.ofSeconds(TCP_KEEPALIVE_IDLE))  
                       .interval(Duration.ofSeconds(TCP_KEEPINTVL))  
                       .count(TCP_KEEPCNT)  
                       .build())  
               .tcpUserTimeout(SocketOptions.TcpUserTimeoutOptions.builder()  
                       .enable()  
                       .tcpUserTimeout(Duration.ofSeconds(TCP_USER_TIMEOUT))  
                       .build())  
               .build();  
 LettuceClientConfiguration lettuceClientConfiguration = LettuceClientConfiguration.builder().clientOptions(  
               ClientOptions.builder().socketOptions(socketOptions).build()).build();  
       return new LettuceConnectionFactory(config, lettuceClientConfiguration);  
   }  
 @Bean  
   RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory connectionFactory) {  
       RedisTemplate<String, Object> template = new RedisTemplate<>();  
       template.setConnectionFactory(connectionFactory);  
       return template;  
   }  
}