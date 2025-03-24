 
```java
package com.solax.phoebus.interfaces.serialization;  
  
import com.fasterxml.jackson.core.JsonParser;  
import com.fasterxml.jackson.core.JsonProcessingException;  
import com.fasterxml.jackson.core.JsonToken;  
import com.fasterxml.jackson.databind.DeserializationContext;  
import com.fasterxml.jackson.databind.JsonDeserializer;  
import com.solax.phoebus.v5.constants.ConstantsV5;  
  
  
import java.io.IOException;  
import java.time.LocalTime;  
import java.time.format.DateTimeFormatter;  
  
/**  
 * @author yedongdong  
 * date 2024/01/05 10:21 * description 
 */
public class HourMinuteDeserializer extends JsonDeserializer<LocalTime> {  
  
    @Override  
    public LocalTime deserialize(JsonParser jsonParser, DeserializationContext deserializationContext) throws IOException, JsonProcessingException {  
        JsonToken currentToken = jsonParser.getCurrentToken();  
        if (currentToken.equals(JsonToken.VALUE_STRING)) {  
            String text = jsonParser.getText().trim();  
            DateTimeFormatter formatter = DateTimeFormatter.ofPattern(ConstantsV5.HOUR_MINUTE);  
            return LocalTime.parse(text, formatter);  
        }  
        return null;  
    }  
}
```