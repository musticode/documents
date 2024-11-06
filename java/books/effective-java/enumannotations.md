# Enums & Annotations

## int yerine enum kullanmak

- Önceden belli bazı türleri ifade edebilmek için enum'lar kullanılıyor

```java
public static final int APPLE_FUJI         = 0;
public static final int APPLE_PIPPIN       = 1;
public static final int APPLE_GRANNY_SMITH = 2;

public static final int ORANGE_NAVEL  = 0;
public static final int ORANGE_TEMPLE = 1;
public static final int ORANGE_BLOOD  = 2;
```

Üstteki gibi int olarak tanımlanan final değerler çok yanlıştır. Typesafe olarak hatalıdır, eksiktir. Elma yerine portakal geçersek şikayet etmeyecektir. `==`ile bir karşılaştırma bile yapılabilir.

`int i = (APPLE_FUJI - ORANGE_TEMPLE) / APPLE_PIPPIN;` şeklinde bir kullanımla meyve suyu bile tanımlanabilir.

- Enumlar derleme anında tür güvenliği sağlarlar. Eğer bir metot Apple türünden bir parametre bekliyorsa, siz bu metoda Apple tanımında belirlenmiş 3 enum sabitinden birini veya null geçebilirsiniz. Orange gibi başka bir türden nesne referansı geçmeye çalışırsanız derleyici izin vermeyecektir. Bir enum türünü başka bir enum türüne atamaya çalışırsanız veya == ile eşitlik kontrolü yapmanıza derleyici izin vermez
- Son olarak enum türlerinin toString metodunu çağırarak yazdırılabilir bir String elde edebilirsiniz.
- Enumlar Comparable ve Serializable interface'lerini uygularlar. Bütün bunlar biz fazladan tek satır yazmadan derleyici tarafından yazılmaktadır.

## Ordinal yerine nesne alanları kullanmak

- `System.out.println(Color.RED.ordinal());` şeklinde bir kullanım olabilir. İlk değeri 0 olarak anlar ve sırasıyla devam eder.
- Hatalı ordinal kullanımı

```
enum Ensemble {
    SOLO,   DUET,   TRIO, QUARTET, QUINTET,
    SEXTET, SEPTET, OCTET, NONET,  DECTET;

    public int numberOfMusicians() {
        return ordinal() + 1;
    }
}
```

bu kullanım oldukça yanlıştır. numberOfMusicians methodunun bakımı çok zor olaacaktır. Gelecekte bu ensemble sırası bile değişse inanılmaz bir karışıklık yaratacaktır. Bakım maaliyeti oldukça fazladır

## Bit alanları yerine EnumSet kullanmak

- Diziler ile generic türler uyumlu olmadığı için tür dönüşümü gereklidir. Ordinal'ler ile iş yapmak tehlikeli olacaktır.
- Bir problem olarak dizi indisleri için bir int değeri üreten ordinal metodunu kullandığımız için, programın her yerinde doğru int değerlerini kullanmak sizin zorunluluğunuzdadır. Temel int türü enumlar gibi tür güvenliği sağlayamaz. Yanlış bir değer kullanırsanız program sessiz bir biçimde yanlış sonuçlar üretebilir ama şanslıysanız ArrayIndexOutOfBoundsException fırlatır
- java.util.EnumMap adında bir sınıf vardır, EnumMap kullanarak ilerlemek daha sağlıklı olacaktır.
- EnumMap kullanırsak yanlış indis sorunları ile uğraşmamıza gerek kalmayacaktır. Gereksiz dizi kullanmadığımız için indisleri de kontrol altında tutmak gibi bir dert yaşamayacağız.
- EnumMap bize kullanım kolaylığı ve tür güvenliği vermesinin yanında kendi içinde kullandığı dizi sayeesinde yüksek performans da sağlamaktadır. Ayrıca, EnumMap türünün yapıcı metodunda anahtar olarak kullanılmak amacıyla bir Class nesnesi kabul edilmektedir. Bu da sınırlandırılmış tür belirtecine örnektir.

```java
System.out.println(Arrays.stream(garden)
        .collect(groupingBy(p -> p.lifeCycle,
                () -> new EnumMap<>(LifeCycle.class), toSet())));
```

- Maddenin üç halini temsil eden program

