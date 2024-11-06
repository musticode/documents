# General Programming

## Yerel değişkenlerin etki alanlarını kısıtlamak

- Yerel değişkenlerin etki alanını (scope) kısıtlayarak hem kodunuzun okunabilirliğini ve bakım yapılmasını kolaylaştırırsınız, hem de hata yapılmasının önüne geçmiş olursunuz.
- Yerel değişkenlerin etki alanının kısıtlanması konusunda uygulanabilecek en iyi yöntem, değişkeni ilk defa kullanıldığı yerde tanımlamaktır. Bir değişken kullanılmadan önce tanımlanmışsa, kodu dağınık göstermekten başka bir işe yaramayacaktır. Kodu okuyup anlamaya çalışan kişilerin dikkatini bozacaktır. Okuyucu, değişkenin kullanıldığı satıra geldiğinde belki de değişkenin tipini ve değerini hatırlamayacaktır.
- Bir değişkeni gereğinden önce tanımlamak etki alanının (scope) sadece erken başlamasına değil aynı zamanda geç bitmesine sebebiyet verebilir. Etki alanı değişkenin tanımlandığı anda başlar ve içinde bulunduğu blok sonlanana kadar devam eder. Eğer değişken kullanıldığı bloğun dışında tanımlanmışsa, değişkene ihtiyaç kalmadığı durumda bile erişilebilir olacaktır. Bu durum değişkenin yanlış bir noktada yanlışlıkla kullanılması ihtimalini doğurur, bu da kötü sonuçlara sebebiyet verebilir
- For döngülerinin while döngülerine göre bir diğer avantajı da daha kısa ve anlaşılır olmalarıdır. Aşağıdaki döngü de yine yerel değişkenlerin etki alanlarının sınırlandırılmasına güzel bir örnektir:
- Yerel değişkenlerin etki alanını kısıtlamak için kullanılabilecek bir diğer yöntem de metotları kısa tutmak ve aynı metotta birden fazla iş yapmamaktır. Eğer bir metotta iki veya daha fazla iş yapmaya kalkarsanırz, bunlardan birini yaparken tanımladığınız değişkenlerin etki alanı diğer işlemleri yaparken kullandığınız değişkenlerle karışabilir. Bunu önlemek için bu işlemleri farklı metotlarda yapabilirsiniz.



## Klasik for döngüleri yerine for-each tercih etmek

- Bazı durumlarda stream kullanmak avantajlı olurken bazen de yinelemeye (iteration) başvurmak daha mantıklı olmaktadır
```java
// Koleksiyon elemanlarını tarayan klasik döngü kodu
for (Iterator<Element> i = c.iterator(); i.hasNext(); ) {
    Element e = i.next();
    ... // e ile bir işlem yapabilirsiniz
}
```


```java
// Dizi elemanlarını tarayan klasik döngü kodu
for (int i = 0; i < a.length; i++) {
    ... // a[i] ile bir işlem yapın
}
```

Bu kullanımlar while döngülerinden daha iyi olsa da mükemmel değiller. Iterator ve döngü değişkenleri son derece gereksiz, tek ihtiyacımız olan elemanların kendileri. Bu gereksiz değişkenler döngüyü kurarken ve elemanlara erişirken hata yapma olasılığı doğuruyor ve kodun okunabilirliğini azalatmaktadır. Ayrıca dizi ve koleksiyon elemanlarına erişim için döngüler farklı şekilde kuruluyor. Bunun yerine for-each döngülerini kullanarak bu problemleri aşabiliriz. Bu döngüler Iterator ve döngü değişkenleri gizlenmiş olduğu için hata yapma olasılığı ortadan kalkar ve kodun okunabilirliği artar.

