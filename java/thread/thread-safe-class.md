# Java Thread Safe class implementation

1. Stateless class
- if a class does not maintain any state, fields it's automatically thread-safe
```java
public class MathHelper{
    public int add(int a, int b){
        return a + b;
    }

    public static int multiply(int a, int b){
        return a * b;
    }
}
```


2. Immutable classes : read only
- immutabilty is a powerful way to achieve thread safety. If an object can not be modified after creation there is no risk of concurrent modifications
```java
public final class ImmutablePoint{
    private final int x;
    private final int y;

    public ImmutablePoint(int x, int y){
        this.x = x;
        this.y = y;
    }

    public int getX(){
        return x;
    }

    public int getY(){
        return y;
    }

    public ImmutablePoint translate(int dx, int dy){
        return new ImmutablePoint(x + dx, y + dy);
    }
    
}
```
The `String` class is the perfect example of an immutable, thread-safe class. This is why you never have to worry about synchronization when using strings. Use final keyword whenever possible, as it helps in unintentional modification and thread safety.

3. Encapsulation and synchronization
For classes that need mutable state, proper encapsulation combined with synchronization is essential
**Make fields private**
- publicly accessible fields are thread-safety violation
**Identify non-atomic operations and synchronize them**
- Once your fields are private, you need to ensure methods that modify state do so atomically
```java
public class SafeCounter{
    private int count;
    public synchronized void increment(){
        count++;
    }

    public synchronized void decrement(){
        count--;
    }

    public synchronized int getCount() { 
        return count; 
    } 
}
```

The synchronized keyword ensures that only one thread can execute these methods on a particular instance at any given time. This prevents race conditions, but it does come with a performance cost due to locking overhead.

**Volatile keyword for visibility**
Sometimes you do not need full synchronization but do need to ensure that changes made by one thread are visible to other threads.
```java
public class StatusChecker{
    private volatile boolean running = true;

    public void stop (){
        running = false;
    }

    public void performTask(){
        while(running){
            // do something
        }
    }
}
```

The volatile keyword ensures that changes to the variable are immediately visible to other threads, preventing visibility issues though it doesn't help with atomicity

**Coarse-grained locking VS fine grained locking**

- Synchronizing entire method will work, but it has performance impact. It locks entire method and let's say the execution takes time, then many other thread will be waiting for long to enter inside the method.

- Coarse-grained locking : means using fewer, broader locks that protect larger sections of code or data structures. Like locking an entire list when you only need to modify one element, or locking entire method where only a sub-portion of method need to be synchronized.
- Fine-grained locking : means using many smaller, targeted locks that protect specific components or operations. Like locking just the specific list element you're modifying


```java
// coarse-grained locking
public synchronized void transferMoney(Account from, Account to, int amount){
    from.debit(amount);
    to.credit(amount);
}
```

```java
// fine-grained locking
public void transferMoney(Account from, Account to, int amount){
    synchronized(from){
        synchronized(to){
            from.debit(amount);
            to.credit(amount);
        }
    }
}
```

> Performance : fine-grained typically allows higher concurrency and throughput as multiple threads can access different parts simultaneously. Coarse-grained can create bottlenecks.
> Complexity : Fine-grained is more complext to implement correctly and increases risk of deadlocks. Coarse-grained is simpler and safer to implement.
> Overhead : fine-grained can have higher overhead from managing many locks. Coarse-grained has less lock management overhead but higher contention costs.

4. Leveraging thread-safe libraries
   
to achieve fine-grained locking, java provides thread-safe collections and utilities that can simplify building thread-safe classes.

```java
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.atomic.AtomicInteger;
```

```java
public class ThreadSafeUserManager{

    private final ConcurrentHashMap<String, User> users = new ConcurrentHashMap<>();

    private final AtomicInteger userCount = new AtomicInteger(0);
}
```

This approach uses fine grained locking, rather than coarse grained locking. potentially improving perfomance under contention.

Common components : 
 - Collections: ConcurrentHashMap, CopyOnWriteArrayList, ConcurrentLinkedQueue
 - Atomic Variables: AtomicInteger, AtomicLong, AtomicReference
 - Queues: LinkedBlockingQueue, ArrayBlockingQueue
 - Synchronizers: CountDownLatch, CyclicBarrier, Semaphore

5. Thread confinement : each thread gets its own copy

Sometimes the best wat to avoid sharing is not to share at all. Thread confinement means ensuring eache thread has its own independent copy of data : 

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
```

ScopedValue(introduced in java 21), is the modern replacement ThreadLocal and offers better perfomance and cleaner semantics, especially with virtual threads.

```java
public class ScopedValueExample{
    private static final ScopedValue<String> CURRENT_USER = ScopedValue.newInstance();

    public static void main(String[] args){
                // Run code with a specific value bound to the ScopedValue
        ScopedValue.where(CURRENT_USER, "Alice").run(() -> {
            processRequest();
            
            // Value remains available in downstream methods
            auditAction("data_access");
        });
        // Multiple bindings in nested scopes
        ScopedValue.where(CURRENT_USER, "Bob").run(() -> {
            System.out.println("Outer scope: " + CURRENT_USER.get());
            
            // Can rebind in an inner scope
            ScopedValue.where(CURRENT_USER, "Charlie").run(() -> {
                System.out.println("Inner scope: " + CURRENT_USER.get());
            });
            
            // Outer binding is preserved
            System.out.println("Back to outer: " + CURRENT_USER.get());
        });
    }

    private static void processRequest() {
    // Access the ScopedValue from another method
    System.out.println("Processing request for: " + CURRENT_USER.get());
    }

    private static void auditAction(String action) {
        // Get user from ScopedValue without having to pass it as a parameter
        System.out.println("User " + CURRENT_USER.get() + " performed action: " + action);
    }
}
```

6. Defensive copying : protect your internals 
   When your class holds references to mutable object, you should making defensive copies when accepting or returning those objects: 

```java
public class DefensiveCalendar {
    private final Date startDate;
    
    public DefensiveCalendar(Date start) {
        // Defensive copy to prevent the caller from modifying our state
        this.startDate = new Date(start.getTime());
    }
    
    public Date getStartDate() {
        // Defensive copy to prevent the caller from modifying our state
        return new Date(startDate.getTime());
    }
}
```

Without these defensive copies, a caller could modify the Date object even after passing it to your class, breaking encapsulation and potentially thread safety. 

Conclusion

Building thread-safe classes in Java requires careful consideration of how your class will be used in concurrent environments. Let’s summarize our key strategies:

- Stateless classes eliminate shared state entirely
- Immutable classes prevent modifications after construction
- Proper encapsulation with synchronization protects mutable state
- Thread-safe libraries provide building blocks for complex classes
- Thread confinement isolates state to individual threads
- Defensive copying protects against external modifications
- Lock granularity choices balance safety and performance.