```java
enum Phase {
    SOLID, LIQUID, GAS;

    enum Transition {
        MELT, FREEZE, BOIL, CONDENSE, SUBLIME, DEPOSIT;
        private static final Transition[][] TRANSITIONS = {
                {null, MELT, SUBLIME},
                {FREEZE, null, BOIL},
                {DEPOSIT, CONDENSE, null}
        };

        public static Transition from(Phase from, Phase to) {
            return TRANSITIONS[from.ordinal()][to.ordinal()];
        }

    }
}
```

Bu program çalışır hata gözümüze hoş da görünebilir ancak aldatıcıdır. Bitki örneğindeki gibi int türünden dizi indisleri kullandığı için Phase veya Phase.Transition enumlarında bir değişiklik yapıp iki boyutlu diziyi olması gerektiği gibi güncellemezseniz programınız çalışma zamanında çökecektir.

```java
    enum Transition{
        MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID);
        private final Phase from;
        private final Phase to;
        Transition(Phase from, Phase to){
            this.from = from;
            this.to = to;
        }

        private static final Map<Phase, Map<Phase, Transition>> m =
                Stream.of(values()).collect(
                        groupingBy(t -> t.from, () ->
                                        new EnumMap<>(Phase.class),
                                toMap(t -> t.to, t -> t,
                                        (x, y) -> y, () -> new EnumMap<>(Phase.class))));

        public static Transition from(Phase from, Phase to) {
            return m.get(from).get(to);
        }

    }
```

Burada ilk dikkati çeken şey Transition enum sabitlerinin ypaıcı metotlarına maddenin hangi iki hali arasındaki geçişi temsil ettiğini göstermek üzere from ve to isimli Phase sabitleri geçmesidir

- Geçişleri temsil eden veri yapısının yaratılması biraz karmaşık. Buradaki m nesnesinin türü Map<Phase, Map<Phase, Transition>> olarak belirlenmiştir. Bu veri yapısında dıştaki Phase anahtarı from(maddenin ilk hali), içteki Phase anahtarı da to(maddenin son hali), Transition ise geçiş sabitini temsil etmektedir. Bu iç içe Map yapısını kurmak için stream ve kademeli collector kullanılmıştır.
- Şimdi varsayalım ki maddednin başka bir hali olan plazmayı eklemek istiyoruz. Bu yeni hal için daha karmaşık bir geçiş tanımlamak gerekecektir. Burda eğer 2 boyutlu dizi kullanarak  yapıyor olsaydık Phase enum'una bir tane, Transition enum'una da 2 tane daha ekleme yapardık ancak 9 elemanlı diziyi 4x4=16 elemanlı diziye çıkarıyor olacaktık. En ufak hatada programımız çökecekti. EnumMap kullandığımız yapıda is tek gereken Phase enum türüne PLASMA isimli bir yeni değer, Transition içine ise IONIZE(GAS,PLASMA) ve DEIONIZE(PLASMA,GAS) olacak şekilde yeni değerler eklemek
```java
SOLID, LIQUID, GAS, PLASMA;
enum Transition{
    MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID),
    BOIL(LIQUID, GAS), CONDENSE(GAS, LIQUID),
    SUBLIME(SOLID, GAS), DEPOSIT(GAS, SOLID),
    IONIZE(GAS, PLASMA), DEIONIZE(PLASMA, GAS);
```
Programın geri kalanında hiçbir değişiklik yapmadan çalışacaktır.

- Özetle, ordinal değerlerini dizi indisi olarak kullanmak çok nadir durumlar haricinde önerilmez, bunun yerine EnumMap kullanın. Eğer ifade etmek istediğiniz ilişkiler çok boyutlu ise iç içe `EnumMap<..., EnumMap<...>>` kullanın.  

## Enum'larda kalıtımı arayüz olarak kullanarak taklit etmek

