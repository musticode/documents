# Asyn Configuration in Springboot

## Understanding @Async in springboot

### Asynchronous approach : 
- with the high development of hardware & software modern applications, become much more complex and demanding.

### @Async in spring
- The async annotaiton in spring enables asynchronous processing of a method call. It instructs the framework to execute the method in a sepereate thread, allowing the caller to proceed without waiting for the methot to complete. This improves the overall responssiveness and throughput of an application. 
- To use @Async, you must first enable asynchous processing in your application by adding the @EnableAsync annotation to a configuration class :

```java
@Configuration
@EnableAsync
public class AsyncConfig{
}
```
- Next, annotate the method you want to execute asnychronously with the async annotation,
```java
@Service
public class AsyncService{

    @Async
    public void asyncMethod(){

    }
}
```
### How is @Async different from multithreading and conccurrency
- Sometimes it might seem confusing to differentiate multithreading and concurrency from parallel execution however, both are related to parallel execution. Each of them has their use case and implementation 
- @Async annotation is Spring Framework-specific abstraction, which enables asynchronous execution. It gives the ability to use async with ease, handling all hard work in the background, such as thread creation, management, and execution. This allows users to focus on business logic rather than low-level details.
- Multithreading is a general concept, commonly referring to the ability of an OS or program to manage multiple threads concurrently. As @Async helps us to do all hard work automatically, in this case, we can handle all this work manually and create a multithreading environment. Java has necessary classes such as Thread and ExecutorService to create and work with multithreading.
- Concurrency is a much broader concept, and it covers both multithreading and parallel execution techniques. It is the
ability of a system to execute multiple tasks simultaneously, on one or more processors across.

In summary, async is a higher level abstraction that simplifies asynchronous processing for devs, on the other hand, multithreading and concurrency are more about to manual management of parallel execution


### When to use @Async and whenm to avoid it
**Use @Async when:**

- You have independent, time-consuming tasks that can run concurrently without affecting the application's responsiveness.
- You want a simple and clean way to enable asynchronous processing without diving into low-level thread management.


**Avoid using @Async when:**

- The tasks you want to execute asynchronously have complex dependencies or need a lot of coordinating. In such cases, you might need to use more advanced concurrency APIs, like CompletableFuture, or reactive programming libraries like Project Reactor.
- You must have precise control over how threads are managed., such as custom thread pools or advanced synchronization mechanisms. In these cases, consider using Java's ExecutorService or other concurrency utilities.


###  Using @Async in a Spring Boot application. 
- Enable async annotation to main class or app config class
```java
@SpringBootApplication
@EnableAsync
public class AsyncDemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(AsyncDemoApplication.class, args);
  }
}

// other example
@Configuration
@EnableAsync
public class ApplicationConfig {}
```
1. For the optimal solution, what we can do is create a custom Executor bean and customize it as per our needs in the same config class
 
```java
@Configuration
@EnableAsync
public class ApplicationConfig {

    @Bean
    public Executor getAsyncExecutor() {
    ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
    executor.setCorePoolSize(5);
    executor.setMaxPoolSize(10);
    executor.setQueueCapacity(100);
    executor.setThreadNamePrefix("AsyncThread-");
    executor.initialize();
    return executor;
    }
}
```
2. example order service class: 

```java
@Service
public class OrderService {

  @Async
  public void saveOrderDetails(Order order) throws InterruptedException {
    Thread.sleep(2000);
    System.out.println(order.name());
  }

  @Async
  public CompletableFuture<String> saveOrderDetailsFuture(Order order) throws InterruptedException {
    System.out.println("Execute method with return type + " + Thread.currentThread().getName());
    String result = "Hello From CompletableFuture. Order: ".concat(order.name());
    Thread.sleep(5000);
    return CompletableFuture.completedFuture(result);
  }

  @Async
  public CompletableFuture<String> compute(Order order) throws InterruptedException {
    String result = "Hello From CompletableFuture CHAIN. Order: ".concat(order.name());
    Thread.sleep(5000);
    return CompletableFuture.completedFuture(result);
  }
}
```

- What we did here is create 3 different Async methods. First saveOrderDetails service is a straightforward asynchronous
service, which will start doing the computing asynchronously. If we want to use modern asynchronous Java features
like CompletableFuture, we can achieve it with saveOrderDetailsFuture service. With this service, we can call a thread to wait for the result of an @Async. It should be noted that CompletableFuture.get() will block until the result is available. If we want to perform further asynchronous operations when the result is available, we can use thenApply, thenAccept, or other methods provided by CompletableFuture.

- The server returns a response right away, we do not need to wait for 5 seconds, and computation will be done background. The most important point, in this case, is a call to async service, in our case compute() must be done from the outside of the same class. If we use @Async on a method and call it within the same class, it won't work. This is because Spring uses proxies to add asynchronous behavior, and calling the method internally bypasses the proxy. To make it work, we can either:

Move the @Async methods to a separate service or component.
Use ApplicationContext to get the proxy and call the method on it.



> https://dev.to/ilkin0/understanding-async-in-spring-boot-2hjp#asynchronous-approach
  