```java
// Tercih edilen döngü kullanımı
for (Element e : elements) {
    ... // e elemanını kullanabilirsiniz..
}
```
- İç içe kullanımlarda aşağıdaki gibi kullanmak daha okunabilir olacaktır
```java
// İç içe döngülerlerde tercih edilmesi gereken for-each kullanımı
for (Suit suit : suits) {
    for (Rank rank : ranks) {
        deck.add(new Card(suit, rank));
    }
}
```
- for-each döngülerini kullanmanın mümkün olmadığı 3 tane durum var:
  - Eğer bir koleksiyon üzerinde gezerken aynı zamanda eleman silmek gerekiyorsa iterator kullanmak gerekir ve remove metodu çağırmak gerekiyor. Ancak Collections'a eklenen removeIf metodunu kullanarak döngüye gereke kalmadan da bunu ypaabiliriz.
  - Eğer bir liste veya dizi elemanları üzerinde gezip bir veya daha fazla elamanın değerini değiştirmek istiyorsak, bir liste iteratörü veya dizi indisi kullanmak gerekecektir.
  - Eğer birden fazla koleksiyon üzerine aynı anda iterasyon yapaaksak veya dizi indisine gerek duyarsak çünkü koleksiyonlar üzerinde ilerleme yaparken bunların birbirleriyle uyumlu çalışması gerekir
- Eğer sıfırdan bir Iterator gerçekleştirimi yapıyorsanız bu arayüzü uygulamak biraz zahmetli olabilir. Ancak bir grup elemanı temsil eden bir tür yazıyorsanız Iterator arayüzünü uygulamayı ciddi ciddi düşünün. Böylece kullanıcılarınız tür üzerinde for-each döngüsüyle tarama yapabilirler ve bunun için size minnet duyacaklardır.
- Özetle, for-each döngüleri klasik for döngülerine göre daha okunaklıdır, hata yapma olasılığını düşürürler ve daha esnektirler. Herhangi bi performans kaybına da sebep olmadıklarından, mümkün olan her yerde kullanılmalıdırlar.


## Kütüphaneleri öğrenmek ve kullanmak

- Bu hataları çözen ve daha doğru çalışan bir random metodu yazabilmek için ciddi matematik bilgisi gerekmektedir. Neyse ki buna gerek yok, çünkü yazılmışı var: Random.nextInt(int) Bu konulara oldukça hakim uzman bir mühendis, bu metodu tasarlamak, yazmak ve test etmek için büyük çaba ve zaman harcadı. Üstüne başka uzmanlara da danışarak metodun doğruluğunu teyit ettirdi. Daha sonra bu kütüphane defalarca test edildi, yayınlandı ve neredeyse yirmi yıldır milyonlarca yazılımcı tarafından kullanıldı. Bugüne kadar bir hata bulunamadı, ancak bulunursa da bir sonraki versiyonda çözüleceğini biliyoruz. Standart bir kütüphane kullandığınızda, bunu yazan uzmanların bilgisinden ve daha önce kullanan yazılımcıların tecrübelerinden istifade etmiş olursunuz.
- Java 7’den itibaren artıkRandom kullanmamalısınız. Rastgele sayı üretmek için çoğu durumda ThreadLocalRandom tercih edilmelidir. Çünkü hem bu işi daha iyi yapmaktadır hem de daha hızlı çalışmaktadır. Benim makinamda ThreadLocalRandom, Random‘a kıyasla 3,6 kat daha hızlı çalışmaktadır. Fork-join havuzları ve paralel streamler içinse SplittableRandom kullanmalı
- **Standart kütüphaneleri kullanmanın ikinci bir avantajı da, uygulama koduna odaklanmanıza olanak sağlamasıdır. Bu şekilde standart işleri halletmek için yeniden kod yazmanıza gerek kalmaz, zamanınızı ve enerjinizi sizi asıl ilgilendiren uygulamanın kritik parçalarına ayırabilirsiniz.**
- **Standart kütüphaneleri kullanmanın üçüncü avantajı ise zaman geçtikçe performans iyileştirmeleri yapılmasıdır. Bunları kullanan çok sayıda kişi ve kuruluş olduğu için ve sık sık performans testlerinden geçirilir ve gerekirse yeniden yazılırlar. Java kütüphanelerinin bir çoğu zaman içerisinde defalarca yeniden yazılmış ve ciddi performans kazanımları elde edilmiştir. Bunları kullanan programcılar çaba sarf etmeden bu iyileştirmelerden faydalanabilirler.**
- Dördüncü bir avantaj ise bu kütüphanelere zaman içerisinde yeni özellikler eklenmesidir. Bir eksik bulunduğunda yazılımcı toplulukları bunu rapor ederler ve bir sonraki versiyonda bu eksiklik giderilebilir.
- Son olarak, standard kütüphaneler kullandığınızda kodunuz daha çok yazılımcı tarafından kolayca anlaşılabilecek ve bakım yapılması kolaylaşacaktır.
- Özetle, tekerleği yeniden icat etmeyin! İhtiyacınız olan şeyin başka yazılımcılar tarafından da yaygın olarak kullanılabileceğini düşünüyorsanız, büyük ihtimalle bir kütüphane bunu sağlıyor olacaktır. İhtiyacınızı gideren bildiğiniz bir kütüphane varsa kullanın, yoksa da araştırıp bulmaya çalışın. Genel olarak konuşmak gerekirse, bu kütüphanelerde bulacağınız kod sizin kendi yazacağınız koddan daha iyi olacaktır ve zamanla daha da gelişecektir. Bu sizin yazılımcı olarak yeteneklerinizi küçümsemek anlamına asla gelmez. Kütüphane kodları sizin tek başınıza verebileceğinizden çok daha fazla ilgi ve özenle yazıldığı için, daha kaliteli olması son derece doğaldır.

