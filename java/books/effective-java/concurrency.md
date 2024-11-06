# Concurrency & Multithreading

## Synchronize access to shared mutable data

- `synchronized` keyword'ü bir metod'u ya da bir kod bloğunu aynı anda tek thread'in çalıştıracağını garanti eder. Birçok programcı senkronizasyonu yalnızca bir `mutual exclusion` yöntemi olarak düşünür, bir nesnenin bir iş parçacığı tarafından tutarsız bir durumda görülürken başka bir iş parçacığı tarafından değiştirilmesini önlemek için. Bir nesne tutarlı bir durumda oluşturulur ve ona erişen metodlar tarafından kilitlenir. Bu metotlar durumu gözlemler ve isteğe bağlı 0olarak bir state transition'a neden olarak nesneyi tutarlı bir durumdan diğerine dönüştürür. Senkronizasyonun doğru kullanımı, hiçbir yöntemin nesneyi tutarsız bir durumda gözlemlemeyeceğini garanti eder.
- Bu görüş doğrudur, ancak hikayenin yalnızca yarısıdır. Senkronizasyon olmadan, bir iş parçacığının değişiklikleri diğer iş parçacıkları tarafından görülemeyebilir. Senkronizasyon, iş parçacıklarının tutarsız bir durumdaki bir nesneyi gözlemlemesini engellemekle kalmaz, aynı zamanda senkronize bir metoda veya bloğa giren her iş parçacığının aynı kilit tarafından korunan tüm önceki değişikliklerin etkilerini görmesini sağlar.
- Performans artırmak için atomik verileri okurken veya yazarken senkronizasyondan vazgeçmek gerektiğini duymuş olabilir fakat bu çok yanlıştır. Dil, bir thread'in bir alanı okurken keyfi bir değer görmeyeceğini garanti ederken bir thread taraından yazılan bir değerin başka biri taraından görülebileceğini garanti etmez. 
- Senkronizayon, thread'ler arasında güvenilir iletişim ve mutual exclusion için gereklidir. Bunun nedeni, bir thread tarafından yapılan değişikliklerin diğerleri tarafından ne zaman ve nasıl görülebileceğini belirten memory model olarak bilinen dil belirtiminin bir parçasıdır.
- Hem yazma hem de okuma yöntemlerini senkronize etmediğimiz sürece yeterli olmayacaktır ve garanti sağlamayacaktır.
- `volatile` ifadesi kullanırken dikkatli olmalıyız. “volatile” anahtar kelimesi değişkenin sakladığı değerin Thread'ler tarafından okunmaya çalışıldığında hepsinde aynı değerin okunacağının garantisini verir.
```java
private static volatile int nextSerialNumber = 0;

public static int generateSerialNumber() {
    return nextSerialNumber++;
}
```
Yöntemin amacı, her çağrının benzersiz bir değer döndürmesini garantilemektir (232'den fazla çağrı olmadığı sürece). Yöntemin durumu, atomik olarak erişilebilir tek bir alandan oluşur, nextSerialNumber ve bu alanın tüm olası değerleri yasaldır. Bu nedenle, değişmezlerini korumak için senkronizasyona gerek yoktur. Yine de, senkronizasyon olmadan yöntem düzgün çalışmayacaktır.
Sorun, artırma operatörünün (++) atomik olmamasıdır. nextSerialNumber alanında iki işlem gerçekleştirir: önce değeri okur ve sonra eski değer artı bire eşit yeni bir değer geri yazar. İkinci bir iş parçacığı, bir iş parçacığının eski değeri okuması ve yeni bir değer geri yazması arasındaki sürede alanı okursa, ikinci iş parçacığı birinciyle aynı değeri görür ve aynı seri numarasını döndürür. Bu bir güvenlik hatasıdır: program yanlış sonuçları hesaplar.
Yöntemi kurşun geçirmez hale getirmek için int yerine long kullanın veya nextSerialNumber sarmak üzereyse bir istisna oluşturun.
Daha da iyisi, Madde 59'daki tavsiyeyi izleyin ve java.util.concurrent.atomic'in bir parçası olan AtomicLong sınıfını kullanın. Bu paket, tek değişkenlerde kilitsiz, iş parçacığı güvenli programlama için ilkel öğeler sağlar. Volatile yalnızca senkronizasyonun iletişim etkilerini sağlarken, bu paket aynı zamanda atomiklik de sağlar. Bu, generateSerialNumber için tam olarak istediğimiz şeydir ve senkronize edilmiş sürümden daha iyi performans göstermesi muhtemeldir:

```java
// Lock-free synchronization with java.util.concurrent.atomic
private static final AtomicLong nextSerialNum = new AtomicLong();
public static long generateSerialNumber() {
    return nextSerialNum.getAndIncrement();
}
```

- Bu maddede tartışılan sorunlardan kaçınmanın en iyi yolu mutable verileri paylaşmamaktır. Ya da immutable verileri paylaşmaktır. Başka bir deyişle mutable verileri tek bir thread ile sınırlayın. Bu politikayı benimsemek önemlidir. Kullanılan kütüphaneler hakkında da derin bir bilgiye sahip olmak gerekir.
- Obje referansını güvenli şekilde paylaşmanın birkaç yolu vardır: static bir alan, volatile bir alan, final field ya da normal locking ile ulaşılabilen normal bir alan. Ya da concurrent collection'a bu obje referansını koyabiliriz.
- Özetle, eğer birden fazla thread, mutable bir datayı paylaşmak isterse her thread okuma ve yazma yaparken senkronizasyonu sağlamalıdır. Senkronizasyonun olmadığı durumlarda thread'in değiştirdiği değişiklikler ve diğer thread'e görünen verilerin aynı olacağının garantisi yoktur. Paylaşılan mutable verileri senkronize edemiyorsak liveness ve safety hatalarıyla karşılaşabiliriz. Bu hatalar da ayıklaması en zor olanlardır. Aralıklı ve zamanlamaya bağlı olabilirler ve program davranışı bir VM'den diğerine kökten değişebilir. Yalnızca `(inter-thread communication)`'e ihtiyacınız varsa ve karşılıklı dışlamaya ihtiyacınız yoksa, `volatile` değiştiricisi kabul edilebilir bir senkronizasyon biçimidir, ancak doğru şekilde kullanılması zor olabilir. 


## Avoid excessive synchronization

- Bu konuyla ilgili tartışmanın son konusu, değiştirilebilir bir sınıf oluştururken karşımıza çıkan karardır. İş parçacığı güvenliği söz konusu olduğunda sahip olduğumuz iki seçenek, sınıfın dışında senkronizasyon sağlanmasını zorunlu kılmak veya sınıfa dahili olarak iş parçacığı güvenliği eklemektir. Dahili senkronizasyon yalnızca bunu yaparak çok daha yüksek bir eşzamanlılık elde edebiliyorsanız kullanılmalıdır. Hiçbir şeyin bir bedeli yoktur. Birçok erken Java çekirdek sınıfı dahili senkronizasyon tarafında hata yaptı ve bu nedenle tek iş parçacıklı bir kullanım durumunda kullanılsa bile bir senkronizasyon maliyeti ödedi. Bunu aşmak için orijinal iş parçacığı güvenli uygulamaları değiştirmek üzere birçok modern alternatif oluşturuldu. Bunlara örnek olarak StringBuffer yerine StringBuilder, Hashtable yerine HashMap ve Random yerine ThreadLocalRandom verilebilir. Şüpheniz varsa bu daha yeni sınıfların önerdiği rehberliği izleyin, sınıfınız içinde senkronizasyon yapmayın ve sınıfınızı iş parçacığı güvenli olmadığı şeklinde belgelendirin.
- Sınıfınızın senkronizasyonu mantıklıysa daha karmaşık bir yola gireceksiniz. Performanslı ve güvenli iş parçacığı güvenli kod yazmak zor olabilir. Bunu kolaylaştıracak kalıplar ve uygulamalar bu blog yazısının kapsamının ötesindedir ancak öğrenebileceğiniz birçok harika kaynak vardır.
- Her şeyi bir araya getirerek, kritik bir bölüm içinden sağlanan işlevleri çağırmayın. Bunu yaparak kodunuzu çıkmazlara ve veri bozulmasına açıyorsunuz. Kritik bölümlerinizi küçük tutun. İş parçacığı güvenli veri türleri kullanmak, kodunuza iş parçacığı güvenliği eklemenin yararlı bir yöntemi olabilir. Değişebilirliğe sahip bir sınıf oluştururken senkronizasyon sağlaması gerekip gerekmediğini düşünün.


## Prefer Executors, Tasks, and Streams to Threads

- Her geliştiricinin bir tür iş kuyruğu gerektiren bir sorunla karşılaşacağı anlaşılıyor. Bu iş kuyrukları, üzerinde çalışılacak görevlerin bir koleksiyonunu belirli bir sırayla depolamak ve devam etmek için kullanılabilir. Bu iş kuyruklarının konsepti basit olsa da, bunlardan birini güvenli ve performanslı bir şekilde geliştirmek zor ve hataya açık olabilir. Neyse ki Java diline yerleştirilmiş bir çözümümüz var.
- java.util.concurrent paketinde, bu işleri yapabileceğimiz Executor Framework var
```java
ExecutorService executor = Executors.newSingleThreadExecutor();
executor.execute(runnable);
executor.shutdown();
```
- Bir görevin bitmesini bekleyebilir, bir görev grubunun herhangi birinin veya tamamının bitmesini bekleyebilir, yürütücünün bitmesini bekleyebilir, eşzamansız görevlerinizin sonuçlarını alabilir, belirli görevleri bir zamanlamaya göre çalıştırabilir ve daha fazlasını yapabilirsiniz.
- Executor service bizi Thread kullanmaktan uzak tutacaktır. Çünkü thread kullanmak tehlikelidir. Executor interface 2 tip task alabilir, 1.si Runnable : bu taskın dönüş türü yok. 2.si ise Callable : dönüş tipi var
- Executor service fork-join task'lar için ilerletilmiştir.


## Prefer Concurrency Utilities Over wait and notify

- Java dilinde objeler için 3 temel metot vardı, bunlar üzerinden low level'da concurrency işleri yapılabilir. wait, notify, notifyAll.
- Java concurrent paketinde utilities ile senkronizasyon kontrolleri için 3 yöntem bulunuyor. Executor framework önceki item'de anlatıldı, concurrent collections implementations ve synchronizers.
- Bu yerleşik eşzamanlı yetenekli koleksiyonlar, yerleşik senkronizasyon toplama işlemlerini (Collections.synchronizedMap vb.) etkili bir şekilde geçersiz kılar. Bu kullanımlardan birini eşzamanlı karşılığıyla değiştirmek büyük bir performans avantajı olabilir. Bu eşzamanlı koleksiyonlar, önceki maddede tartışıldığı gibi, BlockingQueue'nin ThreadPoolExecutor'daki kullanımı gibi, eşzamanlı paket tarafından işini kolaylaştırmak için dahili olarak bile kullanılır.
- java.util.concurrent paketi tarafından sağlanan bir diğer eşzamanlı yardımcı program türü de eşzamanlayıcılardır. Bunlar, bir iş parçacığının başka bir iş parçacığında beklemesine izin veren nesnelerdir. Bu oldukça temel bir kavram olmasına rağmen güçlü bir şekilde kullanılabilir. En yaygın eşzamanlayıcılar CountdownLatch ve Semaphore'dur ancak CyclicBarrier, Exchanger ve Phaser gibi daha gelişmiş eşzamanlayıcılar da vardır.
- Bu örnekte, biraz karışık olabilen ancak işleri oldukça ayrı tutan üç farklı CountdownLatch kullanıyoruz. Her iş parçacığının giriş yaptığı hazır mandalımız ve ana iş parçacığının beklediği bir mandalımız var. Tüm iş parçacıkları giriş yaptığında, zamanlayıcı başlar ve tüm çalışan iş parçacıklarının beklediği başlatma mandalını tetikler. Tüm çalışan iş parçacıkları işlerini yapar ve daha sonra bitmiş mandalı aracılığıyla giriş yapar. Ana iş parçacığı bitmiş mandalının açılmasını bekler ve daha sonra bitmiş işlem süresini işaretler.
- Bu örnek hakkında birkaç şeyi daha ele alalım. İş parçacığı sayısıyla eşleşmeyen bir eşzamanlılık sayısı geçirirsek ne olur? Çok az olsaydı, zamanımızı erken başlatır ve zamanlayıcımızı bitirirdik. İş parçacığı sayımızdan yüksek olsaydı, iş parçacığı açlığı çıkmazı olarak bilinen bir çıkmaza girerdik. Ayrıca, InterruptedException'ı yakaladığımızı da fark edeceksiniz. Bu istisna türü yakalandığında, kural gereği, iş parçacığının sahibine iş parçacığının kesildiğini bildirmek ve uygun gördüğü şekilde bunu işlemesine izin vermek için Thread.currentThread().interrupt()'ı çağırmalıyız. Son olarak, System.nanoTime()'ın System.currentTimeMillis() gibi bir şeye kıyasla kullanımını fark edeceksiniz. Bunun nedeni, daha doğru olması ve sistemin gerçek zamanlı saatinden etkilenmemesidir. Ayrıca, Runnable önemli miktarda iş temsil etmediği sürece, bu işlevin çok ilginç işler döndürmeyeceği de unutulmamalıdır. Bunun nedeni, System.nanonTime()'ın bile mikro kıyaslama için yeterince doğru olmamasıdır. Bu nedenle, JMH gibi araçlar bu özel amaç için mevcuttur.
- Bu, çekirdek diline yerleştirilmiş eşzamanlı yardımcı programlar tarafından sağlanan yardımcı programları kapsamaya başlar. Bu yardımcı programları daha derinlemesine incelemekten çekinmeyin.
- Wait ve notify'ı doğrudan kullanmaktan daha iyi yöntemler olmasına rağmen, bu işlevleri kullanan kodu korumamız gerekebilir. Wait yöntemi, adından da anlaşılacağı gibi, bir iş parçacığının bir koşul için beklemesini sağlamak için kullanılır. Çağrıldığı yöntemi kilitleyen senkronize bir bölgede çağrılmalıdır. Günlük kullanım şu şekildedir:
```java
synchronized(obj) {
  while(<condition does not hold) {
    obj.wait();
  }
  // perform action now that condition holds.
}
```

- Dikkat edilmesi gereken noktalar, beklediğimiz koşulun doğru olup olmadığını kontrol etmek için her zaman bir döngü içinde wait'i çağırmamız gerekir. Beklediğimiz koşul doğruysa ve notify veya notifyAll yöntemi wait yöntemi çağrılmadan önce çağrılırsa, iş parçacığının uyanacağının garantisi yoktur. Kontrolü bir döngüye koyarak güvenliği sağlarız. İş parçacığı koşul geçerli olmadan önce wait'i geçerse, değişmezimizin korumasını kaybederiz. Bir iş parçacığının beklediği koşul doğru olmadığında uyandırılmasının birkaç yolu vardır.

    - Başka bir iş parçacığı da bilgilendirilebilir ve kilidi alabilir.
    - Başka bir iş parçacığı notify'ı yanlış bir şekilde çağırmış olabilir.
    - Çağıran iş parçacığı, notify'ı aslında hazır olmadan çok erken tetiklemiş olabilir.
    - Nadir durumlarda, bekleyen bir iş parçacığı notify çağrısı olmadan bile uyandırılabilir.
- Wait and notify tartışılırken ortaya çıkan bir konu, notify veya notifyAll kullanılıp kullanılmayacağıdır. Hatırlatma olarak notify bir bekleyen iş parçacığını uyandırır ve notifyAll tüm bekleyen iş parçacıklarını uyandırır. Tüm bekleyen iş parçacıklarını uyandırmak güvenli ve muhafazakar bir seçimdir. Her zaman uyandırılması gereken tüm iş parçacıklarını uyandırmanızı garanti eder. Aslında ihtiyacınızdan daha fazla iş parçacığını uyandırıyor olabilirsiniz, ancak bekledikten sonra devam etmeden önce koşulu düzgün bir şekilde kontrol ediyorsanız, uyandırılan bu ek iş parçacıkları basitçe beklemeye geri dönecektir. Yalnızca notify kullanmak biraz optimizasyona yol açabilir, ancak uzun vadede buna değmez.
- Basitçe söylemek gerekirse, wait ve notify'ın yeni kodda nadiren kullanılması gerekir. Modern eşzamanlı yardımcı programları kullanarak çok daha basit, daha güvenli ve muhtemelen daha performanslı kodlar elde edebiliriz. Wait and notify kullanan bir kod sürdürürsek, işlevselliği doğru kullandığımızdan emin olmalıyız. Devam etmeden önce her zaman kontrol edin, koşul karşılanmazsa döngüye alın ve notify yerine notifyAll'ı tercih edin.

## Document thread safety

- (https://blog.scaledcode.com/blog/effective-java/document_thread_safety/)
- Yazdığımız sınıfların kullanıcıları sınıfların nasıl davrandığını bilmelidir. Bir kullanıcının bilmesi gereken önemli bir konu da sınıfımız thread safe olup olmadığıdır.
- Thread safety level'ları:
 
  - Immutable objects : Nesneler değiştirilemez olduğu için bunlar doğası gereği thread safe'tir. String, Long, BigInteger
  - Unconditionally thread-safe objects : AtomicLong ve ConcurrentHashMap gibi örnekler
  - Not thread-safe objects : Nesneler değiştirilebilir veriler içierir ve senkronizasyon için bir çaba gösterilmez. Burda thread safety isteniyorsa kullanıcı kendisi bu senkronizasyonu sağlamalıdır. ArrayList, HashMap gibi örnekler
  - Thread-hostile : Bu nesneler genellikle bilerek bu şekilde uygulanmaz ve mükemmel harici senkronizasyon gerçekleştirseniz bile kullanımı güvenli değildir. Bunun meydana gelebileceği en yaygın yol, statik değerlerle güvenli olmayan etkileşimdir.
- İş parçacığı güvenliği, bir uygulama içinde zor bir sorun olabilir. Kullandığımız sınıflar, hangi iş parçacığı güvenliği düzeyini uyguladıklarını belgelemediğinde çok daha zor hale gelir. Doğru veya yanlış iş parçacığı güvenliği düzeyi yoktur, farklı davranış türleri farklı iş parçacığı güvenliği düzeylerine katkıda bulunabilir. Durumlarını belgeleyebildiğimiz seviye ne olursa olsun. Tüm kilit nesnelerinizi nihai ve mümkün olduğunda özel yapmayı unutmayın. Bu yönergeleri izleyerek eş zamanlı bir ortamda çalışmanın zorluklarından bazılarını azaltabiliriz.


## Use Lazy Initialization Judiciously

- Lazy initialization, bir nesnenin veya işlemin oluşturulmasını ihtiyaç duyulana kadar erteleme kalıbıdır. Bu kalıbın arkasındaki fikir, nesneye asla ihtiyaç duymayacağınız ve böylece başlatma maliyetlerinden tasarruf edeceğinizdir. Lazy initialization'ın kullanılmasının temel nedeni bir optimizasyon olarak kullanılmasıdır. Lazy initialization'ın diğer kullanımı, kodunuzdaki zorlu dairesel bağımlılıkları(circular dependencies) kırmaktır.
- Double check idiom'a bakmak gerekir.
```java
private volatile FieldType field;

private FieldType getField() {
  FieldType result = field;
  if (result == null) {
    synchronized(this) {
      result = field;
      if (result == null) {
        field = result = computeFieldValue();
      }
    }
  }
  return result;
}
```

- single check idiom: 
```java
private volatile FieldType field;

private FieldType getField() {
  FieldType result = field;
  if (result == null) {
    field = result = computeFieldValue();
  }
  return result;
}
```

- Lazy initializatio'ı ele almaya çalışırken birçok komplikasyon vardır. Mümkünse bundan kaçınmalıyız. Bununla birlikte, testten sonra faydalı olacağı doğrulanırsa, tembel başlatma performansı iyileştirmek veya dairesel bir bağımlılığı kırmak için yararlı olabilir. Yukarıda sunulan, eş zamanlı bir ortamda bile güvenliği ve performansı dengeleyen kanıtlanmış yöntemler mevcuttur.


## Don't Depend on the Thread Scheduler

- Bu konuyu özetlemek gerekirse, uygulamanıza doğruluk sağlamak için JVM'nizin iş parçacığı zamanlama algoritmasına güvenmeyin. Bu, Thread.yield ve iş parçacığı önceliklerine güvenmeyi içerir. Bunun yerine, çalıştırılabilir iş parçacıklarının sayısını, sisteminizin eşzamanlı olarak çalıştırabileceği yürütülebilir işlem sayısı civarında tutmaya çalışın.


## 