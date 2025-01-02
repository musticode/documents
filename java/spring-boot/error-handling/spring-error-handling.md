# SpringBoot Error Handling

## Synch Exception Handling

**Custom exception class**

```java
public class ResourceNotFoundException extends RuntimeException {
    public ResourceNotFoundException(String message) {
        super(message);
    }
}
```

**Error response class**

This class represents the structure of the error response.
```java
public class ErrorResponse {
    private String message;
    private int status;
    private long timestamp;

    public ErrorResponse(String message, int status) {
        this.message = message;
        this.status = status;
        this.timestamp = System.currentTimeMillis();
    }
}
```


**Global exception handler with RestControllerAdvice**

This class handles exceptions globally and returns a custom error response with the appropriate status code.
```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleResourceNotFoundException(ResourceNotFoundException ex) {
        ErrorResponse errorResponse = new ErrorResponse(ex.getMessage(), HttpStatus.NOT_FOUND.value());
        return new ResponseEntity<>(errorResponse, HttpStatus.NOT_FOUND);
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGenericException(Exception ex) {
        ErrorResponse errorResponse = new ErrorResponse("Internal Server Error", HttpStatus.INTERNAL_SERVER_ERROR.value());
        return new ResponseEntity<>(errorResponse, HttpStatus.INTERNAL_SERVER_ERROR);
    }
}
```

## Asynchronous Exception Handling
Asynch errors occuur when working with @Async methods, where exceptins may not be caught immediately. Implementing AsyncUncaughtExceptionHandler helps capture these errorrs.

```java
@Configuration
@EnableAsync
public class AsyncConfig implements AsyncConfigurer {

    @Override
    public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
        return (ex, method, params) -> {
            System.err.println("Exception in async method: " + ex.getMessage());
        };
    }
}
```

## Authentication and Authorization Errors
Auth and authorization errors occur when a user tries to access a resource without valid credentials or the necessary permissions. Handling these errors can improve security and user experience by providing meaningful feedback

Custom exception for unauthorized access:

```java
public class UnauthorizedAccessException extends RuntimeException {
    public UnauthorizedAccessException(String message) {
        super(message);
    }
}
```

Global exception handler : 

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(UnauthorizedAccessException.class)
    public ResponseEntity<ErrorResponse> handleUnauthorizedAccess(UnauthorizedAccessException ex) {
        ErrorResponse errorResponse = new ErrorResponse(ex.getMessage(), 401);
        return ResponseEntity.status(HttpStatus.UNAUTHORIZED).body(errorResponse);
    }
}
```

Usage in controller 

```java
@GetMapping("/secure-data")
public ResponseEntity<String> getSecureData() {
    // Check for authentication or role
    if (!isAuthenticated()) {
        throw new UnauthorizedAccessException("You are not authorized to access this resource");
    }
    return ResponseEntity.ok("Secure Data");
}
```

## Rate Limiting Errors

Rate limiting provents users from overloading the application with too many requests. If users exceed their limit, we should respond with an appropriate error message.

exception and handler

```java
public class RateLimitExceededException extends RuntimeException {
    public RateLimitExceededException(String message) {
        super(message);
    }
}

@ExceptionHandler(RateLimitExceededException.class)
public ResponseEntity<ErrorResponse> handleRateLimitExceeded(RateLimitExceededException ex) {
    ErrorResponse errorResponse = new ErrorResponse(ex.getMessage(), 429);
    return ResponseEntity.status(HttpStatus.TOO_MANY_REQUESTS).body(errorResponse);
}
```

- Explanation: If the user exceeds the allowed request rate, the RateLimitExceededException is thrown, and the client receives a 429 (Too Many Requests) response.