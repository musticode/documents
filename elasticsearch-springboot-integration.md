# SpringBoot ElasticSearch Integration
### Docker
> After creating docker compose file, `docker compose up -d`  in folder path. ElasticSearch will be started.


docker-compose.yml file:

```
  elasticsearch:
    image: elasticsearch:8.8.0
    ports:
      - 9200:9200
      - 9300:9300
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false
```

### SpringBoot
> Below dependencies will be added to spring boot pom.xml file

dependency:

```
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-elasticsearch</artifactId>
		</dependency>

		<dependency>
			<groupId>org.elasticsearch.client</groupId>
			<artifactId>elasticsearch-rest-high-level-client</artifactId>
			<version>7.17.14</version> <!-- Use the appropriate version for your Elasticsearch cluster -->
		</dependency>
		<dependency>
			<groupId>org.apache.httpcomponents</groupId>
			<artifactId>httpclient</artifactId>
			<version>4.5.13</version> <!-- Use a version compatible with the Elasticsearch client -->
		</dependency>
```
**These httpclient and rest-high level-client dependencies are added to solve the incopatible version problems**

> Configuration class will be implemented as:

ElasticSearchConfig.java:

```
@Configuration
@EnableElasticsearchRepositories(basePackages = "com.example.shoppingcart.repository.es")
public class ElasticSearchConfig {

    @Bean
    public RestClient getRestClient() {
        RestClient restClient = RestClient.builder(
                new HttpHost("localhost", 9200)).build();
        return restClient;
    }

    @Bean
    public ElasticsearchTransport getElasticsearchTransport() {
        return new RestClientTransport(
                getRestClient(), new JacksonJsonpMapper());
    }

    @Bean
    public ElasticsearchClient getElasticsearchClient(){
        ElasticsearchClient client = new ElasticsearchClient(getElasticsearchTransport());
        return client;
    }
}

```
`basePackages` parameter should be implemented as elastic repository path. 

> Model class will be implemented as:

```
@Data
@Builder
@AllArgsConstructor
@NoArgsConstructor
@Document(indexName = "carts")
public class CartEs {
    @Id
    @GeneratedValue(strategy = GenerationType.UUID)
    private String id;

    @Field(type = FieldType.Text)
    private String name;

    @Field(type = FieldType.Integer)
    private int itemCount;

}
```



> Repository interface class will be implemented as:

```
@Repository
public interface CartEsRepository extends ElasticsearchRepository<CartEs, String> {
}

```

