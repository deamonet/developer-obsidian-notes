这应该都是些兼容性的问题，注意Spring Cloud官网对于 cloud 和 boot 的兼容性表格

[Spring Cloud](https://spring.io/projects/spring-cloud)

Table 1. Release train Spring Boot compatibility

| Release Train        | Boot Version                          |
|----------------------|---------------------------------------|
| 2021.0.x aka Jubilee | 2.6.x, 2.7.x (Starting with 2021.0.3) |
| 2020.0.x aka Ilford  | 2.4.x, 2.5.x (Starting with 2020.0.3) |
| Hoxton               | 2.2.x, 2.3.x (Starting with SR5)      |
| Greenwich            | 2.1.x                                 |
| Finchley             | 2.0.x                                 |
| Edgware              | 1.5.x                                 |
| Dalston              | 1.5.x                                 |


完美解决Error creating bean with name ‘configurationPropertiesBeans‘ defined in class path resource

相知于花海

于 2021-03-11 17:43:46 发布

30065
 收藏 10
分类专栏： Nacos 版本问题 loadbalancer 文章标签： spring boot elasticsearch maven
版权

Nacos
同时被 3 个专栏收录
1 篇文章0 订阅
订阅专栏

版本问题
1 篇文章0 订阅
订阅专栏

loadbalancer
1 篇文章0 订阅
订阅专栏
文章目录
项目场景
问题描述
原因分析
解决方案
项目场景
SpringBoot启动报错，大致报错信息如下：

2021-03-11 12:38:15.512 ERROR 16912 --- [           main] o.s.boot.SpringApplication               : Application run failed

org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'configurationPropertiesBeans' defined in class path resource [org/springframework/cloud/autoconfigure/ConfigurationPropertiesRebinderAutoConfiguration.class]: Post-processing of merged bean definition failed; nested exception is java.lang.IllegalStateException: Failed to introspect Class [org.springframework.cloud.context.properties.ConfigurationPropertiesBeans] from ClassLoader [sun.misc.Launcher$AppClassLoader@18b4aac2]
1
2
3
先把项目中主要依赖的版本梳理一遍，这些版本都是这个时间段（2021年3月上旬）的最新版本：

依赖	版本
Spring Boot	2.4.3
Spring Cloud	2020.0.1
com.alibaba.cloud » spring-cloud-starter-alibaba-nacos-discovery	2.2.5.RELEASE
com.alibaba.cloud » spring-cloud-starter-alibaba-nacos-config	2.2.5.RELEASE
org.springframework.cloud » spring-cloud-starter-loadbalancer	3.0.1
org.elasticsearch.client » elasticsearch-rest-high-level-client	7.11.1
其中nacos discovery去除了org.springframework.cloud » spring-cloud-starter-netflix-ribbon这个依赖，这又是另一个问题：

问题描述
今天在项目的pom.xml中添加elasticsearch-rest-high-level-client依赖之后，SpringBoot启动就报错了，引入依赖之前项目启动是正常的，详细报错信息如下：

2021-03-11 12:38:15.512 ERROR 16912 --- [           main] o.s.boot.SpringApplication               : Application run failed

org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'configurationPropertiesBeans' defined in class path resource [org/springframework/cloud/autoconfigure/ConfigurationPropertiesRebinderAutoConfiguration.class]: Post-processing of merged bean definition failed; nested exception is java.lang.IllegalStateException: Failed to introspect Class [org.springframework.cloud.context.properties.ConfigurationPropertiesBeans] from ClassLoader [sun.misc.Launcher$AppClassLoader@18b4aac2]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:579) ~[spring-beans-5.3.4.jar:5.3.4]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:524) ~[spring-beans-5.3.4.jar:5.3.4]
	at org.springframework.beans.factory.support.AbstractBeanFactory.lambda$doGetBean$0(AbstractBeanFactory.java:335) ~[spring-beans-5.3.4.jar:5.3.4]
	at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:234) ~[spring-beans-5.3.4.jar:5.3.4]
	at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:333) ~[spring-beans-5.3.4.jar:5.3.4]
	at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:213) ~[spring-beans-5.3.4.jar:5.3.4]
	at org.springframework.context.support.PostProcessorRegistrationDelegate.registerBeanPostProcessors(PostProcessorRegistrationDelegate.java:270) ~[spring-context-5.3.4.jar:5.3.4]
	at org.springframework.context.support.AbstractApplicationContext.registerBeanPostProcessors(AbstractApplicationContext.java:761) ~[spring-context-5.3.4.jar:5.3.4]
	at org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:566) ~[spring-context-5.3.4.jar:5.3.4]
	at org.springframework.boot.SpringApplication.refresh(SpringApplication.java:767) ~[spring-boot-2.4.3.jar:2.4.3]
	at org.springframework.boot.SpringApplication.refresh(SpringApplication.java:759) ~[spring-boot-2.4.3.jar:2.4.3]
	at org.springframework.boot.SpringApplication.refreshContext(SpringApplication.java:426) ~[spring-boot-2.4.3.jar:2.4.3]
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:326) ~[spring-boot-2.4.3.jar:2.4.3]
	at org.springframework.boot.builder.SpringApplicationBuilder.run(SpringApplicationBuilder.java:144) [spring-boot-2.4.3.jar:2.4.3]
	at org.springframework.cloud.bootstrap.BootstrapApplicationListener.bootstrapServiceContext(BootstrapApplicationListener.java:212) [spring-cloud-context-2.2.5.RELEASE.jar:2.2.5.RELEASE]
	at org.springframework.cloud.bootstrap.BootstrapApplicationListener.onApplicationEvent(BootstrapApplicationListener.java:117) [spring-cloud-context-2.2.5.RELEASE.jar:2.2.5.RELEASE]
	at org.springframework.cloud.bootstrap.BootstrapApplicationListener.onApplicationEvent(BootstrapApplicationListener.java:74) [spring-cloud-context-2.2.5.RELEASE.jar:2.2.5.RELEASE]
	at org.springframework.context.event.SimpleApplicationEventMulticaster.doInvokeListener(SimpleApplicationEventMulticaster.java:176) [spring-context-5.3.4.jar:5.3.4]
	at org.springframework.context.event.SimpleApplicationEventMulticaster.invokeListener(SimpleApplicationEventMulticaster.java:169) [spring-context-5.3.4.jar:5.3.4]
	at org.springframework.context.event.SimpleApplicationEventMulticaster.multicastEvent(SimpleApplicationEventMulticaster.java:143) [spring-context-5.3.4.jar:5.3.4]
	at org.springframework.context.event.SimpleApplicationEventMulticaster.multicastEvent(SimpleApplicationEventMulticaster.java:131) [spring-context-5.3.4.jar:5.3.4]
	at org.springframework.boot.context.event.EventPublishingRunListener.environmentPrepared(EventPublishingRunListener.java:82) [spring-boot-2.4.3.jar:2.4.3]
	at org.springframework.boot.SpringApplicationRunListeners.lambda$environmentPrepared$2(SpringApplicationRunListeners.java:63) [spring-boot-2.4.3.jar:2.4.3]
	at java.util.ArrayList.forEach(ArrayList.java:1257) ~[na:1.8.0_251]
	at org.springframework.boot.SpringApplicationRunListeners.doWithListeners(SpringApplicationRunListeners.java:117) [spring-boot-2.4.3.jar:2.4.3]
	at org.springframework.boot.SpringApplicationRunListeners.doWithListeners(SpringApplicationRunListeners.java:111) [spring-boot-2.4.3.jar:2.4.3]
	at org.springframework.boot.SpringApplicationRunListeners.environmentPrepared(SpringApplicationRunListeners.java:62) [spring-boot-2.4.3.jar:2.4.3]
	at org.springframework.boot.SpringApplication.prepareEnvironment(SpringApplication.java:362) ~[spring-boot-2.4.3.jar:2.4.3]
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:320) ~[spring-boot-2.4.3.jar:2.4.3]
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:1311) ~[spring-boot-2.4.3.jar:2.4.3]
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:1300) ~[spring-boot-2.4.3.jar:2.4.3]
	at com.xxxx.xxx.search.SearchApplication.main(SearchApplication.java:11) ~[classes/:na]

