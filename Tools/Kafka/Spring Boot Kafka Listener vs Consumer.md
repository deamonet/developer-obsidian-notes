https://stackoverflow.com/questions/55497363/spring-boot-kafka-listener-vs-consumer

The `@KafkaListener` is a high level API for the `ConcurrentMessageListenerContainer`, which spawns several internal listeners around `KafkaConsumer`.

The difference is that that `KafkaConsumer` API is _pollable_ on demand when you call its `poll()` whenever you need. The listener abstraction is about to have an infinite loop around that `poll()` and it produces messages for records whenever they appear from the `poll()`. We have there a task executor which runs a logic like this:

```
        while (isRunning()) {
            try {
                pollAndInvoke();
            }
            catch (@SuppressWarnings(UNUSED) WakeupException e) {
                // Ignore, we're stopping
            }
            catch (NoOffsetForPartitionException nofpe) {
                this.fatalError = true;
                ListenerConsumer.this.logger.error("No offset and no reset policy", nofpe);
                break;
            }
            catch (Exception e) {
                handleConsumerException(e);
            }
            catch (Error e) { // NOSONAR - rethrown
                Runnable runnable = KafkaMessageListenerContainer.this.emergencyStop;
                if (runnable != null) {
                    runnable.run();
                }
                this.logger.error("Stopping container due to an Error", e);
                wrapUp();
                throw e;
            }
        }
```

The `KafkaConsumer.poll()` is called in that `pollAndInvoke();`.