[](https://stackoverflow.com/posts/40941845/timeline)

I'm writing a Spring-Boot application to monitor a directory and process files that are being added to it. I start a thread by creating a [ApplicationRunner](http://docs.spring.io/spring-boot/docs/current/api/index.html?org/springframework/boot/ApplicationRunner.html) in my `Application` class that calls a method annotated with `@Async`:

```java
@SpringBootApplication
@EnableAsync
public class Application {

    @Autowired
    private DirectoryMonitorService directoryMonitorService;

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

    @Bean
    public ApplicationRunner startDirectoryMonitorService() {
        return args -> directoryMonitorService.monitorSourceDirectoty();
    }
}
```

Here is the code for `DirectoryMonitorService` that has a method annotated with `@Async`:

```java
@Service
public class DirectoryMonitorService {

    private static final Logger logger = LogManager.getLogger(DirectoryMonitorService.class);

    @Value("${timeout}")
    private long timeout;

    @Autowired
    private WatchService watchService;

    @Async
    public void monitorSourceDirectoty() {
        while (true) {
            WatchKey watchKey;
            try {
                watchKey = watchService.poll(timeout, TimeUnit.SECONDS);
            } catch (ClosedWatchServiceException | InterruptedException e) {
                logger.error("Exception occured while polling from source file", e);
                return;
            }

            // process the WatchEvents

            if (!watchKey.reset()) {
                break;
            }
        }
    }
}
```

Finally here is where I create the [ThreadPoolTaskExecutor](http://docs.spring.io/spring/docs/current/javadoc-api/index.html?org/springframework/scheduling/concurrent/ThreadPoolTaskExecutor.html):

```java
public class AsyncConfig extends AsyncConfigurerSupport {

    private static final Logger logger = LogManager.getLogger(AsyncConfig.class);

    private static final String THREAD_NAME_PREFIX = "Parser-";

    @Value("${corePoolSize}")
    public int corePoolSize;

    @Value("${maxPoolSize}")
    public int maxPoolSize;

    @Value("${queueCapacity}")
    public int queueCapacity;

    @Override
    public Executor getAsyncExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(corePoolSize);
        executor.setMaxPoolSize(maxPoolSize);
        executor.setQueueCapacity(queueCapacity);
        executor.setThreadNamePrefix(THREAD_NAME_PREFIX);
        executor.initialize();

        return executor;
    }

    @Override
    public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
        return (Throwable ex, Method method, Object... params) -> {
            logger.error("Exception message - " + ex.getMessage());
            logger.error("Method name - " + method.getName());
            for (Object param : params) {
                logger.error("Parameter value - " + param);
            }
        };
    }
}
```

Somehow I feel this is not most elegant way of starting a main thread. Does anybody have a better solution?

Also I would rather have replace `while (true)` with a `Boolean` variable that I can set to false when Spring-Boot shuts down. Does anybody know which interface I need to implement for this?

-   [java](https://stackoverflow.com/questions/tagged/java "show questions tagged 'java'")
-   [multithreading](https://stackoverflow.com/questions/tagged/multithreading)
-   [spring-boot](https://stackoverflow.com/questions/tagged/spring-boot "show questions tagged 'spring-boot'")