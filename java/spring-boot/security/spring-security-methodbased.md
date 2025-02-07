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

## Method based 2

**@Secured Annotation**

The @Secured annotation is used to specify a list of roles that are allowed to access a particular method. 

```java
@Secured("ROLE_ADMIN") 
public void deleteUser(int userId) { 
// Method logic here 
}
```

**@PreAuthorize Annotation**

The @PreAuthorize annotation is used to specify more complex security constraints using SpEL (Spring Expression Language). 

```java
@PreAuthorize("hasRole('ROLE_ADMIN') or (hasRole('ROLE_USER') and #userId == principal.userId)") 
public void updateUser(int userId) { 
// Method logic here 
}
````

This annotation allows only users with the ROLE_ADMIN role or users with the ROLE_USER role and the same userId as the principal(the currently authenticated user) to access the updateUser() method. By default, method-level securitty is not enabled in spring security. To enable it, developers need to add the <global-method-security> element to the spring security configuration file and set the pre-post annotations attribute to "enabled".

In summary, method level security in spring security allows develoopers to apply security constraints to specific methods within a class, rather than applying them to the entire class or application. This feature is implemented using the @Secured and @PreAuthorize annotations and is more fine-grained control over access to specific parts of the application.

```xml
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

```java
@Configuration
@EnableGlobalMethodSecurity(prePostEnabled = true) 
public class SecurityConfig extends WebSecurityConfigurerAdapter { 
// Other security configurations here 
}
```

```java
@Service
public class UserService { 
	
    @PreAuthorize("hasRole('ROLE_ADMIN')") 
    public void deleteUser(int userId) { 
    // Method logic here 
    } 
        
    @PreAuthorize("hasRole('ROLE_USER') and #userId == principal.userId") 
    public void updateUser(int userId) { 
    // Method logic here 
    } 
	
}
```

Similarly, in this example as well, the deleteUser() method can only be accessed by users with the ‘ROLE_ADMIN’ role, while the updateUser() method can only be accessed by users with the ‘ROLE_USER’ role.