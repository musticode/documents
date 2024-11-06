# Methods


## Parametrelerin geçerliliğini kontrol etmek

- Çoğu metot ve yapıcı metot parametre geçilen değerler için kısıtlamalar koyar. Örneğin dizin (index) değerlerinin pozitif tamsayı olması ve nesne referanslarının null olmaması gibi kısıtlamalarla sıklıkla karşılaşırız. Bu kısıtlamaları metot gövdesinin başında uygulamalı ve açıkça belgelemelisiniz. Bunu yaptığımız taktirde hataların olabildiğince erken saptanması mümkün olabilir, aksi taktirde bunu geciktirmiş oluruz ve bir hata ile karşılaştığımızda nereden kaynaklandığını bulmak zorlaşır.
- Bir metoda geçersiz bir parametre verildiğinde, eğer metot kod işletimine başlamadan önce parametrelerin geçerliliğini kontrol ediyorsa erkenden hata üretecek ve mantıklı bir aykırı durum fırlatarak işletimi durduracaktır. Bunu yapmadığı taktirde birkaç farklı durumla karşılaşabiliriz. Birincisi uygulamanız işletimin farklı bir yerinde kafa karıştırıcı bir aykırı durum fırlatabilir. Daha kötüsü, metot işletimini tamamlar ancak yanlış bir sonuç üretebilir. En kötüsü ise bir nesnenin durumunu bozarak ileride tamamen alakasız bir kod parçası işletilirken hataya yol açabilir. 
- public ve protected erişim belirtecine sahip metotlar için Javadoc’un @throws etiketini kullanarak parametreler geçersiz olduğunda fırlattığınız aykırı durumu belgeleyebilirsiniz. (Madde 74) Genellikle bu aykırı durumlar IllegalArgumentException, IndexOutOfBoundsException, veya NullPointerException olacaktır.

```java
    /**
     * @param m (modulus) pozitif olmalı
     * @return this mod m
     * @throws ArithmeticException m sıfıra eşitse ya da küçükse
     * */
    public BigInteger mod(BigInteger m){
        if (m.signum() <= 0){
            throw new ArithmeticException("Modulus <= 0" + m);
        }

        // hesaplamalar...
        return BigInteger.ONE;
    }
```
Bu aykırı durum metodun içinde bulunduğu BigInteger sınıfının kendi dokümantasyonunda belgelenmiştir, bu sebeple metot için tekrar yazmaya gerek yoktur. Bu yöntemi kullanarak sınıf içindeki her bir metot için ayrı ayrı NullPointerException belgelemekten kaçınabilirsiniz.

- Bazı durumlarda ise bir hesaplama yapmak için kullanacağımız parametrelerin geçerliliğini önceden kontrol etmek çok da mantıklı olmayabilir. Buna örnek olarak, kendisine verilen nesneleri sıralayan Collections.sort(List) metodunu verebiliriz. Sıralamanın yapılabilmesi için verilen listedeki nesnelerin birbiriyle karşılaştırılabilir olması gerekmektedir. Eğer sıralama esnasında buna aykırı bir durum bulunursa ClassCastException hatası üretilecektir ve aslında sort metodunun yapması gereken de budur. Dolayısıyla, listedeki elemanları birbirleriyle karşılaştırılabilir olup olmadığını önceden kontrol etmek bize çok bir fayda sağlamayacaktır.
- Bu maddeden metot parametrelerine zorunlu olmayan kısıtlamalar koymanın iyi bir şey olduğu sonucunu çıkartmayın. Tam tersine, metotlarınız geçilen parametre değerleriyle mantıklı bir işlem yapabildiği sürece kısıtlamalardan kaçınmaya çalışın ki daha geniş bir kullanım alanı bulabilsin
- Özetle, bir metot veya yapıcı metot yazarken parametreler üzerinde ne gibi kısıtlamalar olması gerektiğini iyice düşünün. Bu kısıtlamaları belgeleyin ve metodun hemen başında gerekli denetimleri uygulayın. Bunu bir alışkanlık haline getirmek önemlidir. Sarf edeceğiniz bu küçük çabanın karşılığını geçerlilik denetimleri hata bulduğu zaman fazlasıyla alacaksınız. 

## Gerektiğinde koruyucu kopyalar yaratmak

