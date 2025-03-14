# Redis - SpringBoot Integration

## Redis cache and springboot integration guide

docker compose file : 

```yml
services:
    redis:
    image: redis:latest
    restart: always
    ports:
      - "6379:6379"
    environment:
      - REDIS_PASSWORD=my-password
      - REDIS_PORT=6379
      - REDIS_DATABASES=16
```

maven dependencies : 

```
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```


applications property file : 

```yml
spring:
  cache:
    type: redis
    redis:
      host: localhost
      port: 6379

cache:
  config:
    entryTtl: 60
    jwtToken:
      entryTtl: 30
```

In application yml file, cache type and redis host, port informations are defined.

Redis config class

```java
@Configuration
@EnableCaching
@Slf4j
public class RedisConfig {

    @Value("${spring.cache.redis.host}")
    private String redisHost;

    @Value("${spring.cache.redis.port}")
    private int redisPort;

    @Bean
    public LettuceConnectionFactory redisConnectionFactory() {
        RedisStandaloneConfiguration configuration = new RedisStandaloneConfiguration(redisHost, redisPort);
        configuration.setPassword("my-password");
        log.info("Connected to : {} {}", redisHost, redisPort);

        return new LettuceConnectionFactory(configuration);
    }

    @Bean
    public RedisCacheManager cacheManager(RedisConnectionFactory connectionFactory) {
        return RedisCacheManager.create(connectionFactory);
    }
    
    @Bean
    public RedisTemplate<String, Object> redisTemplate(){
        final RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        redisTemplate.setHashKeySerializer(new GenericToStringSerializer<Object>(Object.class));
        redisTemplate.setHashValueSerializer(new JdkSerializationRedisSerializer());
        redisTemplate.setValueSerializer(new StringRedisSerializer());
        redisTemplate.setConnectionFactory(redisConnectionFactory());
        return redisTemplate;
    }
}
```

## Working with Cache Annotations

To enable caching in your application, you can use the @EnableCaching annotation and then annotate methods that should be cached with @Cacheable, @CachePut, and @CacheEvict.


```java
@GetMapping("/product/{id}")  
@Cacheable(value = "product", key = "#id")
public Product getProductById(@PathVariable long id) {...}
```

 1. @Cacheable is employed to fetch data from the database, storing it in the cache. Upon future invocations, the method retrieves the cached value directly, eliminating the need to execute the method again.
The value attribute establishes a cache with a specific name, while the key attribute permits the use of Spring Expression Language to compute the key dynamically. Consequently, the method result is stored in the ‘product’ cache, where respective ‘product_id’ serves as the unique key. This approach optimizes caching by associating each result with a distinct key.
2. @CachePut is used to update data in the cache when there is any update in the source database.

```java
@PutMapping("/product/{id}")
@CachePut(cacheNames = "product", key = "#id")
public Product editProduct(@PathVariable long id, @RequestBody Product product) {...}
```
The cacheNames attribute is an alias for value, and can be used in a similar manner.
3. @CacheEvict is used for removing stale or unused data from the cache.

```java
@DeleteMapping("/product/{id}")
@CacheEvict(cacheNames = "product", key = "#id", beforeInvocation = true)
public String removeProductById(@PathVariable long id) {...}
```

We use cacheName and key to remove specific data from the cache. The beforeInvocation attribute allows us to control the eviction process, enabling us to choose whether the eviction should occur before or after the method execution.


```java
@DeleteMapping("/product/{id}")
@CacheEvict(cacheNames = "product", allEntries = true)
public String removeProductById(@PathVariable long id) {...}
```
Alternatively, all the data can also be removed for a given cache by using the allEntries attribute as true. The annotation allows us to clear data for multiple caches as well by providing multiple values as cacheName.
4. @Caching is used for multiple nested caching on the same method.

```java
@PutMapping("/{id}")
@Caching(
     evict = {@CacheEvict(value = "productList", allEntries = true)},
     put = {@CachePut(value = "product", key = "#id")}
)
public Product editProduct(@PathVariable long id, @RequestBody Product product) {...}
```

5. @CacheConfig is used for centralized configuration.

```java
@CacheConfig(cacheNames = "product")
public class ProductController {...}
```

## Working with RedisTemplate

