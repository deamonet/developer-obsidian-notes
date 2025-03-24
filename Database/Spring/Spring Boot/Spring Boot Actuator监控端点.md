目录

[TOC]

微服务的这种架构虽然解决了单体应用的一些劣势，但它也面临一些挑战，比如对运维的要求更高了。一个微服务架构中可能有几十个上百个应用构成，要保证这些应用都正常运行，相互协调是比较麻烦的事情，因此我们需要一个组件来对这些应用进行监控和管理。  
`spring-boot-starter-actuator` 就是Spring Boot提供这个功能的模块。

# [](https://hjwjw.github.io/posts/297fa226/#示例 "示例")示例

运行环境：

> Spring Boot 2.0.3.RELEASE

**1、引入依赖**  

<dependency>  
	<groupId>org.springframework.boot</groupId>  
	<artifactId>spring-boot-starter-actuator</artifactId>  
</dependency>  

**2、启动应用**  
重新启动应用访问 `http://localhost:8000/actuator/health` 会显示如下信息：  

{  
	status: "UP"  
}  

Actuator监控管理默认的访问路径是在 `/actuator` 下。在测试Spring Boot 1.5.9版本时是直接访问端点路径，不需要加 `/actuator`

# [](https://hjwjw.github.io/posts/297fa226/#配置 "配置")配置

除了health端点外，Actuator还为我们提供了很多端点，有些可以直接访问，有些需要授权或通过配置才能访问。

## [](https://hjwjw.github.io/posts/297fa226/#端点列表 "端点列表")端点列表

端点

描述

`actuator`

为其他端点提供基于超媒体的“发现页面”。要求Spring HATEOAS在类路径上

`auditevents`

公开当前应用程序的审核事件信息

`autoconfig`

显示自动配置报告，显示所有自动配置候选项以及它们“未被”应用的原因

`beans`

显示应用程序中所有Spring bean的完整列表

`configprops`

显示所有配置信息。

`dump`

打印线程栈

`env`

查看所以环境变量

`health`

显示应用程序运行状况信息

`info`

显示应用信息

`loggers`

显示和修改应用程序中记录器的配置

`liquibase`

显示已应用的任何Liquibase数据库迁移

`metrics`

显示当前应用程序的“指标”信息

`mappings`

显示所有@RequestMapping路径的整理列表

`shutdown`

允许应用程序正常关闭（默认情况下不启用）

`trace`

显示跟踪信息（默认情况下是最近的100个HTTP请求

## [](https://hjwjw.github.io/posts/297fa226/#Actuator配置 "Actuator配置")Actuator配置

**自定义默认路径**  

management.endpoints.web.base-path = /application  

> 修改后访问端点的默认路径不再是 `/actuator` 而是 `/application`

**自定义访问端口号**  

management.server.port= 8012  

> 修改后我们查看Actuator需要修改成8012端口进行访问，如`http://localhost:8012/actuator/health`

**关闭验证**  

management.security.enabled= false  

> 默认情况下只开放了`health` 与 `info` 端口，关闭验证后，其它的也可以访问了，但不安全，最好添加 `security` 验证

**控制端点是否开放**  

management.endpoints.web.exposure.include= 'info'  

> 表示只暴露`info`端口，如添加其它端口使用 `,` 分隔，暴露所有端口使用 `*`

**端口属性配置**  

management.endpoint.端口名.属性=值  

> 如 `management.endpoint.health.show-details= always` 表示显示 `health`端口的详细信息，大多数端口可以这样配置。

## [](https://hjwjw.github.io/posts/297fa226/#整合Spring-Security "整合Spring Security")整合Spring Security

监控端点的很多信息住信比较隐私，不能让没有权限的人随意查看，因此可以添加Srping Security进行控制。  
**添加依赖**  

<dependency>  
	<groupId>org.springframework.boot</groupId>  
	<artifactId>spring-boot-starter-security</artifactId>  
</dependency>  

**配置Security**  

spring:  
  security:  
    user:  
      name: user  
      password: 123  

> 配置security后，访问端口需要进行登陆验证。

# [](https://hjwjw.github.io/posts/297fa226/#总结 "总结")总结

另外还有其它的端口配置，还可以自定义端口等，这里不一一总结，以后需要时查查资料。贴上一个学习链接： [http://blog.didispace.com/spring-boot-actuator-1/](http://blog.didispace.com/spring-boot-actuator-1/)