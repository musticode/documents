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
* [https://www.youtube.com/watch?v=bSe_JBSk5w4&ab_channel=Theo'sTechTips](https://www.youtube.com/watch?v=bSe_JBSk5w4&ab_channel=Theo%27sTechTips)


## Example tutorial 2

<https://howtodoinjava.com/spring-data/redis-pub-sub-with-spring-boot/>


## Redis config - Subscriber


```java
package com.howtodoinjava.demo.subscriber.pubsub.config;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.listener.ChannelTopic;
import org.springframework.data.redis.serializer.GenericJackson2JsonRedisSerializer;

@Configuration
public class RedisConfiguration {

    @Value("${redis.pubsub.topic:order-events}")
    private String redisPubSubTopic;

    @Bean
    public ChannelTopic topic() {
        return new ChannelTopic(redisPubSubTopic);
    }

    @Bean(name = "pubsubRedisTemplate")
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory connectionFactory) {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(connectionFactory);
        template.setValueSerializer(new GenericJackson2JsonRedisSerializer());
        return template;
    }
}
```


redis subscriber configuration 


```java
package com.howtodoinjava.demo.subscriber.pubsub.config;

import lombok.AllArgsConstructor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.MessageListener;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.listener.ChannelTopic;
import org.springframework.data.redis.listener.RedisMessageListenerContainer;
import org.springframework.data.redis.listener.adapter.MessageListenerAdapter;

@Configuration
@AllArgsConstructor
public class RedisSubscriberConfiguration {

  private final MessageListener messageListener;
  private final RedisConnectionFactory connectionFactory;

  @Bean
  public MessageListenerAdapter messageListenerAdapter() {
    return new MessageListenerAdapter(messageListener);
  }

  @Bean
  public RedisMessageListenerContainer redisContainer(ChannelTopic topic) {
    RedisMessageListenerContainer container = new RedisMessageListenerContainer();
    container.setConnectionFactory(connectionFactory);
    container.addMessageListener(messageListener, topic);
    return container;
  }
}
```


Message Listener service implementation class

```java
package com.howtodoinjava.demo.subscriber.pubsub.subscriber;

import com.fasterxml.jackson.databind.ObjectMapper;
import com.howtodoinjava.demo.model.OrderEvent;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.Message;
import org.springframework.data.redis.connection.MessageListener;
import org.springframework.data.redis.core.RedisTemplate;

import java.io.IOException;
import org.springframework.stereotype.Service;

@Slf4j
@Service
public class OrderEventListener implements MessageListener {

  @Autowired
  private ObjectMapper objectMapper;

  @Autowired
  @Qualifier("pubsubRedisTemplate")
  private RedisTemplate<String, Object> redisTemplate;

  @Override
  public void onMessage(Message message, byte[] pattern) {

    try {
      log.info("New message received: {}", message);
      OrderEvent orderEvent = objectMapper.readValue(message.getBody(), OrderEvent.class);
      redisTemplate.opsForValue().set(orderEvent.getOrderId(), orderEvent);
    } catch (IOException e) {
      log.error("error while parsing message");
    }
  }
}
```


* <https://github.com/lokeshgupta1981/Spring-Boot-Examples/blob/master/spring-redis-pubsub/order-events-subscriber/src/main/java/com/howtodoinjava/demo/subscriber/pubsub/subscriber/OrderEventListener.java>



