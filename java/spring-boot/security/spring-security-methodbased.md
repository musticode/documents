# Spring boot method level security


Simply put, Spring Security supports authorization semantics at the method level.

## Enabling Method Security

- spring-security-config

```xml
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-config</artifactId>
</dependency>
```

- spring security starter

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

- global method security

```java
@Configuration
@EnableGlobalMethodSecurity(
  prePostEnabled = true, 
  securedEnabled = true, 
  jsr250Enabled = true)
public class MethodSecurityConfig 
  extends GlobalMethodSecurityConfiguration {
}
```


- The prePostEnabled property enables Spring Security pre/post annotations.
- The securedEnabled property determines if the @Secured annotation should be enabled.
- The jsr250Enabled property allows us to use the @RoleAllowed annotation.


## Applying method security

- The @Secured annotation is used to specify a list of roles on a method. So, a user only can access that method if she has at least one of the specified roles.

```java
@Secured("ROLE_VIEWER")
public String getUsername() {
    SecurityContext securityContext = SecurityContextHolder.getContext();
    return securityContext.getAuthentication().getName();
}
```


```java
@Secured({ "ROLE_VIEWER", "ROLE_EDITOR" })
public boolean isValidUsername(String username) {
    return userRoleRepository.isValidUsername(username);
}
```

- The @RolesAllowed annotation is the JSR-250’s equivalent annotation of the @Secured annotation.
Basically, we can use the @RolesAllowed annotation in a similar way as @Secured.
This way, we could redefine getUsername and isValidUsername methods:

```java
@RolesAllowed("ROLE_VIEWER")
public String getUsername2() {
    //...
}
    
@RolesAllowed({ "ROLE_VIEWER", "ROLE_EDITOR" })
public boolean isValidUsername2(String username) {
    //...
}
```


### Using @PreAuthorize and @PostAuthorize Annotations

Both @PreAuthorize and @PostAuthorize annotations provide expression-based access control. So, predicates can be written using SpEL (Spring Expression Language).
The @PreAuthorize annotation checks the given expression before entering the method whereas the @PostAuthorize annotation verifies it after the execution of the method and could alter the result

```java
@PreAuthorize("hasRole('ROLE_VIEWER')")
public String getUsernameInUpperCase() {
    return getUsername().toUpperCase();
}
```
The @PreAuthorize(“hasRole(‘ROLE_VIEWER’)”) has the same meaning as @Secured(“ROLE_VIEWER”), which we used in the previous section. Feel free to discover more security expressions details in previous articles.

Consequently, the annotation @Secured({“ROLE_VIEWER”,”ROLE_EDITOR”}) can be replaced with @PreAuthorize(“hasRole(‘ROLE_VIEWER’) or hasRole(‘ROLE_EDITOR’)”):

```java
@PreAuthorize("hasRole('ROLE_VIEWER') or hasRole('ROLE_EDITOR')")
public boolean isValidUsername3(String username) {
    //...
}
```

with method argument 

```java
@PreAuthorize("#username == authentication.principal.username")
public String getMyRoles(String username) {
    //...
}
```

> [Spring Method Security](https://www.baeldung.com/spring-security-method-security)