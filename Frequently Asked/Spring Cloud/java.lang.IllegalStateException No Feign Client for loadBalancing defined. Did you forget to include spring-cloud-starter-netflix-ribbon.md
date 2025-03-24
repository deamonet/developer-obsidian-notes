
spring cloud 2.2.8 release
spring boot 2.3.10 release
openfeign 


[](https://stackoverflow.com/posts/64222260/timeline)

You have to decide, which client load balancer to use: (1)**Spring Cloud Loadbalancer** or (2)**Ribbon**.

> Spring Cloud Loadbalancer is a generic abstraction that can do the work that we used to do with Netflix’s Ribbon project. Spring Cloud still supports Netflix Ribbon, but Netflix Ribbons days are numbered, like so much else of the Netflix microservices stack, so we’ve provided an abstraction to support an alternative

Check here: [https://spring.io/blog/2020/03/25/spring-tips-spring-cloud-loadbalancer](https://spring.io/blog/2020/03/25/spring-tips-spring-cloud-loadbalancer)

(1) **Spring Cloud Load Balancer**:

```java
spring:
  cloud:
    loadbalancer:
       ribbon:
        enable: false

# And... inform the "url" attribute at FeignClient
@FeignClient(name = "student", url = "student") 
```

(2) **Ribbon**: Add the dependency:

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
</dependency>

# And (optionally)... @application.yaml

   spring:
      cloud:
        loadbalancer:
           ribbon:
            enable: true
```