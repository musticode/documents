# CompletableFuture Java

### get()

- Bir completable future sonucunu almak için get() metodunu kullanır. Bu future tamamlanana kadar o thread blocklanır
- String result = completableFuture.get();

### complete()

- Future’ı elle manuel olara ktamamlamak için complete() metodu kullanılır
- Örnek olarak bir servis down vaaziyette olabilir, bu durumda da eldeki son cevabı dönmek isteyebiliriz bunun için kullanılır.

### runAsync()

- Bazı background taskları aschronous olarak çalıştırmak ve task tamamlandıktan sonra herhangi bir şey döndürmek istemiyorsak CompletableFuture.runAsync() metodunu kullanabiliriz. Callback function olarak bir Runnable object parametresi alır ve CompletableFuture<Void> değerini döndürür

```python
CompletableFuture<Void> future = CompletableFuture.runAsync(() -> {
      try {
          System.out.println("Main thread'den ayrı bir threadde calisirim ve bir deger return etmem.");
          TimeUnit.SECONDS.sleep(3);
      } catch (InterruptedException e) {
          throw new IllegalStateException(e);
      }
});
```

### supplyAsync()

- runAsync’ten tek farkı bir şey return etmesidir

```python
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    try {
        System.out.println("Main thread'den farkli bir thread'de calisirim ve bir deger return ederim.")
        TimeUnit.SECONDS.sleep(3);
    } catch (InterruptedException e) {
        throw new IllegalStateException(e);
    }
    return "Herhangi bir deger";
});

String result = future.get();
System.out.println(result);
```

Not : CompletableFuture taskları global ForkJoinPool.commonPool nesnesinden elde edilen bir thread’de yürütür. Ancak isteğe bağlı olarak ayrıca kendimizin configure ettiği bir Thread Pool oluşturabilir ve taskları bu thread Pool’dan alınan bir threadden yürütmek için runAsync() ve supplyAsync() metotlarına iletebiliriz.

```python
Executor executor = Executors.newFixedThreadPool(20);
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    try {
        TimeUnit.SECONDS.sleep(3);
    } catch (InterruptedException e) {
        throw new IllegalStateException(e);
    }
    return "Herhangi bir deger";
}, executor);
```

## thenApply() - thenAccept() - thenRun()

- get() metodu blocking’dir. Future tamamlanana kadar bekler ve tamamlanmasından sonra sonucu döndürür. Ama istediğimiz bu değilse?
- Async sistemler oluşturmak için CompletableFurue’a Future tamamlandığında otomatik olarak çağrılması gereken bir callback function ekleyebilmeliyiz. Bu şekilde sonucu beklememiz gerekmeyecek ve Future’ın tamamlanamsından sonra callback fonksiyonumuz içinde yürütlmesi gereken logic’i yazabiliriz.

### thenApply()

- Completablefuture soncu geldiğinde process etmek ve transform etmek için thenApply() metodunu kullanabiliriz. thenApply ile future’dan gelen değeri alırız iligili işlemlerimizi yapıp tekrar return edebiliriz.

  \

  ```java
  CompletableFuture<String> filmFuture = CompletableFuture.supplyAsync(() -> {
     try {
         TimeUnit.SECONDS.sleep(3);
     } catch (InterruptedException e) {
         throw new IllegalStateException(e);
     }
     return "Kim Book Joo";
  });

  CompletableFuture<String> resultFuture = filmFuture.thenApply(name -> {
     return "Film " + name;
  });
  ```

- test 2

```java
CompletableFuture<String> filmFuture = CompletableFuture.supplyAsync(() -> {
    try {
        TimeUnit.SECONDS.sleep(3);
    } catch (InterruptedException e) {
       throw new IllegalStateException(e);
    }
    return "Kim Book Joo";
}).thenApply(name -> {
    return "Film " + name;
}).thenApply(greeting -> {
    return greeting + ", Super bir film";
});

System.out.println(filmFuture.get());
```

### thenAccept() - thenRun()

- Bir önce anlatılan thenApply() metodunda callback function’umuz bir şey return eder. Bizim ihtiyacımız da callback function’dan bir şey döndürmek istemiyorsak ve yalnızca Future tamamlandıktan sonra bir parça kod çalıştırmak istiyorsak thenAccept() ve thenRun() metodlarını kullanabiliriz.
- CompletableFuture.thenAccept() bir Consumer<T> alır ve ComletableFuture<Void> değerini return eder. thenAccept metodu bağlı olduğu ComletableFuture sonucuna erişebilir

```java
CompletableFuture.supplyAsync(() -> {
	return shipService.getShip(shipId);
}).thenAccept(ship -> {
	System.out.println("Ship Name " + ship.getShipName())
});
```

### thenRun()

- thenAccept’ten farkı ise bağlı olduğu CompletableFuture’ın sonucuna istese d eerişemez.

```java
CompletableFuture.supplyAsync(() -> {
    // Run some computation
}).thenRun(() -> {
    // Computation Finished.
})
```

- thenRun, thenAccept, thenApply gibi callback fonksiyonaların ayrıca async olanları da vardır. Bu async callback variation’ları sayesinde callback taskları ayrı bir thread’de yürüterek hesaplamalarımızı daha fazla paralelleştirmemize yardımcı olur.

```java
CompletableFuture.supplyAsync(() -> {
    try {
       TimeUnit.SECONDS.sleep(3);
    } catch (InterruptedException e) {
      throw new IllegalStateException(e);
    }
    return "Herhangi bir sey"
}).thenApply(result -> {
    /*
    Ya supplyAsync() task'ının yürütüldüğü aynı threadde yürütülür
	  yada supplyAsync() taskı hemen tamamlanırsa main threadde yürütülür.
    */
    return "Result"
})
```

- Yukarıdaki durumda thenApply içindeki task, supplyAsync taskının yürütüldüğü aynı threadde veya supplyAsync taskı hemen tamamlanırsa main threadde yürütülür. İhtiyacımıza göre callback taskını execute eden thread üzerinde daha fazla kontrole sahip olmak isteyebiliriz. Bu durumda javanın bize sağlamış oldğu diğer varyasyon olan async callback’leri kullanmalıyız. thenApplyAsync() callback’ini kullanırsak fork join pool nesnesinden alınan farklı bir threadde yürütülür.

```java
CompletableFuture.supplyAsync(() -> {
    return "Herhangi bi sey"
}).thenApplyAsync(result -> {
    // ForkJoinPool.commonPool() nesnesinden alınan farklı bir threadde execute edilir.
    return "Result"
})
```

- Ayrıca bir executor’u thenApplyAsync callback’ine argüman olarak ilertirsek task executor thread poolundan elde eilen bir thread içinde yürütülür.

```java
Executor executor = Executors.newFixedThreadPool(2);
CompletableFuture.supplyAsync(() -> {
    return "Herhangi bi sey"
}).thenApplyAsync(result -> {
    // Vermis oldugunuz executor'dan elde edilen thread'de yurutur.
    return "Result"
}, executor);
```
