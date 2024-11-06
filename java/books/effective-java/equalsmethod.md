## Item 10 - Obey the general contract when overriding equals

- equals methodunu override ederken yanlış bir uygulamada ciddi sorunlar ortaya çıkabilir. Bu problemlerden kaçınmanın en kolay yolu override etmemektir. Bu tarz problemlerden kaçınmanın yolu override etmemektir
- Peki o zaman `equals()` methodunu ne zaman overrride etmemiz gerekiyor? Basit olarak, nesnelerin arasında mantıksal eşitlik uygulanabilen sınıflar için `equals()` override edilebilr
- Bu tür sınıflar genellikle `Date`, `Integer` gibi değer sınıflarıdır.
- Eğer bir programcı equals methodunu kullanarak ik Integer nesnesini karşılaştırıyorsa o iki nesnenin aynı nesne olup olmadığından ziyade aynı değeri taşıyıp taşımadığını test ediyordur
- Bu durumlarda equals methodunu override etmek sadece programcının beklentisini karşılamakla kalmaz aynı zamanda bu nesnelerin `Map` için anahtar(key) veya `Set`elemanı olarak doğru biçimde kullanılabilmesini sağlar

#### Kurallar :

- **Dönüşlülük** : Bir nesne kendisine eşit olmalıdır. `x.equals(x)`
- **Tutarlılık** : Eğer iki nesne birbirine eşitse, bu nesnelerden en az birisi değişmediği sürece hep eşit kalmalıdırlar. Başka bir deyişle, kararsız(mutable) nesneler değişik zamanlarda değişik nesnelere eşit olabilirken, kararlı(immutable) nesneler olamazlar. Bu yüzden bir sınıf yazarken kararlı olup olmaması gerekdiğini iyi düşünmek gerekir.

  - Bir sınıf kararlı da kararsız da olsa , equals metodunun sonucu güvenililr olmayan kaynaklara asla bağlı olmamalıdır. Bu kural ihlal edildiğinde tutarlılık prensibini sağlamak zorlaşır. Örnek olarak java.net.URL sınıfındaki equals metodu, url ile ilişkili olan ip dareslerinin dikkate alarak bir sonuç üretir.
  - Bu IP adreslerini de üretmek ağ bağlantısı gerektirebilir ve üstelik zaman içerisinde de aynı sonucu üreteceğinin bir garantisi yoktur. Maalesef bu durum URL sınıfının equals() methodunun tutarsız çalışmasına sebep olmaktadır.
  - Çok ekstra durum olmadıkça equals() methodu dış kaynaklara bağımlı olmadan her zaman aynı sonucu üretebilecek bir durumda olmalıdır.
- **Null değerler**
- `equals()` sözleşmesinin son maddesine göre hiçbnir nesne null değerine eşit değildir. `o.equals(null)` gibi bir metod çağrısından yanlışlıkla true döndürmek zor bir ihtimal gibi görünse de NullPointerException fırlatmak gibi bir hata yapabiliriz. Birçok sınıf bu hatayı önlemek için açık bir biçimde null testi yapmaktadır.

```java
@Override
public boolean equals(Object o) {
    if (o == null)
        return false;
}
```

Bu testi yapmak gereksizdir. Geçilen argümanı eşitlik için kontrol etmek istediğiimzde ilk başta doğru türe döndürmek gerekir ki o nesnenin variable'larına erişebilelim. `instanceof` operatörü kullanarak doğru türde bir nesnenin geçildiğinden emin olunması gerekiyor.

```java
@Override
public boolean equals(Object o) {
    if (!(o instanceof MyType))
        return false;
    MyType mt = (MyType) o;
}
```

Eğer `instaceof` ile tür kontrolü yapmasaydık cast işlemi sırasında `ClassCastException` ile sonuçlanırdı, bu da equals sözleşmesin aykırı olurdu. `instanceof` operatörü eğer solundaki değer null ise false döndürecek şekilde tasarlanmıştır. Bundan dolayı sağındaki tür değerinin bir önemi yoktur. Dolayısıyla metoda null geçildiği taktirde zaten false sonuç üretecektir. Fazladan bir null kontrolü yapmaya gerek yoktur.

Sonuç olarak;

1. `==` operatörünü kullanarak geçilen argümanın bu nesneye bir referans olup olmadığını kontrol edin.  Eğer öyleyse true döndürün. Bu iyileştirme, eğer karşılaştırma işlemi pahalı bir işlemse performans artışı sağlayabilir.
2. `instaneof` ile doğru türe çevirmek gerekiyor. Eğer aynı türde değilse false döndürmek gerekli.
3. Eşitlik kontrolü yaparken sınıf içerisindeki bütün anlamlı alanları karşılaştırın. Eğer bütün alanlar eşitse o zaman true, değilse false döndürün.
4. Transitive özelliği sağlamak için class'ı inherit etmek yerine has-a ilişkisiyle, class'ın içine variable olarak verdiğimizde sorun olmayacaktır

```java
class ColorPoint {
  private final Point point;

  public ColorPoint(int x, int y, Color color) {
    point = new Point(x, y);
    this.color = Objects.requireNonNull(color);
  }
}
```

Bazı türler null alabilir. Bu durumda NullPointerException hatasını engellemek adına `Objects.equals(Object, Object)` kullanmak gerekiyor. equals metodunun performansa etkileri değişebilir, en iyi perfomans için farklı olması ve karşılaştırılması kolay olan alanları seçmek gerekiyor. Nesnenin o anki mantıksal durumunu yansıtmayan, Lock gibi senkronizasyon işlemleri için kullanılan alanları karşılaştırmamak gerekiyor. Sınıfın içinde başka alanları kullanarak türetilen alanlar varsa da bunları kullanmak çok doğru olmaz.


Özet olarak, `equals()` metodunu mecbur değilseniz geçersiz kılmayın. Çoğu durumda, `Object` sınıfından gelen davranış sizin ihtiyacınızı karşılayacaktır.** Kendiniz yazıyorsanız da, yukarıda incelenen sözleşmenin bütün kurallarını sağladığınıza emin olun.
