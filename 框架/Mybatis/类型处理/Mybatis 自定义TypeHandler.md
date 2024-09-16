# Mybatis 自定义TypeHandler

**1、简介**

       相信大家用Mybatis这个框架至少一年以上了吧，有没有思考过这样一个问题：数据库有自己的数据类型，Java有自己的数据类型，那么Mybatis是如何把数据库中的类型和Java的数据类型对应的呢？本篇文章就来讲讲Mybatis中的 黑匣子TypeHandler(类型处理器) ，说它是黑匣子一点都不为过，总是在默默的奉献着，但是不为人知。本篇文章讲的一切内容都是基于Mybatis3.5 和SpringBoot-2.3.3.RELEASE 。

**2、什么是TypeHandler？**

顾名思义，类型处理器，将入参和结果转换为所需要的类型，Mybatis中对于内置了许多类型处理器，实际开发中已经足够使用了，如下图：

![](media/1419013-20221024154555898-2002475795.png)

类型处理器这个接口其实很简单，总共四个方法，一个方法将入参的Java类型的数  据转换为JDBC类型，三个方法将返回结果转换为Java类型。源码如下：

```java
public interface TypeHandler<T> {    //设置参数，java类型转换为jdbc类型
   void setParameter(PreparedStatement ps, int i, T parameter, JdbcType jdbcType) throws SQLException;    //将查询的结果转换为java类型
    T getResult(ResultSet rs, String columnName) throws SQLException;
    T getResult(ResultSet rs, int columnIndex) throws SQLException;
    T getResult(CallableStatement cs, int columnIndex) throws SQLException;
}

```

**3、如何自定义并使用TypeHandler？**

实际应用开发中的难免会有一些需求要自定义一个TypeHandler ，比如这样一个需求：前端传来的年龄是 男, 女，但是数据库定义的字段却是int 类型（ 1男2女）。此时可以自定义一个年龄的类型处理器，进行转换。

**自定义的方式有两种：**

1. 一种是实现TypeHandler 这个接口。
2. 另一个就是继承BaseTypeHandler 这个便捷的抽象类。

下面直接继承BaseTypeHandler 这个抽象类，定义一个年龄的类型处理器，如下：

```java
@MappedJdbcTypes(JdbcType.INTEGER)
@MappedTypes(String.class)
public class GenderTypeHandler extends BaseTypeHandler {      //设置参数，这里将Java的String类型转换为JDBC的Integer类型
      @Override      public void setNonNullParameter(PreparedStatement ps, int i, Object parameter, JdbcType jdbcType) throws SQLException {
           ps.setInt(i, StringUtils.equals(parameter.toString(),"男")? 1:2);
       }
      //  以下三个参数都是将查询的结果转换
      @Override      public Object getNullableResult(ResultSet rs, String columnName) throws SQLException {          return rs.getInt(columnName)==1?"男":"女";
     }
     @Override
     public Object getNullableResult(ResultSet rs, int columnIndex) throws SQLException {       return rs.getInt(columnIndex)==1?"男":"女";
     }
     @Override
     public Object getNullableResult(CallableStatement cs, int columnIndex) throws SQLException {          return cs.getInt(columnIndex)==1?"男":"女";
     }
}
```

##### 这里涉及到两个注解，如下：

**@MappedTypes ：**指定与其关联的 Java 类型列表。 如果在 javaType 属性中也同时指定，则注解上的配置将被忽略。

**@MappedJdbcTypes ：**指定与其关联的 JDBC 类型列表。 如果在jdbcType 属性中也同时指定，则注解上的配置将被忽略。

**4、如何将其添加到Mybatis中？**

##### Mybatis在与SpringBoot整合之后一切都变得很简单了，其实这里有两种配置方式，下面将会一一介绍。

第一种：只需要在配置文件application.properties 中添加一行配置即可，如下：

