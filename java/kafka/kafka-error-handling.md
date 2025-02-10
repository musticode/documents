# Kafka Error Handling



Kafka provides mechanisms for handling failures using Dead Letter Queues (DLQ) and retries.


```java
@KafkaListener(topics = "my-topic", groupId = "my-group", errorHandler = "customErrorHandler")
public void listen(String message) {
    throw new RuntimeException("Error processing message!");
}

@Bean
public KafkaListenerErrorHandler customErrorHandler() {
    return (m, e) -> {
        System.out.println("Error handling message: " + m.getPayload());
        return null; // Custom retry or DLQ logic
    };
}
```