- Enum türleri kalıtılamayan `final` sınıf türlerine dönüşüyor. Dolayısıyla bir enum türünü başka bir enum kalıtamaz.
- Bu duruma rağmen kalıtım ihtiyacı duyabiliriz, interface'ler ile bu durumu oluşturabiliriz
```java
interface Operation{
    double apply(double x, double y);
}

enum BasicOperation implements Operation {
    PLUS("+") {
        @Override
        public double apply(double x, double y) {
            return x + y;
        }
    },
    MINUS("-") {
        @Override
        public double apply(double x, double y) {
            return x - y;
        }
    },
    TIMES("*") {
        @Override
        public double apply(double x, double y) {
            return x * y;
        }
    },

    DIVIDE("/") {
        @Override
        public double apply(double x, double y) {
            return x / y;
        }
    },

    ;

    private final String symbol;

    BasicOperation(String symbol) {
        this.symbol = symbol;
    }

    @Override
    public String toString() {
        return super.toString();
    }

}
```
Eğer bir kullanıcı BasicOperation türünde tanımlı aritmetik işlemlere eklemeler yapmak isterse bu enumu kalıtamaz ancak Operation arayüzünü uygulayabilir. Yeni bir enum ile devam ettirebilir. Örnek olarak üst alma ve bölme işleminden kalanı hesaplama işlemlerini de eklemek isteyebiliriz, bunun için kalıtım kullanamayız ama Operation interface'ini implemente edebiliriz.
```java
enum ExtendedOperation implements Operation{
    EXP("^"){
        @Override
        public double apply(double x, double y) {
            return Math.pow(x,y);
        }
    },
    REMAINDER("%"){
        @Override
        public double apply(double x, double y) {
            return x % y;
        }
    };
    private final String symbol;

    ExtendedOperation(String symbol) {
        this.symbol = symbol;
    }

    @Override
    public String toString() {
        return "ExtendedOperation{" +
                "symbol='" + symbol + '\'' +
                '}';
    }
}
```

- Bu kodların test edilebilecek metodu aşağıdaki gibi yazılmıştır.Metottaki opEnumType parametresinin T extends Enum.. gibi tür tanımını da geçilebilecek Class nesnesinin hem bir enum olması hem de operation interface'ini implemente etmesi gerektiğini belirtiyor. 
```java
    private static <T extends Enum<T> & Operation> void test(Class<T> opEnumType, double x, double y){
        for (Operation op : opEnumType.getEnumConstants()){
            System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y));
        }
    } 
```

- Arayüz kullanarak kalitılabilirliği taklit etme yönteminin bir dezavantajı bir enumda tanımlı mettotların diğer metot tarafından kullanılamamasıdır. Bu metodun davranışı nesnenin durumuna bağımlı değilse arayüz içinde varsayılan mettot olarak yazılabilir. Bizim örneğimizde BasicOperation ve ExtendedOperation enumları symbol alanının saklanması ve okunması amacıyla yazılmış bölümleri tekrar etmektedilerler. Bu örnekte tekrar edilen kod miktarı çok az olduğu için farketmez ama artarsa ortak kısımları bir yardımcı sınıf veya statik bir yardımcı metot içine taşıyabilirsiniz. Örneğin java.nio.file.LinkOption enum türü CopyOption ve OpenOption arayüzlerini uygulamaktadır.
- Özetle, kalıtılabilen enum türleri yazamasak da bu etkiyi verebilmek için bir arayüz tanımlayıp yazdığımız enumda bunu uygulayabiliriz. 


## Annotation'ları enum'a isimlendirme modellerine tercih etmek 

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@interface Test{

}
```

- Bu Test notasyonu tanımlanırken başka notasyonlar da kullanıldı. Bunlar meta annotation'dur. @RetentionPolicy.RUNTIME tanımladığımız test notasyonunun çalışma zamanında korunması gerektiğini belirtmektedir. Aksi taktirde notasyonumuz test kütüphanesi tarafından görülemez
- @Target(ElementType.METHOD) is bu notasyonun sadece metotlar üzerinde kullanılabileceğini berlirtmektedir. Sınıflar, alanlar veya başka program elemanları ile kullanmak derleme hatalarına yol açacaktır.
- Exception'lar için test interface'i

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@interface ExceptionTest{
    Class<? extends Throwable> value();
}
```
- Bu notasyonun Class<? extends Throwable> türünde bir parametresi vardır. Yani parametre olarak Throwable sınıfını kalıtan bir sınıfın Class nesnesini kullanabiliriz. Bütün aykırı durumlar (exception) Throwable sınıfını kalıttığı için bu notasyonu kullanan kişiler istedikleri aykırı durum türünü parametre olarak kullanabilirler.
```java
    @ExceptionTest(ArithmeticException.class)
    public static void m2(){
        int i = 0;
        i = i / i;
    }
```

- Bu maddedde yazdığımız test çatısı basittir ancak yine de notasyonların isimlendirme modellerine göre daha üstün olduğunu görmemiz için yeterli olacaktır. Yazılan kaynak koda herhangi bir bilgi eklemesini gerektiren araçlar yazarsak isimlendirme modelleri değil mutlaka notasyonları kullanmak gerekir.

