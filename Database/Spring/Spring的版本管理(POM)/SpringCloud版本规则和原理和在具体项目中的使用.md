# [SpringCloud版本规则和原理和在具体项目中的使用](https://www.cnblogs.com/hzhuxin/p/12393456.html)

　　伴随着微服务框架的流行，SpringCloud项目也变得越来越庞大，由于它是由诸多子项目组成，为了更好管理SpringCloud的发布版本与各个子项目的版本关系，SpringCloud采用了一种新的版本规则来控制整个SpringCloud的版本发布。先来了解下SpringCloud的几个相关网址：

　　项目主页：[https://spring.io/projects/spring-cloud](https://spring.io/projects/spring-cloud)   

　　各个子项目的github地址： [https://github.com/spring-cloud](https://github.com/spring-cloud)

       在子项目中有一个spring-cloud-release子项目，项目地址 [https://github.com/spring-cloud/spring-cloud-release](https://github.com/spring-cloud/spring-cloud-release) ，该项目的作用就是管理SpringCloud版本的发布，主要做了二件事，一个是分布版本的规则，二是管理各个发布版本的子项目的版本的映射关系。下面就两方面进行分析。

**一.发布版本的命名规则**

发布版本不采用通常的版本号的方式，而是采用英国伦敦的地铁站名的站名（按字母顺序）来命名发布的版本名称。不过这个版本名称后面还会以.xxx 的形式加一个额外的版本说明，

这个说明通常是以下形式：

SNAPSHOT： 快照版本，随时可能修改

M： MileStone，M1表示第1个里程碑版本，一般同时标注PRE，表示预览版版。

SR： Service Release，SR1表示第1个正式版本，一般同时标注GA：(GenerallyAvailable),表示稳定版本。

例如当一个版本的Spring Cloud项目的发布内容积累到临界点或者解决了一个严重bug后，就会发布一个“service releases”版本，简称SRX版本，其中X是一个递增数字。这个SRX会以 .形式跟在版本名称的后面。

按如下规则，当前最新的SpringCloud发布版本就是 Hoxton.SR1

**二. 子项目版本的映射管理**

Spring Cloud项目通过 [spring-cloud-dependencies](https://github.com/spring-cloud/spring-cloud-release/tree/master/spring-cloud-dependencies "spring-cloud-dependencies") 子项目的pom文件来管理不同的SpringCloud发布版本下的各个SpringCloud子项目的版本。以下是某个版本的spring-cloud-dependencies项目的pom文件的部分内容：

<dependencies>
            <!-- bom dependencies at the bottom so they can be overridden above -->
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-commons-dependencies</artifactId>
                <version>${spring-cloud-commons.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-netflix-dependencies</artifactId>
                <version>${spring-cloud-netflix.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-stream-dependencies</artifactId>
                <version>${spring-cloud-stream.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            ............  

![复制代码](media/复制代码.gif)

可以看到在pom文件中引入了许多spring-cloud-xxx-dependencies这样的管理子项目的依赖的项目，我们随便打开其中一个，例如 spring-cloud-netflix-dependencies的pom文件的片断来看下：

![复制代码](media/复制代码.gif)

<properties>
        <archaius.version>0.7.6</archaius.version>
        <concurrency-limits.version>0.1.6</concurrency-limits.version>
        <eureka.version>1.9.8</eureka.version>
        <hystrix.version>1.5.18</hystrix.version>
        <ribbon.version>2.3.0</ribbon.version>
        <servo.version>0.12.21</servo.version>
        <zuul.version>1.3.1</zuul.version>
        <rxjava.version>1.2.0</rxjava.version>
        <rxjava-reactive-streams.version>1.2.1</rxjava-reactive-streams.version>
        <turbine.version>1.0.0</turbine.version>
        <eureka-jersey.version>1.19.1</eureka-jersey.version>
        <xstream.version>1.4.10</xstream.version>
        <okhttp3.version>3.8.1</okhttp3.version>
    </properties>
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-netflix-eureka-client</artifactId>
                <version>${project.version}</version>
            </dependency>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-netflix-archaius</artifactId>
                <version>${project.version}</version>
            </dependency>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-starter-netflix-archaius</artifactId>
                <version>${project.version}</version>
            </dependency>

![复制代码](media/复制代码.gif)

可以看到里面出现了各个子项目真正依赖的一些具体jar的版本。 当然这些jar都是以可选列表的方式提供出来的，也就是通过<dependencyManagement/>提供出来的。并不会真正被项目引入。下面通过一个具体的springCloud注册中心的案例来看下，在具体项目pom文件中如何使用，有两种方式：

**不继承父项目的方式**

![复制代码](media/复制代码.gif)

<properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Greenwich.RELEASE</spring-cloud.version>
    </properties>

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

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>
    </dependencies>

![复制代码](media/复制代码.gif)

**继承父项目的方式**

![复制代码](media/复制代码.gif)

<parent> 
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-parent</artifactId>
        <version>${spring-cloud.version}</version>
    </parent>
  <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Greenwich.RELEASE</spring-cloud.version>
    </properties>

   

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>
    </dependencies>

![复制代码](media/复制代码.gif)

总结一下，分三个步骤：

1. 在 <properties/>中定义springCloud的版本常量

2. 以bom机制引入spring-cloud-dependencies，其版本由步骤1指定

3. 在 <dependencies>依赖列表中引入具体要用到的springCloud的组件依赖，即子项目，此时无需关注版本了

好文要顶 关注我 收藏该文 ![](media/icon_weibo_24.png) ![](media/wechat.png)

[![](media/sample_face.gif)](https://home.cnblogs.com/u/hzhuxin/)

[杭州胡欣](https://home.cnblogs.com/u/hzhuxin/)  
[粉丝 - 114](https://home.cnblogs.com/u/hzhuxin/followers/) [关注 - 3](https://home.cnblogs.com/u/hzhuxin/followees/)  

+加关注

0

0

[«](https://www.cnblogs.com/hzhuxin/p/12371043.html) 上一篇： [为什么要使用Mycat](https://www.cnblogs.com/hzhuxin/p/12371043.html "发布于 2020-02-27 11:13")  
[»](https://www.cnblogs.com/hzhuxin/p/14373803.html) 下一篇： [why spring?](https://www.cnblogs.com/hzhuxin/p/14373803.html "发布于 2021-02-04 17:11")

posted @ 2020-03-02 03:02  [杭州胡欣](https://www.cnblogs.com/hzhuxin/)  阅读(1350)  评论(0)  [编辑](https://i.cnblogs.com/EditPosts.aspx?postid=12393456)  收藏  举报