- Güvenli bir dil kullanırken, programın başka yerlerinde ne olursa olsun yazdığınız bir sınıftan yaratılan nesnelerin durumlarının sizin koyduğunuz kurallar çerçevesinde şekillenmesini sağlayabilirsiniz. Bunu yapmak bütün belleği kocaman bir diziymiş gibi ele alan dillerde mümkün değildir.
- Güvenli dillerde bile, diğer sınıfların etkilerinden korunabilmek için biraz çaba sarfetmeniz gerekir. Bu sebeple, sınıfınızın bütün istemcilerinin kötü niyetli bir biçimde nesneyi olmaması gereken durumlara sokmaya çalışacakları varsayımı ile defansif kodlama yapmak gerekmektedir. Sistemlerin güvenliğini kırmaya çalışan kişilerin sürekli arttığını düşünürsek bu daha da önemli hale gelmektedir. Ancak bundan daha yaygın olanı iyi niyetli programcıların yanlışlıkla yapabileceği hatalar sonucu programda yol açabileceği sorunlardır. Her ne olursa olsun, sınıf yazarken bu olasılıkları göz önünde bulundurmalı, hatalara ve saldırılara karşı dirençli kodlar yazmalıyız.

```java
class Period{
    private final Date start;
    private final Date end;

    Period(Date start, Date end) {
        this.start = new Date(start.getTime());
        this.end = new Date(end.getTime());

        if (this.start.compareTo(this.end) > 0) {
            throw new IllegalArgumentException(this.start + " after " + this.end);
        }
    }

    // problemli
//    Period(Date start, Date end) {
//        if (start.compareTo(end) > 0){
//            throw new IllegalArgumentException(start + " after "+ end);
//        }
//        this.start = start;
//        this.end = end;
//    }

    public Date getStart() {
        return start;
    }

    public Date getEnd() {
        return end;
    }
}
```
Düzeltilmiş yapıcı metodu kullandığımızda, bir önceki istemci kodu Period nesnesi üzerinde bir değişikliğe sebep olmayacaktır. Dikkat ederseniz parametrelerin geçerliliğini kopyaları yarattıktan sonra test ettik (Madde 49). Bu garip görünse de aslında gereklidir. Aksi durumda parametreler test edildikten hemen sonra ve kopya oluşturmadan hemen önceki zaman diliminde, başka bir thread bu parametreleri keyfi bir biçimde değiştirebilirdi. Bilgisayar güvenliği alanında bu saldırı denetim-zamanı/kullanım-zamanı (time-of-check/time-of-use) saldırısı olarak bilinir.

```java
// Düzeltilmiş erişim metotları - koruyucu kopya döndürüyoruz
public Date start() {
    return new Date(start.getTime());
}
public Date end() {
    return new Date(end.getTime());
}
```

Düzelttiğimiz yapıcı metot ve erişim metotları sayesinde artık Period sınıfının nesneleri değiştirilemez hale gelmiştir. Böylece istemci kötü niyetli de olsa, hata da yapsa bütün Period nesnelerinde start değerinin end değerinden büyük olamayacağını garanti etmiş olduk. (reflection veya native metotlar kullanılarak bu yine de mümkün olabilir) Çünkü Period nesnelerinin değiştirilebilir alanlarına istemcilerin erişimi engellenmiş, bu alanlar sınıf içerisinde kapsüllenmiştir (encapsulation). 

