不太懂什么意思，大致上的理解，devtools跟不能用于生产环境。


开发者工具，可以在开发SpringBoot的时候，自动的实现实时开发特性。使用过程需要引入如下的依赖：

对于maven来说：

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <optional>true</optional>
    </dependency>
</dependencies>

对于Gradle来说：

dependencies {
    developmentOnly("org.springframework.boot:spring-boot-devtools")
}

当进行全量打包（fully packaged）的时候， Developer Tools会自动的被禁用。比如，当你用 java -jar 或者制定了类加载器的时候，则会被当成是 生产应用 （production application）。生产环境上，通常不允许启用devTools工具，因为这是一个严重的安全问题。如果想启用 Devloper Tools，需要配置 spring.devtools.restart.enabled 系统属性为 true。