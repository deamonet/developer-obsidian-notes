```xml
<?xml version="1.0" encoding="UTF-8"?>  
  
<!-- 从高到低：OFF，FATAL，ERROR，WARN，INFO，DEBUG，TRACE，ALL -->  
<!-- 日志输出规则：输出比当前ROOT级别高的日志，另外亦可考虑filter级别过滤器 -->  
  
<!--  
   scan=true，一旦配置文件发生改变，将会被重新加载，默认为true；  
   scanPeriod，监测配置文件是否有修改的时间间隔，默认单位是毫秒，scan为true才生效，默认的时间间隔为1分钟；  
   debug=true，打印logback内部日志信息，实时查看logback运行状态，默认为false；  
-->  
<configuration scan="true" scanPeriod="60 seconds" debug="false">  
    <!-- 定义日志文件位置 -->  
    <property name="log_dir" value="/solax/application/logs/installer"/>  
    <!-- 日志最大历史30天 -->  
    <property name="maxHistory" value="30"/>  
    <!-- 单个文件最大 -->  
    <property name="maxFileSize" value="100MB" />  
    <!-- 总文件大小 -->  
    <property name="totalSizeCap" value="1GB" />  
  
    <springProperty scope="context" name="appName" source="spring.application.name"/>  
  
    <!-- 控制台输出日志 -->  
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">  
        <!-- 对日志进行格式化 -->  
        <layout class="ch.qos.logback.classic.PatternLayout">  
            <pattern>%d{yyyy-MM-dd HH:mm:ss} [%thread] [%X{x-transaction-id}] %-5level %L %logger - %msg%n</pattern>  
        </layout>        <encoder class="ch.qos.logback.core.encoder.LayoutWrappingEncoder">  
            <charset>UTF-8</charset>  
            <layout class="ch.qos.logback.classic.PatternLayout">  
                <pattern>%blue(%d{yyyy-MM-dd HH:mm:ss.SSS}) %cyan([${appName}]) %yellow([%thread]) %magenta([%X{x-transaction-id}]) %highlight(%-5level) %green(%L %logger) - %msg%n</pattern>  
            </layout>        </encoder>    </appender>  
    <appender name="INFO_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">  
        <file>${log_dir}/info.log</file>  
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">  
            <FileNamePattern>${log_dir}/info-%d{yyyy-MM-dd}-%i.log</FileNamePattern>  
            <maxHistory>${maxHistory}</maxHistory>  
            <maxFileSize>${maxFileSize}</maxFileSize>  
            <totalSizeCap>${totalSizeCap}</totalSizeCap>  
        </rollingPolicy>        <layout class="ch.qos.logback.classic.PatternLayout">  
            <pattern>%d{yyyy-MM-dd HH:mm:ss} [%thread] [%X{x-transaction-id}] %-5level %L %logger - %msg%n</pattern>  
        </layout>        <filter class="ch.qos.logback.classic.filter.LevelFilter">  
            <level>INFO</level>  <!-- 只打印Info日志 -->  
            <onMatch>ACCEPT</onMatch>  
            <onMismatch>DENY</onMismatch>  
        </filter>        <encoder class="ch.qos.logback.core.encoder.LayoutWrappingEncoder">  
            <charset>UTF-8</charset>  
            <layout class="ch.qos.logback.classic.PatternLayout">  
                <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] [%X{x-transaction-id}] %-5level %L %logger - %msg%n</pattern>  
            </layout>        </encoder>    </appender>  
    <appender name="WARN_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">  
        <file>${log_dir}/warn.log</file>  
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">  
            <FileNamePattern>${log_dir}/warn-%d{yyyy-MM-dd}-%i.log</FileNamePattern>  
            <maxHistory>${maxHistory}</maxHistory>  
            <maxFileSize>${maxFileSize}</maxFileSize>  
            <totalSizeCap>${totalSizeCap}</totalSizeCap>  
        </rollingPolicy>        <layout class="ch.qos.logback.classic.PatternLayout">  
            <pattern>%d{yyyy-MM-dd HH:mm:ss} [%thread] [%X{x-transaction-id}] %-5level %L %logger - %msg%n</pattern>  
        </layout>        <filter class="ch.qos.logback.classic.filter.LevelFilter">  
            <level>WARN</level>  <!-- 只打印Warn日志 -->  
            <onMatch>ACCEPT</onMatch>  
            <onMismatch>DENY</onMismatch>  
        </filter>        <encoder class="ch.qos.logback.core.encoder.LayoutWrappingEncoder">  
            <charset>UTF-8</charset>  
            <layout class="ch.qos.logback.classic.PatternLayout">  
                <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] [%X{x-transaction-id}] %-5level %L %logger - %msg%n</pattern>  
            </layout>        </encoder>    </appender>  
    <appender name="ERROR_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">  
        <file>${log_dir}/error.log</file>  
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">  
            <FileNamePattern>${log_dir}/error-%d{yyyy-MM-dd}-%i.log</FileNamePattern>  
            <maxHistory>${maxHistory}</maxHistory>  
            <maxFileSize>${maxFileSize}</maxFileSize>  
            <totalSizeCap>${totalSizeCap}</totalSizeCap>  
        </rollingPolicy>        <layout class="ch.qos.logback.classic.PatternLayout">  
            <pattern>%d{yyyy-MM-dd HH:mm:ss} [%thread] [%X{x-transaction-id}] %-5level %L %logger - %msg%n</pattern>  
        </layout>        <filter class="ch.qos.logback.classic.filter.LevelFilter">  
            <level>ERROR</level>  <!-- 只打印Error日志 -->  
            <onMatch>ACCEPT</onMatch>  
            <onMismatch>DENY</onMismatch>  
        </filter>        <encoder class="ch.qos.logback.core.encoder.LayoutWrappingEncoder">  
            <charset>UTF-8</charset>  
            <layout class="ch.qos.logback.classic.PatternLayout">  
                <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] [%X{x-transaction-id}] %-5level %L %logger - %msg%n</pattern>  
            </layout>        </encoder>    </appender>  
    <appender name="SQL_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">  
        <file>${log_dir}/slow_sql.log</file>  
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">  
            <FileNamePattern>${log_dir}/sql-%d{yyyy-MM-dd}-%i.log</FileNamePattern>  
            <maxHistory>${maxHistory}</maxHistory>  
            <maxFileSize>${maxFileSize}</maxFileSize>  
            <totalSizeCap>${totalSizeCap}</totalSizeCap>  
        </rollingPolicy>        <layout class="ch.qos.logback.classic.PatternLayout">  
            <pattern>%d{yyyy-MM-dd HH:mm:ss} [%thread] [%X{x-transaction-id}] %-5level %L %logger - %msg%n</pattern>  
        </layout>        <filter class="ch.qos.logback.classic.filter.LevelFilter">  
            <level>INFO</level>  
            <onMatch>ACCEPT</onMatch>  
            <onMismatch>DENY</onMismatch>  
        </filter>        <!--日志文件输出格式-->  
        <encoder class="ch.qos.logback.core.encoder.LayoutWrappingEncoder">  
            <charset>UTF-8</charset>  
            <layout class="ch.qos.logback.classic.PatternLayout">  
                <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] [%X{x-transaction-id}] %-5level %L %logger - %msg%n</pattern>  
            </layout>        </encoder>    </appender>  
  
    <!-- 异步输出控制台日志 -->  
    <appender name="CONSOLE_ASYNC" class="ch.qos.logback.classic.AsyncAppender">  
        <!-- 不丢失日志.默认的,如果队列的80%已满,则会丢弃TRACT、DEBUG、INFO级别的日志 -->  
        <discardingThreshold>0</discardingThreshold>  
        <!-- 更改默认的队列的深度,该值会影响性能.默认值为256 -->  
        <queueSize>512</queueSize>  
        <!-- 添加附加的appender,最多只能添加一个 -->  
        <appender-ref ref="CONSOLE"/>  
        <!-- 开启记录信息 不开启不会显示代码行数 -->  
        <includeCallerData>true</includeCallerData>  
    </appender>  
    <!-- 异步输出INFO日志 -->  
    <appender name="INFO_ASYNC" class="ch.qos.logback.classic.AsyncAppender">  
        <!-- 不丢失日志.默认的,如果队列的80%已满,则会丢弃TRACT、DEBUG、INFO级别的日志 -->  
        <discardingThreshold>80</discardingThreshold>  
        <!-- 更改默认的队列的深度,该值会影响性能.默认值为256 -->  
        <queueSize>512</queueSize>  
        <!-- 添加附加的appender,最多只能添加一个 -->  
        <appender-ref ref="INFO_FILE"/>  
        <!-- 开启记录信息 不开启不会显示代码行数 -->  
        <includeCallerData>true</includeCallerData>  
    </appender>  
    <!-- 异步输出WARN日志 -->  
    <appender name="WARN_ASYNC" class="ch.qos.logback.classic.AsyncAppender">  
        <!-- 不丢失日志.默认的,如果队列的80%已满,则会丢弃TRACT、DEBUG、INFO级别的日志 -->  
        <discardingThreshold>80</discardingThreshold>  
        <!-- 更改默认的队列的深度,该值会影响性能.默认值为256 -->  
        <queueSize>512</queueSize>  
        <!-- 添加附加的appender,最多只能添加一个 -->  
        <appender-ref ref="WARN_FILE"/>  
        <!-- 开启记录信息 不开启不会显示代码行数 -->  
        <includeCallerData>true</includeCallerData>  
    </appender>  
    <!-- 异步输出ERROR日志 -->  
    <appender name="ERROR_ASYNC" class="ch.qos.logback.classic.AsyncAppender">  
        <!-- 不丢失日志.默认的,如果队列的80%已满,则会丢弃TRACT、DEBUG、INFO级别的日志 -->  
        <discardingThreshold>0</discardingThreshold>  
        <!-- 更改默认的队列的深度,该值会影响性能.默认值为256 -->  
        <queueSize>512</queueSize>  
        <!-- 添加附加的appender,最多只能添加一个 -->  
        <appender-ref ref="ERROR_FILE"/>  
        <!-- 开启记录信息 不开启不会显示代码行数 -->  
        <includeCallerData>true</includeCallerData>  
    </appender>  
    <!-- 异步输出SQL日志 -->  
    <appender name="SQL_ASYNC" class="ch.qos.logback.classic.AsyncAppender">  
        <!-- 不丢失日志.默认的,如果队列的80%已满,则会丢弃TRACT、DEBUG、INFO级别的日志 -->  
        <discardingThreshold>0</discardingThreshold>  
        <!-- 更改默认的队列的深度,该值会影响性能.默认值为256 -->  
        <queueSize>512</queueSize>  
        <!-- 添加附加的appender,最多只能添加一个 -->  
        <appender-ref ref="SQL_FILE"/>  
        <!-- 开启记录信息 不开启不会显示代码行数 -->  
        <includeCallerData>true</includeCallerData>  
    </appender>  
    <!-- package log level configure -->  
    <logger name="org.springframework" level="WARN"/>  
    <logger name="com.apache.ibatis" level="WARN"/>  
    <logger name="java.sql.Connection" level="WARN"/>  
    <logger name="java.sql.Statement" level="WARN"/>  
    <logger name="java.sql.PreparedStatement" level="WARN"/>  
    <logger name="ch.qos.logback" level="WARN"/>  
    <logger name="com.solax" level="INFO"/>  
    <logger name="com.alibaba.nacos" level="ERROR"/>  
    <!-- SQL日志 additivity="false" 避免重复输入到root节点-->  
    <logger name="SQL_LOG" level="INFO" additivity="false">  
        <appender-ref ref="SQL_ASYNC"/>  
        <appender-ref ref="CONSOLE_ASYNC"/>  
    </logger>  
    <springProfile name="dev">  
        <!-- root级别  -->  
        <root level="INFO">  
            <!-- 控制台输出-->  
            <appender-ref ref="CONSOLE_ASYNC"/>  
            <!-- 异步文件输出 -->  
            <appender-ref ref="INFO_ASYNC"/>  
            <appender-ref ref="WARN_ASYNC"/>  
            <appender-ref ref="ERROR_ASYNC"/>  
        </root>    </springProfile>  
    <springProfile name="test">  
        <!-- root级别  -->  
        <root level="INFO">  
            <!-- 控制台输出-->  
            <appender-ref ref="CONSOLE_ASYNC"/>  
            <!-- 异步文件输出 -->  
            <appender-ref ref="INFO_ASYNC"/>  
            <appender-ref ref="WARN_ASYNC"/>  
            <appender-ref ref="ERROR_ASYNC"/>  
        </root>    </springProfile>  
    <springProfile name="prod">  
        <!-- root级别  -->  
        <root level="ERROR">  
            <!-- 控制台输出-->  
            <appender-ref ref="CONSOLE_ASYNC"/>  
            <!-- 异步文件输出 -->  
<!--            <appender-ref ref="INFO_ASYNC"/>-->  
<!--            <appender-ref ref="WARN_ASYNC"/>-->  
            <appender-ref ref="ERROR_ASYNC"/>  
        </root>    
   </springProfile>
</configuration>
```