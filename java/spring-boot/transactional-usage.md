# Transactional Usage

@Transactional is a spring annotation used to manage database transactions. It ensures that a group of operations either complete successfully or are rolled back on failure.

- All-or-nothing
- Reduces boilerplate
- Declarative transaction management

```javascript
@Service
public class OrderService{

  @Transactional
  public void placeOrder(Order order){
    saveOrder(order);
    updateInventory(order);
  }
}
```

If any exception occurs, the entire placeOrder transaction will roll back.

## Use on public methods only

- Transactional only works on public methods. Spring uses proxies that don’t apply to private or protectred methods.

```java
@Service
public class OrderService{

  @Transactional
  public void placeOrder(Order order){

  }
}
```

## Avoid self-invocation

- Calling a @Transactional method from withinn the same class won’t trigger transaction behavior.
- Move the transaction al method to another bean/service

  \

```java
  @Transactional
  public void inner(Order order){

  }

  // wrong
  public void outer(){
    inner();
  }
```

## Don’t use on read-only methods

- Adding @transactional on read-only metrhods add unnecessary overhead

  \

## Choose the right propagation

Use apropriate propagation types depending on the use case

- REQUIRED(default) : join current or create new
- REQUIRED_NEW : always start new transaction
- NESTED : savepoint mechanism

## Rollback rules

By default, springh only rolls back on unchecked exceptions(runtime exception)

To roll back on checked exceptions, specify explicitly :

```java
  @Transactional(rollbackFor = IOException.class)
  public void inner(Order order){

  }
```

## Use with caution in Async methods

- Transactional doesn’t work weel with @Async out of the box. Why? Async runs in a different thread → proxy-based transaction lost.
- Use TransactionTemplate if async transactions are needed
