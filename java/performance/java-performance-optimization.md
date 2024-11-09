# Java Performance Optimization Notes


### Choose the Right Data Structure for Performance

- Choosing an efficient data structure can significantly improve the performance of your application, especially when dealing with large datasets or time-critical operations. Using the correct data structure minimizes access time, optimizes memory usage, and reduces processing time.

- HashSet - ArrayList
```java
// Inefficient - O(n) complexity for contains() method 
List<String> names = new ArrayList<String>();
names.add("Alice");
names.add("Bob");
// Checking if "Alice" exists - time complexity O(n)
if (names.contains("Alice")) {
    System.out.println("Found Alice");
}

// Efficient - O(1) complexity for contains() check
Set<String> namesSet = new HashSet<>();
namesSet.add("Alice");
namesSet.add("Bob");
// Checking if "Alice" exists - time complexity O(1)
if (namesSet.contains("Alice")) {
    System.out.println("Found Alice");
}
```

In this example, HashSet provides an average time complexity of O(1) for contains() operations, while ArrayList requires O(n) as it must iterate through the list. Therefore, for frequent lookups, a HashSet is more efficient than an ArrayList.


### Check Exception Conditions at the Beginning

- You can avoid unnecessary processing overhead by checking the fields to be used in the method that should not be null at the beginning of the method. It is more effective in terms of performance to check them at the beginning, instead of checking for null checks or illegal conditions in later steps in the method.

```java
public void processOrder(Order order) {
    if(Objects.isNull(order)){
        throw new IllegalArgumentException("Order must not be null");
    }

    if (order.getItems().isEmpty()){
        throw new IllegalStateException("Order must contain items");
    }

    // process starts
    processItems(order.getItems));
}
```

### Avoid Creating Unnecessary Objects

- Creating unnecessary objects in Java applications can negatively affect performance by increasing Garbage Collection time. The most important example of this is String usage.

- This is because the String class in Java is immutable. This means that each new String modification creates a new object in memory. This can cause a serious performance loss, especially in loops or when multiple concatenations are performed.

```java
String result = "";
for (int i = 0; i < 1000; i++) 
    result += "number " + i;
```

- We can avoid unnecessary object creation by doing the same with StringBuilder. StringBuilder improves performance by modifying the existing object:

```java
StringBuilder result = new StringBuilder();

for (int i = 0; i < 1000; i++){
    result.append("number ").append(i);
}

String finalResult = result.toString();
```


### Use Flatmap for Nested Loops

- `flatMap()` function included with the Java Stream API are powerful tool for optimizin operations on collections. Nested loops can lead to performance loss and more complex code. By using this method, you can make your code readable and gain performance. 
- `flatMap()` usage : 
  - `map` : Operates on each element and returns another element as a result
  - `flatMap()` :Operates on each element, converts the results into a flat structure and provides a simpler data structure
- Nested loop example : 
```java
List<List<String>> listOfList = new ArrayList<>();
for (List<String> list : listOfList){
    for (String item : list){
        System.out.println(item);
    }
}
```

- using flatMap : 
```java
listOfList
        .stream()
        .flatMap(Collection::stream)
        .forEach(System.out::println);
```

For this example, we convert each list into a flat stream with flatMap and then process the elements with forEach. This method makes the code both shorter and more performance efficient.


### Prefer Lightweight DTOs Instead of Entities


- Returning data pulled from the database directly as Entity classes can lead to unnecessary data transfer. This is a very faulty method both in terms of security and performance. 

```java
@Getter
@Setter
public class UserDTO {
    private String name;
    private String email;
}
```

### Speaking of databases; Use EntityGraph, Indexing, and Lazy Fetch Feature

- Database performance directly affects the speed of the application. Performance optimization is is especially important when retrieving data between related tables. To avoid, lazy fetching is important. 

#### Optimized data extraction with EntityGraph

- EntityGraph allows you to control the associated data in database queries. You pull only the data you need, avoiding the costs of eager fetching. Eager fetching is when the data in the related table automatically comes with the query.

```java
@EntityGraph(attributePaths = {"addresses"})
User findUserByUserId(@Param("userId") Long userId);
```
In this example, address-related data and user information are retrieved in the same query. Unnecessary additional queries are avoided.


#### Improve Performance with Indexing

- Indexing is one of the most effective methods to improve the performance of database queries. A table in a database consists of rows and columns, and when a query is made, it is often necessary to scan all rows. Indexing speeds up this process, allowing the database to search faster over a specific field.


### Lighten the Query Load by Using Cache

- Caching is the process of temporarily storing frequently accessed data or computational results in a fast storage area such as memory. Caching aims to provide this information more quickly when the data or computation result is needed again. Especially in database queries and transactions with high computational costs, the use of cache can significantly improve performance.
  
```java
@Cacheable("users")
public User findUserById(Long userId) {
    return userRepository.findById(userId);
}
```