- Koruyucu kopyalar sadece değiştirilemez sınıflar için değildir. İstemciden alınan bir nesne referansını sakladığınız her durumda, bu nesnenin değiştirilebilir olup olmadığını kontrol edin. Eğer öyle ise, siz nesne referansını sakladıktan sonra nesnede yapılacak değişiklikleri sınıfınızın tolere edip edemeyeceğini düşünün. Eğer bu durum sizin sınıfınızda problem yaratacaksa, orijinal nesne yerine bir kopya oluşturup onu kullanmalısınız. Örneğin, istemci tarafından geçilen bir nesne referansını içeride bir Set nesnesine ekleyecekseniz veya bir Map için anahtar olarak kullanacaksanız, bu nesnenin sınıfınızın dışında değiştirilmesi sorunlara yol açabilir.
- Bu durum nesne alanlarını istemcilere geri döndürürken de geçerlidir. Sınıfınızın değiştirilemez (immutable) olması gerekmese bile, değiştirilebilir bir nesneyi istemciye döndürürken iki kez düşünün. Unutmayın ki, diziler (arrays) de değiştirebilir yapılardır, dolayısıyla bunları istemciye döndürürken kopya oluşturmalısınız veya Madde 15’de anlatıldığı gibi dizinin değiştirilemez bir görüntüsünü (immutable view) de döndürebilirsiniz. 
- Burada çıkartılması gereken ders aslında şudur: mümkün olduğu sürece nesnelerinizin bileşenlerini değiştirilemez nesnelerden seçmeye çalışın. Böylece koruyucu kopyalar yaratmaktan kurtulabilirsiniz. (Madde 17) Bizim Period sınıfı örneğinizde, Java 8 veya üzeri kullanıyorsanız Date yerine Instant nesneleri kullanabilirsiniz. Önceki versiyonları kullanıyorsanız da Date.getTime() metodunun döndürdüğü long değerini Date referansı yerine kullanabilirsiniz.
- Koruyucu kopyalar yaratmanın bir performans kaybına sebep olabileceğini de söyleyelim. Eğer bir sınıf istemcilerinin nesne üzerinde keyfi değişiklikler yapmayacağından eminse (mesela sınıf ve istemcisi aynı paketin içindeyse), koruyucu kopyalardan vazgeçilebilir. Bu gibi durumlarda sınıfın gerekli belgelemeyi yapması ve istemcilerini uyarması doğru olacaktır.
- Bu şekilde istemciden gelen nesneyi kendi kontrolüne alan sınıflar kötü niyetli istemcilere karşı kendilerini savunamazlar. Bu sebeple ancak istemci ve sınıf arasında bir güven söz konusu ise veya nesnede oluşacak değişiklikler sadece istemcide hasara yol açacaksa kullanılmalıdırlar.
- Özetle, eğer bir sınıf istemciden gelen veya döndürülen değiştilebilir bileşenler içeriyorsa, bu bileşenlerin koruyucu kopyaları yaratılmalıdır. Bu eğer ciddi performans kaybına yol açıyorsa veya sınıf istemcilerinin uyugunsuz değişiklikler yapmayacağından eminse, koruyucu kopyalar yerine gerekli belgeleme yapılarak istemciler uyarılmalıdır.

## Metot imzalarını dikkatli tasarlamak

- Metot isimlerini dikkatli seçin. İsimler her zaman için standart isimlendirme modellerini takip etmelidir. (Madde 68) Ana hedefiniz anlaşılabilir ve aynı pakette kullanılan diğer isimlerle tutarlı olan isimler seçmek olmalıdır. Uzun metot isimlerinden kaçının. Şüpheye düştüğünüz zaman Java kütüphanesinde yer alan API’lara bakın. Burada her ne kadar bazı tutarsızlıklar olsa da bu kütüphanelerin büyüklüğü ve kapsamı düşünüldüğünde çoğunlukla üzerinde fikir birliği kurulmuş isimlendirmeler olduğunu söyleyebiliriz.
- Yardımcı metotlar yazarken aşırıya kaçmayın. Çok fazla sayıda yardımcı metot yazmak sınıfın bakımını, anlaşılmasını ve belgelenmesini zorlaştırır. Bu durum arayüzler için de geçerlidir. Geçerli kılınması gereken çok sayıda metot programcıların işini zorlaştırır. Sadece sıklıkla kullanılacağını düşündüğünüz yardımcı metotları ekleyin, eğer emin değilseniz eklemeyin.
- Çok sayıda parametreden kaçının. Dört veya daha az sayıda parametre ile yetinmeye çalışın. Birçok programcı bundan daha fazlasıyla çalışırken hata yapabilir. Bu şekilde çok sayıda metodunuz varsa istemcileriniz sürekli olarak dokümantasyona başvurmak zorunda kalabilir.
- Parametre sayısını azaltmak için üç yöntem vardır. Bunlardan bir tanesi metodu parçalamaktır. Böylece her biri az sayıda parametreye sahip birden fazla metot olacaktır. Ancak bunu da çok abartırsanız metot sayısı çok artabilir, bu yüzden dikkatli olunmalıdır. List arayüzü bu anlamda akıllıca tasarlanan örneklerden biridir.
- Parametre sayısını azaltmak için kullanılabilecek ikinci yöntem ise yardımcı sınıflar oluşturarak bir grup parametrenin beraber tutulmasını sağlamaktır. Bu yardımcı sınıflar genelde statik üye sınıf olarak tanımlanırlar. (Madde 24) Eğer belli bir grup parametre sıklıkla tekrar ediyorsa ve bunlar mantıksal olarak bir bütünü ifade edebiliyorlarsa bu yöntemi kullanabiliriz.
- Üçüncü yöntem ise Madde 2’de anlattığımız Builder tasarım deseni. Eğer çok sayıda parametre isteyen bir metodunuz varsa ve özellikle de bu parametrelerin bazıları isteğe bağlı ise bütün bu parametreleri tek tek geçmek yerine bunları temsil eden bir sınıf yaratılabilir ve istemci Builder tasarım desenini kullanarak bir nesne yaratıp metoda tek bir parametre geçebilir. 
- Parametre türleri için sınıflar yerine arayüz türlerini tercih edin. (Madde 64) Eğer sınıf türü yerine kullanabileceğiniz ve sınıfın uyguladığı bir arayüz var ise onu kullanın. Mesela, girdi olarak HashMap türünde parametre alan bir metot yazmanın bir gereği yoktur, bunun yerine Map arayüzünü kullanın. Böylece istemciler sadeceHashMap değil, TreeMap, ConcurrentHashMap veya başka bir Map gerçekleştirimini de gönderebilirler. Arayüz yerine sınıf türü kullanmak istemcileri kısıtlar ve esnekliği azaltır.

