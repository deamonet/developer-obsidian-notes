admin 340 2022-12-06

本文转载自网络公开信息

详解SpringBoot自定义配置与整合Druid

目录SpringBoot配置文件优先级yaml的多文档配置扩展SpringMVC添加自定义视图解析器自定义DruidDataSourcesAbout Druid添加依赖配置数据源其他配置Druid配置类测试类数据源监控监控过滤器filter配置

![详解SpringBoot自定义配置与整合Druid](media/详解SpringBoot自定义配置与整合Druid.png)

SpringBoot配置文件

优先级

前面SpringBoot基础有提到，关于SpringBoot配置文件可以是properties或者是yaml格式的文件，但是在SpringBoot加载application配置文件时是存在一个优先级的。优先级如下：

file:./config/ ==> 项目路径下的config目录下

file:./ ==> 项目路径下

classpath:/config/ ==> 资源路径下的config目录下

classpath:/ ==> 项目路径下

yaml的多文档配置

yaml可以通过---达到在一个文件中写入多套配置文件的效果

server:

port: 8081

spring:

profiles: dev

---

server:

port: 8082

spring:

profiles: test

@canditionalon注解，Spring底层的注解， 用于判断是否符合条件，符合条件才会自动装配。

扩展SpringMVC

添加自定义视图解析器

ViewResolver 试图解析器，实现了该接口的类都可以称作试图解析器

candidateViews 候选视图，getBestView 得到最优视图

