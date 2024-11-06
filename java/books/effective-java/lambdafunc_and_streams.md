# Lambda Fonksiyonlar ve Stream'ler

## Lambda fonksiyonlarını isimsiz sınıflara tercih etmek

- Eskiden beri tek bir soyut metodu olan arayüzler fonksiyon türleri olarak kullanılmıştır. Bunların nesneleri de fonksiyon neneleri olarak bilinirler ve bir işlevi veya eylemi temsil ederler. Javanın ilk sürümünden itibaren fonksiyon nesneleri yaratmanın birincil yolu isimsiz sınıflar(anonymous class) kullanmaktı. Aşağıda da bir words listesini alıp karakter sayısına göre sıralayan kod parçası görülüyor.

```java
Collections.sort(words, new Comparator<String>() {
    public int compare(String s1, String s2){
      return Integer.compare(s1.length(), s2.length());
    }
});
```

- İsimsiz sınıflar yazarken ortaya çıkan gereksiz ve uzun kodlar java'yı fonksiyonel programlama yapmak için elverişsiz bir hale getiriyordu.
- Java 8'le birlikte tek bir soyut metodu olan interface'lere özel bir anlam yüklenmiştir ve bunlara artık fonksiyonel interface denmektedir. Bunlardan nesne yaratmak için de lambda ifadeleri dile eklenmiştir. Lambdalar işlevsel olarak isimsiz sınıflara benzerdir ancak çok daha kısa bir biçimde ifade edilirler.

```java
Collections.sort(words, (s1, s2) -> Integer.compare(s1.length(), s2.length()));
```