You can use the RedisTemplate to interact with Redis. For example, to save data in Redis: 

- Save data to Redis

```java
@Autowired
private RedisTemplate<String, Object> redisTemplate;

public void saveData(String key, Object data) {
    redisTemplate.opsForValue().set(key, data);
}
```

- Retrieve data from Redis: 

```java
public Object getData(String key) {
    return redisTemplate.opsForValue().get(key);
}
```

- Example service class with Redis Template

```java

@Service
public class CacheTokenService {

    private final StringRedisTemplate stringRedisTemplate;
    private final RedisTemplate<String, Object> redisTemplate;

    public CacheTokenService(StringRedisTemplate stringRedisTemplate, RedisTemplate<String, Object> redisTemplate) {
        this.stringRedisTemplate = stringRedisTemplate;
        this.redisTemplate = redisTemplate;
    }

    public void saveData(String key, Object data) {
        redisTemplate.opsForValue().set(key, data);
    }

    public Object getData(String key) {
        return redisTemplate.opsForValue().get(key);
    }

    public void cacheToken(String username, String token) {
        stringRedisTemplate
        .opsForValue()
        .set("token:" + username, token, Duration.ofMinutes(60));
    }

    public String getCachedToken(String username) {
        // Retrieve the token from Redis cache
        return stringRedisTemplate
        .opsForValue()
        .get("token:" + username);
    }

}
```

## PUB/SUB Messaging

Adding configurations like : 

```java
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
    public MessageListenerAdapter messageListener( ) {
        return new MessageListenerAdapter(redisMessageHandler);
    }
```

last version of configuration class : 


```java
@Configuration
@EnableCaching
@Slf4j
public class RedisConfig {

    @Value("${spring.cache.redis.host}")
    private String redisHost;

    @Value("${spring.cache.redis.port}")
    private int redisPort;


    private final RedisMessageHandler redisMessageHandler;

    public RedisConfig(RedisMessageHandler redisMessageHandler) {
        this.redisMessageHandler = redisMessageHandler;
    }

    @Bean
    public LettuceConnectionFactory redisConnectionFactory() {
        RedisStandaloneConfiguration configuration = new RedisStandaloneConfiguration(redisHost, redisPort);
        configuration.setPassword("my-password");
        log.info("Connected to : {} {}", redisHost, redisPort);


        return new LettuceConnectionFactory(configuration);
    }

    @Bean
    public RedisCacheManager cacheManager(RedisConnectionFactory connectionFactory) {
        return RedisCacheManager.create(connectionFactory);
    }
    @Bean
    public RedisTemplate<String, Object> redisTemplate(){
        final RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        redisTemplate.setHashKeySerializer(new GenericToStringSerializer<Object>(Object.class));
        redisTemplate.setHashValueSerializer(new JdkSerializationRedisSerializer());
        redisTemplate.setValueSerializer(new StringRedisSerializer());
        redisTemplate.setConnectionFactory(redisConnectionFactory());
        return redisTemplate;
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
    public MessageListenerAdapter messageListener( ) {
        return new MessageListenerAdapter(redisMessageHandler);
    }
}
```

Publisher interface : 

```java
public interface MessagePublisher {
    void publish(String message);
}
```

Publisher service implementation class : 


```java
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.listener.ChannelTopic;
import org.springframework.stereotype.Service;

@Service
@Slf4j
public class RedisMessagePublisher implements MessagePublisher{

    private final RedisTemplate<String, Object> redisTemplate;
    private final ChannelTopic topic;

    public RedisMessagePublisher(RedisTemplate<String, Object> redisTemplate, ChannelTopic topic) {
        this.redisTemplate = redisTemplate;
        this.topic = topic;
    }

    @Override
    public void publish(String message) {
        redisTemplate.convertAndSend(topic.getTopic(), message);
        log.info("Published : {}", topic.getTopic());
    }

}
```


RedisMessageHandler class is the listener class and shown belove : 

```java
@Service
@Slf4j
public class RedisMessageHandler implements MessageListener {

    @Override
    public void onMessage(Message message, byte[] pattern) {
        log.info("Message : {}, from channel : {}", message, message.getChannel());
        // message handle logic
    }
    
}
```

Usage: after injecting `redisMessagePublisher` messages can be published


`redisMessagePublisher.publish("test-message");`