# SpringBoot - MongoDB Integration

### Docker compose file

`mongo` and `mongo-express`
```yml
services:
  mongo:
    image: 'mongo:7.0.5'
    ports:
      - 27017:27017
    volumes:
      - my-data:/var/lib/mongodb/data

  mongo-express:
    image: 'mongo-express:1.0.2'
    ports:
      - 8082:8081
    environment:
      ME_CONFIG_BASICAUTH_USERNAME: username
      ME_CONFIG_BASICAUTH_PASSWORD: password
```

### Dependencies

**pom.xml**
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
```

**application.yml** file
```yml
server:
  port: 8080
spring:
  application:
    name: AUTH-SERVICE
  data:
    mongodb:
      uri: mongodb://localhost:27017
```





### Configuration Classes

**MongoConfig**


```java
@Configuration
public class MongoConfig {

    @Value("${spring.data.mongodb.uri}")
    private String connectionString;


    @Bean
    public MongoClient mongoClient() {
        CodecRegistry pojoCodecRegistry = fromProviders(PojoCodecProvider.builder().automatic(true).build());
        CodecRegistry codecRegistry = fromRegistries(MongoClientSettings.getDefaultCodecRegistry(), pojoCodecRegistry);
        return MongoClients.create(MongoClientSettings.builder()
                .applyConnectionString(new ConnectionString(connectionString))
                .codecRegistry(codecRegistry)
                .build());
    }

}
```