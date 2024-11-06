# Generics

- Generic türlerde tür bilgisi çalışma zamanında silinmektedir. Peki program çalışır mı? Evet ama derleyici bundan emin olamıyor. Siz buna kendinizi ikna edip SuppressWarning notasyonu ile gizleyebilirsiniz, ancak uyarıdan da kurtulmak en doğrusu olacaktır. Derleyici hatasını gidermek için dizi yerine liste kullanabiliriz.

```java
class ListChooser<T>{
    private final List<T> choiceList;

    ListChooser(List<T> choiceList) {
        this.choiceList = new ArrayList<>(choiceList);
    }

    public T choose(){
        Random rnd = ThreadLocalRandom.current();
        return choiceList.get(rnd.nextInt(choiceList.size()));
    }
}
```

Yukarıdaki versiyonda biraz performans kaybı bulunsa da çalışma zamanında ClassCastException hatası fırlatmayacağını garanti ediyor. Bundan dolayı tercih sebebi olabilir.

- Özetle, generic türler ve dizilerin uymak zorunda olduğu tür kuralları birbirlerinden farklıdır. Bu farklılıklar yüzünden, diziler çalışma zamanında tür güvenliği sağlarken derleme zamanında sağlayamazlar. Generic türler içinse bu durum tam tersidir. Bu yüzden de bu ikisi birbiriyle tam uyumlu çalışamazlar. Eğer bunları beraber kullanmaya çalışırken derleyiciden hata ve uyarı alırsak dizileri listelere dönüştürmeyi denemek gerekir.

## Generic türleri tercih etmek

- Bazı sınıflar direkt olarak generic olmalıdır.
- Stack sınıfı en başından beri üreysel olması gerekirdi. Ama sonradan mevcut clientlere zarar vermeden generic türe dönüştürmek de mümkün olaiblecektir. Bu haliyle, client'lar stackten okudukları değerleri kullandıkları türe kendileri dönüştürmek zorundadır. Bu dönüşümler de başarısız olabilir, exception fırlatılabilir.
- Bir sınıfı generic'e çevirmek için ilk adım, sınıf tanımına bir veya daha çok tür parametresi eklemektir. Bizim örneğimizde eleman türünü temsil edecek şekilde tek bir tür parametresi yeterlidir. Eleman türünü temsil eden tür parametreleri genellilkle E olarak isimlendirilir.

```java
elements = new E[DEFAULT_INITIAL_CAPACITY];
```

derlenmez

* E gibi tür parametrelerinden dizi yaratamayız. Bu problem arka planda dizi kullanan generic türler yazmaya çalıştığınızda her zaman karşınıza çıkacaktır. Mantıklı bir şekilde çözmek için iki yol bulunur. Birincisi diziyi E türünde yaratmak yerine Object türünde yaratıp sonradan E türüne dönüştürebiliriz.

```java
elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
```

bu kullanım geçerlidir fakat yine de type safety'yi garantilemez.

* Burada söz konusu olan elements dizisi private bir alanda saklanıor ve istemciye asla döndürülmüyor. Diziye eleman eklenen tek yer push metodu. Burada da E türünde geçilen parametreler diziye eklendiği için tür güvenliğini sıkıntıya sokacak bir durum yok. Tür dönüşümü güvenlidir.
* Verilen stack örneği diziler yerine listeler kullanımı önerilen madde ile çakışabiliyor. Generic türler içerisinde dizi yerine liste kullanmak her zaman mümkün olmayabilir veya istenmeyebilir. ArrayList gibi bazı generic türler mecburen dizileri kullanacak şekilde yazılmıştır, HashMap gibi örnekler de performans kazanımı için dizilerden faydalanmıştır.
* Bizim stack örneğinde olduğu gibi generic türlerin büyük çoğunluğu herhangi bir kısıtlamaya gitmeden istediğiniz tür parametrelerini kullanmamıza izin verirler. Örnek olarak `Stack<Object>` `Stack<int[]>` gibi kullanımların hepsi geçerli olaiblir. İstediğininiz herhangi bir referans türünden elemanlar tutan bir Stack yaratabilirsiniz. Ancak int double gibi temel türleri tutan bir stack yaratmaya çalışırsak derleme hatası alırız. Bu, Java'da generic tür sisteminin kısıtlarından biridir. Ancak bunların yerine Integer, Double gibi boxed primitive type kullanarak bu sorunu aşabiliriz.
* Kullanılabilecek tür parametrelerinde kısıtlama yapan generic türler de mevcuttur. Örnek olarak `java.util.concurrent.DelayQueue` sınıfının tanımı aşağıdaki gibidir:

