# Embedable & Embedding in hibernate

Jpa providdes the @Embedable annotation to declare that a class will be embedded by other entities.


```java
@Embeddable
public class ContactPerson {

    private String firstName;

    private String lastName;

    private String phone;

    // standard getters, setters
}
```


* Embedded

The other annotation @Embedded is used to embed a type into another entity. Company class:


```java
@Entity
public class Company {

    @Id
    @GeneratedValue
    private Integer id;

    private String name;

    private String address;

    private String phone;

    @Embedded
    private ContactPerson contactPerson;

    // standard getters, setters
}
```


* There are two types of objects in hibernate, value objects and entities
* Value objects are the objects which can not stand alnone. Take `Address` for example. If you say address, people will ask whose address is this. So it can not stand alone
* Entity objects are those who can stand alone like `College` and Student . 
* So in case of value objects preferred way is to embed them into an entity object. 
* To answer why we are creating two different classes: first of all, it’s a OOPS concept that you should have loose coupling and high cohesion among classes. That means you should create classes for specialized purpose only. For example, your Student class should only have the info related to Student.
* Second point is that by creating different classes you promote reusability, When we define the value object for the entity class we use @Embeddable. When we uyse value type object in entity class we use @Embedded


