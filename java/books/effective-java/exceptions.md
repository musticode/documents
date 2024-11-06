# Exceptions

## Use exceptions for only exceptional conditions

- Exception'lar sadece istisnai durumlarda kullanılmalıdır. Diğer türlü noktalarda performans kayıplarıyla sonuçlanacaktır.
- İyi tasarlanmış bir API, istemcilerini sıradan kontrol akışı için 'istisnalar' kullanmaya zorlamamalıdır.
- Özetle, istisnalar istisnai koşullar için tasarlanmıştır. Bunları sıradan kontrol akışı için kullanmayın ve başkalarını bunu yapmaya zorlayan API'ler yazmayın.

## Use checked exceptions for recoverable conditions and runtime exceptions for programming errors

- Checked ya da unchecked exception kullanmakta karar noktası şudur : caller'ın kontrol edebileceği hatalar için checked exception'lar kullanmak gerekir.
- Kontrol edilmiş bir istisna atarak, çağıranı istisnayı bir catch cümlesinde işlemeye veya dışarıya yaymaya zorlarsınız. Bir yöntemin atmak üzere bildirdiği her kontrol edilmiş istisna, bu nedenle API kullanıcısına ilişkili koşulun yöntemi çağırmanın olası bir sonucu olduğuna dair güçlü bir göstergedir.
- herhangi bir yeni Error alt sınıfı uygulamamak en iyisidir. Bu nedenle, uyguladığınız tüm denetlenmemiş atılabilirler RuntimeException'ı (doğrudan veya dolaylı olarak) alt sınıfa ayırmalıdır. Sadece Error alt sınıflarını tanımlamamalısınız, aynı zamanda AssertionError hariç, bunları atmamalısınız.
  Exception, RuntimeException veya Error'ın bir alt sınıfı olmayan bir atılabilir tanımlamak mümkündür. JLS bu tür atılabilirleri doğrudan ele almaz, ancak örtük olarak bunların sıradan denetlenmiş istisnalar gibi davrandığını belirtir (bunlar Exception'ın alt sınıflarıdır, ancak RuntimeException değildir). Peki böyle bir canavarı ne zaman kullanmalısınız? Tek kelimeyle, asla. Sıradan denetlenmiş istisnalara göre hiçbir avantajları yoktur ve yalnızca API'nizin kullanıcısını şaşırtmaya yararlar.
- Özetlemek gerekirse, kurtarılabilir koşullar için `checked exceptions` ve programlama hataları için `unchecked exceptions` fırlatın. Şüpheniz olduğunda, `unchecked exceptions` fırlatın. Ne `checked exceptions` ne de `runtime exceptions` olan hiçbir fırlatılabilir tanımlamayın. Kurtarmaya yardımcı olmak için checked exceptions'larınızda yöntemler sağlayın.

## Avoid unnecessary use of checked exceptions

- Kontrol edilen bir istisnayı ortadan kaldırmanın en kolay yolu, istenen sonuç türünün bir opsiyonelini döndürmektir (Madde 55). Kontrol edilen bir istisna atmak yerine, yöntem basitçe boş bir opsiyonel döndürür. Bu tekniğin dezavantajı, yöntemin istenen hesaplamayı gerçekleştirememe durumunu ayrıntılarıyla açıklayan herhangi bir ek bilgi döndürememesidir.
- Özetle, seyrek kullanıldığında, denetlenen istisnalar programların güvenilirliğini artırabilir; aşırı kullanıldığında, API'lerin kullanımını zorlaştırır. Çağrı yapanlar arızalardan kurtaramayacaksa, denetlenmeyen istisnalar atın. Kurtarma mümkün olabilirse ve çağrı yapanları istisnai koşulları işlemeye zorlamak istiyorsanız, önce isteğe bağlı bir değer döndürmeyi düşünün. Yalnızca bu, arıza durumunda yeterli bilgi sağlamayacaksa denetlenen bir istisna atmalısınız.

## Favor the use of standard exceptions

- Standart istisnaları yeniden kullanmanın birçok faydası vardır. Bunların en önemlisi, programcıların zaten aşina olduğu yerleşik kurallara uyduğu için API'nizi öğrenmeyi ve kullanmayı kolaylaştırmasıdır. İkinci olarak, API'nizi kullanan programların, aşina olunmayan istisnalarla dolu olmadıkları için okunması daha kolaydır. Son olarak (ve en az önemlisi), daha az istisna sınıfı daha küçük bir bellek alanı ve sınıfları yüklemek için harcanan daha az zaman anlamına gelir.
- İyi bir neden olmadan kendi exception class'ımızı yazmamıza gerek yoktur.

## Throw exceptions appropriate to the abstraction

