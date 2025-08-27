# Hibernate N+1 Problem

- İki tane tablo var, user ve posts. Her user’ın bir post listesi var. Sadece kullanıcıyı çekmek istiyoruz ama JPA arkada gidip postları da mı getirecek yoksa sadece user tablosunu mu sorgulayacak? Bu davranışı belirleyen şey fetch stratejisi.
- Bu konu farklı şekilde performans problemleri oluşturabilir. Bazen gereksiz yere fazla sorgu atılmış olacaktır. İhtiyaç olmayan verileri çekmemiz anlamına gelecektir.

## Fetch type nedir?

- İlişkili verilerin ne zaman yükleneceğini belirleyen mekanizmadır. Yani bir entity çağırıldığında ilişkili tabloların veirleri hemen mi gelmeli yoksa ihtiyaç oldğunda mı çekilsin buna karar veren mekanizmadır. Lazy ve eager olarak 2 şekilde hibernate’de bulunuyor

## Lazy fetch

- Lazy : JPA’nın çoğu ilişki için varsayılan tercihidir. Mantığı çok basit şekilde “ilişkili veriye ihtiyaç duymadan boşuna sorgu atma”

```java
@Entity
public class User {
    @Id
    private Long id;

    private String name;

    @OneToMany(mappedBy = "user", fetch = FetchType.LAZY)
    private List<Post> posts;
}
```

Burada userRepository.findById(1) denildiğinde sadece user tablosuna sorgu atılır. Kullanıcının postlarına erişilemez, ama sonrasında user.getPosts() şeklinde metot çağırıldığında hibernate gidip o kullanıcının postlarının ayrı bir sorgu ile çeker.

- Avantaj : kullanılmayan veriler boşa çekilmez.
- Dezavantaj : Eğer bir liste üzerine ço kfazla işlem yapıyorsak her entity için ayrı query atıldığı için performans düşebilir. (N+1 problem burda başlıyor)

## Eager Fetch

- Bu strateji daha açgözlü şekilde davranır, entity her çağırıldğında ilişkili verileri de yanında getirir.

  ```java
  @OneToMany(mappedBy = "user", fetch = FetchType.EAGER)
  private List<Post> posts;
  ```

userRepository.findById(1) dendiği zaman burda hem kullanıcı hem de onun postlarını joinleyerek çeker.

- Avantaj : ilişkili verilere kesin ihtiyacın varsa tek sorguda hepsini getirir.
- Dezavantaj : kullanılmayan verileri de getirir. Veri çok fazlaysa bellek şişer

## İlişki türlerine göre varsayılan fetch :

- @OneToMany → LAZY
- @ManyToMany → LAZY
- @ManyToOne → EAGER
- @OneToOne → EAGER

Yani “çoktan-bire” ve “bire-bir” ilişkiler genelde hemen yüklenir, “birden-çoğa” ve “çoktan-çoğa” ilişlkiler ise ihtiyaç olunca yüklenir.

## N+1 Problemi :

- hibernate ile çalışırken sık yaşanılan problemdir. Diyelim ki elimizde bir user tablosu var ve içinde 10 tane kullanıcı kayırlı. Kullanıcıları çekmek için `select * from user` sorgusu attık diyelim. Buraya kadar sorun yok, tek sorgu çalışıyor ve 10 kullanıcı geliyor. Burdan sonra her kullanıcı için ‘bu kişinin postlarını da getir’ dendiğinde hibernate aşağıdaki gibi çalışıyor.

  user 1 → select\*from post where user_id = 1

  user 2 → select \* from post where user_id = 2

  ….

  user 10 → select \* from post where user_id = 10

- Yani 10 kullanıcı için 10 ayrı sorgu daha atılıyor. Toplamda da; ilk başta kullanıcıları çektik(1 sorgu), sonra her kullanıcı için ayrı post sorgusu atıldı(10 sorgu). Toplamda 11 sorgu atılmış oluyor. Burda da N+1 problemi ortaya çıkıyor.
- 10 kişi örneğinde sorun yaratacak bir problem olmayabilir ama 10 bin user için 10001 tane istek atılacak demektir. Bu da performans sorunları oluşturacaktır.

## Çözüm Yöntemleri

- JOIN FETCH kullanmak : JPQL

  ```javascript
  @Query("SELECT u FROM User u JOIN FETCH u.posts")
  List<User> findAllWithPosts();
  ```

- @EntityGraph kullanmak

  ```javascript
  @EntityGraph(attributePaths = "posts")
  @Query("SELECT u FROM User u")
  List<User> findAllWithPosts();
  ```

## Sonuç

- Varsayılan fetch türleirni bilmek, projenin davranışını anlamak için çok önemli
- N+1 problemi performansın en büyük düşmanlarından biri
- Tekrar eden sorgulardan kaçın
- Gereksiz eager yüklemeden uzak dur
- Kodun amacını net yansıtan repository metodları yaz

[https://medium.com/@berkan.akbulut/jpada-eager-lazy-ve-n-1-problemi-performans-tuzaklarını-önlemek-d934d77d0590](https://medium.com/@berkan.akbulut/jpada-eager-lazy-ve-n-1-problemi-performans-tuzaklar%C4%B1n%C4%B1-%C3%B6nlemek-d934d77d0590)
