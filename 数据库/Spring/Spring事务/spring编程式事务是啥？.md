发布于 2020-09-14 16:12:12

3K

0

举报

本文开始，大概用10篇左右的文章来详解spring中事务的使用，吃透spring事务。

## **本文内容**

详解spring中编程式事务的使用。

## **spring中使用事务的2种方式**

spring使事务操作变的异常容易了，spring中控制事务主要有2种方式

- **编程式事务**：硬编码的方式
- **声明式事务**：大家比较熟悉的注解@Transaction的方式

## **编程式事务**

### **什么是编程式事务？**

通过硬编码的方式使用spring中提供的事务相关的类来控制事务。

### **编程式事务主要有2种用法**

- 方式1：通过PlatformTransactionManager控制事务
- 方式2：通过TransactionTemplate控制事务

## **方式1：PlatformTransactionManager**

这种是最原始的方式，代码量比较大，后面其他方式都是对这种方式的封装。

直接看案例，有详细的注释，大家一看就懂。

### **案例代码位置**

![](media/3hdwzgsaa4.png)

### **准备sql**

```javascript
DROP DATABASE IF EXISTS javacode2018;
CREATE DATABASE if NOT EXISTS javacode2018;

USE javacode2018;
DROP TABLE IF EXISTS t_user;
CREATE TABLE t_user(
  id int PRIMARY KEY AUTO_INCREMENT,
  name varchar(256) NOT NULL DEFAULT '' COMMENT '姓名'
);
```

### **maven配置**

```javascript
<!-- JdbcTemplate需要的 -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>5.2.3.RELEASE</version>
</dependency>
<!-- spring 事务支持 -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-tx</artifactId>
    <version>5.2.3.RELEASE</version>
</dependency>
```

### **测试代码**

代码中会用到JdbcTemplate，对这个不理解的可以看一下：[**JdbcTemplate使用详解**](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzA5MTkxMDQ4MQ%3D%3D%26mid%3D2648936449%26idx%3D2%26sn%3Dda1e98e5914821f040d5530e8ca9d9bc%26scene%3D21%23wechat_redirect)

```javascript
@Test
public void test1() throws Exception {
    //定义一个数据源
    org.apache.tomcat.jdbc.pool.DataSource dataSource = new org.apache.tomcat.jdbc.pool.DataSource();
    dataSource.setDriverClassName("com.mysql.jdbc.Driver");
    dataSource.setUrl("jdbc:mysql://localhost:3306/javacode2018?characterEncoding=UTF-8");
    dataSource.setUsername("root");
    dataSource.setPassword("root123");
    dataSource.setInitialSize(5);
    //定义一个JdbcTemplate，用来方便执行数据库增删改查
    JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);
    //1.定义事务管理器，给其指定一个数据源（可以把事务管理器想象为一个人，这个人来负责事务的控制操作）
    PlatformTransactionManager platformTransactionManager = new DataSourceTransactionManager(dataSource);
    //2.定义事务属性：TransactionDefinition，TransactionDefinition可以用来配置事务的属性信息，比如事务隔离级别、事务超时时间、事务传播方式、是否是只读事务等等。
    TransactionDefinition transactionDefinition = new DefaultTransactionDefinition();
    //3.开启事务：调用platformTransactionManager.getTransaction开启事务操作，得到事务状态(TransactionStatus)对象
    TransactionStatus transactionStatus = platformTransactionManager.getTransaction(transactionDefinition);
    //4.执行业务操作，下面就执行2个插入操作
    try {
        System.out.println("before:" + jdbcTemplate.queryForList("SELECT * from t_user"));
        jdbcTemplate.update("insert into t_user (name) values (?)", "test1-1");
        jdbcTemplate.update("insert into t_user (name) values (?)", "test1-2");
        //5.提交事务：platformTransactionManager.commit
        platformTransactionManager.commit(transactionStatus);
    } catch (Exception e) {
        //6.回滚事务：platformTransactionManager.rollback
        platformTransactionManager.rollback(transactionStatus);
    }
    System.out.println("after:" + jdbcTemplate.queryForList("SELECT * from t_user"));
}
```

### **运行输出**

```javascript
before:[]
after:[{id=1, name=test1-1}, {id=2, name=test1-2}]
```

### **代码分析**

代码中主要有5个步骤

**步骤1：定义事务管理器PlatformTransactionManager**

事务管理器相当于一个管理员，这个管理员就是用来帮你控制事务的，比如开启事务，提交事务，回滚事务等等。

spring中使用PlatformTransactionManager这个接口来表示事务管理器，