## Net sonuçlar almak için double ve float kullanmaktan kaçınmak

- Java’da double ve float bilimsel ve mühendislik hesaplamaları için tasarlanmış türlerdir. Bu türler, ikili kayan noktalı işlemler (binary floating-point arithmetic) yapmak için tasarlarlanmıştır ancak net sonuçlar almak istediğinizde doğru çalışmazlar. Özellikle de parasal hesaplamalar yapmak için hiç uygun değildirler çünkü 0.1 değerini (ya da 10’un diğer negatif kuvvetlerini) double veya float kullanarak ifade etmek imkansızdır.
- Özetle, bir matematiksel hesaplama yaparken tam cevaplar almak istiyorsanız float veya double kullanmaktan kaçınmalısınız. Eğer ondalık değerlerle uğraşmak istemiyorsanız ve performans kaybı önemli değilse BigDecimal kullanabilirsiniz. BigDecimal kullanmanın bir avantajı da size sekiz farklı yuvarlama modu sunmasıdır. Eğer yasal sebeplerden ötürü belli bir yuvarlama davranışı kullanmanız gerekiyorsa bunun faydasını görebilirsiniz. Eğer performans çok önemliyse ve ondalık değerleri kendiniz takip edebiliyorsanız, değerlerin büyüklüğüne göre int veya long kullanın.


## Boxed türler yerine temel türleri kullanmak