`public enum TemperatureScale { FAHRENHEIT, CELSIUS }`
- İstemciler için nesne yaratırken Thermometer.newInstance(TemperatureScale.CELSIUS) kullanmak Thermometer.newInstance(true) ifadesine göre çok daha anlaşılır olacaktır. Dahası, ileride TemperatureScale enum türüne KELVIN türünü eklemek çok kolaydır. Ayrıca enum türleri metot tanımlamaya izin verdiği için ileride eklenebilecek özellikler için büyük esneklik sağlar. (Madde 34)


## Overloading'i dikkatli kullanmak

```java
// Bozuk! Bu programın ne sonuç üreteceğini biliyor musunuz?
class CollectionClassifier {
    public static String classify(Set<?> s) {
        return "Set";
    }

    public static String classify(List<?> lst) {
        return "List";
    }

    public static String classify(Collection<?> c) {
        return "Unknown Collection";
    }
}
```

- **Bu programın davranışı biraz kafa karıştırıcı çünkü aşırı yüklenen metotlar arasında yapılan seçim statik, geçersiz kılınan (override) metotlar arasındaki seçim ise dinamik olarak yapılıyor. Geçersiz kılınan metotlar söz konusu olduğunda, çalışma zamanındaki tür bilgisine göre hareket edilerek hangi metodun çalıştırılacağı belirleniyor. Hatırlatmak gerekirse, bir metodun geçersiz kılınması üst türde var olan bir metodun aynı metot imzası kullanılarak alt bir türde yeniden tanımlanmasıdır. Böyle bir durumda eğer bir metot alt türden yaratılmış bir nesne üzerinden çağırılıyorsa, derleme anındaki tür bilgisi üst türe işaret etse de alt türde tanımlanmış olan metot çağrılacaktır.** 
- Az önceki metodu düzeltmek için instanceof ile tür kontrolü yapmak gerekiyor. Yoksa çalışma zamanında hangi metodun seçileceğine karar verileceği için problem olacaktır.

```java
public static String classify(Collection<?> c) {
    return c instanceof Set  ? "Set" :
            c instanceof List ? "List" : "Unknown Collection";
}
```

- Geçersiz kılınan metotların nasıl davrandığına aşina olan programcılar aşırı yüklemenin de benzer biçimde çalışacağını düşünebilirler. Ancak örneğimizden anlaşıldığı üzere bu beklenti gerçeği yansıtmamaktadır. Programcıların kafasını karıştıracak kodlar yazmak pek tavsiye edilmez, özellikle de API yazıyorsanız daha dikkatli olmanız gerekir. Eğer API kullanıcısı aşırı yüklenmiş birkaç metottan hangisinin çalıştırılacağını bilemiyorsa, büyük ihtimalle hatalı kod yazacaktır. Bu hatalar derleme anında değil çalışma anında ortaya çıkacak, hatanın bulunmasını zorlaştıracaktır. Bu sebeple aşırı yüklemenin kafa karışıklığına sebep olabilecek kullanımlarından kaçınmalıyız.
- Aşırı yüklemenin kafa karıştırıcı kullanımlarının hangileri olduğu tartışmaya açık bir konudur. Temkinli sayılabilecek yaklaşımlar, aşırı yüklenen metotların parametre sayısının birbirinden farklı olması gerektiğini savunur. Eğer metotlar değişken sayılı parametre (varargs) kullanıyorsa hiç aşırı yükleme yapmamak daha garanti bir yoldur. (Madde 53 buna bir istisnadır) Bu tavsiyelere uyulursa programcılar hiçbir zaman hangi aşırı yüklenmiş metodun çağrılacağı konusunda bir tereddüt yaşamazlar. Ayrıca, bu tavsiyelere uymak çok zahmetli bir iş de değildir, en nihayetinde metotları aşırı yüklemek yerine farklı isimler kullanmak mümkündür.
- Yapıcı metotlar için farklı isimler kullanmak gibi bir şansımız olmadığı için, birden fazla yapıcı metot tanımlamak istiyorsak aşırı yükleme yapmak zorundayız. Alternatif olarak yapıcı metotlar yerine statik fabrika metotları yazabiliriz. (Madde 1) Ancak aynı parametre sayısına sahip birden fazla yapıcı metot tanımlamamız gereken durumlar da karşımıza çıkacaktır. Bu yüzden bunu nasıl güvenli bir biçimde yapabileceğimizi öğrenmek önemlidir.
- Hatalı kod ve doğru kod
```java
new Thread(System.out::println).start();

ExecutorService executorService = Executors.newCachedThreadPool();
executorService.submit(System.out::println);
```

