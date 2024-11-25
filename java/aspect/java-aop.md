# Aspect Oriented Programming in SpringBoot



**dependency**
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```


### Using @around annotation


- Create an annotation interface in java
```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface LogExecutionTime {
}
```


- @Target(ElementType.METHOD): This annotation indicates that the LogExecutionTime annotation can be applied only to elements of type METHOD, i.e. methods.
- @Retention(RetentionPolicy.RUNTIME): This annotation specifies that the LogExecutionTime annotation shall be available at runtime, meaning that it can be accessed through reflection at runtime.
- @interface : Is used in Java to define a new custom annotation. Basically, it is a way to create your own annotations with specific metadata that can be applied to code elements such as classes, methods or variables.


**Aspect component**

```java
@Aspect
@Component
@Slf4j
public class LoggingAspect {

    @Around("(@annotation(org.springframework.web.bind.annotation.GetMapping) || @annotation(org.springframework.web.bind.annotation.PostMapping) || " +
            "@annotation(org.springframework.web.bind.annotation.PatchMapping) || @annotation(org.springframework.web.bind.annotation.DeleteMapping)) " +
            "&& @annotation(com.ocpp.ocpp_socket_service.aop.annotation.LogExecutionTime)")
    public Object executionTime(ProceedingJoinPoint joinPoint) throws Throwable {
        String methodName = joinPoint.getSignature().getName();
        log.info("Starting execution method {}", methodName);
        Object result = joinPoint.proceed();
        log.info("Finished execution method {}", methodName);
        return result;
    }
    
}
```


**Using of the aspect**

```java
@RestController
@RequestMapping("/api/charge-box")
@RequiredArgsConstructor
public class ChargeBoxController {

    private final ChargeBoxService chargeBoxService;

    @GetMapping("/test")
    @LogExecutionTime
    public ResponseEntity<String> getFirst(){
        return new ResponseEntity<>("TEST MESSAGE", HttpStatus.OK);
    }

}
```


- @Aspect : This annotation marks the LoggingAspect class as an aspect in AOP. An aspect is a module that encapsulates a cross-cutting behaviour that can be applied to various breakpoints in a system.
- @Component: This annotation marks the LoggingAspect class as a Spring component, allowing it to be detected and managed by the Spring container.

-----

The logExecutionTime method is the breakpoint of the aspect and is executed when a certain condition defined in the @Around annotation is met. Here is its step-by-step explanation:

- @Around : The logExecutionTime method is the breakpoint of the aspect and is executed when a certain condition defined in the @Around annotation is met.
Inside the Around annotation we are indicating that the logic of the logExecutionTime method for Get, Post, Delete, Patch requests will be executed and that they must also be annotated with the annotation we create.