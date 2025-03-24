原因

IDEA会自动扫描代码的上下文, 如果类前有 Component, Controller , Service ,Repository 之中任意注解的类, 自动注册到Spring的Bean管理库中. IDEA会智能的提示.

但是, 在 mapper 中, 没有用到上面的那些注解. 使用 @Mapper 注解. 这个注解的作用, 源码中注释没有写, 比较遗憾.

![To Do](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABAAAAAQCAYAAAAf8/9hAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsMAAA7DAcdvqGQAAAAySURBVDhPYxQqm/WfgQIANuBFSwKUSxqQqFnAwARlkw1GDRg1AARGDYDmRiibDMDAAACSDAmB2sfKTAAAAABJRU5ErkJggg==) 但是可以猜测是和 xml 中的配置有相同的作用.

在Spring-MVC中使用一下配置, 来声明一个映射器:

<bean id="userMapper" class="org.mybatis.spring.mapper.MapperFactoryBean">

  <property name="mapperInterface" value="org.mybatis.spring.sample.mapper.UserMapper" />

  <property name="sqlSessionFactory" ref="sqlSessionFactory" />

</bean>

    MapperFactoryBean 创建的代理类实现了 UserMapper 接口,并且注入到应用程序中。 因为代理创建在运行时环境中(Runtime,译者注) ,那么指定的映射器必须是一个接口,而 不是一个具体的实现类。

mybatis 的 xml 配置解释参考官网, 没必要贴出来

spring-mybatis官网

SpringBoot-mybatis官网

因为 IDEA 不认为 @Mapper 注解是用来声明一个Bean, 所以就会提示无法找到Bean.

解决

方式一: 降低 IDEA 错误提示的等级

因为这是 IDEA 自动提示

在这里插入图片描述

方式二: 在映射器接口上添加@Component注解

当然, 添加 Component, Controller , Service ,Repository 四个注解中的任意一个都可以解决. 功能是相同的. 但是为了语意明确, 应该使用 Component ,Repository 中的一个.

方式三: 使用构造器

Spring5中autowired注解官方文档

用谷歌翻译一下:

在这里插入图片描述

也就是说, 当类中, 只有一个构造器的时候, Spring会自动将通过构造器的方式注入Bean.

生成构造器也有好几种方式

    利用编辑器自动生成

    用 lombok 的 @AllArgsConstructor 注解

注意: 用构造器的方式注入Bean, 要防止循环依赖