其中有getCandidateViews[方法](https://www.finclip.com/news/tags-157.html)，先遍历所有的视图解析器，之后封装成view对象，添加到candidateViews候选视图解析器数组中。

自定视图解析器需要实现ViewResolver接口并重写resolveViewName方法

@Configuration

public class MyMvcConfig implements WebMvcConfigurer {

@Bean

public ViewResolver myViewResolver(){

return new MyViewResolver();

}

public static class MyViewResolver implements ViewResolver {

@Override

public View resolveViewName(String viewNaem, Locale locale) throws Exception {

return null;

}

}

}

想自定义其他功能也是同理，按格式写好组件交给SpringBoot自动装配即可。

自定义DruidDataSources

About Druid

Druid：https://github.com/alibaba/druid/

Druid是alibaba开源平台上一个数据库连接池实现，结合了C3P0,DBCP等DB池的优点，同时也有Web监控界面。

Druid可以很好的监控DB池连接和SQL执行的情况，为监控而生的DB连接池。

SpringBoot2.0以上默认使用Hikari数据源，下面记录下如何用SpringBoot整合配置Druid

添加依赖

com.alibaba

druid

1.1.21

配置数据源

因为SpringBoot2.0以上默认使用Hikari数据源，所以需要用 spring.datasource.type 指定数据源。

spring:

datasource:

username: root

password: 123456

url: jdbc:mysql://localhost:3306/springboot?serverTimezone=UTC&useUnicode=true&characterEncoding=utf-8

driver-class-name: com.mysql.cj.jdbc.Driver

type: com.alibaba.druid.pool.DruidDataSource # 自定义数据源

可以起一个测试类查看

@Test

public void druidTest() throws SQLException {

//查看默认数据源

System.out.println(dataSource.getClass());

//获得数据库连接

Connection connection = dataSource.getConnection();

System.out.println(connection);

//close

connection.close();

}

其他配置

具体其他配置可参考官方文档，简单列举一些：

#Spring Boot 默认是不注入这些[属性](https://www.finclip.com/news/tags-91.html)值的，需要自己绑定

#druid 数据源专有配置

initialSize: 5

minIdle: 5

maxActive: 20

maxWait: 60000

timeBetweenEvictionRunsMillis: 60000

minEvictableIdleTimeMillis: 300000

validationQuery: SELECT 1 FROM DUAL

testWhileIdle: true

testOnBorrow: false

testOnReturn: false

poolPreparedStatements: true

#配置监控[统计](https://www.finclip.com/news/tags-832.html)拦截的filters，stat:监控统计、log4j：日志记录、wall：防御sql注入

#如果允许时报错 [java](https://www.finclip.com/news/tags-44.html).lang.ClassNotFoundException: org.apache.log4j.Priority

#则导入 log4j 依赖即可，Maven 地址：https://mvnrepository.com/artifact/log4j/log4j

filters: stat,wall,log4j

maxPoolPreparedStatementPerConnectionSize: 20

useGlobalDataSourceStat: true

connectionProperties: druid.stat.mergeSql=true;druid.stat.slowSqlMillis=500

关于监控的配置是Druid特点。比如可配置log4j以及自带wall防止sql注入

Druid配置类

一般在config包下，与自定义组件类似，通过@ConfigurationProperties注解与配置文件中datasource的配置绑定并交给SpringBoosZUngGFft自动装配。

@Configuration

public class DruidConfig {

/*

将自定义的 Druid数据源添加到容器中，不再让 Spring Boot 自动创建

绑定全局配置文件中的 druid 数据源属性到 com.alibaba.druid.pool.DruidDataSource从而让它们生效

@ConfigurationPropersZUngGFfties(prefix = "spring.datasource")：作用就是将 全局配置文件中

前缀为 spring.datasource的属性值注入到 com.alibaba.druid.pool.DruidDataSource 的同名参数中

*/

@ConfigurationProperties(prefix = "spring.datasource")

@Bean

public DataSource druidDataSource() {

return new DruidDataSource();

}

}

测试类

@Test

public void druidTest() throws SQLException {

//查看默认数据源

System.out.println(dataSource.getClass());

//获得数据库连接

Connection connection = dataSource.getConnection();

System.out.println(connection);

//close

connection.close();

DruidDataSource druidDataSource = (DruidDataSource) dataSource;

System.out.println("druidDataSource 数据源最大连接数：" + druidDataSource.getMaxActive());

System.out.println("druidDataSource 数据源初始化连接数：" + druidDataSource.getInitialSize());

}

数据源监控

还是在同一个配置类文件中写入，这里对于审计或者渗透测试中的重点其实就是用户名密码了和其访问限制了

package com.zh1z3ven.hellospringboot.config;

import com.alibaba.druid.pool.DruidDataSource;

import com.alibaba.druid.support.http.StatViewServlet;

import org.springframework.boot.context.properties.ConfigurationProperties;

import org.springframework.boot.web.servlet.ServletRegistrationBean;

import org.springframework.context.annotation.Bean;

import org.springframework.context.annotation.Configuration;

import javax.sql.DataSource;

import java.util.HashMap;

import java.util.Map;

@Configuration

public class DruidConfig {

/*

将自定义的 Druid数据源添加到容器中，不再让 Spring Boot 自动创建

绑定全局配置文件中的 druid 数据源属性到 com.alibaba.druid.pool.DruidDataSource从而让它们生效

@ConfigurationProperties(prefix = "spring.datasource")：作用就是将 全局配置文件中

前缀为 spring.datasource的属性值注入到 com.alibaba.druid.pool.DruidDataSource 的同名参数中

*/

@ConfigurationProperties(prefix = "spring.datasource")

@Bean

public DataSource druidDataSource() {

return new DruidDataSource();

}

//配置 Druid 监控管理后台的Servlet；

//内置 Servlet 容器时没有web.xml文件，所以使用 Spring Boot 的注册 Servlet 方式

@Bean

public ServletRegistrationBean statViewServlet() {

ServletRegistrationBean bean = new ServletRegistrationBean(new StatViewServlet(), "/druid/*");

// 这些参数可以在 com.alibaba.druid.support.http.StatViewServlet

// 的父类 com.alibaba.druid.support.http.ResourceServlet 中找到

Map initParams = new HashMap<>();

initParams.put("loginUsername", "admin"); //后台管理界面的登录账号

initParams.put("loginPassword", "123456"); //后台管理界面的登录密码

//后台允许谁可以访问

//initParams.put("allow", "localhost")：表示只有本机可以访问

//initParams.put("allow", "")：为空或者为null时，表示允许所有访问

initParams.put("allow", "");

//deny：Druid 后台拒绝谁访问

//initParams.put("kuangshen", "192.168.1.20");表示禁止此ip访问

//设置初始化参数

bean.setInitParameters(initParams);

return bean;

}

}

配置完后重启项目，访问测试

监控过滤器filter配置

//配置 Druid 监控 之 web 监控的 filter

//WebStatFilter：用于配置Web和Druid数据源之间的管理关联监控统计

@Bean

public FilterRegistrationBean webStatFilter() {

FilterRegistrationBean bean = new FilterRegistrationBean();

bean.setFilter(new WebStatFilter());

//exclusions：设置哪些请求进行过滤排除掉，从而不进行统计

Map initParams = new HashMap<>();

initParams.put("exclusions", "*.js,*.css,/druid/*,/jdbc/*");

bean.setInitParameters(initParams);

//"/*" 表示过滤所有请求

bean.setUrlPatterns(Arrays.asList("/*"));

return bean;

}

所有内容仅限于维护网络安全学习参考