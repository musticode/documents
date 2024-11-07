# Singleton Design Pattern

Singleton modeli, Single Responsibility Principle'ı ihlal ederek aynı anda iki sorunu çözer:

- Bir sınıfın yalnızca tek bir örneği(instance) olmalıdır.Neden biri bir sınıfın kaç tane örneği olduğunu kontrol etmek ister ?Bunun en yaygın sebebi, örneğin bir veritabanı veya dosya gibi bazı paylaşılan kaynaklara erişimi kontrol etmektir.

Diyelim ki bir nesne oluşturdunuz,daha sonra başka bir yerde daha bu nesneye ihtiyacınız oldu ve yeniden yaratmaya karar verdiniz.Bu noktada size yeni bir nesne yerine var olan nesne dönecektir.

Bu davranış normal bir oluşturucu (constructor) ile mümkün değildir, çünkü tasarımı gereği oluşturucular her zaman yeni bir nesne döndürmek zorundadır.



## Singleton

```java
class Singleton{
    private static Singleton singleton = new Singleton();
    
    private Singleton(){
        System.out.println("Singleton sınıfından bir nesne oluşturuldu.");
    }
    
    public static Singleton getInstance(){
        return singleton;
    }
}

public class Main{
    public static void main(String[] args) {
        
        Singleton singleton1 = Singleton.getInstance();
        Singleton singleton2 = Singleton.getInstance();
        
        if(singleton1==singleton2){
            System.out.println("Bu nesneler birbiri ile aynı");
        }
        else{
            System.out.println("Bu nesneler birbirlerinden farklı");
        }
    }
}
```
- Görüldüğü gibi Singleton sınıfına ait iki farklı referansa getInstance() metodu ile değer atadık. Burada değer atama işlemi iki defa gerçekleştirilmesine rağmen Constructor içerisinde bulunan mesajı yalnızca bir kez ekranda görüntüledik. Buradan Constructor metodun sadece bir kez çalıştığını anlayabiliyoruz. Ayrıca referansların eşitlik kontrolüne baktığımızda nesnelerin birbiri ile aynı olduğunu gördük. Demek ki gerçekten bu sınıftan yalnızca bir nesne oluşturuyoruz ve dışarıdan Constructor metodun kullanımını engellemiş oluyoruz.

## Lazy singleton

- Singleton Design Pattern tasarımında gereksiz nesne oluşturma işleminin önüne geçmek için gerçekleştirilir. Temelde, nesne yaratma işleminin getInstance() metodu içerisinde gerçekleştirilmesidir. Bu gerçekleştirmede yine sadece tek bir nesne oluşturmak adına bir adet “null” kontrolü yapılır. Eğer nesne “null” değere sahip ise yalnızca tanımlanmış ancak yaratılmamıştır. Bu durumda nesne yaratma gerçekleştirilir. Bununla beraber, eğer nesne null değere sahip değil ise nesne yaratma işlemi hiç gerçekleştirilmez. Böylece tek nesne oluşumu garantilenmiş olur. Lazy Singleton yapısı aşağıdaki gibidir :

```java
class LazySingleton{
    private static LazySingleton singleton;
    
    private LazySingleton(){
        System.out.println("Singleton sınıfından bir nesne oluşturuldu.");
    }
    
    public static LazySingleton getInstance(){
        if(singleton==null){
            singleton = new LazySingleton();
        }
        return singleton;
    }
}

public class Main{
    public static void main(String[] args) {
        
        LazySingleton singleton1 = LazySingleton.getInstance().getInstance();
        LazySingleton singleton2 = LazySingleton.getInstance();
        
        if(singleton1==singleton2){
            System.out.println("Bu nesneler birbiri ile aynı");
        }
        else{
            System.out.println("Bu nesneler birbirlerinden farklı");
        }
    }
}
```

- Tek nesne oluşturuldu ve bu işlem sadece nesne kullanımına ihtiyaç duyulması halinde gerçekleştirildi. Bu yüzden temel singleton yapısındaki performans eksikliği giderildi.
- İhtiyacımızı karşılamış gibi görünsek de Lazy Singleton yapısı güvenilirlik açısından kusursuz değildir. Multi-thread kullanımının olduğu bir ortamda tek nesne oluşumu garanti edilemez. 

Lazy Singleton yapısındaki “null” kontrolünün aynı anda yalnızca tek bir iş parçacığı tarafından çalıştırılmasını sağlamak amacıyla “ synchronized “ anahtarı kullanılır :

```java
public static ThreadedLazySingleton getInstance(){ 
        synchronized(ThreadedLazySingleton.class){
            if(singleton==null){
                singleton = new ThreadedLazySingleton();
            }
        }
        return singleton;
}
```
`getInstance()`metodu yukarıdaki gibi güncellendiği taktirde daime tek nesne oluşacaktır. Bu şekilde güvnelik açığı kapatılıyor. 

- İhtiyacımızı karşılamış gibi gözüksek de yukarıdaki Lazy Singleton yapısı yine performans olarak bizi tatmin etmez. Bunun sebebi ise yukarıdaki tasarımın iş parçacıkları arasında gereksiz kontrol yapmaya sebebiyet vermesidir. Dolayısıyla, çok iş parçacığı kullanılan yazılımlarda performans açısından verim oldukça düşük olacaktır. Bu performans düşüşünden kurtulmak için ise yapıya bir kontrol daha eklenir. Bu işleme Double-Checked Locking adı verilir.


### Double-checked Locking

- Lazy Singleton tasarımındaki performans düşüşünü engellemek için gerçekleştirilir. Temelde, getInstance() metodu içerisinde synchronized olarak gerçekleştirilen işlemleri belirli bir kurala bağlamaktır. Bu kural singleton nesnesinin null olmaması durumudur. Eğer singleton null ise synchronized bir şekilde çalışma gerçekleştirilirken; nesnenin null olmaması durumunda herhangi bir işlem gerçekleştirilmeyecektir. Double-Checked Locking yapısı aşağıdaki gibidir :

```java
class DoubleCheckedLockingSingleton{
    private static DoubleCheckedLockingSingleton singleton;
    
    private DoubleCheckedLockingSingleton(){
        //Constructor işlemleri
    }
    
    public static DoubleCheckedLockingSingleton getInstance(){
        if(singleton==null){
            synchronized(DoubleCheckedLockingSingleton.class){
                if(singleton==null){
                    singleton = new DoubleCheckedLockingSingleton();
                }
            }   
        }
        
        return singleton;
    }
    
}
```

Bu şekilde getInstance metoduna bir adet daha null kontrolü eklenirç. Bu şekilde gereksiz `synchronized` çalışmaların önüne geçilir ve performans artırılabilir.

