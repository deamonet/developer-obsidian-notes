***************************
APPLICATION FAILED TO START
***************************

Description:

Spring MVC found on classpath, which is incompatible with Spring Cloud Gateway.

Action:

Please set spring.main.web-application-type=reactive or remove spring-boot-starter-web dependency.

Disconnected from the target VM, address: '127.0.0.1:3012', transport: 'socket'

Process finished with exit code 1

spring cloud gateway 项目不能添加 spring-boot-starter-web 依赖。所以。