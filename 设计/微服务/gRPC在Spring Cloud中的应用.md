

发布于2020-04-01 19:37:14阅读 3.6K0

**1、前言**

在微服务开发中，服务间的调用一般有两种方式：Feign、RestTemplate，但在实际使用过程中，尤其是Feign，存在各种限制及局限性，如：HTTP请求方式、返回类型等限制，有时会让你觉得那那都别扭。在微服务项目中，服务间的调用，是非常普遍频繁的，其性能也不是很理想。

为了解决上述问题，进行反复对比，最终服务间的调用方式采取了gRPC方式，其显著特点就是性能之高(通信采用Netty)，通过proto文件定义的接口也是非常清晰而又灵活。本文主要就gRPC在Spring Cloud项目中的使用进行说明实战。

关于gRPC相关基础知识可以参考上一篇文章[gRPC的使用](http://mp.weixin.qq.com/s?__biz=MzA5MzUwOTY4NQ==&mid=2247484376&idx=1&sn=bdc952861a4b7445252cda9895e433e1&chksm=905d8282a72a0b94efedd9e62e0443925beba363f9f2a1221ff5390f0f8e46407b9953458220&scene=21#wechat_redirect)。

**2、gRPC在Spring Cloud中的使用**

看过上一篇文章gRPC的使用的话，你就清楚如果直接使用gRPC，显得有些吃力，因此借助一些开源的框架变得尤为必要。gRPC在Spring Cloud中使用开源项目grpc-spring-boot-starter，便于在Spring Cloud项目中开发应用。

(_grpc-spring-boot-starter虽然存在一些问题，但集成Sping Cloud项目已经相当高了，还是不错之选。如果你有时间，精力，还是又必要在源码基础上进行开发。_)

下面以实际demo来说明grpc-spring-boot-starter的应用。

**2.1 特点**

-   使用@ GrpcService自动创建并运行一个 gRPC 服务，内嵌在 spring-boot 应用中
-   使用@ GrpcClient自动创建和管理你的客户端
-   支持Spring Cloud（向Consul或Eureka注册服务并获取gRPC[服务器](https://cloud.tencent.com/product/cvm?from=10680)信息）
-   支持Spring Sleuth 进行链路跟踪
-   支持对于server、client 分别设置全局拦截器或单个的拦截器
-   支持Spring-Security
-   支持metric (micrometer / actuator)

（看了上面这些特点，就知道为啥选择这个开源项目了）

**2.2 使用DEMO**

**2.2.1 定义gRPC接口**

基于protobuf来声明数据模型和RPC接口服务。

创建一个公共字模块项目spring-boot-grpc-common，用于定义存放gRPC接口(proto)，便于gRPC服务端和客户端使用。以helloworld.proto(src\main\proto\helloworld.proto)为例：

```javascript
syntax = "proto3";



option java_multiple_files = true;

option java_package = "com.xcbeyond.springboot.grpc.lib";

option java_outer_classname = "HelloWorldProto";



// The greeting service definition.

service Simple {

    // Sends a greeting

    rpc SayHello (HelloRequest) returns (HelloReply) {

    }

}



// The request message containing the user's name.

message HelloRequest {

    string name = 1;

}



// The response message containing the greetings

message HelloReply {

    string message = 1;

}
```

复制

根据proto的命令可以转换成对应的语言的代码，生成java代码，也可以借助maven插件，在编译时自动生成。这里通过mavent插件，可以在pom.xml中增加如下依赖：

```javascript
<build>

        <extensions>

            <extension>

                <groupId>kr.motd.maven</groupId>

                <artifactId>os-maven-plugin</artifactId>

                <version>${os.plugin.version}</version>

            </extension>

        </extensions>

        <plugins>

            <plugin>

                <groupId>org.xolstice.maven.plugins</groupId>

                <artifactId>protobuf-maven-plugin</artifactId>

                <version>${protobuf.plugin.version}</version>

                <configuration>

                    <protocArtifact>com.google.protobuf:protoc:${protoc.version}:exe:${os.detected.classifier}</protocArtifact>

                    <pluginId>grpc-java</pluginId>

                    <pluginArtifact>io.grpc:protoc-gen-grpc-java:${grpc.version}:exe:${os.detected.classifier}</pluginArtifact>

                </configuration>

                <executions>

                    <execution>

                        <goals>

                            <goal>compile</goal>

                            <goal>compile-custom</goal>

                        </goals>

                    </execution>

                </executions>

            </plugin>

        </plugins>

</build>
```

复制

在maven中进行package编译打包，将会在target中看见根据proto自动生成的Java类。(编译过程中可能会报错，此时可以忽略)

![](media/9m918enq21.jpeg.jpg)

**2.2.2 gRPC服务端**

maven依赖：

> <dependency> <groupId>net.devh</groupId> <artifactId>grpc-server-spring-boot-starter</artifactId> <version>2.2.1.RELEASE</version> </dependency>

实现 gRPC server 的业务逻辑，使用注解**@GrpcService**定义gRPC服务端，如下所示：

```javascript
package com.xcbeyond.springboot.grpc.server.service;



import com.xcbeyond.springboot.grpc.lib.HelloReply;

import com.xcbeyond.springboot.grpc.lib.HelloRequest;

import com.xcbeyond.springboot.grpc.lib.SimpleGrpc;

import io.grpc.stub.StreamObserver;

import net.devh.boot.grpc.server.service.GrpcService;



/**

 * @Auther: xcbeyond

 * @Date: 2019/3/6 18:15

 */

@GrpcService

public class GrpcServerService extends SimpleGrpc.SimpleImplBase {

    @Override

    public void sayHello(HelloRequest request, StreamObserver<HelloReply> responseObserver) {

        System.out.println("GrpcServerService...");

        HelloReply reply = HelloReply.newBuilder().setMessage("Hello ==> " + request.getName()).build();

        responseObserver.onNext(reply);

        responseObserver.onCompleted();

    }

}
```

复制

application.yml配置：

gRPC 的 host 跟 port ，默认的监听的 host 是 0.0.0.0，默认的 port 是 9090，配置为0将会自动分配未使用的端口。

> grpc: server: port: 0

**2.2.3 gRPC客户端**

maven依赖：

> <dependency> <groupId>net.devh</groupId> <artifactId>grpc-client-spring-boot-starter</artifactId> <version>2.2.1.RELEASE</version> </dependency>

使用注解**@GrpcClient**来调用服务端接口，通过

> HelloReply response = simpleBlockingStub.sayHello(HelloRequest.newBuilder().setName(name).build());

直接向服务端发起请求，和调用本地接口一样。

```javascript
package com.xcbeyond.springboot.grpc.client.service;



import com.xcbeyond.springboot.grpc.lib.HelloReply;

import com.xcbeyond.springboot.grpc.lib.HelloRequest;

import com.xcbeyond.springboot.grpc.lib.SimpleGrpc.SimpleBlockingStub;

import io.grpc.StatusRuntimeException;

import net.devh.boot.grpc.client.inject.GrpcClient;

import org.springframework.stereotype.Service;



/**

 * @Auther: xcbeyond

 * @Date: 2019/3/7 09:10

 */

@Service

public class GrpcClientService {



    @GrpcClient("spring-boot-grpc-server")

    private SimpleBlockingStub simpleBlockingStub;



    public String sendMessage(String name) {

        try {

            HelloReply response = simpleBlockingStub.sayHello(HelloRequest.newBuilder().setName(name).build());

            return response.getMessage();

        } catch (final StatusRuntimeException e) {

            return "FAILED with " + e.getStatus().getCode();

        }

    }

}
```

复制

application.yml配置：

spring-boot-grpc-server，即：服务端应用名，结合spring cloud Eureka[注册中心](https://cloud.tencent.com/product/tse?from=10680)，通过服务名将会找到服务端的ip，进行通信，实际上是netty通信。

> grpc: client: spring-boot-grpc-server: enableKeepAlive: true keepAliveWithoutCalls: true negotiationType: plaintext

文章分享自微信公众号：

![](media/code-3.jpg)

程序猿技术大咖

复制公众号名称

本文参与 [腾讯云自媒体分享计划](https://cloud.tencent.com/developer/support-plan) ，欢迎热爱写作的你一起参与！

作者：xcbey0nd

原始发表时间：2019-03-16

如有侵权，请联系 cloudcommunity@tencent.com 删除。

[RPC](https://cloud.tencent.com/developer/tag/10759?entry=article)[Spring](https://cloud.tencent.com/developer/tag/10323?entry=article)[Spring Cloud](https://cloud.tencent.com/developer/tag/10773?entry=article)[Maven](https://cloud.tencent.com/developer/tag/10300?entry=article)[开源](https://cloud.tencent.com/developer/tag/10667?entry=article)

举报

点赞 4分享

登录 后参与评论

_0_ 条评论

### 相关文章

-   [](https://cloud.tencent.com/developer/article/1439021?from=article.detail.1608265&areaSource=106000.1&traceId=s7A0m-r9k7hkEe3QS-j6q)
    
    ### 机器学习在SAP Cloud for Customer中的应用
    
    关于机器学习这个话题，我相信我这个公众号1500多位关注者里，一定有很多朋友的水平比Jerry高得多。如果您看过我以前两篇文章，您就会发现，我对机器学习仅仅停留...
    
    [Jerry Wang](https://cloud.tencent.com/developer/user/1306491)
    
-   [](https://cloud.tencent.com/developer/article/2027047?from=article.detail.1608265&areaSource=106000.2&traceId=s7A0m-r9k7hkEe3QS-j6q)
    
    ### 剖析 SPI 在 Spring 中的应用
    
    SPI（Service Provider Interface），是Java内置的一种服务提供发现机制，可以用来提高框架的扩展性，主要用于框架的开发中，比如Dub...
    
    [2020labs小助手](https://cloud.tencent.com/developer/user/4821640)
    
-   [](https://cloud.tencent.com/developer/article/1441340?from=article.detail.1608265&areaSource=106000.3&traceId=s7A0m-r9k7hkEe3QS-j6q)
    
    ### Freemarker在spring boot中的应用
    
    Hello，又到周一，搜狗测试小编华安又和大家见面啦。今天我们聊一聊java的模板引擎之一-----Freemarker！Let's go!
    
    [用户5521279](https://cloud.tencent.com/developer/user/5521279)
    
-   [](https://cloud.tencent.com/developer/article/1008538?from=article.detail.1608265&areaSource=106000.4&traceId=s7A0m-r9k7hkEe3QS-j6q)
    
    ### Redis在Spring Boot中的应用
    
    最近项目中用到Redis，上网查了很多示例，发现或多或少都有问题。踩过很多坑，终于在Spring Boot中成功实现了Redis存储。记录如下，方便别人，也方便...
    
    [YGingko](https://cloud.tencent.com/developer/user/1131318)
    
-   [](https://cloud.tencent.com/developer/article/1331798?from=article.detail.1608265&areaSource=106000.5&traceId=s7A0m-r9k7hkEe3QS-j6q)
    
    ### 机器学习在SAP Cloud for Customer中的应用
    
    关于机器学习这个话题，我相信我这个公众号1500多位关注者里，一定有很多朋友的水平比Jerry高得多。如果您看过我以前两篇文章，您就会发现，我对机器学习仅仅停留...
    
    [Jerry Wang](https://cloud.tencent.com/developer/user/1306491)
    
-   [](https://cloud.tencent.com/developer/article/1821250?from=article.detail.1608265&areaSource=106000.6&traceId=s7A0m-r9k7hkEe3QS-j6q)
    
    ### gRPC在C#中的未来属于grpc-dotnet
    
    grpc-dotnet（Grpc.Net.Client[1]和Grpc.AspNetCore.Server[2] nuget 包）现在是.NET/C#中推荐的 ...
    
    [CNCF](https://cloud.tencent.com/developer/user/6781541)
    
-   [](https://cloud.tencent.com/developer/article/1532912?from=article.detail.1608265&areaSource=106000.7&traceId=s7A0m-r9k7hkEe3QS-j6q)
    
    ### 详解设计模式在Spring中的应用
    
    http://itxxz.com/a/javashili/tuozhan/2014/0601/7.html
    
    [java思维导图](https://cloud.tencent.com/developer/user/1203634)
    
-   [](https://cloud.tencent.com/developer/article/1397269?from=article.detail.1608265&areaSource=106000.8&traceId=s7A0m-r9k7hkEe3QS-j6q)
    
    ### Swagger UI 在Spring boot中的应用
    
    Swagger UI是一个自动生成Java web接口文档的库。Swagger UI可以帮助前端开发者和后端开发者方便地进行沟通，后端开发者可以因此节省很多写接...
    
    [leaforbook](https://cloud.tencent.com/developer/user/1759025)
    
-   [](https://cloud.tencent.com/developer/article/1758188?from=article.detail.1608265&areaSource=106000.9&traceId=s7A0m-r9k7hkEe3QS-j6q)
    
    ### 赠书：Kotlin在Spring Boot中的应用
    
    随着Kotlin在移动端开发的普及，它也逐步走入后端开发者的视野。Kotlin是JVM体系的语言，和Java有着良好的互操作性，上手较容易，且可以使用Java强...
    
    [程序猿DD](https://cloud.tencent.com/developer/user/1258501)
    
-   [](https://cloud.tencent.com/developer/article/1072851?from=article.detail.1608265&areaSource=106000.10&traceId=s7A0m-r9k7hkEe3QS-j6q)
    
    ### Spring Security 入门（五）：在 Spring-Boot中的应用
    
    ? 前言 本文作为入门级的DEMO，完全按照官网实例演示； 项目目录结构 ? Maven 依赖 <parent> <groupId>org.sprin...
    
    [程序猿DD](https://cloud.tencent.com/developer/user/1258501)
    
-   [](https://cloud.tencent.com/developer/article/1196125?from=article.detail.1608265&areaSource=106000.11&traceId=s7A0m-r9k7hkEe3QS-j6q)
    
    ### 在静态方法中应用spring注入的类
    
    最近在一次项目的重构中，原项目需要在静态方法中调用service，现在需要更换框架，service需要自动注入，无法再静态方法中调用
    
    [二十三年蝉](https://cloud.tencent.com/developer/user/1222012)
    
-   [](https://cloud.tencent.com/developer/article/1414276?from=article.detail.1608265&areaSource=106000.12&traceId=s7A0m-r9k7hkEe3QS-j6q)
    
    ### Spring Cloud在Netflix后时代的走向？
    
    如果有人问你关于Spring Cloud的问题，那么你首先想到的可能是Netflix OSS的支持。对Eureka，Zuul或Ribbon等工具的支持不仅由Sp...
    
    [本人秃顶程序员](https://cloud.tencent.com/developer/user/5198734)
    
-   [](https://cloud.tencent.com/developer/article/1081590?from=article.detail.1608265&areaSource=106000.13&traceId=s7A0m-r9k7hkEe3QS-j6q)
    
    ### Spring Cloud中Hystrix的请求合并
    
    在微服务架构中，我们将一个项目拆分成很多个独立的模块，这些独立的模块通过远程调用来互相配合工作，但是，在高并发情况下，通信次数的增加会导致总的通信时间增加，同时...
    
    [江南一点雨](https://cloud.tencent.com/developer/user/1516773)
    
-   [](https://cloud.tencent.com/developer/article/1081593?from=article.detail.1608265&areaSource=106000.14&traceId=s7A0m-r9k7hkEe3QS-j6q)
    
    ### Spring Cloud中Hystrix的请求缓存
    
    高并发环境下如果能处理好缓存就可以有效的减小服务器的压力，Java中有许多非常好用的缓存工具，比如Redis、EHCache等，当然在Spring Cloud的...
    
    [江南一点雨](https://cloud.tencent.com/developer/user/1516773)
    
-   [](https://cloud.tencent.com/developer/article/1081532?from=article.detail.1608265&areaSource=106000.15&traceId=s7A0m-r9k7hkEe3QS-j6q)
    
    ### Spring Cloud中的负载均衡策略
    
    在上篇博客(Spring Cloud中负载均衡器概览)中，我们大致的了解了一下Spring Cloud中有哪些负载均衡器，但是对于负载均衡策略我们并没有去详细了...
    
    [江南一点雨](https://cloud.tencent.com/developer/user/1516773)
    
-   [](https://cloud.tencent.com/developer/article/1081575?from=article.detail.1608265&areaSource=106000.16&traceId=s7A0m-r9k7hkEe3QS-j6q)
    
    ### Spring Cloud中Feign的继承特性
    
    上篇文章我们了解了Feign的基本使用，在HelloService类中声明接口时，我们发现这里的代码可以直接从服务提供者的Controller中复制过来，这些可...
    
    [江南一点雨](https://cloud.tencent.com/developer/user/1516773)
    
-   [](https://cloud.tencent.com/developer/article/1380099?from=article.detail.1608265&areaSource=106000.17&traceId=s7A0m-r9k7hkEe3QS-j6q)
    
    ### Hystrix断路器在微服务网关中的应用（Spring Cloud Gateway）
    
    在之前的一篇文章：微服务网关Zuul迁移到Spring Cloud Gateway，我们讲解了如何从Zuul迁移到新的组件：Spring Cloud Gatew...
    
    [aoho求索](https://cloud.tencent.com/developer/user/1446357)
    
-   [](https://cloud.tencent.com/developer/article/2051099?from=article.detail.1608265&areaSource=106000.18&traceId=s7A0m-r9k7hkEe3QS-j6q)
    
    ### RabbitMQ入门：在Spring Boot 应用中整合RabbitMQ
    
    在上一篇随笔中我们认识并安装了RabbitMQ，接下来我们来看下怎么在Spring Boot 应用中整合RabbitMQ。
    
    [全栈程序员站长](https://cloud.tencent.com/developer/user/8223537)
    
-   [](https://cloud.tencent.com/developer/article/1989418?from=article.detail.1608265&areaSource=106000.19&traceId=s7A0m-r9k7hkEe3QS-j6q)
    
    ### Spring Cloud应用的优雅下线与灰度发布
    
    前言 在生产环境中，如何保证在服务升级的时候，不影响用户的体验，这个是一个非常重要的问题。如果在我们升级服务的时候，会造成一段时间内的服务不可用，这就是不够优雅...
    
    [程序猿DD](https://cloud.tencent.com/developer/user/1258501)
    

[更多文章](https://cloud.tencent.com/developer/column)

### 作者介绍

[](https://cloud.tencent.com/developer/user/3106400)

### [xcbeyond](https://cloud.tencent.com/developer/user/3106400 "xcbeyond")

架构师

关注[专栏](https://cloud.tencent.com/developer/column/84515)

-   [
    
    文章
    
    232
    
    ](https://cloud.tencent.com/developer/user/3106400)
-   [
    
    阅读量
    
    164.2K
    
    ](https://cloud.tencent.com/developer/user/3106400)
-   [
    
    获赞
    
    636
    
    ](https://cloud.tencent.com/developer/user/3106400)
-   [
    
    作者排名
    
    110
    
    ](https://cloud.tencent.com/developer/rank)

### 精选专题

-   [
    
    ### 腾讯云原生专题
    
    云原生技术干货，业务实践落地。
    
    
    
    ](https://cloud.tencent.com/developer/special/TencentCloudNative)

### 活动推荐

-   [
    
    ### 社区创作者年终回顾
    
    参与活动，赢取限量周边礼品
    
    立即参加
    
    ](https://cloud.tencent.com/developer/article/2210043)
-   [
    
    ### 邀请好友加入自媒体分享计划
    
    邀请好友，同享奖励 30 / 100 / 180 元云服务器代金券
    
    立即邀请
    
    ](https://cloud.tencent.com/developer/support-plan-invitation)

### 运营活动

[![活动名称](media/活动名称-1.jpg)](https://cloud.tencent.com/act/pro/2022double11_video?from=16978)

广告关闭

## 关注

腾讯云_开发者_公众号

将获得

10元无门槛代金券

洞察腾讯核心技术

剖析业界实践案例

![腾讯云开发者公众号二维码](media/腾讯云开发者公众号二维码.png)

-   ### 社区
    
    -   [专栏文章](https://cloud.tencent.com/developer/column)
    -   [阅读清单](https://cloud.tencent.com/developer/inventory)
    -   [互动问答](https://cloud.tencent.com/developer/ask)
    -   [技术沙龙](https://cloud.tencent.com/developer/salon)
    -   [技术视频](https://cloud.tencent.com/developer/video)
    -   [团队主页](https://cloud.tencent.com/developer/teams)
    -   [腾讯云TI平台](https://cloud.tencent.com/developer/timl)
    
-   ### 活动
    
    -   [自媒体分享计划](https://cloud.tencent.com/developer/support-plan)
    -   [邀请作者入驻](https://cloud.tencent.com/developer/support-plan-invitation)
    -   [自荐上首页](https://cloud.tencent.com/developer/article/1535830)
    -   [技术竞赛](https://cloud.tencent.com/developer/competition)
    
-   ### 资源
    
    -   [技术周刊](https://cloud.tencent.com/developer/specials)
    -   [社区标签](https://cloud.tencent.com/developer/tags)
    -   [开发者手册](https://cloud.tencent.com/developer/devdocs)
    -   [开发者实验室](https://cloud.tencent.com/developer/labs)
    
-   ### 关于
    
    -   视频介绍
    -   [社区规范](https://cloud.tencent.com/developer/article/1006434)
    -   [免责声明](https://cloud.tencent.com/developer/article/1006435)
    -   [联系我们](mailto:cloudcommunity@tencent.com)
    -   [友情链接](https://cloud.tencent.com/developer/friendlink)
    
-   -   [](https://cloud.tencent.com/developer/ask/archives.html)
    -   [](https://cloud.tencent.com/developer/column/archives.html)
    -   [](https://cloud.tencent.com/developer/news/archives.html)
    -   [](https://cloud.tencent.com/developer/information/all.html)
    -   [](https://cloud.tencent.com/developer/devdocs/archives.html)
    -   [](https://cloud.tencent.com/developer/devdocs/sections_p1.html)
    

### 腾讯云开发者

![](media/a8907230cd5be483497c7e90b061b861.png)

扫码关注腾讯云开发者

领取腾讯云代金券

### 热门产品

-   [域名注册](https://dnspod.cloud.tencent.com/)
-   [云服务器](https://cloud.tencent.com/product/cvm)
-   [区块链服务](https://cloud.tencent.com/product/tbaas)
-   [消息队列](https://cloud.tencent.com/product/mq)
-   [网络加速](https://cloud.tencent.com/product/dsa)
-   [云数据库](https://cloud.tencent.com/product/tencentdb-catalog)
-   [域名解析](https://cloud.tencent.com/product/cns)
-   [云存储](https://cloud.tencent.com/product/cos)
-   [视频直播](https://cloud.tencent.com/product/css)

### 热门推荐

-   [人脸识别](https://cloud.tencent.com/product/facerecognition)
-   [腾讯会议](https://cloud.tencent.com/product/tm)
-   [企业云](https://cloud.tencent.com/act/pro/enterprise2019)
-   [CDN 加速](https://cloud.tencent.com/product/cdn-scd)
-   [视频通话](https://cloud.tencent.com/product/trtc)
-   [图像分析](https://cloud.tencent.com/product/tiia)
-   [MySQL 数据库](https://cloud.tencent.com/product/cdb)
-   [SSL 证书](https://cloud.tencent.com/product/symantecssl)
-   [语音识别](https://cloud.tencent.com/product/asr)

### 更多推荐

-   [数据安全](https://cloud.tencent.com/solution/data_protection)
-   [负载均衡](https://cloud.tencent.com/product/clb)
-   [短信](https://cloud.tencent.com/product/sms)
-   [文字识别](https://cloud.tencent.com/product/ocr)
-   [云点播](https://cloud.tencent.com/product/vod)
-   [商标注册](https://tm.cloud.tencent.com/)
-   [小程序开发](https://cloud.tencent.com/solution/la)
-   [网站监控](https://cloud.tencent.com/product/cat)
-   [数据迁移](https://cloud.tencent.com/product/cdm)

Copyright © 2013 - 2023 Tencent Cloud. All Rights Reserved. 腾讯云 版权所有 [京公网安备 11010802017518](http://www.beian.gov.cn/portal/registerSystemInfo?recordcode=11010802020287) [粤B2-20090059-1](http://beian.miit.gov.cn/)

扫描二维码