```
class DelayQueue<E extends Delayed> implements BlockingQueue<E>
````

Burada parametre olarak tanımlanan `<E extends Delayed>`ifadesinin anlamı şudur: istemci tür parametresi olan E yerine sadece Delayed türünün bir alt türünü geçebilir. Bu şekilde DelayQueue sınıfı ve bu sınıfın istemcileri, DelayQueue'nun elemanları için Delayed türünden gelen metotları da tür dönüşümüne gerek kalmadan ve ClassCastException riski olmadan kullanabilirler. Buradaki tür parametresi E aynı zamanda sınırlandırılmış tür parametresi(Bounden type parameter) olarak da anılır.

* Özet olarak, istemcilerin kullanırken tür dönüşümü yapmak zorunda kaldığı türlere kıyasla üreysel türleri kullanmak hem daha güvenli hem de istemcilere kullanım kolaylığı sağlar. Yeni türler tanımlarken istemcilerin tür dönüşümü yapmadan bunları kullanabileceğinden emin olmak gerekiyor. Bu çoğu zaman bu türlerin generic olarak tasarlanması anlamına gelmektedir. Eğer generic olması gerektiği halde olmayan türleriniz varsa bunları güvenle generic'e dönüştürebiliriz. Çünkü mevcut istemciler etkilenmeyecek ve yeni istemciler için kolaylık sağlanacaktır.

## Generic metotları tercih etmek

- Aynı sınıflar gibi metotlar da generic olarak yazılabilir. Parametreli türlerle çalışan statik yardımcı metotlar(utility methods) genellikle generictir. Örnek olarak Collections içerisindeki binarySearch ve sort gibi algorima metotlarının hepsi generictir.

```java
    // Generic method
    public static <E> Set<E> unionSet(Set<E> s1, Set<E> s2){
        Set<E> result = new HashSet<>(s1);
        result.addAll(s2);
        return result;
    }
```

Generic `union` metodunda hem metot parametreleri hem de dönüş tipi aynı türden olmak zorundadır. Bunu biraz esnekleştirmek için sınırlandırılmış joker tür(bounded wildcard type ) kullanılabilir.

- Bazı durumlarda, immutable ama birçok türle birlikte kullanılabilen nesneler yaratmak isteyebiliriz. Java'da generic mekanizması çalışma zamanında tür bilgisini silecek şekilde tasarlandığı için(type erasure) birçok parametreli türleri yaratacak static factory method yazmak gerekir. Bu tasarım generic singleton factory olarak bilinir.
- Birçok static yardımcı metot Comparable interface'ini uygulayan bir COllection parametresi alarak sıralama, arama, en küçük ve en büyük değerleri hesaplama gibi işlemler yapar. Bunu yapabilmek için Collection içinde yer alan elemanların birbirleriyle karşılaştırılabilir olması gerekir. Bu koşulu aşağıdaki gibi ifade eedebiliriz:

`public static <E extends Comparable<E>> E max(Collection<E> c);`

```java
     // generic tür sınırlaması kullanarak collectin'daki en büyük değer bulunur
     public static <E extends Comparable<E>> E max(Collection<E> c){
         if (c.isEmpty()){
             throw  new IllegalArgumentException("Empty collection");
         }

         E result = null;
         for (E e : c){
             if (result == null || e.compareTo(result) > 0){
                 result = Objects.requireNonNull(e);
             }
         }

         return result;
     }
```

## Wildcard kullanımı

- Aşağıdaki kodu stack class'ına ekleyebiliri. Derleme olarak sorun olmaz. Ama eğer `Iterable<Integer> integers` ile eklemek istersek hata alabiliriz. Yani `src` hem integer hem de String değerler ile eklenebilir.

```java
    public void addAll(Iterable<E> src){
        for (E e : src){
            push(e);
        }
    }

