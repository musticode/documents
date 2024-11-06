# Inheritance Notlar

- Şimdiye kadar anlatılanlardan, bir sınıfı kalıtıma izin verecek şekilde tasarlamanın çok zahmetli bir iş olduğunu ve sınıf üzerinde ciddi kısıtlamalara yol açtığını anlamışsınızdır. Bu sebeple bu kararı alırken ciddi kafa yormanız gerekmektedir. Soyut sınıflar (abstract class) gibi durumlarda bu açıkça yapılması gereken bir şey olsa da, değiştirilemez sınıflar (immutable class) söz konusuysa kalıtımı düşünerek tasarım yapmak anlamsızdır. (Madde 17)
- Cloneable ve Serializable arayüzleri kalıtım için tasarlanan sınıflar için daha büyük güçlükler doğurmaktadır. Genel olarak, kalıtım için tasarlanan sınıfların bu arayüzleri uygulaması iyi bir fikir değildir çünkü bu sınıfı kalıtacak programcılara ağır bir yük yüklenmiş olur. Ama yine de bunu yapmak istiyorsanız, Madde 13 ve Madde 86’da anlatıldığı şekilde kullanabileceğiniz özel yöntemler mevcuttur. Bu arayüzlerden birini uygulamaya karar verdiyseniz, yapıcı metotlardaki kısıtlamanın clone ve readObject metotları için de geçerli olduğunu unutmayın: clone ve readObject içerisinde geçersiz kılınabilen başka metotları doğrudan veya dolaylı olarak çağırmayın!
- Somut sınıflarda genel olarka kalııtım uygulandığında problem oluşabilir. Bu tarz sınıf yazarken çok dikkatli olmak gerekiyor
- Kalıtım için tasarlanmamış ve belgelenmemeiş sınıfıların kalıtılmasını yasaklamak en iyi çözüm olacaktır
- İki yol vardır: kolay olanı sınıfı final olarak tanımlamaktır. İkinci yol ise bütün constructor metotları private ya da pacakage private olarak tanımlayıp, public static factory metotlar aracılığıyla nesne yartılmasına izin vermek gerekiyor. Yani her fonksiyon için sınf içinde yeni nesne yaratmak.
- Eğer somut bir sınıf standart bir interface uygulamıyorsa kalıtımı kısıtlamak suretiyle bazı insanları zora sokabiliriz. Bu tarz sınıfılarda kendimizi kalıtıma izin vermek zorunda hissedebiliriz. En azından sınıfın geçersiz kılınabilen metotlarının hiç birisini çağırmadığından emin olmak gerekir. Kalıtılması güvenli bir sınıf yazmış olabiliriz
- Sınıfın geçersiz kılınabilen metotlarını çağırmasını davranış değişiklipğiine sebep olmadan engellemenin yolu mevcut. Geçersiz kılınabilen yani override edilebilen her metodun içeriğini private bir yardımcı metoda(helper)'a taşıyın ve override edilebilen her metottan kendisine ait olan yardımcı metodu çağırın. Daha sonra sınıfın içerisinde geçersiz kılınabilen metotları çağıran kısımları değiştirip direkt olarak onlara karşılık gelen private yardımcı metotları çağırın.
- Özetle bir sınıfı kalıtıma uygun tasarlamak zor iştir. Sınfın kendi içindeki kullanımları çok iyi belgelemek gerekiyor ve bunlara sınıfın ömrü boyunca riayet etmek gerekiyor. Bunu yapmazsak çocuk sınfılar ata sınıfın gerçekleştirme detaylarına bağımlı olur ve bu gerçekleştirim değiştiğinde hata vermeye başlarlar. Diğer programcılara verimli çocuk sınflar yazmalarına izin verebilmek için bazı metotları protected olarak dışarı açmak gerekebilir. Sınıfınızdan çocuk sınıflar yaratılması konusunda ciddi bir gereklilik hissetmiyorsak kalıtıtımı yasaklamak daha faydalı olacaktır.

## İşaretli Sınıflar - 23

- İşaretli bir sınıfı sınıf hiyerarşisine çevirmek için öncelikle soyut bir sınıf tanımlamak gerekir. Bu sınıfa metot olarak her bir tür için uygulanması gereken ve gerçekleştirimi türe göre değişiklik gösteren metotları yazın. Örnekteki area metodu buna örnektir. Çünkü alan hesaplama türe göre değişiklik göstermektedir. Bu soyut sınııf, hiyerarşisinin tepesinde olacaktır. Gerçekleştirimi türe göre değişmeyen, başka bir deyişle bütün türler için kodu aynı olan metotları bu sınıfa somut metot olarak ekleyin. Yine bütün türler için kullanılan ortak alanlar varsa onları da ekleyin. Bizim Figure örneğimizde bütün türler tarafından ortak olarak kullanılabilecek bir metot veya alan yoktur.
- Daha sonrasında bizim gerçekleştirmek istediğimiz her tür için bir somut sınıf yaratarak soyut sınıfı kalıtın. `Circle` ve `Rectangle` olmak üzere iki tane somut sınıf oluşturulması gerekir. Bu sınıflar için kullanılacak alanları da ekleyin: yani `Circle` sınıfında sadece `radius`, Rectangle sınıfında ise `length` ve `width` alanları olması gerekir. Son olarak ata sınıfta soyut tanımladığınız metodu geçersiz kılarak o tür için olması gereken kodu yazın. Yukarıdaki örneği buna çevirecek olursak

```java

abstract class Figure{
    abstract double area();
}

class Circle extends Figure{
    final double radius;
  Circle(double radius) {
      this.radius = radius;
  }

  @Override
  double area() {
      return Math.PI * (radius * radius);
  }
}

class Rectangle extends Figure{
  final double length;
  final double width;

  Rectangle(double length, double width) {
      this.length = length;
      this.width = width;
  }

  @Override
  double area() {
      return length * width;
  }
}
```

- Burda herhangi bir switch bloğu kullanmamıza gerek yoktur. Her tür kendi sınıfında tanımalndığı için kendisini ilgilendirmeyen alanlarla ilgilenmek zorunda değğildir. Bütün alanları final tanımlayabilmek de mümkün olmmuştur.
- Bu tür hiyerarşilerinin başka bir avantajı da alt türlerin birbiri arasındaki hiyerarşik düzeni kurabilmektir. Örneğin buraya bir kare türü eklemek istersek, kareyi dikdörtgenden kalıtıp özel bir dikdörtgen olduğunu aşağıdaki gibi hiyerarşik düzene yansıtabiliriz.
```java
// özel dikdörtgen class'ı
class Square extends Rectangle{
    Square(double side) {
        super(side, side);
    }
}
```
- Özetle, işaretli sınıfların kullanımı birçok durum için uygunsuzdur. Eğer böyle bir sınıf yazmayı aklınızdan geçiriyorsanız, işaret alanını kaldırıp bunu bir tür hiyerarşisine çevirip çeviremeyeceğimizi düşünmemiz gerekiyor. Böyle sınıflarla karşılaşırsak da tür hiyerarşisi önemli olur

## Static Üye Sınıflar - 24