## 设置自定义的Typehandler所在的包，启动的时候会自动扫描配置到Mybatis中
mybatis.type-handlers-package=cn.cb.demo.typehandler

第二种：其实任何框架与Springboot整合之后，只要配置文件中能够配置的，在配置类中都可以配置（除非有特殊定制，否则不要轻易覆盖自动配置）。如下：

![复制代码](media/复制代码.gif)

```java
@Bean("sqlSessionFactory")
public SqlSessionFactory sqlSessionFactory(DataSource dataSource) throws Exception {
   SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
   sqlSessionFactoryBean.setDataSource(dataSource);  
   sqlSessionFactoryBean.setMapperLocations(new PathMatchingResourcePatternResolver().getResources(MAPPER_LOCATOIN));
org.apache.ibatis.session.Configuration configuration = new org.apache.ibatis.session.Configuration();
// 自动将数据库中的下划线转换为驼峰格式
  configuration.setMapUnderscoreToCamelCase(true);  
  configuration.setDefaultFetchSize(100);  
  configuration.setDefaultStatementTimeout(30);  
  sqlSessionFactoryBean.setConfiguration(configuration);
 // 将typehandler注册到mybatis
  GenderTypeHandler genderTypeHandler = new GenderTypeHandler();
  TypeHandler[] typeHandlers=new TypeHandler[]{genderTypeHandler};
  sqlSessionFactoryBean.setTypeHandlers(typeHandlers); return sqlSessionFactoryBean.getObject();
}
```

![复制代码](media/复制代码.gif)

**5、XML文件中如何指定TypeHandler？**

##### 上面的两个步骤分别是自定义和注入到Mybatis中，那么如何在XML 文件中使用呢？

使用其实很简单，分为两种，一种是 更新，一种 查询，下面将会一一介绍。

更新：删除自不必说了，这里讲的是update 和insert 两种，只需要在#{} 中指定的属性typeHandler 为自定义的全类名即可，代码如下：

```sql
<insert id="insertUser">
   insert into user_info(user_id,his_id,name,gender,password,create_time)
   values(#{userId,jdbcType=VARCHAR},#{hisId,jdbcType=VARCHAR},#{name,jdbcType=VARCHAR},
     #{gender,jdbcType=INTEGER,typeHandler=cn.cb.demo.typehandler.GenderT ypeHandler},  
     #{password,jdbcType=VARCHAR},now())
</insert>
```



查询：查询的时候类型处理会将JDBC类型的转化为Java类型，因此也是需要指定typeHandler ，需要在resultMap 中指定typeHandler 这个属性，值为 全类名，如下：

```sql
<resultMap id="userResultMap" type="cn.cb.demo.domain.UserInfo">
    <id column="id" property="id"/>
    <result column="user_id" property="userId"/>
    <result column="his_id" property="hisId"/>
      <!-- 指定typeHandler属性为全类名-->
    <result column="gender" property="gender" typeHandler="cn.cb.demo.typehandler.GenderTypeHandler"/>
    <result column="name" property="name"/>
    <result column="password" property="password"/>
</resultMap>

<select id="selectList" resultMap="userResultMap">   
   select * from user_info where status=1 and user_id in
    <foreach collection="userIds" item="item" open="(" separator="," close=")" >
       #{item}
       </foreach>
</select>
```


**6、源码中如何执行TypeHandler？**

 既然会使用TypeHandler 了，那么肯定要知道其中的执行原理了，在Mybatis中类型处理器是如何在JDBC 类型和Java 类型进行转换的，下面的将从源码角度详细介绍。

### 6.1、入参如何转换？

这个肯定是发生在设置参数的过程中，详细的代码在PreparedStatementHandler 中的parameterize() 方法中，这个方法就是设置参数的方法。源码如下：


```java
@Override
public void parameterize(Statement statement) throws SQLException {  // 实 际 调 用 的 是 DefaultParameterHandler   
  parameterHandler.setParameters((PreparedStatement) statement);
}
```



