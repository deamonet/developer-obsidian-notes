```java

import com.fasterxml.jackson.core.JsonParser;  
import com.fasterxml.jackson.core.JsonProcessingException;  
import com.fasterxml.jackson.databind.DeserializationContext;  
import com.fasterxml.jackson.databind.JsonNode;  
import com.fasterxml.jackson.databind.deser.std.StdDeserializer;  
import lombok.extern.slf4j.Slf4j;  
  
import java.io.IOException;  
import java.time.ZoneId;  
  
/**  
 * @author yedongdong * 
 * date 2024/12/10 20:13 
 * project electric-price-api * description 
*/
@Slf4j  
public class TimeZoneDeserializer extends StdDeserializer<ZoneId> {  
  
    public TimeZoneDeserializer() {  
        this(null);  
    }  
  
    public TimeZoneDeserializer(Class<?> vc) {  
        super(vc);  
    }  
  
    @Override  
    public ZoneId deserialize(JsonParser jp, DeserializationContext ctxt) throws IOException, JsonProcessingException {
    // 这里的JsonNode就是指加上@JsonDeserialize(using = TimeZoneDeserializer.class)的变量
        JsonNode node = jp.getCodec().readTree(jp);  
        try {  
            return ZoneId.of(node.asText());  
        } catch (Exception e) {  
            log.error("反序列化时区错误,node={}", node, e);  
            return ZoneId.of("Europe/Berlin");  
        }  
    }  
}
```