- İstisna çevirisi, alt katmanlardan gelen istisnaların düşüncesizce yayılmasından daha üstün olsa da, aşırı kullanılmamalıdır. Mümkün olduğunda, alt katmanlardan gelen istisnalarla başa çıkmanın en iyi yolu, alt düzey yöntemlerin başarılı olmasını sağlayarak bunlardan kaçınmaktır. Bazen bunu, daha üst düzey yöntemin parametrelerinin geçerliliğini alt katmanlara geçirmeden önce kontrol ederek yapabilirsiniz.
- Özetle, alt katmanlardan gelen istisnaları önlemek veya işlemek mümkün değilse, alt düzey yöntemin tüm istisnalarının üst düzeye uygun olmasını garantilemesi haricinde, istisna çevirisini kullanın. "Zincirleme" her iki dünyanın da en iyisini sunar: uygun bir üst düzey istisna atmanıza izin verirken, arıza analizinin altında yatan nedeni yakalamanızı sağlar

## Document all exceptions thrown by each method

- Her zaman kontrol edilen exception'ları ayrı ayrı bildirmek ve hangi koşullar altında atıldığını Javadoc @throws etiketi kullanarak kesin olarak belgelendirmek gerekir.
- Dil, programcıların bir yöntemin fırlatabileceği denetlenmemiş istisnaları bildirmesini gerektirmese de, bunları denetlenmiş istisnalar kadar dikkatli bir şekilde belgelemek akıllıca olacaktır. Denetlenmemiş istisnalar genellikle programlama hatalarını temsil eder (Madde 70) ve programcıları yapabilecekleri tüm hatalarla tanıştırmak, bu hataları yapmaktan kaçınmalarına yardımcı olur. Bir yöntemin fırlatabileceği denetlenmemiş istisnaların iyi belgelenmiş bir listesi, başarılı bir şekilde yürütülmesi için ön koşulları etkili bir şekilde açıklar.
- Bir metodun fırlatabileceği her exception'u belgelemek için Javadoc @throws etiketini kullanın. Ancak unchecked exception'larda throws anahtarını kullanmamak gerekir. API'yi kullanan programcıların hangi exception'ların kontrol edildiğini ve hangilerinin kontrol edilmediğini bilmeleri önemlidir, çünkü programcıların sorumlulukları bu iki durumda farklıdır. Method bildiriminde karşılık gelen bir throws ifadesi olmadan Javadoc @throws etiketi tarafından oluşturulan belgeler, programcıya bir istisnanın kontrol edilmediğine dair güçlü bir görsel ipucu sağlar.
- Bir sınıftaki bir çok method tarafından aynı sebepten öütürü bir exception fırlatılırsa, exception'u her method için ayrı ayrı belgelemek yerine sınıfın belge yorumunda belgeleyebiliriz. Yaygın örnek olarak NullPointerException gösterilebilir. Döküman kısmında 'Bu sınıftaki tüm method'lar herhangi bir parametre null geçilirse NullPointerException fırlatır' şeklinde ifade edilmelidir.
- Özetle, yazdığımız her method için fırlatılan exception'u belgelemek önemlidir. Bu yöntem hem checked hem de unchecked exception'lar için geçerlidir. @throws tag'i ile bunu javadoc yorumlarında belirtebiliriz. Method'larınızın atabileceği istisnaları belgelemezseniz, başkalarının sınıflarınızı ve arayüzlerinizi etkili bir şekilde kullanması zor veya imkansız olacaktır.


## Include failure-capture information in detail messages

- Bir program yakalanamayan bir exception'dan kaynaklı başarısız olursa sistem otomatik olarak exception'un stack trace'ini yazdırır. Bu stack trace exception'un toString methodundaki sonuçtur. Bu da hatayla alakalı görülebilecek tek bilgidir. Bundan dolayı exception'un toString methodunda hatanın nedeni hakkında mümkün olduğunca fazla bilgi döndürmesi kritik öneme sahiptir. Her exception'un ayrıntılı mesajı sonraki analiz için hatayı yakalamalıdır.(capture the failure)
- Bir hatayı yakalamak için, bir istisnanın ayrıntı mesajı, istisnaya katkıda bulunan tüm parametrelerin ve alanların değerlerini içermelidir. Örneğin, bir IndexOutOfBoundsException'ın ayrıntı mesajı, alt sınırı, üst sınırı ve sınırlar arasında yer almayan dizin değerini içermelidir.
- Bir uyarı, güvenlik açısından hassas bilgilerle ilgilidir. Yığın izleri, yazılım sorunlarını teşhis etme ve düzeltme sürecinde birçok kişi tarafından görülebileceğinden, ayrıntılı mesajlara parolaları, şifreleme anahtarlarını ve benzerlerini dahil etmeyin.


## Strive for failure atomicity

