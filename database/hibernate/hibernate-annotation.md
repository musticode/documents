# Annotations

- @Entity : Entity anotasyonu eklendiği sınıfın bir varlık olduğunu ifade eder. Bu anotasyonun yer aldığı sınıfların veritabanında mutlaka bir tablo karşılığı vardır.
- @Table(name = "table_names") : İlgili varlığa karşılık gelen tabloyu belirtmek için kullanılır.
- @Id : Birincil anahtarı niteler.
- @GeneratedValue : Birincil anahtarların değerleri için üretim stratejilerini belirtir. TABLE, SEQUENCE, IDENTITY, UUID, AUTO değerleri alabilir.
- @Column(name = "test_data_id") : Bir alanı database'deki sütun ile eşleştirmeyi sağlar. Kullanılabilecek özellikler : 
  - name -> alanın db'deki karşılık geldiği sütun adını belirtir
  - length -> alanın boyutunu belirtir. Default olarak 255
  - nullabl -> default değğeri true'dur. null olarak işaretlenmeyi anlatır
  - unique -> default false, unique olup olmamayı belirtir
  - insertable -> default true, false olduğu zamanda alanın db'de insert işlemi sırasında eklenmesine izin verilmediğini belirtir.
  - updatable -> default true, false olduğu zamanda update edilmeye izin verilmediğini belirtir.
  `@Column(name = "FIRST_NAME", nullable = false, length = 200) private String FirstName;`
- @Size : alanın minimum ve maximum değerlerini belirtmek için kullanılır. Column anotasyonunda yer alan length özelliklerinden farklı olarak validasyon yapmayı sağlar
  `@Size(min = 2, max = 200)`
- @Temporal : alanın date, time, timestamp olarka naısl saklanacağını belirtmek için kullanılır
```
    @Temporal(TemporalType.DATE)
    @Column(name = "DATE_OF_BIRTH")
    private Date DateOfBirth;
```
- @NotNull : Alanın null değer alamayacağını ifade eder. Notnull anotasyonu veritabanına gitmeden bu kontrolü sağlar, değerin null olduğu durumda veritabanına gitmeden hatayı gösterir. @Column anotasyonunun notnull özelliği ise tablo alanını notnull olarak işaretler. Fakat her defasında veritabanına gider, veritabanına gittikten sonra hatayı döner. Aslında veritabanı hatası oluşturur.
- @Transient : Veritabanına kaydedilmek istenmeyen alanları belirtmek için kullanılır. (Uygulama mantığı ile ilgili değişkenler, kaydedilmek istenmeyen hesaplamalar gibi.)
  
