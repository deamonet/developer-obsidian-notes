[乾坤刀](https://blog.51cto.com/881206524)2018-05-28 11:20:38博主文章分类：[spring/cloud](https://blog.51cto.com/881206524/category13)©著作权

**_文章标签_[spring](https://blog.51cto.com/topic/spring-1.html)[boot](https://blog.51cto.com/topic/boot.html)****_文章分类_[软件架构](https://blog.51cto.com/nav/software-architecture)[软件研发](https://blog.51cto.com/nav/software)****_阅读数_**1.5万****

spring boot 虽然不强制使用特殊的依赖。但是其提供了一些非常高效的依赖。其中最有如下几个：

-   spring-boot-starter-parent

```html
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.0.0.RELEASE</version>
		<relativePath/>
	</parent>
```

spring-boot-starter-parent这是一个依赖管理器的pom文件。它的作用就是管理boot需要的所有依赖，从而统一各种jar的版本号，避免了版本不一致而出现的问题。所以，引入其他的依赖就可以省略版本号。当然也可以加上指定的版本号，从而取代默认的。

-   spring-cloud-dependencies

```html
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
```

spring-cloud-dependencies也是一个依赖管理器的pom文件，与spring-boot-starter-parent的作用一样，不同的是spring-cloud-dependencies是对cloud的依赖管理。如：spring-cloud-starter-config、spring-cloud-starter-netflix-eureka-server

-   spring-boot-starter-web

```html
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

spring-boot-starter-web会自动嵌入tomcat容器。同时，springboot也会根据classpath中的dependency来自动配置。比如：spring-boot-starter-web会自动装配tomcat容器；并且会自动从application.properties中读取web应用的配置，如：server.port；如果application.properties没有配置相关的参数，则采用默认的配置信息，如：8080。

-   spring-boot-starter-data-jpa数据库连接的依赖。

```html
<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

spring-boot-starter-data-jpa数据库连接的依赖。

-   spring-cloud-config-server

```html
<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-config-server</artifactId>
</dependency>
```

spring-cloud-config-server配置中心；

-   spring-cloud-starter-netflix-eureka-server

```html
 <dependency>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

spring-cloud-starter-netflix-eureka-server注册中心。是spring cloud的核心架构。

说明：spring boot提供的一系列spring-boot-starter-*和spring-cloud-starter-*依赖，其实是相关功能依赖的整合，即引入一个start依赖，就引入多个相应的jar。同时需要注意的是，spring boot提供的starter都是spring-boot-starter-*和spring-cloud-starter-_这样开头的，如果想自定义starter,则命名格式应该是：_-spring-boot-starter.