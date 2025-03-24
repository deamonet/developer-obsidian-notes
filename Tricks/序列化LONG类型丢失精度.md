# 修复Long类型太长，而Java序列化JSON丢失精度问题的方法

2018-03-16 4149

版权

**简介：** Java序列化JSON时long型数值,会出现精度丢失的问题。 原因： java中得long能表示的范围比js中number大,也就意味着部分数值在js中存不下(变成不准确的值). 解决办法一： 使用ToStringSerializer的注解，让系统序列化 时，保留相关精度 @JsonSerialize(using=ToStringSerializer.class) private Long createdBy; 上述方法需要在每个对象都配上该注解，此方法过于繁锁。

Java序列化JSON时long型数值,会出现精度丢失的问题。  
原因：  
java中得long能表示的范围比js中number大,也就意味着部分数值在js中存不下(变成不准确的值).  
解决办法一：  
使用ToStringSerializer的注解，让系统序列化  
时，保留相关精度

```
    @JsonSerialize(using=ToStringSerializer.class)
    private Long createdBy;
```

上述方法需要在每个对象都配上该注解，此方法过于繁锁。

解决办法(二）：  
使用全局配置，将转换时实现自动ToStringSerializer序列化

```
Override
public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
    MappingJackson2HttpMessageConverter jackson2HttpMessageConverter = new MappingJackson2HttpMessageConverter();

    ObjectMapper objectMapper = new ObjectMapper();
    /**
     * 序列换成json时,将所有的long变成string
     * 因为js中得数字类型不能包含所有的java long值
     */
    SimpleModule simpleModule = new SimpleModule();
    simpleModule.addSerializer(Long.class, ToStringSerializer.instance);
    simpleModule.addSerializer(Long.TYPE, ToStringSerializer.instance);
    objectMapper.registerModule(simpleModule);

    jackson2HttpMessageConverter.setObjectMapper(objectMapper);
    converters.add(jackson2HttpMessageConverter);
}

```

方法二比较完美，强烈推荐使用！