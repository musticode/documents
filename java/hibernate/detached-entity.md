# Detached Entity Error

## Solution 1

Remove cascading from the child entity Transaction, it should be just:

```java
@Entity class Transaction {
    @ManyToOne // no cascading here!
    private Account account;
}
```

(FetchType.EAGER can be removed as well as it's the default for @ManyToOne)

That's all!

Why? By saying "cascade ALL" on the child entity Transaction you require that every DB operation gets propagated to the parent entity Account. If you then do persist(transaction), persist(account) will be invoked as well.

But only transient (new) entities may be passed to persist (Transaction in this case). The detached (or other non-transient state) ones may not (Account in this case, as it's already in DB).

Therefore you get the exception "detached entity passed to persist". The Account entity is meant! Not the Transaction you call persist on.


## Solution 2

Removing child association cascading


- https://stackoverflow.com/questions/13370221/persistentobjectexception-detached-entity-passed-to-persist-thrown-by-jpa-and-h