## Override Annotation'ununu sürekli olarak kullanmak

```java
class Bigram{
    private final char first;
    private final char second;
    Bigram(char first, char second){
        this.first = first;
        this.second = second;
    }

    public boolean equals(Bigram b) {
        return b.first == first && b.second == second;
    }

    @Override
    public int hashCode() {
        return 31 * first + second;
    }
}
```

- Yukarıdaki kod parçası hatalıdır. Sonuç 26 çıkması gerekirken 260 olarak çıktı verecektir. Sorun şurdadır: sınıfı yazan kişi equals metodunu geçersiz kılıp override etmek istemiş ve hatta hashCOde'u da override etmesi gerektiğini unutmamıştır. Ancak equals metodunu override etmeye çalışan programcı yanlışlıkla overloading yapmıştır. Çünkü equals metodu Object sınıfında ngelen parametre türü olarak Object beklerken bu class'ta Bigram sınıfındaki equals metodunda, Object yerine Bigram parametresi geçilmiştir.
- Bu gibi sorunları `@Override` notasyonu ile çözebiliriz. Bir üst class'taki versiyon geçersiz kılınacaktır.
- Üstteki kodu aşağıdaki gibi düzeltebiliriz.
```java
@Override
public boolean equals(Object o) {
    if (!(o instanceof Bigram)) {
        return false;
    }

    Bigram b = (Bigram) o;
    return b.first == first && b.second == second;

}
```

- Özetle, Override notasyonu kullanıldığı zaman derleyici sizi türlü hatalardan korur. Bu notasyonu üst türlerdeki metotları geçersiz kılmak istediğiniz durumlarda kullanın. Somut sınıflarda soyut metotları geçersiz kılıyorsanız kullanmanız şart değildir ancak bir zararı da yoktur.

## İşaretçi arayüzleri(marker interface) tür tanımlamak için kullanmak

- Marker interface'ler içlerinde hiçbir metot bulunmayan interface'lerdir. Bunları uygulayan sınıfların amacı sahip oldukları bir özelliği ortaya çıkarmak için kendilerini işaretlemektir. Serializable interface'i buna örnek olarak verilebilir. 
- İşaretçi arayüzlerin iki tane üstünlüğü vardır. Birincisi ve en önemlisi işaretçi arayüzler uygulayan sınıflar için bir tür görevi görürler ancak notasyonlarda bu yoktur. İşaretçi arayüz türünün varlığı hataları derleme anında yakalamamızı sağlar. İşaretçi notasyonlar kullanıldığında ise çalışma zamanına kadar hataları farkedemeyiz.
- Ne zaman marker interface ne zaman annotation kullanmak gerekir? Sınıf veya arayüz harici program elemanlarını işaretlemek istiyorsak notasyon kullanmak gerektiği açıktır. Çünkü marker interface'ler sadece sınıflar veya başka interface'ler uygulayabilir. İşaretçi eğer sadece sınıflar ve arayüzler içinse, kendinize şu soruyu sorun: “Sadece bu işaretçiye sahip olan nesneleri kabul edecek metotlar yazmam gerekir mi?” Cevabınız evetse işaretçi arayüz kullanmanız gerekir. Bu sayede işaretçi arayüz tür olacağı için metot parametrelerinde kullanabilir ve derleme anında tür uyumunu garanti edebilirsiniz. Ancak yukarıdaki soruya net bir biçimde hayır yanıtını verebiliyorsanız, işaretçi notasyon kullanmanız daha mantıklı olabilir. Ek olarak, tanımladığınız işaretçi notasyonları sıklıkla kullanan bir çatının (framework) parçası olacaksa yine işaretçi notasyonları kullanmanız gerekir.
- Özetle, işaretçi arayüzlerin de işaretçi notasyonların da kullanım alanları vardır. Eğer amacınız hiçbir metot eklemeden yeni bir tür tanımlamaksa o zaman işaretçi arayüz kullanmalısınız. Sınıf veya arayüz haricindeki program elemanlarını işaretlemek ve zaten notasyonları ağırlıkla kullanan bir çatıya işaretçi eklemek içinse işaretçi notasyonları kullanın. İşaretçi notasyon tanımlarken hedef (target) olarak ElementType.TYPE kullanıyorsanız bunun gerçekten notasyon mu yoksa işaretçi arayüz mü olması gerektiğini iki kere düşünün.