- Java’da tür sistemi iki parçadan oluşur: int, double, boolean gibi temel türler ve String, List gibi referans türleri. Her temel türe karşılık gelen birer tane de kutulanmış temel tür (boxed primitive type) bulunmaktadır. Örneğin, int, double ve boolean temel türlerine karşılık gelen kutulanmış türler Integer, Double ve Boolean olmaktadır.
- temel türlerle kutulanmış türler arasında yapılan otomatik dönüştürme işlemleri (autoboxing ve auto-unboxing), bu türler arasındaki farkı bir nebze saklasa da ortadan kaldırmaz. Bu türler arasında önemli farklar vardır ve kod yazarken bunlardan hangisini kullandığınızın farkında olmanız, bu ikisi arasında seçim yaparken dikkatli olmanız gerekmektedir.
- Temel türler ve kutulanmış türler arasında üç tane önemli fark vardır. Birincisi, temel türlerin sadece değerleri vardır ancak kutulanmış türlerin hem değerleri hem de bundan bağımsız kimlikleri (identity) vardır. Başka bir deyişle, iki tane kutulanmış tür nesnesi aynı değere ama farklı kimliklere sahip olabilirler. İkinci olarak, temel türlerde sadece işlevsel değerlerden bahsedebiliyoruz ancak kutulanmış türlerde buna ek olarak bir de işlevsel olmayan bir null değeri mümkündür. Son olarak, temel türlerin kutulanmış türlere kıyasla daha verimli çalıştıklarını söyleyebiliriz. Bu üç farklılık da eğer dikkatli olmazsanız başınıza büyük sorunlar açabilir.
`naturalOrder.compare(new Integer(42), new Integer(42))`
Burada her iki Integer değeri de 42 değerine sahiptir ve dolayısıyla compare metodu 0 döndürmelidir değil mi? Beklentimiz bu ancak sonuç 1 çıkmaktadır yani birinci Integer ikinciden daha büyüktür!
- Peki buradaki problem nedir? naturalOrder içindeki i < j aslında doğru çalışmaktadır. Bu karşılaştırma için i ve j ile temsil edilen Integer değerleri int türüne dönüştürülmekte (auto-unboxing) ve ikisi de 42 değerine sahip olduğu için i < j false değerini üretmektedir. Daha sonra ikinci karşılaştırma, yani i == j işletilmektedir. Bu ifade ile değer karşılaştırması değil kimlik karşılaştırması yapılmaktadır. Her ne kadar i ve j nesneleri aynı değeri taşısalar da, farklı Integer nesneleri oldukları için bu karşılaştırma false döndürecektir ve metodun sonucu 1 olarak döndürülecektir. Kutulanmış türlere == operatörünün uygulanması neredeyse her zaman yanlış sonuç almanıza neden olacaktır.
- Boxed türlere `==` operatörünün uygulanması neredeyse her zaman yanlış sonuçlar almamıza neden olacaktır.
- Peki ne zaman kutulanmış türleri kullanmalıyız? Bu türlerin birkaç tane kullanım alanı vardır. Bunlardan bir tanesi koleksiyonlar içerisinde anahtar (key) veya değer (value) olarak kullanmaktır. Koleksiyonlar içerisine temel türleri koyamadığımız için mecburen kutulanmış tür kullanmamız gerekmektedir. Bununla beraber, üreysel tür ve metotları tanımlarken de kutulanmış türler kullanılır. Örneğin, ThreadLocal<int> şeklinde bir kullanıma izin verilmez, bu sebeple ThreadLocal<Integer> kullanmalıyız. Son olarak, reflection (yansıma) kullanarak yapılan metot çağrılarında da kutulanmış türleri kullanmak gerekmektedir
- Özetle, seçme şansımız olan her yerde boxed türler yerine ilkel türleri kullanmak gerekir. Eğer boxed tipleri kullanmak zorundaysak çok dikkatli olmamız gerekiyor. Autoboxing kodun kalabalığını azaltsa da, boxed türlerin sebep olabileceği sorunları engellemez. Program eğer boxecd türler ile == operatörü kullanıyorsa burda yapılan şey kimlik karşılaştırması olur ve genelde istenen şey bu değildir. . Eğer null değere sahip bir kutulanmış değişkenin temel türe dönüştürülmesi gerekirse (auto-unboxing), sonuçta NullPointerException fırlatılacaktır. Son olarak da, siz farkında olmadan temel türdeki değişkenleriniz kutulanmış türlere dönüştürülüyorsa, gereksiz nesne yaratmış olursunuz ve performans kayıpları görebilirsiniz.

## Diğer türlerin daha elverişli olduğu durumlarda String kullanmaktan kaçınmak

- String türü, metinleri temsil etmek için tasarlanmıştır ve bu işi gayet iyi yapmaktadır. Ancak stringler çok yaygın ve kullanımı kolay olduğundan, uygun olmadıkları durumlar için de kullanıldığını görmekteyiz. Bu maddede, stringleri kullanmamanız gereken durumlardan bahsedeceğiz.
- Stringler enum türlerinin yerine kullanılmamalıdır, enum türleri kullanarak numaralandırılmış tür sabitlerini tanımlamak çok daha doğru olacaktır.
- Stringler birden fazla bileşenden oluşan türleri temsil etmek için kullanılmamalıdır.
`String compoundKey = className + "#" + i.next();`
  Bu kullanımın çok sayıda dezavantajı vardır. İki değeri birbirinden ayırmak için kullanılan “#” karakteri bu değerlerden birinde bulunursa problem çıkartır. Birleştirdikten sonra bu iki değeri tekrar elde etmek istersek stringi parçalamamız gerekir ve başka hatalara yol açabilir. Ayrıca kendi equals, toString veya compareTo metotlarınızı yazamazsınız çünkü String sınıfından gelenleri kullanmaya mecbur kalırsınız.

