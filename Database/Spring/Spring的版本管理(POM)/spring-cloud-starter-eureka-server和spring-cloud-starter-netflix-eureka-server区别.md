关于spring-cloud-starter-eureka-server和spring-cloud-starter-netflix-eureka-server的区别

现在在看项目源码，发现spring-boot项目有些jar包，或者配置文件有些写法不一样，今天恰巧看到了有的项目里关于eureka用的是spring-cloud-starter-eureka-server，有的项目用的是spring-cloud-starter-netflix-eureka-server。

在spring-boot的1.4的时候还是spring-cloud-starter-eureka-server，

在这之后不知道哪个版本就换成了spring-cloud-starter-netflix-eureka-server，

去maven的中央仓库看了看，地址：[https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-eureka-server](https://links.jianshu.com/go?to=https%3A%2F%2Fmvnrepository.com%2Fartifact%2Forg.springframework.cloud%2Fspring-cloud-starter-eureka-server)。仓库已经废弃了spring-cloud-starter-eureka-server，推荐使用spring-cloud-starter-netflix-eureka-server。

  

![](media/6006813-a30fd4b02539794d.png)

50人点赞

[微服务](/nb/39111018)

  
  
作者：苗義  
链接：https://www.jianshu.com/p/767274038972  
来源：简书  
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。