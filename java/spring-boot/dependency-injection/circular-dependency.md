# Circular Dependency injection 

How to Solve This, Let’s GO!
- Use @Lazy Annotation:
Annotate one of the dependencies with @Lazy to delay its initialization.

- Refactor the Design:

Extract shared logic into a third service to break the circular dependency.

- Setter Injection:

Use setter-based dependency injection to avoid circular constructor dependencies.
Use ObjectFactory or Provider: (Seen this solution online but havn’t tried it though) — Lazily retrieve the dependent bean when needed.


### @Lazy

Mark one of the services as @Lazy, which delays its initialization until it's actually needed, thus breaking the circular dependency.

```java
@Service
public class UserService {
    private final RetryService retryService;

//I prefer Constructor Injection

public UserService(@Lazy RetryService retryService) {  //Used @Lazy here
        this.retryService = retryService;
    }
}

@Service
public class RetryService {
    private final UserService userService;
    public RetryService(UserService userService) {
        userService = userService;
    }
}
```

