# Efficient Object Handling Springboot



### Dependency Injection

- Singleton Beans : `@Component` or `@Service`. This scope is ideal for stateless services.
- Prototype Beans : `@Scope("prototype")` for beans that need to have a short lifecycle like request-specific data or temporary calculations

```java
@Component
@Scope("prototype")
public class PrototypeBean {
    // Bean logic
}
```
- Avoid Manual Instantiation : rely on spring to handle bean creation and injection instead of using new keyboard. Consistent management

### Efficient Use of Caching

Implementing caching reduces reduntand computations and database hits, thus optimizing performance.


- Enable caching : Configure Springboot caching by annotaiton methods with @Cachable

```java
@Cacheable("items")
public Item getItemById(Long id) {
    return itemRepository.findById(id).orElseThrow();
}
```

- Cache eviction : clear cache entries when underlying data changes, ensuring data consistency

```java
@CacheEvict(value = "items", allEntries = true)
public void updateItem(Item item) {
    itemRepository.save(item);
}
```

- Fine-tune cache configuration : optimize time-to-live(TTL), maximum size and eviction policies based on application needs
  

### Minimized Object Creation

Object creation can be expensive operation, especially in high-troughput applications. To reduce:
- Object pooling : use object pools for expensive-to-create object, such as database connections or threads. Springboot integrates seamlessly with connection pool libraries like HikariCP
- Reusable objects : reuse immutable objects or constant values where possible. For instance, prefer StrinBuilder over repeatedly concatenating strings.


### Use Lazy Initialization

- Lazy initialization defers the creation of beans until they are needed, saving memory and startup time

```java
@Component
@Lazy
public class ExpensiveBean {
    // Bean logic
}
```
- Lazy loading with hibernate : configure hibernate entities to load data lazily to prevent unnecessary fetching of large datasets.

```java
@Entity
public class User {
    @OneToMany(fetch = FetchType.LAZY, mappedBy = "user")
    private List<Order> orders;
}
```

### Use DTOs and Projections for Data Transfer


- DTOs (Data Transfer Objects): Define lightweight classes for specific use cases.
  
```java
public class UserDTO {
    private String name;
    private String email;

// Getters and Setters
}
```
- Projections : use jpa projections to fetch only the required fields directly from the database

```java
public interface UserProjection {
    String getName();
    String getEmail();
}
```

### Optimize Database Queries

- Pagination and sorting : always use pagination for APIs returning large datasets.
```java
Page<User> users = userRepository.findAll(PageRequest.of(0,10));
```
- Avoid N+1 query problem : use @EntityGraph or join fetch to fetch related data in a single query
```java
@EntityGraph(attributePaths = {"orders"})
List<User> findAllWithOrders();
```

- Batch Processing: For bulk operations, use JPA’s batch processing capabilities.
```java
@Modifying
@Transactional
@Query("update User u set u.status = :status where u.id in :ids")
void bulkUpdateStatus(@Param("status") String status, @Param("ids") List<Long> ids)
```


### Monitor and Profile Memory Usage


- Profiling Tools: Use tools like VisualVM, JProfiler, or IntelliJ Profiler to detect memory leaks or excessive object creation.
- Spring Boot Actuator: Enable Actuator endpoints to gather runtime insights into memory and thread usage.

### Use Efficient Data Structures

- Collections: Use ArrayList for random access and LinkedList for frequent insertions/deletions.
- Maps: Use HashMap or TreeMap based on key lookup or sorting requirements.
- Concurrent Collections: For multithreaded scenarios, prefer ConcurrentHashMap or CopyOnWriteArrayList to ensure thread safety.


### Garbage Collection Optimization

- G1 Garbage Collector: Default in most JVMs, suitable for low-latency applications.
- Monitor GC: Use JVM tools to track GC behavior and optimize heap sizes and generations.
`java -XX:+UseG1GC -Xms512m -Xmx2048m -XX:+PrintGCDetails -jar app.jar`


### Avoid Common Pitfalls
- Circular dependencies : redesign beans or use @Lazy to prevent runtime errors
- Unmanaged resources : always release resources like database connections, file handles, or streams. Utilize try-with-resources for auto management

```java
try (BufferedReader br = new BufferedReader(new FileReader("file.txt"))) {
    // Read file
}
```
- Excessive Bean Initialization: Use conditional bean initialization (@Conditional) to load only the required beans based on the application context.
