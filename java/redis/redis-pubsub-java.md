# Redis PubSub messaging integration Springboot

## Configuration class:

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.data.redis.listener.PatternTopic;
import org.springframework.data.redis.listener.RedisMessageListenerContainer;
import org.springframework.data.redis.listener.adapter.MessageListenerAdapter;

@Configuration
public class RedisConfig {
    @Bean
    public RedisMessageListenerContainer container(RedisConnectionFactory connectionFactory, MessageListenerAdapter listenerAdapter) {
        RedisMessageListenerContainer container = new RedisMessageListenerContainer();
        container.setConnectionFactory(connectionFactory);
        container.addMessageListener(listenerAdapter, new PatternTopic(redisChannel()));

        return container;
    }

    @Bean
    public MessageListenerAdapter listenerAdapter(RedisReceiver receiver) {
        return new MessageListenerAdapter(receiver, "receiveMessage");
    }

    @Bean
    public StringRedisTemplate template(RedisConnectionFactory connectionFactory) {
        return new StringRedisTemplate(connectionFactory);
    }

    @Bean
    public String redisChannel() {
        return "exampleapp";
    }
}
```


## Redis message service

```java
@Component
public class RedisMessagingService {
    @Autowired
    private StringRedisTemplate redisTemplate;

    @Autowired
    private String redisChannel;

    public void sendMessage(String message) {
        redisTemplate.convertAndSend(redisChannel, message);
    }
}
```


<<<<<<< HEAD
* <https://github.com/math-boy11/yt-redis-pubsub-demo-spring-boot/blob/main/src/main/java/com/mathboy11/RedisMessagingService.java>
* <https://www.youtube.com/watch?v=bSe_JBSk5w4&ab_channel=Theo%27sTechTips>


=======
>>>>>>> 3af173d48559c35639b6bc82956e1ce2189bd5b3