#### ThreadLocal

- ThreadLocal sınıfı belirlediğimiz nesnelerin sadece aynı thread tarafından erişilebilir olmasını sağlar. Bu sayede thread safe olmayan nesneleri thread safe kullanmış oluruz. Bir ThreadLocal nesnesi içerisine yazdığımız nesne, ThreadLocal‘e yazan thread tarafından çalıştırılan tüm methodlar tarafından okunabilir olacaktır. Gelin bu güçlü aracı avantajları ve dezavantajları ile inceleyelim.
- İlk olarak ThreadLocal teknik olarak nedir ona bakalım. Java’da her yaratılan Thread nesnesi içerisinde o Thread‘e ait ThreadLocal‘lerin tutulduğu bir map bulunmaktadır. İki farklı thread aynı ThreadLocal nesnesine eriştikleri zaman, ThreadLocal nesnesi değerini o an kendisine erişen thread’in içerisindeki Map‘den okur veya yazar. Bu sayede iki farklı Thread aynı ThreadLocal nesnesi üzerinden farklı değerlere ulaşırlar.
- Boş veya dolu farketmeksizin bir ThreadLocal nesnesine set metodunu kullanarak yeni bir değer atayabiliriz. Atanan değer sadece o değeri atadığımız Thread için geçerli olacaktır. Diğer Thread'lerin atayacakları veya okuyacakları değerler tamamen mevcut Thread'inkinden bağımsız olacaktır.
- Bir ThreadLocal üzerinde mevzut Thread için tutulmakta olan değeri almak için get methodu kullanılır. Mevcut Thread tarafından değer atanmamış veya başlangıç değeri bulunmayan bir ThreadLocal üzerindeki kget metodunu çağırmak null değerini dönecektir.
- ThreadLocal nesneleri bir başlangıç değerine sahip olabilir. Bu durumda set çağırmadan get methodunu ilk kez çağıran her bir Thread belirlenen initialValue değerini alırlar. Bu ilk değer her Thread için farklı olabileceği gibi aynı değer de verilebilir. Bu kısım tamamen kullanıcıya kalmış. Aşağıda başlangıç değeri ile bir ThreadLocal nesnesi oluşturmanın iki farklı yolunu görelim.

- Özetle, daha farklı türlerle ifade edebildiğimiz verileri String ile temsil etmekten kaçınmalıyız. Yanlış kullanıldığında stringler esnekliği azaltır, daha yavaştır ve hata yapmamıza sebebiyet verebilirler. Stringler genellikle temel türler, enumlar ve bileşik türlerin yerine yanlış olarak kullanılırlar.

## String birleştirme yaparken performans kayıplarına dikkat etmek

- String birleştirme operatörünü(+) kullanarak birden çok string değerini tek bir string olarak kolayca birleştirmemiz mümkündür. Bu yöntem eğer tek satırlık bir çıktı oluşturmak istiyorsak veya küçük bir nesneyi string formatında ifade etmek istiyorsak iş görebilir, ancak daha büyük işlerde sınıfta kalır. Bunun sebebi ise stringlerin değiştirilemez(immutable) olmasıdır. İki string'i birleştirdiğimiz de her iksinin de kopyalanması gerekir.
- String builder kullanılırken metin boyutu önden geçilebilir, eğer geçilmezse de yine String'leri birleştirmekten çok daha hızlıdır
- String birleştirme operatörünü en fazla birkaç tane stringi birleştirmek için kullanın. Kod performansı önemliyse, bunun yerine StringBuilder.append(); metodunu kullanın. Bunun yerine karakter dizisi (char array) de kullanılabilir ve string'leri birleştirmek yerine tek tek işleyebiliriz.


## Nesneleri arayüz türleri üzerinden kullanmak

