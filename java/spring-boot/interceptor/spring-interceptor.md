# Spring Interceotor

## Detailed Explanation of Spring Boot Interceptor

The interceptor (Interceptor), like the filter, is a specific implementation of aspect-oriented programming — AOP (Aspect-Oriented Programming is just a programming thought).

You can use an Interceptor to perform certain tasks, such as writing logs before the Controller processes requests, adding or updating configurations…

In Spring, when a request is sent to the Controller, before being processed by the Controller, it must pass through Interceptors (zero or more).

The Spring Interceptor is a concept very similar to Servlet Filter.


## Differences between Interceptor and Filter

The main differences between interceptor and filter li in the scope of action and implementation method. 

1. Scope of action
   - Filters act on the entire web application and can filter all requests and responses. It is part of the Servlet specification and is managed by the Servlet container.
   - Interceptors usually act on specific frameworks. For example, in Spring boot, it mainly intercepts request processing flow of a specific framework and processes requests within a specific framework.
2. Implementation method
   - Filters implement the `javax.servlet.Filter` interface and are configured in web.xml or through annpotations. Methods such as init(), doFilter() and destroy() need to be implemented
   - Interceptors have diffrenet implementation methods in different frameworks. For example, in Spring MVC, you can implement the HandlerInterceptor interface or inherit the HandlerInterceptorAdapter class. Methods such as preHandle() and postHandle() and afterCompilation() need to be implemented.

## Role of interceptors

- Authentication and access control: Interceptors can be used to check the user’s authentication status and permissions and perform relevant processing as needed. For example, an interceptor can be used to verify the user’s login status. If not logged in, redirect to the login page or return the corresponding error information.
- Exception handling and unified error handling: Interceptors can capture and handle exceptions that occur during request processing. Appropriate processing can be performed according to the exception type, such as returning a custom error page or error information, or executing specific error handling logic.


https://medium.com/javarevisited/spring-boot-in-depth-understanding-of-the-implementation-and-application-of-interceptor-ffc83e136347

