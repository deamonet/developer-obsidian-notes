注意到这个东西是跟jackson相关的，于是在pom中添加jackson依赖
```
<!-- https://mvnrepository.com/artifact/com.fasterxml.jackson.core/jackson-databind -->  
<dependency>  
    <groupId>com.fasterxml.jackson.core</groupId>  
    <artifactId>jackson-databind</artifactId>  
    <version>2.14.0-rc3</version>  
</dependency>
```

然后报错：Error creating bean with name 'gsonBuilder' defined in class path resource