- Parametre türleri olarak sınıflar yerine interface'leri kullanmak gerekir. Bunu daha da genelleyecek olursak, nesneleri işaret etmek için sınıflar yerine interface'leri kullanmalıyız. Eğer uygun bir arayüz türü mevcutsa, parametreler, dönüş değerleri, değişkenler ve alanlar interface ile tanımlanmalıdır. Bunun tek istisnası constructor metot içerisinde nesne yarattığımız zamandır. Bunu örneklemek için aşağıda örnek eklenmiştir.
`Set<Son> sonSet = new LinkedHashSet<>();`
- Eğer referans türü olarak arayüzleri kullanmayı alışkanlık haline getirirseniz, programlarınız çok daha esnek olacaktır. Eğer sonradan aynı arayüzü uygulayan başka bir sınıf kullanmak isterseniz, tek yapmanız gereken sınıf adını ve yapıcı metot çağrısını değiştirmek olacaktır. Örneğin, yukarıdaki birinci kod örneğini aşağıdaki gibi değiştirmek mümkündür:
`Set<Son> sonSet = new HashSet<>();`
- Programın geri kalanı sorunsuz biçimde çalışmaya devam edecektir. Diğer kodlar kullanılan önceki sınıftan haberdar olmadıkları için yapılan değişiklikten de etkilenmezler.
- Eğer kullanılan ilk sınıfta arayüzde tanımlanmamış bir özellik varsa ve kodun geri kalanı da bu özelliğe bağımlıysa, kullanmak istediğimiz yeni sınıfın da aynı özelliğe sahip olması gerekir. Örneğin, eğer kodun geri kalanı LinkedHashSet ile gelen sıralama özelliğini kullanıyorsa bunu HashSet ile değiştirmek çalışmayacaktır çünkü HashSet elemanlar arasında herhangi bir sıralama yapacağını garanti etmez.
- Peki o zaman neden gerçekleştirim türünü değiştirmek isteyesiniz? Bunun sebebi ikinci gerçekleştirimin ilkine göre daha performanslı çalışması olabilir veya birincide eksik olan bazı özelliklerin eklenmesi olabilir. Örneğin, HashMap türünde bir değişkeniniz varsa bunu EnumMap ile değiştirmek hem hızı artıracaktır hem de anahtar değerler enum türünde olmak zorunda olduğu için enumların bazı avantajlarından faydalanmak da mümkün olur. Eğer anahtar değerler enum olarak tanımlanamıyorsa, EnumMap yerine LinkedHashMap kullanmak isteyebilirsiniz. Bu durumda daha tahmin edilebilir bir iterasyon (yineleme) özelliği kazanmış olursunuz.
- Eğer kullanabileceğiniz uygun bir arayüz yoksa, nesneleri yaratıldıkları sınıf türünde referanslarla tanımlamak gayet normaldir. Örneğin, String ve BigInteger gibi türleri düşünün. Bu tür sınıflar birden fazla gerçekleştirim yazılabileceği düşünerek yazılmamıştır. O yüzden final olarak tanımlanmış ve arayüz kullanılmamıştır. O yüzden bu tür değer sınıflarını (value class) sınıf isimleriyle kullanmakta bir sorun yoktur.
- Buna ikinci bir örnek de sınıf-tabanlı bazı uygulama çatılarının (framework) içerisinde gelen ve temel olarak arayüz yerine sınıfları kullanan türlerdir. Eğer bu şekilde bir sınıftan yaratılmış bir nesneniz varsa, bunları temsil etmek için genellikle abstract olan ata sınıf türlerini kullanmalıyız. OutputStream gibi birçok java.io sınıfı bu kategoridedir.
- Bu üç örnek referans türleri tanımlarken sınıf kullanmanın hangi durumlarda normal olduğunu göstermek için seçilmiştir ancak bunun dışında başka durumlar da olabileceği açıktır. En nihayetinde, bir sınıfı temsil etmek için uygun bir arayüz olup olmadığı çabucak anlaşılabilir. Eğer böyle bir arayüz varsa ve siz de bunu referans türü olarak kullanırsanız, programlarının daha esnek ve zarif olacaktır. Eğer uygun arayüz yoksa, size ihtiyacınız olan bütün özellikleri sağlayan en yukarıdaki ata sınıfı kullanın.








