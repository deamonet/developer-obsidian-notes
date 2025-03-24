

GraphQLUtils.java

```java
import lombok.extern.slf4j.Slf4j;  
import org.springframework.beans.factory.annotation.Autowired;  
import org.springframework.stereotype.Component;  
  
import java.lang.reflect.Field;  
import java.lang.reflect.ParameterizedType;  
import java.lang.reflect.Type;  
import java.util.List;  
  
/**  
 * @author yedongdong * date 2024/12/10 16:45 * project electric-price-api * description */@Slf4j  
@Component  
public class GraphQLUtils {  
  
    private static final String LEFT_BRACELET = "{";  
    private static final String RIGHT_BRACELET = "}";  
    private static final String NEW_LINE = "\n";  
    private static BusinessConfig businessConfig;  
  
    @Autowired  
    public void setBusinessConfig(BusinessConfig businessConfig) {  
        GraphQLUtils.businessConfig = businessConfig;  
    }  
  
    /**  
     * Java Bean 转成 GRAPHQL查询语句  
     *  
     * @param clazz  JAVA CLASS  
     * @param indent 缩进  
     * @param <T>  
     * @return GRAPHQL查询语句  
     */  
    public static <T> String generateGraphQLQueryStringFromJavaBean(Class<T> clazz, int indent) {  
        Field[] fields = clazz.getDeclaredFields();  
        List<String> baseTypes = businessConfig.getGraphQLBaseTypes();  
        StringBuilder queryString = new StringBuilder();  
        String indentation = new String(new char[indent]).replace("\0", " ");  
        queryString.append(LEFT_BRACELET);  
        for (Field field : fields) {  
            String fieldName = field.getName();  
            Class<?> fieldType = field.getType();  
            String[] fieldTypeNames = fieldType.getName().split("\\.");  
            String fieldTypeName = fieldTypeNames[fieldTypeNames.length - 1];  
            if ("List".equals(fieldTypeName)) {  
                ParameterizedType parameterizedType = (ParameterizedType) field.getGenericType();  
                Type genericFieldType = parameterizedType.getActualTypeArguments()[0];  
                fieldType = (Class<?>) genericFieldType;  
                fieldTypeName = genericFieldType.getTypeName();  
            }  
  
            queryString.append(NEW_LINE).append(indentation).append(fieldName);  
            if (!baseTypes.contains(fieldTypeName)) {  
                queryString.append(generateGraphQLQueryStringFromJavaBean(fieldType, indent + 2));  
            }  
        }  
        queryString.append(NEW_LINE).append(indentation).append(RIGHT_BRACELET);  
        return queryString.toString();  
    }  
  
    public static void main(String[] args) {  
        GraphQLUtils.businessConfig = new BusinessConfig();  
        System.out.println(generateGraphQLQueryStringFromJavaBean(TibberData.class, 0));  
    }  
}
```

BusinessConfig.java

```java

import lombok.Data;  
import org.springframework.boot.context.properties.ConfigurationProperties;  
import org.springframework.context.annotation.Configuration;  
  
import java.util.Arrays;  
import java.util.List;  
import java.util.Map;  
  
/**  
 * @author tangyifeng * date  2024/8/24 16:14 * description */@Data  
@Configuration  
@ConfigurationProperties(prefix = "business")  
public class BusinessConfig {  
    private List<String> graphQLBaseTypes = Arrays.asList("String", "Long", "Integer", "Double", "Boolean", "Float", "Date", "LocalDateTime", "Instant");  
    private Map<String, String> octopusArea;  
    private String curDate;  
}
```