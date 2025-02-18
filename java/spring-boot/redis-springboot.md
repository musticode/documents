
### Redis — SpringBoot

docker-compose.yml

```yml
    version: '3.8'  
    services:  
      cache:  
        image: redis:6.2-alpine  
        restart: always  
        ports:  
          - '6379:6379'  
        command: redis-server --save 20 1 --loglevel warning --requirepass eYVX7EwVmmxKPCDmwMtyKVge8oLd2t81  
        volumes:  
          - cache:/data  
    volumes:  
      cache:  
        driver: local
```

pom.xml dependencies:

```xml
<dependency>  
  <groupId>org.springframework.boot</groupId>  
  <artifactId>spring-boot-starter-data-redis</artifactId>  
</dependency>
```

SpringBoot application.properties file:
```
    redis.host=localhost  
    redis.port=6379
```

RedisConfiguration class:

  
```java
@Configuration  
@EnableCaching  
public class RedisConfiguration {  
  
    @Value("${redis.host}")  
    private String redisHost;  
  
    @Value("${redis.port}")  
    private int redisPort;  
  
    @Bean  
    public LettuceConnectionFactory redisConnectionFactory() {  
        RedisStandaloneConfiguration configuration = new RedisStandaloneConfiguration(redisHost, redisPort);  
        return new LettuceConnectionFactory(configuration);  
    }  
  
    @Bean  
    public RedisCacheManager cacheManager(RedisConnectionFactory connectionFactory) {  
        return RedisCacheManager.create(connectionFactory);  
    }  
  
}
```

After configurations are set, simple CRUD implementation is done below

![](https://cdn-images-1.medium.com/max/800/1*ArzG8g3EcNlRpOIMeWXRzg.png)

basic model class

![](https://cdn-images-1.medium.com/max/800/1*OmCdxwatJMdtipyS7pTZQg.png)

repository

![](https://cdn-images-1.medium.com/max/800/1*u3WgPGmu52jyblqE78DVOA.png)

service

  

![](https://cdn-images-1.medium.com/max/800/1*e_s7caljRqDWRBiS3dZHQA.png)

controller

Testing with postman:

![](https://cdn-images-1.medium.com/max/800/1*jE9VYT7KQTM457r5rJffSw.png)

first test

second test:

![](https://cdn-images-1.medium.com/max/800/1*avKYR5B4v-L79bn42nBbKw.png)

cached

Additionally, update cache and delete cache variables

```
    @CachePut(value = "book", key = "#bookId")  
    public Book updateBook(long bookId, Book book) {  
      //update codes  
    }
    
      
    @CacheEvict(value = "book", key="#bookId")  
    public long deleteBook(long bookId) {  
      //delete codes  
    }
```