Burada ikinci submit metodu derlenme hatası verecektir. Bu da submit metodu için Callable T olacak şekilde bir metodun daha overload edilmesinden kaynaklıdır.
Bu konunun detayları şu linkte bulunuyor : [Oracle](https://docs.oracle.com/javase/specs/jls/se8/html/jls-15.html#jls-15.13.1)

- Önemli olan nokta şu: yapıcı metotları veya sınıf metotlarını aşırı yüklerken aynı parametre pozisyonuna farklı fonksiyonel arayüzler koyarsanız derleyicinin kafası karışabilir. Farklı fonksiyonel arayüzler derleyici tarafından tamamen farklı türler olarak algılanmazlar. Bu şekilde problemli bir aşırı yükleme yaptığınızda -Xlint:overloads kullanıldığında derleyici sizi uyacaktır.
- Özetlemek gerekirse, metotları aşırı yükleyebiliyor olmamız bunu gerçekten yapmamız gerektiği anlamına gelmez. Metot imzalarında aynı sayıda parametre olarak şekilde aşırı yükleme yapmaktan kaçınmalıyız. Bazı durumlarda, özellikle de yapıcı metotlarla çalışırken bu mümkün olmayabilir. Bu durumlarda en azından aşırı yüklenmiş farklı metotlarda birbirilerine dönüştürülebilen türler kullanmamalıyız. Eğer bunu da yapamıyorsak, karışıklığa yol açabilecek aşırı yüklenmiş metotları aynı davranışı gösterecek şekilde yazmalıyız. Bunlara uymadığımız taktirde programcılar bu aşırı yüklenmiş metotları kullanırken sorun yaşayacak ve neden istedikleri gibi çalışmadığı anlamakta zorlanacaklardır. 

## Değişken sayılı argümanları (varargs) kullanırken dikkat etmek

- Varargs java dilinde belirli bir türden sıfır veya daha fazla sayıda argümanı bir metoda tek seferde geçmemizi sağlayan bir mekanizmadır. Variable arity, yani değişken sayılı argümanlar ifadesinin kısaltmasıdır. Arka planda bu mekanizma değişken sayıdaki argümanların toplanıp tek bir dizi içerisinde saklanması ve bu dizinin metoda geçilmesi mantığıyla çalışır.
```java
static int sum(int... args){
    int sum = 0;
    for (int arg : args){
        sum += arg;
    }
    return sum;
}
```

- Performans hassasiyetinin yüksek olduğu durumlarda varargs metotları kullanırken daha dikkatli olun. Bir varargs metot her çağrıldığında, arka planda bir dizi oluşturulup içi doldurulur. Deneysel olarak bunu test edip performans kaybını kabul edilemez bulduğunuz taktirde, varargs metot yerine kullanabileceğiniz bir başka yöntem vardır. Diyelim ki metot çağrılarının yüzde 95’inin üç veya daha az argümanla yapıldığını tespit ettiniz. Bu durumda metodun aşırı yüklenmiş beş versiyonunu aşağıdaki gibi tanımlayabilirsiniz:
```java
public void foo() { }
public void foo(int a1) { }
public void foo(int a1, int a2) { }
public void foo(int a1, int a2, int a3) { }
public void foo(int a1, int a2, int a3, int... rest) { }
```
Artık biliyorsunuz ki metot çağrılarının sadece yüzde 5’i varargs kullanırken ortaya çıkabilecek performans kayıplarından etkilenebilir. Geri kalan yüzde 95 varargs metodu değil, aşırı yüklenmiş ve varargs kullanmayan diğer metotlardan birini çağıracaktır. Bu her ne kadar göze hoş görünmese de, performansın kritik olduğu uygulamalarda hayat kurtarabilir.


-  Özetle, varargs metotlar değişken sayılı argüman geçilebilmesini istediğiniz metotlar için çok faydalıdır. Eğer geçilmesini istediğiniz minimum sayıda argüman varsa, bunları varargs parametrenin önüne aynı tipte normal parametreler olarak ekleyin. Varargs metotların kritik durumlarda performans kaybına yol açabileceğini de unutmayın.

## Null yerine boş koleksiyon veya dizi döndürmek

```java
// Boş koleksiyonu simgelemek için null döndürüyor, yanlış!
private final List<Cheese> cheesesInStock = ...;
 
/**
* @return markette bulunan bütün peynirleri içeren bir liste döndürür
* eğer peynir kalmadıysa null döndürür.
*/
public List<Cheese> getCheeses() {
    return cheesesInStock.isEmpty() ? null : 
           new ArrayList<(cheesesInStock);
}
```
Burada eğer peynir listesi boşsa özel durum olarak ele alınmıştır, null olarak istemciye döndürülecektir. Bu son derece yanlıştır. Eğer null dönersek istemci de null kontrolü yapmak zorunda kalacaktır.

- **Boş bir koleksiyon veya dizi yerine null döndüren bütün metotlar bu soruna yol açar. Hata yapmaya çok elverişlidir çünkü istemci null kontrolü yapmayı unutabilir. Çağrılan metot çoğu durumda bir veya daha fazla eleman döndürüyorsa böyle hatalar yıllar boyunca farkedilmeyebilir. Ayrıca, bir metotta boş koleksiyon yerine null döndürmek daha zahmetli bir iştir.**
- Bazen boş koleksiyon döndürmenin null döndürmeye kıyasla performans kaybına sebebiyet vereceği düşünülebilir çünkü boş da olsa bir koleksiyon yaratmanın bir maliyeti vardır. Bu yaklaşımın iki problemi vardır. Birincisi, eğer performans kaybını ölçümlerle destekleyemiyorsanız bu seviyelerde endişelenmenize gerek yoktur. (Madde 67) İkincisi de bellekte ek yer işgal etmeden boş koleksiyonlar döndürmek mümkündür. Çoğu durumda yapmanız gereken tek şey budur:
```java
// Boş koleksiyon döndürmenin doğru yolu
public List<Cheese> getCheeses() {
    return new ArrayList<>(cheesesInStock);
}
```

- Eğer boş koleksiyon üretmenin maliyeti gerçekten uygulamamızın performansını tetkiliyorsa, değiştirilemez(immutable) bir boş koleksiyonu tekrar tekrar kullanabilirsiniz. Çünkü değiştirilemez nesneler serbest bir biçimde paylaşılabilir. Aşağıdaki kod Collections.emptyList metodunu kullanarak bunu yapmaktadır. Küme(set) döndürüyor olsaydık Collections.emptySet, map içinse Collections.emptyMap kullanılabilir.
```java
public List<String> getCheeses(){
    List<String> cheesesInStock = new ArrayList<>();
    return cheesesInStock.isEmpty() ? Collections.emptyList() : new ArrayList<>(cheesesInStock);
}
```

- Dizilerdeki durum da collections ile aynı olacaktır. Eğer bir dizinin elemanı yoksa uzunluk değeri 0 olmalıdır. Bu son derece geçerli bir kullanımdır.
```java
    public String[] getCheeseArr(){
        List<String> cheesesInStock = new ArrayList<>();
        return cheesesInStock.toArray(new String[0]);
    }
```

- Eğer performans etkileniyorsa bir tane static dizi yaratılıp her zaman bu dizi döndürülebilir.

```java
// İyileştirme - boş dizi yaratılmasını engeller
private static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];
public Cheese[] getCheeses() {
    return cheesesInStock.toArray(EMPTY_CHEESE_ARRAY);
}
```

- Özetle, boş koleksiyon veya dizi yerine asla null döndürmeyin. Bu API’ınızın kullanımını hem zorlaştırır hem de hata ihtimalini artırır. Performans katkısı da sağlamaz.


## Optional döndürürken dikkatli olmak

- Java 8'den önce belli durumlarda değer döndüremeyen bir metot yazabilmek için iki farklı yöntem vardı. Birincisi exception fırlatmak, ikincisi ise metodun dönşü türü nesne referansı ise null döndürmekti. Bu yaklaşımlar mükemmel değildir. Exception'lar istisnai durumlar için kullanılmalıdır. Aynı zamanda da yaratılması masraflıdır. Null dönmenin de başka problemi vardır, istemciler her metodu çağırdıklarında null denetimi yapmak zorunda kalacaklardır. Bu unutulduğunda ise çok alakasız şekilde nullpointer exception hatası ortaya çıkacaktır.
- Java 8'de bu tarz metotları yazabilmek için üçüncü bir yol eklenmiştir. Optional<T> sınıfı ya hiçbir şey ya da null olmayan bir T referansı tutabilen değiştirilemez bir container sınıf olarak tanımlanmıştır. Başka bir deyişle bu taşıyıcı sınıf nesneleri boş olabililr veya var olan bir T nesnesine işaret eden bir referans tutabililr. Dolayısıyla en fazla bir elemana sahip olabilirler ve immutable taşıyıcılardır.
- Bir metot eğer T türünde değerler döndürüyorsa ancak hiçbir değer döndüremediği durumlar da varsa dönüş türü olarak Optional<T> kullanılabilir. Böylece metodun değer döndüremediği durumlar için içi boş bir Optional<T> döndürmesi mümkün olur. Optional döndüren metotlar exception fırlatan metotlara göre daha esnektirler ve daha kolay kullanılırlar, null döndüren metotlara göre de hata yapılma ihtimalini düşürürler.
```java
// koleksiyondaki maximum değeri bulur
// eğer eleman yoksa exception fırlatır
public static <E extends Comparable<E>> E max(Collection<E> c){
    if (c.isEmpty()){
        throw new IllegalArgumentException("Empty collection ");
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
Eğer metoda geçilen koleksiyon boş ise, bu metot IllegalArgumentException fırlatmaktadır. Madde 30’da bahsettiğimiz gibi, buna alternatif olarak Optional<E> döndürmeyi deneyebiliriz. Aşağıdaki kod bunu yapmaktadır:

```java
    public static <E extends Comparable<E>> Optional<E> maxOptional(Collection<E> c){
    if (c.isEmpty()){
        return Optional.empty();
    }

    E result = null;
    for (E e : c){
        if (result == null || e.compareTo(result) > 0){
            result = Objects.requireNonNull(e);
        }
    }
    return Optional.of(result);
}
```
Göründüğü gibi optional döndürmek oldukça kolay. Tek yapmak gerekn uygun bir statik factory metodu kullanarak bunu yaratmak.  Optional.empty() boş bir optional, Optional.of(value) ise value değerine sahip bir optional döndürecektir. Ancak burada value değeri null ise NullPointerException fırlatılır. Optional.ofNullable(value) kullanırsanız value değerinin null olması durumunda boş bir optional döndürecektir. Optional dönüş türüne sahip bir metottan asla null döndürmeyin, çünkü optional kullanmaktaki bütün amacımız zaten bunlardan kurtulmak.

- Stream'lerle kullanılan birçok sonlandırıcı işlem optional döndürür. Aynı işlemi stream kullanarak yapıyoruz :


```java
public static <E extends Comparable<E>> Optional<E> maxOptionalStream(Collection<E> c){
    return c.stream().max(Comparator.naturalOrder());
}
```

- Optional metotlar mantık olarak denetimli exceptionlara benzerler.(Checked exception). İstemciyi metottan herhangi bir değer dönmeme durumuyla yüzleşmek zorunda bırakırlar. Metottan null döndürmek veya denetimsiz exception(unchecked exception) fırlatmak istemcinin bu durumu görmezden gelmesine olanak sağlar. Bu da kötü sonuçlar doğurabilir. Denetimli exception fırlatmak ise istemciyi bu durumu ele almaya mecbur bırakır.
- Eğer bir metot optional döndürüyorsa, istemci değer dönmeyen durumda ne yapılması gerektiğini seçebilir ve orElse gibi bir metotla varsayılan (default) bir değer belirleyebilir:
```java
// Varsayılan değer kullanılan optional kullanımı
String lastWordInLexicon = max(words).orElse("No words...");
```

Burada eğer max metodunun döndürdüğü optional boş değilse onun değeri boş ise No words.. değeri kullanılacaktır. Varsayılan bir değer kullanmak yerine uygun bir exception fırlatmak da isteyebiliriz. O kısımda da orElseThrow metoduna bir exception yaratıp geçmek yerine exception yaratan bir factory metot geçiyoruz. Böylece gerekli olmayan durumlarda exceptipon yaratılmasını engellemiş oluyoruz:

- Bazen de varsayılan değer üretiminin masraflı olduğu durumlarla karşılaşabilirsiniz. Bu durumda, varsayılan değeri sadece gerekli olduğunda üretmek için Supplier<T> parametresi kabul eden orElseGet metodunu kullanabilirsiniz. Optional sınıfı başka özel durumlar için tasarlanmış filter, map, flatMap, ve ifPresent gibi farklı metotlar da sunmaktadır. Java 9’da iki tane daha böyle metot eklenmiştir: or ve ifPresentOrElse. Eğer örneklerde kullandığımız nispeten basit metotlar ihtiyacınızı karşılamıyorsa, bu metotların dokümantasyonuna bakarak kullanmayı deneyebilirsiniz.
- Maalesef optional kullanımı bütün türler için faydalı değildir. Koleksiyonlar, streamler, diziler ve map gibi taşıyıcı türler Optional ile sarmalandığında fayda sağlamazlar. Yani Optional<List<T>> döndürmektense boş bir List<T> döndürmek daha mantıklıdır.
- Bu istemciler için de kolaylık sağlayacaktır çünkü istemci optional değerin boş olup olmadığıyla ilgilenmek zorunda kalmayacaktır. 
- Peki bir metodun dönüştürü hangi durumlarda T yerine Optional<T> olmalıdır? Bir kural olarak : eğer bir metodun değer döndüremediği durumlar varsa ve istemciler bu durumu ele almak zorundaysa T değil Optional<T> döndürmeliyiz. Tabi Optional<T> kullanmanın da bir maliyeti vardır. Performansın çok kritik olduğu durumlarda optional kullanımından doğan ek nesne yaratma ve okuma maliyetleri küçük bir fark yaratabilir. Ancak bu performans farkını ölçüm yaparak kanıtlayamıyorsanız endişe etmenize gerek yoktur.
- Kutulanmış (boxed) temel türle sarmalanmış bir optional döndürmek, normal bir temel tür değeri döndürmeye kıyasla oldukça masraflı olmaktadır. Bu sebeple, kütüphane tasarımcıları int, long ve double temel türleri için özel OptionalInt, OptionalLong, ve OptionalDouble sınıfları yazmışlardır. Bu sınıflar Optional<T>‘de bulunan metotların çoğunu barındırırlar. Bu sebeple, kutulanmış temel türleri Optional ile sarmalayıp kullanmayın. Daha küçük temel türler olan Boolean, Byte, Character, Short, ve Float ise bu kuralın istisnası olabilirler.
- Şimdiye kadar nerelerde optional kullanabileceğimizi konuştuk ama nerelerde bunları kullanmamamız gerektiğinden bahsetmedik. Genel olarak söylemek gerekirse, optional nesneleri koleksiyonlarda veya dizilerde eleman, map nesnelerinde anahtar veya değer olarak kullanmak gereksiz karmaşıklığa yol açar ve tavsiye edilmez.
- Bir soru daha var : bir nesne alanı içinde(instance field) optional değer saklamak uyugun olur mu? İlk bakışta bu code smells'e işaret etmektedir. Optional alanlar tanımlamak yerine bunları alt sınıflara taşıyabiliriz. Ancak Optional alanlar tutmak bazen mantıklı olabilir. Bu sınıfın tanımlı alanlarının çoğu zorunlu değildir. Bu alanların her bir kombinasyonu için çok sayıda alt sınıf oluşturmak da çok saçma olurdu. Üstelik bu alanların bazıları temel türdeki değişkenlerle ifade edilmiş. En iyi yol olarak zorunlu olmayan alanlar için yazdığımız erişim metotlarından optinal nesneler döndürülmelidir. Bu sebeple de optional nesne alanları olarak tanımlamak uygun olacaktır.
- Özetle, eğer her durumda bir değer döndürmesi mümkün olmayan bir metot yazıyorsak ve istemcilerin bu ihtimali ele alması gerektiğini düşünüyorsak optional dönüş türü kullanın. Eğer performans konusunda çok hassas isek ve optioanl kullanmanın getirdiği performans kaybını ölçebiliyorsak null döndürmek ve exception fırlatmak bir seçenek olarak kalabilir. Son olarak, optional nesneler çok büyük oranda dönüş türü olarak kullanılırlar. Bunun dışındaki kullanımlara çok nadiren başvurun.

