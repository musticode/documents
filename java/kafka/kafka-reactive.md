# Kafka Reactive

- **dependency**

```xml
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
</dependency>
```

- **Reactive Kafka Producer (Using Project Reactor)**

```java
@Service
public class KafkaProducerService {
    private final ReactiveKafkaProducerTemplate<String, String> reactiveKafkaTemplate;

    public KafkaProducerService(ReactiveKafkaProducerTemplate<String, String> reactiveKafkaTemplate) {
        this.reactiveKafkaTemplate = reactiveKafkaTemplate;
    }
    public Mono<SenderResult<Void>> sendMessageAsync(String topic, String message) {
        return reactiveKafkaTemplate.send(topic, message);
    }
}
```

- **Kafka Consumer (Manual Offset Commit for At-Least-Once)**

```java
@Service
public class KafkaBatchConsumerService {
    @KafkaListener(topics = "my-topic", groupId = "my-group")
    public void listen(@Payload List<String> messages, Acknowledgment acknowledgment) {
        messages.forEach(System.out::println);
        acknowledgment.acknowledge(); // Manual Offset Commit
    }
}
```
Manual offset commits are important for ensuring “at-least-once” processing in scenarios where duplicate messages need to be handled.


**Confluent Schema Registry (Avro/Protobuf for Strongly Typed Messages)**

```java
@Bean
public KafkaAvroSerializer kafkaAvroSerializer() {
    return new KafkaAvroSerializer();
}
```

**Kafka Streams Example**

```java
@Configuration
@EnableKafkaStreams
public class KafkaStreamsConfig {
    @Bean
    public KStream<String, String> kStream(StreamsBuilder builder) {
        KStream<String, String> stream = builder.stream("input-topic");
        stream.mapValues(value -> value.toUpperCase()).to("output-topic");
        return stream;
    }
}
```


**Kafka Transactional Producer (Exactly-Once Processing with EOS v2)**

```java
@Bean
public KafkaTemplate<String, String> transactionalKafkaTemplate() {
    KafkaTemplate<String, String> kafkaTemplate = new KafkaTemplate<>(producerFactory());
    kafkaTemplate.setTransactionIdPrefix("txn-");
    return kafkaTemplate;
}
```


**Idempotent Producer Settings (Preventing Duplicate Messages)**

```java
@Bean
public ProducerFactory<String, String> producerFactory() {
    Map<String, Object> configProps = new HashMap<>();
    configProps.put(ProducerConfig.ENABLE_IDEMPOTENCE_CONFIG, true);
    return new DefaultKafkaProducerFactory<>(configProps);
}
```

## Kafka detailed 

[Detailed Medium link](https://medium.com/coding-odyssey/kafka-with-spring-boot-deep-dive-with-interview-questions-1fad98f7883b)