这是一个版本问题。
原因分析
仔细看报错信息，可以看到这样一个熟悉的版本号“2.2.5.RELEASE”
[spring-cloud-context-2.2.5.RELEASE.jar:2.2.5.RELEASE]
这和 Nacos 的版本号一致

于是我看了一下 Maven 中的 Dependencies

然后去Maven仓库里找了最新的版本

将项目中的版本替换成这个版本：
org.springframework.cloud » spring-cloud-context » 3.0.1

重新启动项目，发现还是报错：

java.lang.IllegalArgumentException: Could not find class [org.springframework.cloud.client.loadbalancer.LoadBalancerProperties]
	at org.springframework.util.ClassUtils.resolveClassName(ClassUtils.java:334) ~[spring-core-5.3.4.jar:5.3.4]
	at org.springframework.core.annotation.TypeMappedAnnotation.adapt(TypeMappedAnnotation.java:446) ~[spring-core-5.3.4.jar:5.3.4]
	at org.springframework.core.annotation.TypeMappedAnnotation.getValue(TypeMappedAnnotation.java:369) ~[spring-core-5.3.4.jar:5.3.4]
	at org.springframework.core.annotation.TypeMappedAnnotation.asMap(TypeMappedAnnotation.java:284) ~[spring-core-5.3.4.jar:5.3.4]
	at org.springframework.core.annotation.AbstractMergedAnnotation.asAnnotationAttributes(AbstractMergedAnnotation.java:193) ~[spring-core-5.3.4.jar:5.3.4]
	at org.springframework.core.type.AnnotatedTypeMetadata.getAnnotationAttributes(AnnotatedTypeMetadata.java:106) ~[spring-core-5.3.4.jar:5.3.4]
	at org.springframework.context.annotation.AnnotationConfigUtils.attributesFor(AnnotationConfigUtils.java:285) ~[spring-context-5.3.4.jar:5.3.4]
	at org.springframework.context.annotation.AnnotationBeanNameGenerator.determineBeanNameFromAnnotation(AnnotationBeanNameGenerator.java:102) ~[spring-context-5.3.4.jar:5.3.4]
	at org.springframework.context.annotation.AnnotationBeanNameGenerator.generateBeanName(AnnotationBeanNameGenerator.java:81) ~[spring-context-5.3.4.jar:5.3.4]
	at org.springframework.context.annotation.ConfigurationClassBeanDefinitionReader.registerBeanDefinitionForImportedConfigurationClass(ConfigurationClassBeanDefinitionReader.java:169) ~[spring-context-5.3.4.jar:5.3.4]
	at org.springframework.context.annotation.ConfigurationClassBeanDefinitionReader.loadBeanDefinitionsForConfigurationClass(ConfigurationClassBeanDefinitionReader.java:150) ~[spring-context-5.3.4.jar:5.3.4]
	at org.springframework.context.annotation.ConfigurationClassBeanDefinitionReader.loadBeanDefinitions(ConfigurationClassBeanDefinitionReader.java:129) ~[spring-context-5.3.4.jar:5.3.4]
	at org.springframework.context.annotation.ConfigurationClassPostProcessor.processConfigBeanDefinitions(ConfigurationClassPostProcessor.java:342) ~[spring-context-5.3.4.jar:5.3.4]
	at org.springframework.context.annotation.ConfigurationClassPostProcessor.postProcessBeanDefinitionRegistry(ConfigurationClassPostProcessor.java:246) ~[spring-context-5.3.4.jar:5.3.4]
	at org.springframework.context.support.PostProcessorRegistrationDelegate.invokeBeanDefinitionRegistryPostProcessors(PostProcessorRegistrationDelegate.java:311) ~[spring-context-5.3.4.jar:5.3.4]
	at org.springframework.context.support.PostProcessorRegistrationDelegate.invokeBeanFactoryPostProcessors(PostProcessorRegistrationDelegate.java:112) ~[spring-context-5.3.4.jar:5.3.4]
	at org.springframework.context.support.AbstractApplicationContext.invokeBeanFactoryPostProcessors(AbstractApplicationContext.java:745) ~[spring-context-5.3.4.jar:5.3.4]
	at org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:563) ~[spring-context-5.3.4.jar:5.3.4]
	at org.springframework.boot.web.servlet.context.ServletWebServerApplicationContext.refresh(ServletWebServerApplicationContext.java:144) ~[spring-boot-2.4.3.jar:2.4.3]
	at org.springframework.boot.SpringApplication.refresh(SpringApplication.java:767) [spring-boot-2.4.3.jar:2.4.3]
	at org.springframework.boot.SpringApplication.refresh(SpringApplication.java:759) [spring-boot-2.4.3.jar:2.4.3]
	at org.springframework.boot.SpringApplication.refreshContext(SpringApplication.java:426) [spring-boot-2.4.3.jar:2.4.3]
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:326) [spring-boot-2.4.3.jar:2.4.3]
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:1311) [spring-boot-2.4.3.jar:2.4.3]
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:1300) [spring-boot-2.4.3.jar:2.4.3]
	at com.xxxx.xxx.search.SearchApplication.main(SearchApplication.java:11) [classes/:na]
	
再打开 Maven 的 Dependencies ，可以看到，loadbalancer 下的spring-cloud-commons与spring-cloud-context版本不一致，spring-cloud-commons用的还是 Nacos 里的版本“2.2.5.RELEASE”。

于是我也将spring-cloud-commons的版本替换成
org.springframework.cloud » spring-cloud-commons » 3.0.1

再重启项目，此时，项目已经成功启动啦！

解决方案
在项目的pom.xml文件中添加两个依赖，用高版本“3.0.1”替换低版本“2.2.5.RELEASE”：

        <!-- https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-context -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-context</artifactId>
            <version>3.0.1</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-commons -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-commons</artifactId>
            <version>3.0.1</version>
        </dependency>