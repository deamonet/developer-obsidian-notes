Make sure that you have `gson` version `2.6` or higher

```
compile group: 'com.google.code.gson', name: 'gson', version: '2.6'
```

You can use `gradle dependencies` command to check dependencies graph.

这好像是gradle的，然而我这是maven项目的报错。不过根据这个提示，似乎是因为gson版本太老导致的问题，所以手动在maven中添加新版本的gson依赖就可以解决

```
<!-- https://mvnrepository.com/artifact/com.google.code.gson/gson -->
<dependency>
    <groupId>com.google.code.gson</groupId>
    <artifactId>gson</artifactId>
    <version>2.10</version>
</dependency>

```

在SpringBoot 项目启动时报一下异常：

 org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'gsonBuilder' 

目测是 com.google.code.gson 版本太低造成的（1.7.2）