```

Bu üstteki şekilde aşağıdaki örnek ile metodu kullanmak istersek hata alırız

```java
       GenericStack<Number> numberStack = new GenericStack<>();
        Iterable<Integer> integers = Arrays.asList(1,2,3,4);
        numberStack.pushAll(integers);
```

Number ile Integer arasında bağ yoktur. Bu hatalardan kurtulabilmek için wildcard kullanmalıyız.

Bizim `pushAll` metodundan istediğimiz aslında sadece `E` türünden değil, `E`‘nin alt türlerinden nesneler içeren bir `Iterator` geçtiğimizde de doğru çalışarak bunları yığıta eklemesi.
`public void pushAll(Iterable<? extends E> src)` ile ekleyebilirsek hata ortadan kalkacaktır. Burdaki ? işareti ile E'yi extend eden her türü alabiliriz manasına geliyor. Class'a integer Number, metoda da Number geçiliyor, dışardan kullanırken de Integer'ı extend eden class'ı da kullanabiliriz anlamına gelmektedir.
Integer class'ının detayına baktığımızda aşağıdaki şekilde göreceğiz
`public final class Integer extends Number` yani Integer class'ı Number'ı extend ediyor

- üst türler için de aşağıdaki gibi kullanabiliriz

```java
    public void popAll(Collection<? super E> dst){
        while (!dst.isEmpty()){
            dst.add(pop());
        }
    }
```

Burada dikkat edilecektir ki extends ifadesi yerine super geldi. Java’da her tür kendisinin üst türü olarak kabul edildiği için bu ifade hem E türünün kendisini de kapsamaktadır.

- Buradaki ders bellidir. Esnekliği artırmak için üretici (producer) ve tüketicileri (consumer) temsil eden girdi parametrelerini (input parameter) joker tür kullanarak tanımlayın. Farklı bir biçimde ifade edecek olursak, metoda geçilen parametreli tür T nesneleri üretmek için kullanılıyorsa <? extends T>, T nesnelerini tüketmek için kullanılıyorsa <? super T> kullanın.
- Bizim pushAll örneğinde parametreli koleksiyon türü yığıt için yeni eleman üretmek için kullanıldığı için (üretici) extends, popAll metodunda ise benzer parametre yığıttan eleman çıkarmak için kullanıldığı için (tüketici) super kullanılmıştır. Şunu da belirmek gerekir ki, bir parametre hem üretici hem de tüketici görevi görüyorsa o zaman joker türü kullanmanın bir anlamı yoktur. Joker tür kullanmadan normal tür parametresi geçmek daha doğru olacaktır.
- Bu kuralı unutmamak için PECS olarak aklımızda tutabiliriz. Bunun açılımı yukarıdaki anlatıma uygun olarak producer-extends, consumer-super olarak düşünülebilir. (producer=üretici, consumer=tüketici)

```java
// T üreticisi görevi gören parametre için joker tür kullanımı
public Chooser(Collection<? extends T> choices)
```

- Peki bu yeni tanım pratikte ne işimize yarayacak? Diyelim ki elimizde Chooser<Number> var ve biz yapıcı metoda List<Integer> geçmek istiyoruz. Orijinal tanımda bu mümkün olmazdı ama yukarıdaki gibi joker tür kullandığımızda sorunsuz çalışacaktır.
- Özetle API tasarlarken joker türler kullanmak esneklik artırabilecektir. Temel PECS de unutulmamalıdır. (Producer extends, consumer super)

## Generic ile varargs kullanırken dikkatli olmak

- Varargs metotların amacı istemcilerin metotlara değişken sayıda argüman geçebilmelerini sağlamaktır. Bir varargs metot çağırıldığı zaman bu değişken sayıdaki argümanlar bir dizi yaratılarak onun içinde tutulur. Bu dizi, her ne kadar bir gerçekleştirim detayı olsa da gizlenmiş değil erişilebilir durumdadır. Sonuçta, varargs parametreler generic türler içerdiği zaman derleyiciden hata alırız.
- Uyarıları engelleyebilmek için SafeVarargs notasyonu dile eklenmiştir. Bu notasyon, programcının metotta tür güvenliğinin var olduğuna dair bir taahhüdüdür, derleyici de uyarı vermeyi bırakır. Bu notasyon kullanılmadan önce metotta gerçekten de tür güvenliği olduğundan emin olmalıyız. Varargs parametreli bir metot çağrıldığında parametre değerlerini tutmak için bir dizi yaratılıyordu, eğer metot bu diziye hiçbir şey yazmıyorsa ve dizi referansını başka metotların erişimine açmıyorsa o zaman tür güvenliği vardır diyebiliriz. Başka deyişle, varargs parametre dizisi sadece parametre değerlerini taşımak için kullanılıyorsa o zaman güvenlidir. Zaten bu metotların amacı budur.
- Generic varargs parametre dizilerinin referansı başka bir metotdun erişimine açılmamalıdır. Bunun iki istisnası vardır. Bu dizinin referansının @SafeVarargs ile işraetlenmiş başka bir varargs metoda veya varargs olmayan ve bu dizinin elemanlarını sadece bir hesaplama yapmak için kullanan bir metoda geçmekte bir sakınca yoktur.
- Güvenli kullanım :

```java
    @SafeVarargs
    static <T> List<T> flatten(List<? extends T>... lists){
        List<T> result = new ArrayList<>();
        for (List<? extends T> list : lists){
            result.addAll(list);
        }
      
        return result;
    }
