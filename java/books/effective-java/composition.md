# Use composition for inheritance

- Metot çağırmanın aksine, kalıtım yapmak sarmalamayı (encapsulation) ihlal eder. Başka bir deyişle, bir çocuk sınıf işlevini gerçekleştirebilmek için ata sınıfın gerçekleştirim detaylarına bağımlıdır. Ata sınıfın içeriği zaman içerisinde yeni sürümlerle birlikte değişebilir, ve bu durumda çocuk sınıfın kodu hiç değişmemiş olsa bile çalışmayabilir. Sonuç olarak, eğer ata sınıf özel olarak kalıtılmak için tasarlanmamışsa, çocuk sınıf değişikliğe ihtiyacı olmasa bile ata sınıfla birlikte gelen değişikliklere uyum sağlayacak şekilde değiştirilmelidir.
- Kalıtım kullanmak sadece çocuk sınıf ata sınıfın bir alt türü ise mantıklıdır. Eğer B sınıfının A sınıfını kalıtması gerektiğini düşünüyorsanız, bunu yapmadan önce mutlaka “B gerçekten bir A mı?” sorusunu sorun. Eğer bu soruya gerçekten evet cevabını veremiyorsanız o zaman B sınıfının A’yı kalıtması yanlış olacaktır. Eğer cevap hayır ise, B sınıfı A türünde private bir referans tanımlamalı ve dışarıya daha küçük ve basit bir API sunmalıdır. Bu durumda A sınıfı B’nin gerekli bir parçası değil gerçekleştirim detayı olarak kalacaktır.
- Kalıtım kullanmaya karar vermeden önce kendinize sormanız gereken son soru da şudur: Kalıtmak istediğim sınıfta bir API hatası var mı? Var ise benim aynı hatayı kendi sınıfıma yansıtmam kabul edilebilir mi? Kalıtım kullandığınızda ata sınıfın API’ını da kalıtmış oluyorsunuz yani hatalar varsa onlar da beraberinde gelecektir, ancak kompozisyon kullanırsanız diğer sınıfın bütün detaylarını saklayıp sıfırdan bir API oluşturabilirsiniz.
- Özetleyecek olursak, kalıtım güçlüdür ancak sarmalamayı (encapsulation) ihlal ettiği için problemlidir. Sadece çocuk sınıf ile ata sınıf arasında bir tür-alt tür ilişkisi varsa kullanılması uygun olur. Ancak o zaman bile, eğer ata sınıf ile çocuk sınıf farklı paketlerde tanımlanmışsa ve ata sınıf kalıtılmak için tasarlanmamışsa yazılımda kırılganlığa yol açar. Bu kırılganlığı önlemek için kompozisyon ve iletim sınıfı kullanmayı tercih edin. Bu şekilde tasarlanan sınıflar hem daha esnek hem de daha sağlam olacaktır.

## Item 5

- Util sınıfları kullanılırsa private constructor oluşturmak gerekir. Private oluşturulan constructor da bu sınıfı değiştirmeyin anlamında olacaktır
- Utility sınıfları initialize edilmemelidir

## Item 6

- Her seferinde nesne oluşturmak garbace collector için problem yaratabilir. Sürekli nesne oluşturulup silinir
- Eager singleton gibi ekleyip sürekli nesne oluşturma probleminden kurtulabiliriz.

```
    private static final Pattern ROMAN = Pattern.compile("asdfff sdadsa");
```

```
    static boolean isNonNumeral(String value){
        return ROMAN.matcher(value).matches();
    }
```

- Wrapper sınıflar kullanıldığında çok fazla şekilde zaman artıyor. Integer kullanıdıldığında her seferinde Integer.valueOf() şeklinde primitive tipe çeviriliyor

```
    ArrayList<Integer> integers = new ArrayList<>();
    for (int i = 0; i < 10; i++) {
        integers.add(Integer.valueOf(i));
    }
```

##  Item 7

- Garbage collection : heap memory oluşturulan memory'ler ile olur. Belirli zamanlarda bu memory'nin temizlenmesi lazım. Bunu yapan da garbace collector
- Stack : bir veri yapısı, son giren ilk çıkar. Bir sepet gibi düşünülebilir
- `System.gc()` -> ile garbage collector manuel olarak tetiklenir
- Statik olmayan dosyalarda garbage collection çalıştığında başlangıçtan daha az bi memory değeri oluşabilir. Yani bir obje statik oluşturulursa GC bunu temizleyemez, statik'lik tehlikeli olabilir

  ```
      public Object pop(){
          if (size == 0){
              throw new EmptyStackException();
          }
          // eğer çıkarabiliyosam sonuncu eleman harici veriyor
          // referans eski kalıyor. en üstteki eleman boş
          return elements[--size];
      }
  ```

  daha iyi olan versiyon:

```
    public Object popWell(){
        if (size == 0){
            throw new EmptyStackException();
        }

        Object result = elements[--size];
        elements[size] = null; // eski objeyi null'a set ettim

        return result;
    }

```

- Buffered read -> Bir connection açıldığında tekrardan kapatılması gerekiyor.
- Caching mekanizmasında da iyi bir yapı kurmak lazım yoksa memory problemleri yaşayabiliriz
- VisualVM kullanmak gerekiyor bkz. [VisualVM](https://https://visualvm.github.io/)

## Item 8

- Finalizers -> garbage collection ile beraber kullanılır. Tahmin edilemez, genellikle tehlikeli ve genellikle de gereksizdir, java 9 ile deprecated oldu yerini cleaners'a bıraktı. Bu da tahmin edilemez. Programı yavaşlatır kullanmaya gerek yok.

## Item 9

- Try with resource -> finally yazmaktan kurtuluyoruz
- Finally -> her zaman çalışır

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
- `equals()` kontratının son maddesine göre hiçbir nesne null değerine eşit değildir.

