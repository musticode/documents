# Redis Pub/Sub Application


---

## Pub/Sub configuration

#### Docker compose file

```yaml
version: '3.8'

services:
  redis:
    image: redis:latest
    container_name: redis
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    command: ["redis-server", "--appendonly yes", "--requirepass", "mysecretpassword"]
    restart: unless-stopped

volumes:
  redis_data:
```

* username : default
* password : mysecretpassword

#### pom.xml file configuration

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

#### application.yml file configuration

```yaml
spring:
  application:
    name: redis-pubsub
  data :
    redis:
      host: localhost
      port: 6379
      password: mysecretpassword
```

#### RedisConfig class

```java
@Configuration
@RequiredArgsConstructor
public class RedisConfig {

    @Value("${spring.data.redis.host}")
    private String redisHost;

    @Value("${spring.data.redis.port}")
    private int redisPort;

    
    private final OrderEventListener orderEventListener;

    @Bean
    public LettuceConnectionFactory redisConnectionFactory() {
        RedisStandaloneConfiguration configuration = new RedisStandaloneConfiguration(redisHost, redisPort);
        configuration.setPassword("mysecretpassword");

        return new LettuceConnectionFactory(configuration);
    }

    @Bean
    public RedisCacheManager cacheManager(RedisConnectionFactory connectionFactory) {
        return RedisCacheManager.create(connectionFactory);
    }
    @Bean
    public RedisTemplate<String, Object> redisTemplate(){
        final RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(redisConnectionFactory());

        // Use JSON serializer for values
        Jackson2JsonRedisSerializer<Object> serializer = new Jackson2JsonRedisSerializer<>(Object.class);
        template.setValueSerializer(serializer);
        template.setHashValueSerializer(serializer);

        // Use String serializer for keys
        template.setKeySerializer(new StringRedisSerializer());
        template.setHashKeySerializer(new StringRedisSerializer());

        template.afterPropertiesSet();

        return template;
    }

    @Bean
    public ChannelTopic channelTopic(){
        return new ChannelTopic("channel_topic");
    }

    @Bean
    public RedisMessageListenerContainer redisMessageListenerContainer(){
        RedisMessageListenerContainer container = new RedisMessageListenerContainer();
        container.setConnectionFactory(redisConnectionFactory());
        container.addMessageListener(messageListener(), channelTopic());
        return container;
    }

    @Bean
    public MessageListenerAdapter messageListener() {
        return new MessageListenerAdapter(orderEventListener);
    }

}
```

* `redisHost` : redis host address, comes from application.yml
* `redisPort` : redis port, comes from application.yml
* `orderEventListener` : comes from OrderEventListener class, injected to RedisConfig. This class will be triggered when redis message is published. OrderEventListener class implements `MessageListener` interface
* `redisConnectionFactory` : redis connection factory, makes connection with redis server
* `cacheManager` : redis cache manager, comes from redis connection factory
* `redisTemplate` : redis template, comes from redis connection factory
* `channelTopic` : redis channel topic, comes from application.yml. This topic will be used to publish and subscribe redis messages, so it can be differ or can be set as a constant
* `redisMessageListenerContainer` : redis message listener container, comes from redis connection factory
* `messageListener` : redis message listener, comes from OrderEventListener class

#### OrderEventListener class

```java
@Service
@RequiredArgsConstructor
@Slf4j
public class RedisMessagePublisher {

    private final RedisTemplate<String, Object> redisTemplate;
    private final ChannelTopic  channelTopic;

    public String publishOrderEvent(@NonNull OrderEvent orderEvent) {
        try {
            redisTemplate.convertAndSend(channelTopic.getTopic(), orderEvent);
            return "Message published successfully";
        } catch (Exception e) {
            log.error("Error occured when publishing message, error : {}",e.getMessage());
            return "Error occured when publishing message";
        }
    }
    
}
```

* `redisTemplate` : redis template, comes from redis connection factory. This template will be used to publish and subscribe redis messages
* `channelTopic` : redis channel topic, comes from the application's class that is injected in RedisConfig. This topic will be used to publish and subscribe redis messages, so it can be differ or can be set as a constant

#### Event listener class

```java
@Service
@RequiredArgsConstructor
@Slf4j
public class OrderEventListener implements MessageListener {

    private final ObjectMapper objectMapper;

    @Override
    public void onMessage(Message message, byte[] pattern) {

        try {
            log.info("onMessage : {}", objectMapper.readValue(message.getBody(), OrderEvent.class));
            final OrderEvent orderEvent = objectMapper.readValue(message.getBody(), OrderEvent.class);
            log.info("onMessage : {}", orderEvent.getPrice());
        } catch (IOException e) {
            log.error(e.getMessage());
        }

    }

}
```

* This class implements `MessageListener` interface to implement and override onMessage method
* `objectMapper` : object mapper, comes from spring framework. This object mapper will be used to convert redis message to java object
* `orderEvent` : order event, comes from redis message
* `readValue` : read value, comes from object mapper. This method will be used to convert redis message to java object


---

## Testing application

* Sending request with simple example order event

```bash
curl --location 'http://localhost:8080/api/events/publish' \
--header 'accept: application/json' \
--header 'x-api-key: 4d73e4a8ce78:c33b1129-16f3-4598-b419-08d678b9a74a' \
--header 'Content-Type: application/json' \
--data '{
    "orderId": 1,
    "userId": 1,
    "productName": "testProduct",
    "price": 122,
    "quantity": 2
}'
```

response :

```json
2025-05-29T00:14:31.471+03:00  INFO 2704 --- [redis-pubsub] [enerContainer-1] c.e.r.service.impl.OrderEventListener    : onMessage : OrderEvent(orderId=1, userId=1, productName=testProduct, price=122, quantity=2)
2025-05-29T00:14:31.474+03:00  INFO 2704 --- [redis-pubsub] [enerContainer-1] c.e.r.service.impl.OrderEventListener    : onMessage : 122
```


* <https://github.com/musticode/redis-pubsub/blob/main/readme.md>