```javascript
public interface PlatformTransactionManager {

 //获取一个事务（开启事务）
 TransactionStatus getTransaction(@Nullable TransactionDefinition definition)
   throws TransactionException;

 //提交事务
 void commit(TransactionStatus status) throws TransactionException;


 //回滚事务
 void rollback(TransactionStatus status) throws TransactionException;

}
```

PlatformTransactionManager多个实现类，用来应对不同的环境

![](media/itggopnujy.png)

**JpaTransactionManager**：如果你用jpa来操作db，那么需要用这个管理器来帮你控制事务。

**DataSourceTransactionManager**：如果你用是指定数据源的方式，比如操作[数据库](https://cloud.tencent.com/solution/database?from_column=20065&from=20065)用的是：JdbcTemplate、mybatis、ibatis，那么需要用这个管理器来帮你控制事务。

**HibernateTransactionManager**：如果你用hibernate来操作db，那么需要用这个管理器来帮你控制事务。

**JtaTransactionManager**：如果你用的是java中的jta来操作db，这种通常是[分布式事务](https://cloud.tencent.com/product/dtf?from_column=20065&from=20065)，此时需要用这种管理器来控制事务。

上面案例代码中我们使用的是JdbcTemplate来操作db，所以用的是`DataSourceTransactionManager`这个管理器。

```javascript
PlatformTransactionManager platformTransactionManager = new DataSourceTransactionManager(dataSource);
```

**步骤2：定义事务属性TransactionDefinition**

定义事务属性，比如事务隔离级别、事务超时时间、事务传播方式、是否是只读事务等等。

spring中使用TransactionDefinition接口来表示事务的定义信息，有个子类比较常用：DefaultTransactionDefinition。

关于事务属性细节比较多，篇幅比较长，后面会专门有文章来详解。

**步骤3：开启事务**

调用事务管理器的`getTransaction`方法，即可以开启一个事务

```javascript
TransactionStatus transactionStatus = platformTransactionManager.getTransaction(transactionDefinition);
```

这个方法会返回一个`TransactionStatus`表示事务状态的一个对象，通过`TransactionStatus`提供的一些方法可以用来控制事务的一些状态，比如事务最终是需要回滚还是需要提交。

执行了`getTransaction`后，spring内部会执行一些操作，为了方便大家理解，咱们看看伪代码：

```javascript
//有一个全局共享的threadLocal对象 resources
static final ThreadLocal<Map<Object, Object>> resources = new NamedThreadLocal<>("Transactional resources");
//获取一个db的连接
DataSource datasource = platformTransactionManager.getDataSource();
Connection connection = datasource.getConnection();
//设置手动提交事务
connection.setAutoCommit(false);
Map<Object, Object> map = new HashMap<>();
map.put(datasource,connection);
resources.set(map);
```

上面代码，将数据源datasource和connection映射起来放在了ThreadLocal中，ThreadLocal大家应该比较熟悉，用于在同一个线程中共享数据；后面我们可以通过resources这个ThreadLocal获取datasource其对应的connection对象。

**步骤4：执行业务操作**

我们使用jdbcTemplate插入了2条记录。

```javascript
jdbcTemplate.update("insert into t_user (name) values (?)", "test1-1");
jdbcTemplate.update("insert into t_user (name) values (?)", "test1-2");
```

大家看一下创建JdbcTemplate的代码，需要指定一个datasource

```javascript
JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);
```

再来看看创建事务管理器的代码

```javascript
PlatformTransactionManager platformTransactionManager = new DataSourceTransactionManager(dataSource);
```

**2者用到的是同一个dataSource，而事务管理器开启事务的时候，会创建一个连接，将datasource和connection映射之后丢在了ThreadLocal中，而JdbcTemplate内部执行db操作的时候，也需要获取连接，JdbcTemplate会以自己内部的datasource去上面的threadlocal中找有没有关联的连接，如果有直接拿来用，若没找到将重新创建一个连接，而此时是可以找到的，那么JdbcTemplate就参与到spring的事务中了。**

**步骤5：提交 or 回滚**

```javascript
//5.提交事务：platformTransactionManager.commit
platformTransactionManager.commit(transactionStatus);

//6.回滚事务：platformTransactionManager.rollback
platformTransactionManager.rollback(transactionStatus);
```

## **方式2：TransactionTemplate**

方式1中部分代码是可以重用的，所以spring对其进行了优化，采用模板方法模式就其进行封装，主要省去了提交或者回滚事务的代码。

### **案例代码位置**

![](media/5u6m3k1bcs.png)

### **测试代码**

```java
@Test
public void test1() throws Exception {
    //定义一个数据源
    org.apache.tomcat.jdbc.pool.DataSource dataSource = new org.apache.tomcat.jdbc.pool.DataSource();
    dataSource.setDriverClassName("com.mysql.jdbc.Driver");
    dataSource.setUrl("jdbc:mysql://localhost:3306/javacode2018?characterEncoding=UTF-8");
    dataSource.setUsername("root");
    dataSource.setPassword("root123");
    dataSource.setInitialSize(5);
    //定义一个JdbcTemplate，用来方便执行数据库增删改查
    JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);
    //1.定义事务管理器，给其指定一个数据源（可以把事务管理器想象为一个人，这个人来负责事务的控制操作）
    PlatformTransactionManager platformTransactionManager = new DataSourceTransactionManager(dataSource);
    //2.定义事务属性：TransactionDefinition，TransactionDefinition可以用来配置事务的属性信息，比如事务隔离级别、事务超时时间、事务传播方式、是否是只读事务等等。
    DefaultTransactionDefinition transactionDefinition = new DefaultTransactionDefinition();
    transactionDefinition.setTimeout(10);//如：设置超时时间10s
    //3.创建TransactionTemplate对象
    TransactionTemplate transactionTemplate = new TransactionTemplate(platformTransactionManager, transactionDefinition);
    /**
     * 4.通过TransactionTemplate提供的方法执行业务操作
     * 主要有2个方法：
     * （1）.executeWithoutResult(Consumer<TransactionStatus> action)：没有返回值的，需传递一个Consumer对象，在accept方法中做业务操作
     * （2）.<T> T execute(TransactionCallback<T> action)：有返回值的，需要传递一个TransactionCallback对象，在doInTransaction方法中做业务操作
     * 调用execute方法或者executeWithoutResult方法执行完毕之后，事务管理器会自动提交事务或者回滚事务。
     * 那么什么时候事务会回滚，有2种方式：
     * （1）transactionStatus.setRollbackOnly();将事务状态标注为回滚状态
     * （2）execute方法或者executeWithoutResult方法内部抛出异常
     * 什么时候事务会提交？
     * 方法没有异常 && 未调用过transactionStatus.setRollbackOnly();
     */
    transactionTemplate.executeWithoutResult(new Consumer<TransactionStatus>() {
        @Override
        public void accept(TransactionStatus transactionStatus) {
            jdbcTemplate.update("insert into t_user (name) values (?)", "transactionTemplate-1");
            jdbcTemplate.update("insert into t_user (name) values (?)", "transactionTemplate-2");

        }
    });
    System.out.println("after:" + jdbcTemplate.queryForList("SELECT * from t_user"));
}
```

### **运行输出**

```javascript
after:[{id=1, name=transactionTemplate-1}, {id=2, name=transactionTemplate-2}]
```

### **代码分析**

TransactionTemplate，主要有2个方法：

**executeWithoutResult：无返回值场景**

`executeWithoutResult(Consumer<TransactionStatus> action)`
：没有返回值的，需传递一个Consumer对象，在accept方法中做业务操作

```java
transactionTemplate.executeWithoutResult(new Consumer<TransactionStatus>() {
    @Override
    public void accept(TransactionStatus transactionStatus) {
        //执行业务操作
    }
});
```

**execute：有返回值场景**

`<T> T execute(TransactionCallback<T> action)`：有返回值的，需要传递一个TransactionCallback对象，在doInTransaction方法中做业务操作

```java
Integer result = transactionTemplate.execute(new TransactionCallback<Integer>() {
    @Nullable
    @Override
    public Integer doInTransaction(TransactionStatus status) {
        return jdbcTemplate.update("insert into t_user (name) values (?)", "executeWithoutResult-3");
    }
});
```

通过上面2个方法，事务管理器会自动提交事务或者回滚事务。

**什么时候事务会回滚，有2种方式**

**方式1**

在execute或者executeWithoutResult内部执行`transactionStatus.setRollbackOnly();`将事务状态标注为回滚状态，spring会自动让事务回滚

**方式2**

execute方法或者executeWithoutResult方法内部抛出任意异常即可回滚。

**什么时候事务会提交？**

方法没有异常  未调用过transactionStatus.setRollbackOnly();

## **编程式事务正确的使用姿势**

如果大家确实想在系统中使用编程式事务，那么可以参考下面代码，使用spring来管理对象，更简洁一些。

先来个配置类，将事务管理器`PlatformTransactionManager`、事务模板`TransactionTemplate`都注册到spring中，重用。

```java
package com.javacode2018.tx.demo3;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;
import org.springframework.transaction.PlatformTransactionManager;
import org.springframework.transaction.support.TransactionTemplate;

import javax.sql.DataSource;

@Configuration
@ComponentScan
public class MainConfig3 {
    @Bean
    public DataSource dataSource() {
        org.apache.tomcat.jdbc.pool.DataSource dataSource = new org.apache.tomcat.jdbc.pool.DataSource();
        dataSource.setDriverClassName("com.mysql.jdbc.Driver");
        dataSource.setUrl("jdbc:mysql://localhost:3306/javacode2018?characterEncoding=UTF-8");
        dataSource.setUsername("root");
        dataSource.setPassword("root123");
        dataSource.setInitialSize(5);
        return dataSource;
    }

    @Bean
    public JdbcTemplate jdbcTemplate(DataSource dataSource) {
        return new JdbcTemplate(dataSource);
    }

    @Bean
    public PlatformTransactionManager transactionManager(DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }

    @Bean
    public TransactionTemplate transactionTemplate(PlatformTransactionManager transactionManager) {
        return new TransactionTemplate(transactionManager);
    }
}
```

通常我们会将业务操作放在service中，所以我们也来个service：UserService。

```java

package com.javacode2018.tx.demo3;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Component;
import org.springframework.transaction.support.TransactionTemplate;

import java.util.List;

@Component
public class UserService {
    @Autowired
    private JdbcTemplate jdbcTemplate;
    @Autowired
    private TransactionTemplate transactionTemplate;

    //模拟业务操作1
    public void bus1() {
        this.transactionTemplate.executeWithoutResult(transactionStatus -> {
            //先删除表数据
            this.jdbcTemplate.update("delete from t_user");
            //调用bus2
            this.bus2();
        });
    }

    //模拟业务操作2
    public void bus2() {
        this.transactionTemplate.executeWithoutResult(transactionStatus -> {
            this.jdbcTemplate.update("insert into t_user (name) VALUE (?)", "java");
            this.jdbcTemplate.update("insert into t_user (name) VALUE (?)", "spring");
            this.jdbcTemplate.update("insert into t_user (name) VALUE (?)", "mybatis");
        });
    }

    //查询表中所有数据
    public List userList() {
        return jdbcTemplate.queryForList("select * from t_user");
    }
}
```

bus1中会先删除数据，然后调用bus2，此时bus1中的所有操作和bus2中的所有操作会被放在一个事务中执行，这是spring内部默认实现的，bus1中调用`executeWithoutResult`的时候，会开启一个事务，而内部又会调用bus2，而bus2内部也调用了`executeWithoutResult`，bus内部会先判断一下上线文环境中有没有事务，如果有就直接参与到已存在的事务中，刚好发现有bus1已开启的事务，所以就直接参与到bus1的事务中了，最终bus1和bus2会在一个事务中运行。

上面bus1代码转换为sql脚本如下：

```javascript
start transaction; //开启事务
delete from t_user;
insert into t_user (name) VALUE ('java');
insert into t_user (name) VALUE ('spring');
insert into t_user (name) VALUE ('mybatis');
commit;
```

来个测试案例，看一下效果

```javascript
package com.javacode2018.tx.demo3;

import org.junit.Test;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class Demo3Test {
    @Test
    public void test1() {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(MainConfig3.class);
        UserService userService = context.getBean(UserService.class);
        userService.bus1();
        System.out.println(userService.userList());
    }
}
```

运行`test1()`输出

```javascript
[{id=18, name=java}, {id=19, name=spring}, {id=20, name=mybatis}]
```

上面代码中，bus1或者bus2中，如果有异常或者执行`transactionStatus.setRollbackOnly()`，此时整个事务都会回滚，大家可以去试试！

## **总结一下**

大家看了之后，会觉得这样用好复杂啊，为什么要这么玩？

的确，看起来比较复杂，代码中融入了大量spring的代码，耦合性比较强，不利于扩展，本文的目标并不是让大家以后就这么用，主要先让大家从硬编码上了解spring中事务是如何控制的，后面学起来才会更容易。

我们用的最多的是声明式事务，声明式事务的底层还是使用上面这种方式来控制事务的，只不过对其进行了封装，让我们用起来更容易些。

下篇文章将详解声明式事务的使用。

## **案例源码**

```javascript
git地址：
https://gitee.com/javacode2018/spring-series

本文案例对应源码模块：lesson-002-tx
```

**路人甲java所有案例代码以后都会放到这个上面，大家watch一下，可以持续关注动态。**