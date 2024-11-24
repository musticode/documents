# ModelMapper for Java

ModelMapper dependency :

```xml
<dependency>
    <groupId>org.modelmapper</groupId>
    <artifactId>modelmapper</artifactId>
    <version>3.1.1</version>
</dependency>
```

To use ModelMapper in your application, configure it as a Spring Bean:

```java
@Configuration
public class ModelMapperConfig {

    @Bean
    public ModelMapper modelMapper() {
        return new ModelMapper();
    }
}
```

Simple entity class :

```java
@Entity
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private String email;
    private String address;

    // Getters and Setters
}
```

Simple DTO or request/response object class

```java
public class UserDTO {
    private String name;
    private String email;

    // Getters and Setters
}
```

Map entity to DTO in a service class

```java
@Service
public class UserService {
    
    private final ModelMapper modelMapper;

    public UserService(ModelMapper modelMapper){
        this.modelMapper = modelMapper;
    }


    public UserDTO convertToDto(User user) {
        return modelMapper.map(user, UserDTO.class);
    }
}
```

- The modelMapper.map() method automatically maps fields from the User entity to the UserDTO.
- It uses conventions like matching field names to perform the mapping.


### Custom mapping

```java
public UserDTO convertToDto(User user) {
    modelMapper.typeMap(User.class, UserDTO.class).addMappings(mapper -> {
        mapper.map(User::getName, UserDTO::setFullName);
    });
    return modelMapper.map(user, UserDTO.class);
}
```

- `typeMap()` allows specifying custom mapping rules.
- `addMappings()` provides a way to define how specific fields are mapped.

### Mapping a List of Entities

```java
public List<UserDTO> convertToDtoList(List<User> users) {
    return users.stream()
                .map(user -> modelMapper.map(user, UserDTO.class))
                .collect(Collectors.toList());
}
```

- stream() iterates through the list of entities.
- Each entity is mapped to a DTO using modelMapper.map().
- collect(Collectors.toList()) gathers the results into a new list.

Mapping to DTO 

```java
public User convertToEntity(UserDTO userDTO) {
    return modelMapper.map(userDTO, User.class);
}
```