实际执行的是DefaultParameterHandler 中的setParameters 方法，如下：

![复制代码](media/复制代码.gif)

 public void setParameters(PreparedStatement ps) {
    ErrorContext.instance().activity("setting parameters").object(mappedStatement.getParameterMap().getId());
    // 获取参数映射
    List<ParameterMapping> parameterMappings = boundSql.getParameterMappings();
    if (parameterMappings != null) {
     // 遍历参数映射，一一设置
      for (int i = 0; i < parameterMappings.size(); i++) {
        ParameterMapping parameterMapping = parameterMappings.get(i);
        if (parameterMapping.getMode() != ParameterMode.OUT) {
          Object value;
          String propertyName = parameterMapping.getProperty();
          if (boundSql.hasAdditionalParameter(propertyName)) { // issue #448 ask first for additional params
            value = boundSql.getAdditionalParameter(propertyName);
          } else if (parameterObject == null) {
            value = null;
          } else if (typeHandlerRegistry.hasTypeHandler(parameterObject.getClass())) {
            value = parameterObject;
          } else {
            MetaObject metaObject = configuration.newMetaObject(parameterObject);
            value = metaObject.getValue(propertyName);
          }
          // 获取类型处理器，如果不存在，使用默认的
          TypeHandler typeHandler = parameterMapping.getTypeHandler();
          JdbcType jdbcType = parameterMapping.getJdbcType();
          if (value == null && jdbcType == null) {
            jdbcType = configuration.getJdbcTypeForNull();
          }
          try {
          // 调用类型处理器中的方法设置参数，将Java类型转换为JDBC类型
            typeHandler.setParameter(ps, i + 1, value, jdbcType);
          } catch (TypeException | SQLException e) {
            throw new TypeException("Could not set parameters for mapping: " + parameterMapping + ". Cause: " + e, e);
          }
        }
      }
    }
  }

![复制代码](media/复制代码.gif)

从上面的源码中可以知道这行代码： typeHandler.setParameter(ps, i + 1,value, jdbcType); 就是调用类型处理器中的设置参数的方法，将Java 类型转换为JDBC 类型。

**6.2、结果如何转换？**

这一过程肯定是发生在执行查询语句的过程中，其中的ResultSetHandler 这个组件就是对查询的结果进行处理的，那么肯定是发生在这一组件中的某个方法。

在PreparedStatementHandler 执行查询结束之后，调用的是ResultSetHandler 中的handleResultSets() 方法，对结果进行处理，如下：

![复制代码](media/复制代码.gif)

public <E> List<E> query(Statement statement, ResultHandler resultHandler) throws SQLException {
     PreparedStatement ps = (PreparedStatement) statement;    // 执 行 SQL   
   ps.execute();    //处理结果
    return resultSetHandler.handleResultSets(ps);  
}

![复制代码](media/复制代码.gif)

最终的在DefaultResultSetHandler 中的getPropertyMappingValue() 方法中调用了TypeHandler 中的getResult() 方法，如下：

![复制代码](media/复制代码.gif)

  private Object getPropertyMappingValue(ResultSet rs, MetaObject metaResultObject, ResultMapping propertyMapping,  
                  ResultLoaderMap lazyLoader, String columnPrefix)throws SQLException {
    if (propertyMapping.getNestedQueryId() != null) {
      return getNestedQueryMappingValue(rs, metaResultObject, propertyMapping, lazyLoader, columnPrefix);
    } else if (propertyMapping.getResultSet() != null) {
      addPendingChildRelation(rs, metaResultObject, propertyMapping);   // TODO is that OK?
      return DEFERRED;
    } else {
      final TypeHandler<?> typeHandler = propertyMapping.getTypeHandler();
      final String column = prependPrefix(propertyMapping.getColumn(), columnPrefix);
      return typeHandler.getResult(rs, column);
    }
  }

![复制代码](media/复制代码.gif)