# [When Spring Boot starts up, it throws the "method names must be tokens" exception](https://stackoverflow.com/questions/38891866/when-spring-boot-starts-up-it-throws-the-method-names-must-be-tokens-exceptio)

[Ask Question](https://stackoverflow.com/questions/ask)

Asked 8 years, 3 months ago

Modified [3 months ago](https://stackoverflow.com/questions/38891866/when-spring-boot-starts-up-it-throws-the-method-names-must-be-tokens-exceptio?lastactivity "2024-08-08 16:45:40Z")

Viewed 179k times

81

[](https://stackoverflow.com/posts/38891866/timeline)

When Spring Boot starts up, it throws the `method names must be tokens` exception

```java
2016-08-11 16:53:54.499  INFO 14212 --- [0.1-8888-exec-1] o.apache.coyote.http11.Http11Processor   : Error parsing HTTP request header
 Note: further occurrences of HTTP header parsing errors will be logged at DEBUG level.

java.lang.IllegalArgumentException: Invalid character found in method name. HTTP method names must be tokens
        at org.apache.coyote.http11.Http11InputBuffer.parseRequestLine(Http11InputBuffer.java:462) ~[tomcat-embed-core-8.5.4.jar!/:8.5.4]
        at org.apache.coyote.http11.Http11Processor.service(Http11Processor.java:994) ~[tomcat-embed-core-8.5.4.jar!/:8.5.4]
        at org.apache.coyote.AbstractProcessorLight.process(AbstractProcessorLight.java:66) [tomcat-embed-core-8.5.4.jar!/:8.5.4]
        at org.apache.coyote.AbstractProtocol$ConnectionHandler.process(AbstractProtocol.java:785) [tomcat-embed-core-8.5.4.jar!/:8.5.4]
        at org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.doRun(NioEndpoint.java:1425) [tomcat-embed-core-8.5.4.jar!/:8.5.4]
        at org.apache.tomcat.util.net.SocketProcessorBase.run(SocketProcessorBase.java:49) [tomcat-embed-core-8.5.4.jar!/:8.5.4]
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142) [na:1.8.0_72]
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617) [na:1.8.0_72]
        at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61) [tomcat-embed-core-8.5.4.jar!/:8.5.4]
        at java.lang.Thread.run(Thread.java:745) [na:1.8.0_72]

2016-08-11 16:53:58.885  INFO 14212 --- [0.1-8888-exec-2] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring FrameworkServlet 'dispatcherServlet'
2016-08-11 16:53:58.888  INFO 14212 --- [0.1-8888-exec-2] o.s.web.servlet.DispatcherServlet        : FrameworkServlet 'dispatcherServlet': initialization started
2016-08-11 16:53:58.922  INFO 14212 --- [0.1-8888-exec-2] o.s.web.servlet.DispatcherServlet        : FrameworkServlet 'dispatcherServlet': initialization completed in 30 ms
11111111-chinadfadf-xxxxxxxx@121.com
```


144

[](https://stackoverflow.com/posts/41728777/timeline)

This exception can occur when you try to execute an HTTPS request from a client on an endpoint that isn't HTTPS enabled. The client will encrypt request data when the server is expecting raw data.

Change `https://` to `http://` in your client url.