- Bir nesne bir exception fırlattıktan sonra, hata bir işlem gerçekleştirilirken meydana gelse bile nesnenin hala iyi tanımlanmış, kullanılabilir durumda olması istenir. Bu özellikle caller'ın kurtarması bneklenen checked exception'lar için geçerlidir. Genel olarak, başarısız method çağrısı nesneyi çağrıdan önceki durumunda bırakmalıdır. Bu özelliğe `failure-atomic` denir
- Buna erişmenin birkaç yolu vardır: En basiti immutable nesneler tasarlamaktır. Bir nesne değişmezse failure atomicity serbesttir. Bir işlem başarısız olursa yeni bir nesnenin oluşturulmasını engelleyebililr, ancak var olan bir nesneyi asla tutarsız durumda bırakmaz. Çünkü her nesnenin durumu oluşturulduğunda tutarlıdır ve sonrasında değiştirlemez.
- Değiştirilebilir nesneler üzerinde çalışan method'lar için failure atomicity elde etmenin en iyi yolu işlem gerçekleştirilmeden önce methodun başında parametrenin geçerliliğini kontrol etmektir. Örnek olarak:
```java
public Object pop() {
    if (size == 0)
        throw new EmptyStackException();
    Object result = elements[--size];
    elements[size] = null; // Eliminate obsolete reference
    return result;
}
```
Bu örnek için, başlangıçta boyut denetimi ortadan kaldırıldığı durumda method boş bir stack'ten bir öğeyi çıkarmaya çalşıtığında yine bir exception fırlatırdı. Ancak boyut alanını negatif(tutarsız) bir durumda bırakırdı. Bu da nesne üzerinde gelecekteki tüm method çağrılarının başarısız olmasına neden olurdu. Ayrıca, pop yöntemi tarafından fırlatılan ArrayIndexOutOf- BoundsException soyutlama için uygunsuz olurdu (Öğe 73).

- Failure atomicity genellikle arzu eilid olsa da her zaman elde edilebilir değildir. Örnek olarak, iki thread aynı nesneyi uygun senkronizasyon olmadan eş zamanlı olarak değiştirmeye çalışırsa nesne tutarsız bir durumda bırakılabilir. Bu nedenle bir nesnenin ConcurrentModificationException yakaladıktan sonra hala kullanılabilir olduğunu varsaymak yanlış olur. Hatalar kurtarılamaz bu nedenle ASsertionError atarken failure atomicity'yi korumaya çalışmaya gerek bile yoktur.
- Failure atomicity mümkün olduğu zamanlarda da her zaman arzu edilen durum olmayabilir. Bazı işlemler için maliyet veya karmaşıklığı önemli ölçüde artıracaktır. Bununla birlikte sorunun farkına vardığımızda failure atomicity'ye ulaşmak genellikle hem ücretsiz hem de kolaydır.
- Özetle bir kural olarak, bir metodun spesifikasyonunun parçası olan herhangi bir exception, nesneyi metod çağırısından önceki haliyle aynı durumda bırakmalıdır. Bu kural ihlal edildiğinde API belgeneleri nesnein hangi durumda bırakılacağını açıkça belirtmelidir.

## Don't ignore exceptions

- Bir API tasarımcıları bir exception fırlatmak için bir metod bildirdiğinde bir şey söylemeye çalışırlar. Bunu görmezden gelmemek gerekir.
```java
try {
    //..
} catch (RuntimeException e) {
    
}
```
Boş bir catch bloğu exception'ların amacını bozar. Bu da bizi istisnai durumlarla başa çıkmaya zorlar. Bir exception'u görmezden gelmek bir yangın alarmını görmezden gelmeye benzer. Sonuçlar felaket olabilir, eğer boş bir catch bloğu görürsek kafamızda alarm oluşmalıdır.

- Bazen bir exception'u görmezden gelmemiz gerekebilir. FileInputStream'i kapatırken bu uygun olabilir. Dosya durumu değiştirmedik, bu nedenle herhangi bir kurtarma işlemi yapmaya gerek yok ve dosyadan ihtiyaç duyulan ifadeleri de okuduk. Bu exception'lar sık sık meydana gelirse bunlar log'lamak gerekebilir. Eğer bir exception'u görmezden gelmek istersek bunun nedenini catch bloğu içinde yorum olarak ifade etmek doğru olacaktır.
- Bu maddedeki tavsiye, işaretli ve işaretsiz istisnalar için eşit şekilde geçerlidir. Bir istisna öngörülebilir bir istisnai durumu veya bir programlama hatasını temsil etsin, onu boş bir yakalama bloğuyla görmezden gelmek, hata karşısında sessizce devam eden bir programla sonuçlanacaktır. Program daha sonra gelecekte keyfi bir zamanda, kodun sorunun kaynağıyla görünürde hiçbir ilgisi olmayan bir noktasında başarısız olabilir. Bir istisnayı düzgün bir şekilde ele almak, başarısızlığı tamamen önleyebilir. Bir istisnanın dışarıya yayılmasına izin vermek bile en azından programın hızla başarısız olmasına neden olabilir ve başarısızlığı ayıklamaya yardımcı olacak bilgileri koruyabilir.