- Dikkat edilirse lambda ifadesi yazılırken lambda fonksiyonunun türü (Comparator<String>), parametrelerin türleri hem s1 hem de s2 String türündedir ve dönüş türü kodda mevcut değildir. Derleyici bu türlerin ne olduğunu tür çıkarsama(type inference) denilen bir teknikle belirler. Bu, kuralları çok karmaşık olduğundan bazı durumlarda bu çıkarım yapılamayabilir. Bu durumlarda türü bizim belirtmemiz gerekir. Lambda parametrelerininn türlerini programın okunabilirliğini kötü etkilemediği sürece yazmayın. Eğer derleyici size tür çıkarımı yyapamadığını belirten bir hata verirse o zaman türleri açıkça belirtin. Nadir de olsa bazen dönüş değeleri veya lambda ifadelerinin bütünü için tür dönüşümü cast yapmamız gerekebilir.
- [Type Inference](https://yteblog.bilgem.tubitak.gov.tr/local_variable_type_inference_var_)
- Tür çıkarsama ile ilgili bir uyarı daha yapalım. Madde 26’da ham (raw) türleri kullanmayın, Madde 29’da üreysel (generic) türlere öncelik verin, Madde 30’da üreysel metotlara öncelik verin gibi tavsiyelerde bulunmuştuk. Bu tavsiyeler lambda fonksiyonları söz konusu olduğunda iki kat daha önem kazanmaktadır çünkü derleyici tür çıkarsama yaparken üreysellik mekanizmasını kullanmaktadır. Bunu yapmadığınız taktirde derleyici tür çıkarsama yapamayacaktır ve sizin türleri belirtmeniz gerekecektir. Örnek verecek olursak, yukarıda lambda kullandığımız örnekte String nesnelerini tutan words değişkeni List<String> yerine List türünde olursa lambda kodu derlenmeyecektir.
- Metotlar ve sınıfların aksine, lambda fonksiyonları isimsizdir ve dökümantasyonları olmaz. Eğer yaptığınız hesaplama ek açıklamaya ihtiyaç duyuyorsa veya birkaç satırdan fazla kod yazmayı gerektiriyorsa lambda kullanmayın. Lambda fonksiyonları için tek satır idealdir, üç satıra kadar da kabul edilebilir. Bunu ihlal ederseniz kodun okunabilirliğine zarar vermiş olursunuz. Eğer bir lambda uzunsa veya okunması zorsa ya bunu basitleştirmeye çalışın ya da tamamen kaldırın. Ayrıca, enum yapıcı metotlarına geçilen argümanlar statik bağlamda değerlendirildiği için lambda fonksiyonları enumların nesne alanlarına ve metotlarına erişemezler. Yani, önceki örnekte kullanılan enum sabitlerine gövde ekleyerek metotları geçersiz kılma yöntemi eğer yapılan hesaplamalar lambda fonksiyonunun okunabilirliğini bozacak kadar uzunsa veya nesne alanlarına/metotlarına erişmesi gerekiyorsa halen geçerlidir.
- Aynı şekilde, lambda fonksiyonları varken artık isimsiz sınıflara da gerek kalmadığını düşünebilirsiniz. Bunun gerçekliği daha fazladır ama yine de isimsiz sınıflarla yapıp lambdalarda yapamadığınız birkaç şey vardır. Lambdalar ancak fonksiyonel arayüzlerle kullanılabilir. Bir soyut sınıftan nesne yaratmak istiyorsanız bunu isimsiz sınıfla yapabilirsiniz ama lambda ile yapamazsınız. Benzer şekilde, birden fazla soyut metodu olan arayüzlerden nesne yaratmak isterseniz de isimsiz sınıflar bunu destekler ama lambda desteklemez. Son olarak, bir lambda fonksiyonu kendisine referans edemez çünkü lambda içinde kullanacağınız this anahtar kelimesi onu çevreleyen nesneyi tutmaktadır. Dolayısıyla lambda içinde fonksiyon nesnesine erişmeniz gerekiyorsa isimsiz sınıf kullanmanız gerekir.
- Lambdalar da aynen isimsiz sınıflarda olduğu gibi serileştirme (serialize) konusunda pek güvenli değillerdir. Bu sebeple lambda fonksiyonlarını ve isimsiz sınıf nesnelerini serileştirmeye kalkmayın. Böyle bir ihtiyaç doğarsa bir private static gömülü sınıf (nested class) tanımlayıp onun nesnelerini kullanın.

## Metot referanslarını lambdalara tercih etmek

- Lambda fonksiyonlarının isimsiz sınıflara göre avantajı daha kıs ave öz olmalarıdır. Java bizlere fonksiyon nesneleri tanımlamak için kullanılabilecek lambdadan daha kısa birr yol sunmaktadır: metot referansları. Elimizde belli anahtarları INteger değerlerine eşleştiren bir map olsun bu Integer değerler anahtarın veri yapısına kaç kere eklendiğini tutmaktadır. Yani ekleme işlemi yapılırken veri yapısına daha önceden hiç eklenmemiş bir anahtar için değer 1 olurken, daha önce eklenmiş anahtarlar için var olan değer artırılmaktadır.

```java
Map<String, Integer> integerMap = new HashMap<>();
// gereksiz yer kaplayan metot
integerMap.merge("key", 1, (count, incr) -> count + incr);
```

Buradaki lambda fonksiyonu yerine `Integer::sum` metodu kullanılarak işlem yapılması daha iyi görüntüye sebep olacaktır.
`integerMap.merge("key", 1, Integer::sum);`

- Ayrıca lambda fonksiyon kodlarının uzadığı durumlarda size bir çıkış yolu sunarlar. Bu durumda lambda kodunu bir metoda taşıyabilir ve bu metodun referansını lambda fonksiyonu ile değiştirebilirsiniz. Metoda da istediğiniz bir isim verip güzelce dokümantasyon yapabilirsiniz.
- Metotların lambda karşılıkları :


| Metot Referans Türü   | Örnek                 | Lambda Karşılığı                                  |
| ----------------------- | ---------------------- | ------------------------------------------------------ |
| Statik                  | Integer::parseInt      | str -> Integer.parseInt(str)                           |
| Bağlı (Bound)         | Instant.now()::isAfter | Instant then = Instant.now();<br/>t -> then.isAfter(t) |
| Serbest (Unbound)       | String::toLowerCase    | str -> str.toLowerCase()                               |
| Sınıf yapıcı metodu | TreeMap<K,V>::new      | () -> new TreeMap<K,V>                                 |
| Dizi yapıcı metodu    | int[]::new             | len -> new int[len]                                    |

- Özetle, metot referansları fonksiyon nesneleri yaratmak için lambdaların yanında ikinci bir seçenek olarak karşımıza çıkarlar. Daha kısa ve okunabilir olduğu durumlarda metot referanslarını kullanın, aksi taktirde lambda kullanmaya devam edin.

## Standart fonksiyonel arayüzlerin kullanımına öncelik vermek

- Java diline lambdalar eklendikten sonra API yazmanın kuralları değişti diyebiliriz. Örneğin template method tasarım desenini gerçekleştirirken çocuk sınıfıların ata sınıftaki soyut metotları geçersiz kılması artık pek de çekici değildir. Bunun daha modern olan alternatifi fonksiyon nesnesi kabul eden bir static factory veya constructor metot yazmak olacaktır. Genel olarak şu söylenebilir, fonksiyon nesnelerini parametre olarak alan dha fazla metot tanımlıyor olacaksınız. Bunun için önce doğru fonksiyon türünü seçmek gerekir.
- LinkedHashMap sınıfını ele alalım. Bu sınıfı cache olarak kullanmak için removeEldestEntry metodunu override etmek isteyebiliriz. Bu metot `LinkedHashMap` nesnesine her yeni anahtar eklendiğinde `put` metodu tarafından çağrılmaktadır ve eğer `true` döndürürse en eski eleman silinmektedir. Biz bu metodu aşağıdaki gibi geçersiz kılarak map nesnesine en fazla yüz tane girdi eklenmesine izin verebiliriz. Bundan sonra eklenen her girdi için en eski girdi silinecek ve toplam sayı yüz olarak kalacaktır.
- `java.util.function` paketinde kırk üç tane standart fonksiyonel arayüz bulunmaktadır. Bunların hepsini hatırlamanız tabii ki beklenmez ama altı tane temel arayüzü bilirseniz gerisini bunlardan çıkarabilirsiniz. Bu temel arayüzler nesne referansları ile çalışırlar. `Operator` arayüzleri argüman ve dönüş türü aynı olan fonksiyonları, `Predicate` ise bir argüman alıp `boolean` değer döndüren bir fonksiyonu temsil eder. `Function` arayüzünde ise argüman türü ve dönüş türü birbirinden farklıdır. `Supplier` (sağlayıcı) arayüzünün hiçbir argümanı yoktur ama bir değer döndürür. `Consumer` (tüketici) ise bir argüman alıp hiçbir şey döndürmeyen bir fonksiyonu simgeler. Aşağıda bunları özetleyen bir tablo görüyorsunuz:


| Arayüz           | Fonksiyon İmzası  | Örnek              |
| ----------------- | ------------------- | ------------------- |
| UnaryOperator<T>  | T apply(T t)        | String::toLowerCase |
| BinaryOperator<T> | T apply(T t1, T t2) | BigInteger::add     |
| Predicate<T>      | boolean test(T t)   | Collection::isEmpty |
| Function<T,R>     | R apply(T t)        | Arrays::asList      |
| Supplier<T>       | T get()             | Instant::now        |
| Consumer<T>       | void accept(T t)    | System.out::println |

- Bu altı standart arayüzün int, long ve double temel türlerini desteklemek için üçer tane de türevi vardır. İsimleri de buna uygun olarak seçilmiştir. Mesela int türü kabul eden bir predicate IntPredicate, iki tane long değeri alıp tek bir long döndüren binary operator fonksiyonu ise LongBinaryOperator adını almıştır.
- `Function` arayüzünün bundan başka ek türevleri de vardır. Örneğin `long` değer alıp `int` döndüren fonksiyonun adı `LongToIntFunction` olarak belirlenmiştir. `int`, `long` ve `double` türleri için bütün bu kombinasyonları bulabilirsiniz: `IntToDoubleFunction`, `DoubleToLongFunction` gibi. Temel türden değerler alıp nesne referansı döndüren fonksiyonlar ise `IntToObjFunction`, `LongToObjFunction` gibi isimler alır.
- Artık standart olanlar işimize görüyorsa kendi arayüzlerimizi yazmamamız gerektiğini biliyoruz. Peki bunları kendimiz ne zaman yazmalıyız ve yazarsak nelere dikkat etmeliyiz? Tabii ki bize lazım olan fonksiyon standart arayüzlerin içinde yoksa kendimiz yazmamız gerekir. Örneğin üç parametreli bir `Predicate` lazım olursa yeni bir arayüz yazmamız gerekir. Ancak bazı durumlarda yapısal olarak bizim ihtiyacımızı karşılayan bir arayüz olsa bile yenisini yazmamız daha doğru olacaktır.
- Eski dostumuz `Comparator<T>` arayüzünü ele alalım. Bu aslında yapısal olarak `ToIntBiFunction<T,T>` arayüzü ile aynıdır. İkisi de iki tane `T` türünde parametre alıp bir `int` döndürmektedir. `Comparator` dile eklendiğinde `ToIntBiFunction` henüz yoktu ama olsa bile bunu kullanmak mantıksız olurdu. Ayrı bir `Comparator` arayüzü yazmamızın birkaç avantajı vardır. Birincisi arayüzün ismi comparator (karşılaştırıcı) arayüzün ne iş yaptığı hakkında bize bilgi vermektedir. İkinci olarak, `Comparator` nesnelerinin hangi özellikleri taşıması gerektiği konusunda çok kapsamlı bir sözleşme (contract) bulundurmaktadır. Bu arayüzü uyguladığınızda sözleşmeye uyduğunuzu da kabul etmiş olursunuz. Üçüncüsü, arayüz içerisinde bulunan çok sayıdaki varsayılan metot birden fazla karşılaştırıcıyı birleştirmenizi veya farklı biçimlere dönüştürmenizi sağlar.
- Özetle, artık lambda fonksiyonları Java diline eklendiğine göre API tasarlarken bunu mutlaka göz önünde bulundurmalıyız. Metot parametrelerinde fonksiyonel arayüz türlerini kabul edin ve/veya bunları dönüş türü olarak kullanın. Java kütüphanelerinin bize sunduğu standart fonksiyonel arayüzleri mümkün olduğunca kullanın, ancak kendi arayüzünüzü yazmanız gereken durumlar olabileceğini de unutmayın.

## Stream'leri akıllıca kullanmak

- Stream API Java 8 ile dile eklenmiştir ve sıralı veya paralel toplu işlemleri (bulk operations) kolaylaştırmak amacıyla kullanılır. Bu API stream denilen sonlu veya sonsuz sayıda eleman ve stream hattı (stream pipeline) denilen ve bu elemanlar üzerinde aşamalı olarak hesaplama yapan iki parçadan oluşur. Stream elemanlarının kaynağı diziler, koleksiyonlar (collections), dosyalar (files), rastgele sayı üreticiler veya başka streamler olabilir. Bu elemanların türleri de nesne referansları veya int, long, double temel türleri olabilir.
- Bir stream hattı sıfır veya daha fazla ara işlem (intermediate operation) ve bir tane sonlandırıcı işlemden (terminal operation) oluşur. Her ara işlem stream elemanları üzerinde bir dönüşüm gerçekleştirir: her elemanı bir fonksiyona sokmak veya bunları belli bir filtrelemeden geçirmek gibi. Her ara işlemin sonucu kendinden sonrakine girdi olarak verilir. Sonlandırıcı işlem ise en son yapılan ara işlemden gelen elemanları son bir işleme tabi tutarak bir sonuç üretir: elemanları bir koleksiyona kaydetmek, bunlardan bir tanesini döndürmek veya hepsini yazdırmak gibi.
- Stream hatlarının işletilmesi geciktirilir (lazy execution), yani hesaplama sonlandırıcı işlem çağrılmadan başlamaz. Asıl sonucu o ürettiği için sonlandırıcı işlem açısından gerekli olmayan elemanları işlemek için zaman harcanmaz. Bu sayede streamlerin sonsuz sayıda elemanla çalışabilmesi mümkün kılınmıştır. Sonlandırıcı işlem içermeyen bir stream hattı hiçbir işlem yapmayacaktır, o yüzden eklemeyi unutmayın.
- Kendi haline bırakırsanız stream hatları sıralı (sequential) olarak işletilirler. Bu işletimi paralele dönüştürmek için tek yapmanız gereken stream hattında parallel metodunu çağırmaktır ancak bu çoğu zaman önerilmez. (Madde 48)
- Stream API kullanarak pratikte aklınıza gelen bütün hesaplamaları yapmanız mümkündür ancak bu her zaman stream kullanmanız gerektiği anlamına gelmez. Doğru kullanıldığında streamler hesaplamaları çok kısaltır ve anlaşılır hale getirir. Yanlış kullanıldığında ise okunabilirliği bozar ve bakım yapmayı zorlaştırır. Ne zaman stream kullanılması gerektiği ile ilgili çok keskin kurallar olmasa da belli önerilerde bulunmak mümkündür.
- Lambda ifadelerinde tür bilgisi bulunmadığı için parametre isimlerinin dikkatli seçilmesi stream hatlarının okunabilirliğini artırmak açısından çok önemlidir.


| 1 | `"Hello world!"``.chars().forEach(System.out::print);` |
| - | ------------------------------------------------------ |

Bu kodun `Hello world!` yazdırmasını beklersiniz ama aslında `721011081081113211911111410810033` yazdırır. Bunun sebebi `"Hello world!".chars()` çağrısından gelen stream elemanlarının `char` değil `int` türünde olmasıdır. Bu sebeple `print` fonksiyonu `int` değerler yazdıracaktır. Bunu düzeltmek için kodu aşağıdaki gibi güncelleyebilirsiniz:


| 1 | `"Hello world!"``.chars().forEach(x -> System.out.print((``char``) x));` |
| - | ------------------------------------------------------------------------ |

Ancak en doğrusu **`char`** değerleri işlemek için stream kullanmaktan kaçınmaktır.

- Streamleri kullanmaya başladıktan sonra bütün döngüleri stream ile değiştirmek isteyebilirsiniz ancak bu doğru bir yaklaşım değildir. Her ne kadar bu mümkün olsa da büyük ihtimalle programın okunabilirliği ve bakım yapılabilirliği zorlaşacaktır. Hesaplama yapmanın kısmen karmaşık olduğu durumlarda bile Anagrams programında olduğu gibi streamler ile yinelemeyi beraber kullanmak daha doğru olacaktır. Bu yüzden streamleri sadece mantıklı olduğu durumlarda kullanın.

* Kod bloğu ile kapsam (*scope*) içindeki yerel değişkenleri okuyabilir ve değiştirebilirsiniz. Lambda ile bunları değiştirmeniz mümkün değildir. Okuyabilmek içinse final veya effectively final olması gerekir. (final anahtar kelimesi ile tanımlanmasa bile ilk değer atamasından sonra değeri değiştirilmeyen değişkenler effectively final olarak kabul edilir.)
* Kod bloğu kullandığınızda çevreleyen metottan çıkış yapabilir, döngülerin işletimini `break` veya `continue` ile değiştirebilir veya metodun tanımladığı bir aykırı durumu fırlatabilirsiniz. Lambda ile bunların hiç birisini yapamazsınız.
* Stream'lerin güçlü olduğu alanlar
  * Bir dizi elemanın homojen olarak dönüşüme tabi tutulması
  * Elemanların filtrelenmesi
  * Tek bir işlemle bir dizi elemanın toplanması, minimum değer hesaplanması, ard arda eklenmesi vs.
  * Bir dizi elemanın gruplanarak bir koleksiyona yazılması
  * Bir dizi eleman içerisinde belli kriterlere göre arama yapılması
* Stream kullanımını zorlaştıran etmenlerden birisi de bir ara işlem yapıldıktan sonra önceki değerlerin kaybolmasıdır. Örneğin ara işlem olarak bir değeri başka bir değere dönüştürürseniz ilk değer kaybolacaktır. Orijinal değerlere ihtiyacınız varsa eski ve yeni değerleri birbirine eşleyen ayrı bir veri yapısı tutabilirsiniz ama bu çok verimli olmayacaktır. Özellikle bunu birden fazla ara işlem için yapmanız gerekiyorsa ortaya karmaşık ve anlaması zor bir kod çıkacaktır. Bunun yerine stream elemanlarının önceki değerlerine erişmemiz gerektiği durumlarda yaptığımız ara işlemi tersinden uygulamak daha mantıklı olacaktır.

- Mersenne asal sayılarını yazdıran program : 2xp - 1

```java
  static Stream<BigInteger> primes(){
      return Stream.iterate(TEN, BigInteger::nextProbablePrime);
  }
```

Burada metot ismi olarak kullanılan primes tesadüfen seçilmiş bir metot ismi değildir. Bu gibi stream döndüren metotlarda çoğul metot isimleri kullanmak stream pipeline'larını okunabilirliğini artırmaktadır. Metot Stream.iterate factory metodunu kullanmakta ve iki parametre almaktadır. İlk parametre streamin ilk elemanını, ikinci parametre ise önceki elemanı kullanarak streamin sonraki elemanlarını hesaplamak içn gerekli fonksiyonu temsil etmektedir. İlk yirmi mersenne asal sayısını hesaplayan program aşağıdadır:

```java
primes()
        .map(p -> TWO.pow(p.intValueExact()).subtract(ONE))
        .filter(mersenne -> mersenne.isProbablePrime(50))
        .limit(20)
        .forEach(System.out::println);
```

Program asal sayıları döndüren primes metodunu çağırarak bir stream edinmekte ve map metodu ile bu asal sayıları Mersenne sayılarına dönüştürmektedir. Mersenne sayıları arasındaki Mersenne asal sayıları filter metodu sayesinde elde ediliyor ve yazdırılıyor. Yirmi tane bulunduğu zaman hesaplama durduruluyor.

- Şimdi varsayalım ki Mersenne asal sayılarını yazdırırken bunun hangi p değeri kullanılarak üretildiğini de yazdırmak istiyoruz. Bu değer sadece en başta elde ettiğimiz stream içerisinde mevcuttur, sonuçları yazdırdığımız sonlandırıcı işlem yapılırken bu değerlere erişim yoktur. Neyse ki bunu hesaplamak için map ile yapılan dönüşümü tersine çevirmek mümkündür. p değeri Mersenne sayısının ikili gösterimindeki bit sayısı ile aynıdır, dolayısıyla sonlandırıcı işlemi aşağıdaki gibi değiştirerek p değerlerini de yazdırabiliriz:
  `.forEach(mp -> System.out.println(mp.bitLength() + ": "+ mp));`
- Stream mi yoksa yineleme mi kullanmak gerektiğine net olarak yanıt bulamadığınız durumlar da olacaktır. Örneğin bir iskambil destesinin yaratılması işlemini düşünelim. Bunun için elimizde değiştirilemeyen (immutable) bir Card sınıfı ve kartların numarasını temsil eden Rank, takımları temsilen de Suit isimli iki enum olsun. Her Card nesnesi Rank ve Suit türlerinde birer değere sahip olacaktır. İskambil destesinin tamamını oluşturmak için bu iki enum türünde tanımlı değerlerin bütün kombinasyonlarını kullanmamız gerekecektir. Matematikte buna kartezyen çarpımı denilmektedir. Aşağıda bu işi yapmak için for-each döngüsü kullanan yinelemeli yaklaşımı görüyorsunuz:

```java
    private static List<Card> newDeck() {
  List<Card> result = new ArrayList<>();
  for (Suit suit : Suit.values()) {
    for (Rank rank : Rank.values()) {
      result.add(new Card(suit, rank));
    }
  }
  return result;
}
```

lambda tabanlı yaklaşım :

```java
    private static List<Card> newDeckStream() {
        return Stream.of(Suit.values())
                .flatMap(suit ->
                        Stream.of(Rank.values())
                                .map(rank -> new Card(suit, rank)))
                .collect(toList());
    }
```

- Burada tanımladığımız iki newDeck metodundan hangisini tercih etmeliyiz? Bu noktada artık kişisel tercihlere ve programlama yaptığımız ortama göre karar vermeliyiz. İlk versiyon daha basit ve doğal bir yaklaşım gibi görünüyor. Birçok programcı bunu anlamakta zorluk çekmeyecektir ama stream tabanlı versiyonu tercih edenler de olacaktır. Stream kullanmaya alışıksanız ve fonksiyonel programlamaya aşinaysanız bu yaklaşım size daha anlaşılır gelebilir. Eğer siz de stream versiyonunu daha anlaşılır buluyorsanız ve kodla ilgilenen diğer programcıların da zorluk çekmeyeceğini düşünüyorsanız o zaman stream kullanmalısınız.
- Özetle, bazı işleri stream bazı işleri de yineleme kullanarak yapmak daha doğrudur. Ancak bu ikisinin birlikte kullanılması gereken durumlar da sıkça karşımıza çıkar. Her ne kadar keskin kurallar olmasa da bu yazıda verilen örneklerden yola çıkarak bir kestirimde bulunabilirsiniz. Birçok durumda hangi yolu kullanmanız gerektiği açıkça ortada olacaktır ancak bazen de arada kalacaksınız. Eğer emin olamazsanız iki yöntemi de deneyip hangisinin daha mantıklı olduğunu gözlemleyebilirsiniz.

## Stream kullanırken yan etkisi olamyan fonksiyonları tercih etmek

- Stream'ler sadece yeni bir API değil, aynı zamanda bir fonksiyonel programlama yaklaşımıdır. Bu yaklaşıomın en önemli tarafı yapmak istenilen hesaplamayı aşamalı bir dönüşüm olarak ifade edebilmektir. Bu dönüşümler yapılırken mümkün olduğu kadar saf fonksiyonlar kullanılmalıdır. Saf fonksiyonlar sonuçlara sadece girdilere(input) bağlı olan ve nesne durumunda değişiklik yapmayan fonksiyonlardır. Bunun mümkün olması için de hem ara hem sonlandırıcı işlem fonksiyonlarının yan etkisiz olması gerekmektedir. Başka bir deyişle fonksiyonun üreteceği sonuç dış etmenlerden etkilenmemeli ve durum değişikliğine sebep olmamalıdır.
- **Java programcıları for döngülerini iyi bilirler ve forEach metodu da buna benzediği için ilk tercih bu olur. Ancak forEach metodu aslında sonlandırıcı stream işlemleri arasında en zayıfı ve fonksiyonel programlama yaklaşımına en aykırı olanıdır. Çünkü esasında yinelemeli kod yazmaya teşvik eder. Ayrıca paralel streamlerle birllikte çalışmaya da uygun değildir. Dolayısıyla forEach metodu sadece önceki hesaplamaları raporlamak için kullanılmalıdır, hesaplama yapmak için değil.**
- Kodda bulunan collect metodu bir toplayıcıyı temsil etmektedir. Bu streamleri doğru kullanmak için öğrenilmesi gereken çok önemli bir kavramdır. Toplayıcıları üreten Collectors API bir çok metoda sahiptir. Burada indirgemeden kastedilen çok sayıdaki stream elemanlarının tek bir nesneyle ifade edilmesidir. Toplayıcı tarafından üretilen bu nesneler genellikle bir collection'dur.
- Stream elemanlarını bir collection içinde toplayan üç tane toplayıcı vardır, toList(), toSet(), toCollection(collectionFactory). Bunlar da sırasıyla bir liste, lüme ve programcı tarafından belirlenen bir koleksiyon türü döndürürler. Bu bilgiyle 'kelimelerin sıklık tablosundan en sık kullanılan on kelimeyi bize döndüren stream' aşağıdaki gibi olacaktır:

```java
Map<String, Long> freq = new HashMap<>();
List<String> topTen = freq.keySet().stream()
        .sorted(Comparator.comparing(freq::get).reversed())
        .limit(10)
        .collect(toList());
```

Dikkat ederseniz toList statik bir metot olmasına rağmen yazarken sınıf adı kullanmadık. Stream hatlarını daha okunabilir hale getirmek için burada olduğu gibi Collectors metotlarının statik olarak import edilmesi mantıklı olacaktır.

- Bu kodda üzerinde konuşulması gereken kısım sorted metoduna geçilen comparing(freq::get).reversed() ifadesidir. Buradaki comparing metodu bir Comparator nesnesi üretmektedir (Madde 14). Argüman olarak geçilen freq::get metot referansı ise tabloda anahtarlara karşılık gelen kelimelerin kaç kere kullanıldığını, yani sıklığını döndürmektedir. Son olarak Comparator üzerinden çağrılan reversed metodu ise sıralamayı tersine çevirmektedir, çünkü biz sıralamayı en sık kullanılandan en aza doğru yapmak istiyoruz. Sonrasında bu hesaplamayı on elemanla kısıtlamak için limit metodu çağrılmakta ve sonuçlar bir listede toplanmaktadır.
- Özetle, stream pipeline'larını programlamanın özü yan etkisi olmayan fonksiyonları kullanmaktır. Sonlandırıcı işlem olan forEach sadece stream pipeline'ı tarafından yapılmış hesaplamayı raporlamak için kullanılmalıdır, hesaplamanın kendisi için değil. Stream'leri doğru kullanabilmek için toplayıcıları(collector) iyi bilmek gerekmektedir. Bunlardan en önemlileri toList, toSet, toMap, groupingBy, joining gibi collector'lerdir.

## Dönüş türü olarak stream yerine collection tercih etmek

- Bir dizi eleman döndüren metotlarla sıkça karşılaşırız, java 8'den önce bu tür metotlar için dönüş türü olarak ya Collection, Set, List gibi collection türleri ya Iterable ya da dizi türleri kullanılırdı. Çoğu zaman da bunların birini seçmek zor olmazdı. Metodun amacı dönüş değerinin for-each döngüsünde kullanılmasını mümkün kılmaksa veya döndürülen nesnelerin Collection arayüzündeki bazı metotları geçersiz kılması mümkün olmuyorsa, Iterable dönüş türü tercih edilirdi. Döndürülen elemanlar temel türlerde ise veya çok ciddi performans gereksinimleri varsa dizi kullanılırdı. Java 8'le streamler dile eklendikten sonra bu metotlarda dönüş türünü belirlemek zorlaştı.
- Bir dizi eleman döndüren metotlar için dönüş türü olarak stream kullanmanın en doğru seçenek olduğunu söyleyenler olabilir. Ancak iyi kod yazabilmek için stream ile yinelemeyi (iteration) beraber kullanmak gerekiyor. Bir API lsadece stream döndürecek şekilde tasarlanırsa bunu for-each döngüsünde kullanmak isteyen bir istemci hayal kırıklığına uğrayabilir. Stream türünün for-each döngülerinde kullanılmasını engelleyen tek şey Stream arayüzünün iterator metoduyla bu işlevselliği sağlamasına rağmen Iterable arayüzünü kalıtmamasıdır.
```java
  // Stream'i iterable'a döndüren metot
  public static <E> Iterable<E> iterableOf(Stream<E> stream){
      return stream::iterator;
  }

  // Iterable<E> yi Stream<E>ye döndüren metot
  public static <E> Stream<E> streamOf(Iterable<E> iterable){
      return StreamSupport.stream(iterable.spliterator(), false);
  }
```
Eğer döndürdüğünüz elemanların bir stream kullanarak işleneceğinden eminseniz tabii ki stream döndürmekte bir sıkıntı yoktur. Aynı şekilde, for-each döngüsünde kullanılacağını bildiğiniz elemanları da Iterable türünde döndürmekte sakınca yoktur. Ancak dışarıya açık (public) bir API yazıyorsanız ve döndürülen elemanların ne şekilde işleneceğinden emin değilseniz, bu iki kullanımı da desteklemek yararlı olacaktır.

- Collection arayüzü Iterable arayüzünü kalıtır ve stream metodu vardır, dolayısıyla hem yinelemeli hem de stream kullanımını destekler. Bu yüzden bir grup eleman döndüren dışa açık metotlar için dönüş türünün Collection veya bunun bir alt türü olması en doğrusudur. Diziler de yineleme ve stream kullanımlarını Arrays.asList ve Stream.of metotları sayesinde desteklerler. Döndürdüğünüz elemanlar belleğe kolayca sığacak kadar küçükse ArrayList veya HashSet gibi standart bir koleksiyon döndürmek mantıklı olacaktır. Ancak sırf koleksiyon türü döndürebilmek için büyük miktarlardaki veriyi belleğe sıkıştırmaya çalışmayın. 
- Özetle, bir grup eleman döndüren metotlar yazarken bazı kullanıcıların bunları stream kullanarak, bazılarının da tarama yaparak yinelemeli biçimde işlemek isteyebileceğini unutmayın. Metodun nasıl kullanılacağından emin değilseniz bu iki kullanımı da desteklemeye çalışın. Bunun için bir koleksiyon döndürebiliyorsanız döndürün. Elemanlar zaten bir koleksiyon içindeyse veya sayıları yeterince azsa ArrayList gibi standart bir koleksiyon döndürün, değilse kuvvet kümesi örneğinde yaptığımız gibi kendiniz bir koleksiyon türü tanımlayın. Collection döndürmek pek makul değilse Stream veye Iterable döndürebilirsiniz. Eğer sonraki Java versiyonlarında Stream arayüzü Iterable arayüzünü kalıtırsa stream döndürmek bir problem yaratmayacaktır çünkü bu durumda hem stream hem de yinelemeli işletimi desteklemiş olacaktır. 

## Stream'leri paralel yaparken dikkatli olmak

-  Java’da paralel işletilen programlar yazmak giderek kolaylaşıyor gibi görünse de, bunu doğru ve yüksek performans alarak yapmak hiç de kolay değildir. Thread güvenliği ve canlılık (liveness) ihlalleri paralel programlamanın doğasında olan sorunlardır ve paralel streamler de bunun bir istisnası değildir. 
- Stream pipeline'larını kafaya göre paralelleştirmek doğru olmayacaktır. Bazı işlemleri paralel nasıl yapılacağı hakkında programın bilgisi olmayabilir.
- Paralel streamlerin yüksek performansla çalışabilmesi için bunların Arraylsit, HashMap, HashSet, ConcurrentHashMapğ nesneleri; dizileri int aralıkları (IntStream.range) veya long aralıkları üzerinden yaratılması gerekir. Bütün bu veri yapılarının ortak noktası istenen küçüklükte parçalara kolayca bölünebilmeleridir. Bu da paralel işletimi kolaylaştıran bir durumdur. Stream kütüphanesi bunu yapmak için Stream ve Iterator'de bulunan splititerator metodunu kullanır.
- Bu veri yapılarının ikinci önemli özelliği ise sıralı bir şekilde işlendikleri zaman referans yerelliği (locality of reference) sunmalarıdır. Başka bir deyişle ard arda gelen elemanların referansları bellekte beraber tutulmaktadır. Ancak referansların beraber olması bunların işaret ettiği nesnelerin de bellekte birbirlerine yakın olacağı anlamına gelmez, bu yerelliği azaltan bir faktördür. Referans yerelliği toplu yapılan işlemlerin paralel işlenmesinde çok önemli bir faktördür. Bu olmadığında threadler verinin bellekten işlemciye aktarılması için beklemek zorunda kalırlar. Referans yerelliğini en iyi sağlayan veri yapıları ise temel türlerdeki dizilerdir, çünkü bunlar verinin kendisini bellekte peş peşe saklarlar.
- Bir stream hattının sonlandırıcı işlemi de paralel işletimin verimini etkiler. Hesaplamanın zaman alan kısmı ara işlemler değil de sonlandırıcı işlemde yapılıyorsa ve bu işlemin doğası gereği peş peşe yapılması gerekiyorsa paralel işletim pek verimli olmaz. Paralel işletime en uygun sonlandırıcı işlemler indirgeme (reduction) işlemleridir. İndirgeme işlemleri bütün stream elemanlarının reduce, min, max, count veya sum gibi metotlar kullanılarak birleştirilmesi sonucu tek sonuç üretirler. anyMatch, allMatch veya noneMatch gibi sonlandırıcı işlemler de paralel işletimde verimlidirler. Ancak Stream.collect metoduyla kullanılan toplayıcı işlemler pek verimli olmaz çünkü stream elemanlarının bir koleksiyonda toplanmasının getirdiği ek yük fazladır.
- Stream hesaplamalarının paralelleştirilmesi kötü performansa sebep olabileceği gibi, programın yanlış sonuçlar üretmesine ve tutarsız davranmasına da sebep olabilir. Bu tür hataların kaynağı stream hatlarında kullanılan fonksiyon nesnelerinin Stream kütüphanesinin tanımladığı bağlayıcı kurallara uymamasıdır. Örneğin, reduce metoduna geçilen toplayıcı ve birleştirici fonksiyonların belirli matematiksel kuralları sağlaması ve durum taşımaması (stateless) gerekir. Bu kurallar ihlal edildiğinde (Madde 46) stream hattı sıralı işletimde düzgün çalışsa bile paralel işletimde büyük ihtimalle çökecektir.
- Streamlerin verimli bir biçimde paralel işletilmesi için burada anlatılan bütün kurallara uysanız bile (doğru veri yapısı ve sonlandırıcı işlem seçimi, fonksiyon nesnelerinin paralel işletime uygun olması gibi) paralel işletimden beklediğiniz performans artışını almanız zordur. Bunun sebebi paralel işletimin kendisinin de bir ek yük getirmesidir. Eğer paralel hesaplamadan elde edilen kazanç, paralel işletimin getirdiği ek yükü fazlasıyla karşılayabiliyorsa o zaman bir performans kazanımı mümkün olur.
- Bir stream hattını paralel yaparken amacın performans iyileştirmesi olduğunu unutmayın. Bu yüzden de iyileştirme yaparken öncesi ve sonrasındaki performans değerlerini ve üretilen sonuçları karşılaştırın (Madde 67). Bu testlerin gerçekçi bir sistem üzerinde yapılması önemlidir. Bütün paralel stream hatları tek bir fork-join havuzunu kullandıklarından bir tanesinde oluşabilecek hata başka stream hatlarında problemlere sebep olabilir.
```java
  // paralele stream'lerin faydalı oldğu bir hesaplama
  static long primeCount(long n){
      return LongStream.rangeClosed(2, n)
              .parallel()
              .mapToObj(BigInteger::valueOf)
              .filter(i -> i.isProbablePrime(50))
              .count();
  }
```

- Eğer rastgele üretilmiş sayılardan oluşan bir stream üzeirnde paralel hesaplama yapmak istiyorsanız ThreadLocalRandom yerine SplittableRandom kullanıın. Bu tam olarak bu amaçla yazılmıştır ve paralel işletime çok uygundur. ThreadLocalRandom ise tek bir thread ile çalışmaya müsaittir. Paralel işletimde de çalışacaktır ancak SplittableRandom kadar hız artışı sağlamayacaktır.
- Özetle, bir stream hattının doğru sonuçları üreteceğinden ve hız artışı sağlayacağından emin değilseniz paralel yapmaya kalkışmayın. Yanlış durumda yapılan paralel işletimin programın çökmesi veya performansın yerlerde sürünmesi gibi etkileri olabileceğini unutmayın. Eğer bir stream hattını paralel yaparak kazanç sağlayabileceğinizi düşünüyorsanız, gerçekçi bir ortamda mutlaka performansı ve üretilen sonuçları test edin. Sadece ve sadece bu testleri geçtiği taktirde bir stream hattını paralel yapmak yararınıza olacaktır.




