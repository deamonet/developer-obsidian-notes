以前用Mybatis插件的形式写了一个[数据脱敏工具](https://cloud.tencent.com/developer/tools/blog-entry?target=http%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzUzMzQ2MDIyMA%3D%3D%26mid%3D2247486391%26idx%3D1%26sn%3Dde5b5d6caf97d5e9aa5d35fa90ea62f3%26chksm%3Dfaa2ee24cdd567320aa12a1f6887785faeed43716d331b2ff932d1e2d8332a8a573b294bc54c%26scene%3D21%23wechat_redirect&objectId=1838819&objectType=1&isNewArticle=undefined)，但是发现有一定的局限性。很多时候我们从ORM查询到的数据有其它逻辑要处理，比如根据电话号查询用户信息，你脱敏了就没有办法来处理该逻辑了。所以脱敏这个步骤需要后置，放在JSON序列化这个阶段比较合适。今天就来实现这个功能。

### **Jackson序列化中脱敏**

改造过程其实就是把脱敏后置到JSON序列化过程中，这里我使用Jackson类库。

原来Mybatis插件中的脱敏注解是这样的：

代码语言：javascript

复制

```javascript
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD)
public @interface Sensitive {
    SensitiveStrategy strategy();
}
```

脱敏的策略是这样的：

代码语言：javascript

复制

```javascript
import java.util.function.Function;

/**
 * 脱敏策略.
 *
 * @author felord.cn
 * @since 11 :25
 */
public enum SensitiveStrategy {
    /**
     * Username sensitive strategy.
     */
    USERNAME(s -> s.replaceAll("(\\S)\\S(\\S*)", "$1*$2")),
    /**
     * Id card sensitive type.
     */
    ID_CARD(s -> s.replaceAll("(\\d{4})\\d{10}(\\w{4})", "$1****$2")),
    /**
     * Phone sensitive type.
     */
    PHONE(s -> s.replaceAll("(\\d{3})\\d{4}(\\d{4})", "$1****$2")),
    /**
     * Address sensitive type.
     */
    ADDRESS(s -> s.replaceAll("(\\S{3})\\S{2}(\\S*)\\S{2}", "$1****$2****"));


    private final Function<String, String> desensitizer;

    SensitiveStrategy(Function<String, String> desensitizer) {
        this.desensitizer = desensitizer;
    }

    public Function<String, String> desensitizer() {
        return desensitizer;
    }
}
```

我将改造它们，让它们在JSON序列化中实现字段属性脱敏。

#### **自定义脱敏序列化**

这里我们首先实现自定义的脱敏序列化逻辑:


```java
import com.fasterxml.jackson.core.JsonGenerator;
import com.fasterxml.jackson.databind.BeanProperty;
import com.fasterxml.jackson.databind.JsonMappingException;
import com.fasterxml.jackson.databind.JsonSerializer;
import com.fasterxml.jackson.databind.SerializerProvider;
import com.fasterxml.jackson.databind.ser.ContextualSerializer;

import java.io.IOException;
import java.util.Objects;

/**
 * @author felord.cn
 * @since 1.0.8.RELEASE
 */
public class SensitiveJsonSerializer extends JsonSerializer<String> implements ContextualSerializer {
    private SensitiveStrategy strategy;

    @Override
    public void serialize(String value, JsonGenerator gen, SerializerProvider serializers) throws IOException {
           gen.writeString(strategy.desensitizer().apply(value));
    }

    @Override
    public JsonSerializer<?> createContextual(SerializerProvider prov, BeanProperty property) throws JsonMappingException {

            Sensitive annotation = property.getAnnotation(Sensitive.class);
            if (Objects.nonNull(annotation)&&Objects.equals(String.class, property.getType().getRawClass())) {
                this.strategy = annotation.strategy();
                return this;
            }
            return prov.findValueSerializer(property.getType(), property);

    }
}
```

其中`createContextual`方法用来获取实体类上的`@Sensitive`注解并根据条件初始化对应的`JsonSerializer`对象；而顾名思义，`serialize`方法执行脱敏序列化逻辑。

#### **改造脱敏注解**

然后就是改造`@Sensitive`，把上面自定义的JSON序列化和脱敏策略绑定到一起。这里用到了Jackson的捆绑注解`@JacksonAnnotationsInside`，它的作用是将多个注解组合到一起；另外一个是序列化注解`@JsonSerialize`，它的作用是声明使用我上面自定义的序列化方法。

代码语言：javascript

复制

```javascript
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD)
@JacksonAnnotationsInside
@JsonSerialize(using = SensitiveJsonSerializer.class)
public @interface Sensitive {
    SensitiveStrategy strategy();
}
```

#### **使用**

这就改造完成了，我们来试一试。我们定义一个需要脱敏的实体类并根据字段标记上对应的脱敏注解：

代码语言：javascript

复制

```javascript
/**
 * @author felord.cn
 * @since 1.0.8.RELEASE
 */
@Data
public class User {

    /**
     * 真实姓名
     */
    @Sensitive(strategy = SensitiveStrategy.USERNAME)
    private String realName;
    /**
     * 地址
     */
    @Sensitive(strategy = SensitiveStrategy.ADDRESS)
    private String address;
    /**
     * 电话号码
     */
    @Sensitive(strategy = SensitiveStrategy.PHONE)
    private String phoneNumber;
    /**
     * 身份证号码
     */
    @Sensitive(strategy = SensitiveStrategy.ID_CARD)
    private String idCard;
}
```

然后Jackson来序列化`User`实例：

代码语言：javascript

复制

```javascript
        User user = new User();

        user.setRealName("张三丰");
        user.setPhoneNumber("13333333333");
        user.setAddress("湖北省十堰市丹江口市武当山");
        user.setIdCard("4333333333334334333");

        ObjectMapper objectMapper = new ObjectMapper();

        String json = objectMapper.writeValueAsString(user);

        System.out.println(json);
```

可以得到：

代码语言：javascript

复制

```javascript
{
    "realName":"张*丰",
    "address":"湖北省****市丹江口市武****",
    "phoneNumber":"133****3333",
    "idCard":"4333****34333"
}
```

效果还是可以的，当然如果能加个开关就更好了，根据不同的场景来决定是否进行脱敏。这个以后在研究研究，好了今天的分享就到这里