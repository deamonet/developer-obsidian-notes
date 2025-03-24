```java
package com.solax.installerapp.interceptors;  
  
import com.solax.installerapp.config.SqlLogConfig;  
import com.solax.installerapp.utils.SqlContext;  
import com.solax.installerapp.utils.ValidateUtil;  
import org.apache.ibatis.executor.statement.StatementHandler;  
import org.apache.ibatis.mapping.BoundSql;  
import org.apache.ibatis.mapping.MappedStatement;  
import org.apache.ibatis.mapping.ParameterMapping;  
import org.apache.ibatis.plugin.*;  
import org.apache.ibatis.reflection.MetaObject;  
import org.apache.ibatis.reflection.SystemMetaObject;  
import org.apache.ibatis.session.Configuration;  
import org.apache.ibatis.session.ResultHandler;  
import org.apache.ibatis.type.TypeHandlerRegistry;  
import org.springframework.stereotype.Component;  
  
import javax.annotation.Resource;  
import java.sql.Statement;  
import java.text.DateFormat;  
import java.util.Date;  
import java.util.List;  
import java.util.Locale;  
import java.util.regex.Matcher;  
  
  
/**  
 * @author Qiu  
 * @version 1.0  
 * @Date 2023/8/31  
 * @description :  拦截Sql语法构建 获取执行完整SQL语句  
 */  
@Component  
@Intercepts({  
        @Signature(type = StatementHandler.class, method = "batch", args = {Statement.class}),  
        @Signature(type = StatementHandler.class, method = "update", args = {Statement.class}),  
        @Signature(type = StatementHandler.class, method = "query", args = {Statement.class, ResultHandler.class}),  
        @Signature(type = StatementHandler.class, method = "queryCursor", args = {Statement.class})  
})  
public class SqlStatementInterceptor implements Interceptor {  
  
    @Resource  
    private SqlLogConfig sqlLogConfig;  
  
    @Override  
    public Object intercept(Invocation invocation) throws Throwable {  
        try {  
            return invocation.proceed();  
        } finally {  
            if (sqlLogConfig.isEnabled()) {  
                StatementHandler statementHandler = (StatementHandler) invocation.getTarget();  
                MetaObject metaObjectHandler = SystemMetaObject.forObject(statementHandler);  
                // 获取查询接口映射的相关信息  
                MappedStatement mappedStatement = (MappedStatement) metaObjectHandler.getValue("delegate.mappedStatement");  
                // 获取请求时的参数  
                Object parameterObject = statementHandler.getParameterHandler().getParameterObject();  
                // 获取sql  
                String sql = showSql(mappedStatement.getConfiguration(), mappedStatement.getBoundSql(parameterObject));  
                String[] path = mappedStatement.getId().split("\\.");  
                String sqlName = path[path.length - 2] + "#" + path[path.length - 1];  
                SqlContext.setSql(sqlName + "=>" + sql);  
            }  
        }  
    }  
  
    @Override  
    public Object plugin(Object target) {  
        if (target instanceof StatementHandler) {  
            return Plugin.wrap(target, this);  
        }  
        return target;  
    }  
  
  
    private static String showSql(Configuration configuration, BoundSql boundSql) {  
        try {  
            // 获取参数  
            Object parameterObject = boundSql.getParameterObject();  
            List<ParameterMapping> parameterMappings = boundSql.getParameterMappings();  
            // sql语句中多个空格都用一个空格代替  
            String sql = boundSql.getSql().replaceAll("[\\s]+", " ");  
            if (ValidateUtil.isNotEmpty(parameterMappings) && parameterObject != null) {  
                // 获取类型处理器注册器，类型处理器的功能是进行java类型和数据库类型的转换  
                TypeHandlerRegistry typeHandlerRegistry = configuration.getTypeHandlerRegistry();  
                // 如果根据parameterObject.getClass(）可以找到对应的类型，则替换  
                if (typeHandlerRegistry.hasTypeHandler(parameterObject.getClass())) {  
                    sql = sql.replaceFirst("\\?", Matcher.quoteReplacement(getParameterValue(parameterObject)));  
                } else {  
                    // MetaObject主要是封装了originalObject对象，提供了get和set的方法用于获取和设置originalObject的属性值,主要支持对JavaBean、Collection、Map三种类型对象的操作  
                    MetaObject metaObject = configuration.newMetaObject(parameterObject);  
                    for (ParameterMapping parameterMapping : parameterMappings) {  
                        String propertyName = parameterMapping.getProperty();  
                        if (metaObject.hasGetter(propertyName)) {  
                            Object obj = metaObject.getValue(propertyName);  
                            sql = sql.replaceFirst("\\?", Matcher.quoteReplacement(getParameterValue(obj)));  
                        } else if (boundSql.hasAdditionalParameter(propertyName)) {  
                            // 该分支是动态sql  
                            Object obj = boundSql.getAdditionalParameter(propertyName);  
                            sql = sql.replaceFirst("\\?", Matcher.quoteReplacement(getParameterValue(obj)));  
                        } else {  
                            // 打印出缺失，提醒该参数缺失并防止错位  
                            sql = sql.replaceFirst("\\?", "缺失");  
                        }  
                    }  
                }  
            }  
            return sql;  
        } catch (Exception exception) {  
            return "get Sql error";  
        }  
    }  
  
    private static String getParameterValue(Object obj) {  
        String value;  
  
        if (obj instanceof String) {  
            value = "'" + obj + "'";  
        } else if (obj instanceof Date) {  
            DateFormat formatter = DateFormat.getDateTimeInstance(DateFormat.DEFAULT, DateFormat.DEFAULT, Locale.CHINA);  
            value = "'" + formatter.format(new Date()) + "'";  
        } else {  
            if (obj != null) {  
                value = obj.toString();  
            } else {  
                value = "";  
            }  
        }  
        return value;  
    }  
}
```


```java
package com.solax.installerapp.config;  
  
import lombok.Data;  
import org.springframework.boot.context.properties.ConfigurationProperties;  
import org.springframework.context.annotation.Configuration;  
  
@Data  
@Configuration  
@ConfigurationProperties(prefix = "sqllog")  
public class SqlLogConfig {  
  
    /**  
     * 是否开启记录SQL日志，默认为false.  
     */    private boolean enabled = true;  
  
    /**  
     * 记录执行时间超过多少毫秒的语句，默认0，记录所有语句.  
     */    private int minCost = 0;  
}
```