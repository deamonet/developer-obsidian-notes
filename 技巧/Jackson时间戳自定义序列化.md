自定义序列化

```java
package com.solax.installerapp.serialization;  
  
import com.fasterxml.jackson.core.JsonGenerator;  
import com.fasterxml.jackson.databind.JsonSerializer;  
import com.fasterxml.jackson.databind.SerializerProvider;  
  
import java.io.IOException;  
import java.time.Instant;  
import java.time.LocalDateTime;  
import java.time.ZoneId;  
import java.time.format.DateTimeFormatter;  
  
import static com.solax.installerapp.utils.DateUtil.YYYY_MM_DD_HH_MM_SS;  
  
/**  
 * @author yedongdong  
 * date 2023/12/05 16:51 * description 序列化时间戳成格式化字符串  
 */  
public class TimestampSerializer extends JsonSerializer<Long> {  
  
    @Override  
    public void serialize(Long time, JsonGenerator jsonGenerator, SerializerProvider serializerProvider) throws IOException {  
        if (time == null) {  
            jsonGenerator.writeString("");  
            return;        }  
  
        Instant instant = Instant.ofEpochMilli(time);  
        LocalDateTime localDateTime = LocalDateTime.ofInstant(instant, ZoneId.systemDefault());  
        jsonGenerator.writeString(localDateTime.format(DateTimeFormatter.ofPattern(YYYY_MM_DD_HH_MM_SS)));  
    }  
}
```

序列化变量注解：JsonSerialize

```java
package com.solax.installerapp.service.dto.device;  
  
import com.fasterxml.jackson.annotation.JsonIgnore;  
import com.fasterxml.jackson.databind.annotation.JsonSerialize;  
import com.fasterxml.jackson.databind.ser.std.ToStringSerializer;  
import com.solax.enums.UploadInterval;  
import com.solax.installerapp.serialization.TimestampSerializer;  
import lombok.Data;  
import lombok.extern.slf4j.Slf4j;  
  
/**  
 * @author yedongdong  
 * @date 2023/08/25 16:23  
 * @description  
 */  
@Data  
@Slf4j  
public class DeviceDTO {  
  
    @JsonSerialize(using = ToStringSerializer.class)  
    protected Long id;  
    /**  
     * 设备名称  
     */  
    protected String deviceName;  
    protected String sn;  
    protected String wifiSn;  
    @JsonSerialize(using = ToStringSerializer.class)  
    protected Long siteId;  
    protected String siteName;  
    @JsonSerialize(using = ToStringSerializer.class)  
    @JsonIgnore  
    protected String createTime;  
    @JsonSerialize(using = TimestampSerializer.class)  
    protected Long lastUpgradeTime;  
}
```