```

Önemli 2 kural

1. Varargs parametre dizisine hiçbir değer yazmamak
2. Diziyi kontrolümüzde olmayan başka kodların erişimine açmamak

- Eğer düzgün ve güvenli bir varargs metot yazamıyorsak bu durumlarda `toArray()` metodu kullanmak yerine `List.of` ile bunu çözebiliriz.
  hatası düzeltilmiş pickTo metodu

```java
    static <T> List<T> pickTwo(T a, T b, T c){
        switch (ThreadLocalRandom.current().nextInt(3)){
            case 0 : return List.of(a, b);
            case 1 : return List.of(a, b);
            case 2 : return List.of(a, b);
        }
        throw new AssertionError();
    }
```

- Özetle varargs ve genericler birlikte düzgün çalışmazlar. Çünkü varargs arka tarafta bir dizi kullanmakta ve bu diziyi dışarı sızdırmaktadır. Genericlerle beraber varargs kullanmak tür güvenliğini tehdit etse de geçerli bir kullanımdır. Böyle bir metot yazmak isterseniz, metodun tür güvenliğinden emin olun ve SafeVarargs notasyonu ile bunu diğer istemcilere ve derleyicilere bildirin

## Tür güvenlikli heterojen taşıyıcılar(Typesafe heterogenous)

- Generic türlerin yaygın olarak bilinenleri Set<E>, Map<K,V> gibi koleksiyonlar ve ThreadLocal<T> ve AtomicReference<T> gibi tek elemanlı taşıyıcılardır. Bütün bu kullanımlarda taşıyıcı olan türün kendisi parametrelerle ifade edilmektedir. Bu bizi her taşıyıcı için sabit bir sayıda tür parametresi kuollanmaya zorlamaktadır. Örneğin Set eleman türünü belirleyen tek bir tür parametresine sahiptir, Map ise iiçndeki girdilerin anahtar ve değerlerini (key-value) tutmak üzere iki tane tür parametresine sahiptir. Bazen bunlardan daha fazla esneklik isteyebiliriz, örneğin database ile işlemleri daha kapsamlı yapabilecek olan sınıflar.
- Bu gibi istekklermizi taşıyıcı tür parametrelerle ifade etmek yerine (parametrize) taşıyıcının içinde tür anahtarı kullanabiliriz. Bu tür anahtarı taşıyıcıya farklı türlerde değer yazmak ve okumak için kullanılacaktır.
örn : 

```java
class Favorites{
    private Map<Class<?>,Object> favorites = new HashMap<>();
    public <T> void putFavorite(Class<T> type, T instance){
        favorites.put(Objects.requireNonNull(type),instance);
    }
    public <T> T getFavorite(Class<T> type){
        return type.cast(favorites.get(type));
    }
}
```
