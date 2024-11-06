# Serialization


## Prefer Alternatives To Java Serialization

- Java'nın built in serialization'ı problemlidir ve bir çok hataya sebep olmuştur.
- her biri 3 veya daha az nesne referansı içeren 201 HashSet örneğinden oluşan bir nesne grafiği oluşturur. Tüm grafik yalnızca 5.744 bayt kaplar ancak serileştirilmesi imkansızdır. Bunun nedeni, bir HashSet'i serileştirmenin tüm öğelerinin karma kodlarının hesaplanmasını gerektirmesidir. Kök HashSet'in iki öğesinin kendileri, 100 seviyeye kadar iki karma kümesine daha sahiptir. Bu, hashCode işlevinin 2^100 kez çağrılmasına neden olur. Bunun ekstra sinir bozucu kısmı, kodun tamamlanmaması dışında bir sorun belirtisi olmamasıdır.
- Bu yüzden Java'nın yerleşik serileştirmesinin birçok tuzağı olduğunu yeterince gösterdiğimizi düşünüyorum. Ne yapmalıyız? Bu sorunu çözmenin en iyi yolu, bundan tamamen kaçınmaktır. Bugün yazdığınız herhangi bir yeni kodda Java serileştirmesini kullanmak için iyi bir neden yoktur. Bunun yerine, başka bir veri aktarım biçimi kullanmalısınız. Bu sistemlerin çapraz platform olma avantajı vardır. Bu gösterimler, Java serileştirmesinden çok daha basit olma ve çok daha küçük bir kapsama sahip olma avantajına sahiptir. Bu, genellikle yalnızca verilere odaklandıkları için çok daha güvenli olmalarını sağlar. Bu formatların bazı örnekleri JSON, Protokol Arabellekleri (Protobuf) ve Avro'dur. Protokol Arabellekleri ve Avro, şema doğrulamasını kolaylaştırabilir ve uzaktan yordam çağrı sistemleri (RPC) için uzantılara sahip olsa da, yalnızca durumu aktarmaya gelince, Java serileştirmesinden çok daha basittir.
- Günümüzde hala serileştirme kullanan çok sayıda Java kodu kullanılmaktadır. Durum böyle olunca, bunun ortaya çıkarabileceği karmaşıklıkları ve sorunları anlamamız gerekir. Ayrıca, bu sorunlara ve bunların sahip olduğu patlama yarıçapına maruz kalmamızı sınırlamak için elimizdeki tüm araçları kullanmalıyız. Genel olarak, Java serileştirmesinden kaçınabiliyorsanız, kaçının. Yeni sistemlerde Java serileştirmesini hiç başlatmayın. Bunun yerine, sistemler arasında veri aktarmak için modern veri değişim biçimlerini kullanın.


## Implement Serializable With Great Caution

- Son olarak, iç (statik olmayan) sınıflar Serializable'ı uygulamamalıdır. Bunların uygulanma şekli, çevreleyen örneğine referansları depolayan ve çevreleyen kapsamdaki yerel değişkenlerin değerlerini depolayan sentetik alanlar kullanmaktır. Bunların tanımlanma şekli iyi tanımlanmamıştır ve bu nedenle kaçınılmalıdır. Ancak statik üye sınıflarında bu sorun yoktur.
- Özetle, Serializable arayüzünü doğru şekilde uygulamak tuzaklarla doludur. Sürümleme ve veri girişlerinin kısıtlandığı ortamınız üzerinde yüksek düzeyde bir kontrole sahip olmadığınız sürece yokuş yukarı bir mücadele içinde olacaksınız. Bu, kalıtım da eklendiğinde daha da zorlaşır.

## Consider Using a Custom Serialized Form

- Serileştirilmiş verileriniz için hangi formu alırsanız alın, geçici olarak işaretlenmemiş bir nesnenin her alanı defaultWriteObject yöntemi çağrıldığında serileştirilecektir. Bu, geçici olarak işaretlenebilecek her alanın serileştirilmesi gerektiği anlamına gelir. Buna türetilmiş alanlar, oluşturulmuş alanlar, önbellek değeri alanları veya o tek çalıştırmaya özgü bir şeye işaret eden alanlar (örneğin yerel bir dosya tanıtıcısı) dahildir. Bir alan geçici olmayan olarak işaretlenmeden önce, bunun sınıfın mantıksal modelinin bir parçası olduğuna kendinizi ikna edebilmelisiniz. Sınıf serileştirme yoluyla oluşturulduğunda geçici olarak işaretlenen alanların varsayılan değerlerine başlatılacağını unutmayın
- Özetle, Java'nın yerleşik serileştirmesini kullanırken, bir sınıfın varsayılan serileştirilmiş biçiminin uygun olup olmadığını göz önünde bulundurun. Mantıksal ve fiziksel gösterimlerinin aynı olup olmadığını belirleyerek uygun olup olmadığını belirleyebilirsiniz. Sınıfınızın serileştirilmiş biçimleri, sınıfınızın genel API'sinin yöntemleri kadar bir parçasıdır ve bu nedenle aynı derecede dikkate alınmalı ve planlanmalıdır.


## Write readObject methods defensively

- Özetlemek gerekirse, bir readObject yöntemi yazdığınız her zaman, hangi bayt akışı verildiğinden bağımsız olarak geçerli bir örnek üretmesi gereken genel bir oluşturucu yazdığınızı düşünün. Bayt akışının gerçek bir serileştirilmiş örneği temsil ettiğini varsaymayın. Bu maddedeki örnekler varsayılan serileştirilmiş formu kullanan bir sınıfla ilgili olsa da, ortaya çıkan sorunların tümü özel serileştirilmiş formlara sahip sınıflar için de geçerlidir. Özet biçiminde, bir readObject yöntemi yazma yönergeleri şunlardır:
  
  - Gizli kalması gereken nesne referans alanlarına sahip sınıflar için, bu tür bir alandaki her nesneyi savunmacı bir şekilde kopyalayın. Değiştirilemez sınıfların değiştirilebilir bileşenleri bu kategoriye girer.
  - Kontroller, savunma amaçlı kopyalamanın ardından yapılmalıdır.
  - Obje deserializasyondan sonra valide edilmelidir.
  - Override edilebilen methodları class'ın içinden direkt olarak ya da farklı şekilde çağırmayın


## For instance control, prefer enum types to readResolve

- Özetlemek gerekirse, mümkün olan her yerde örnek kontrol değişmezlerini zorlamak için enum türlerini kullanın. Bu mümkün değilse ve bir sınıfın hem serileştirilebilir hem de örnek kontrollü olması gerekiyorsa, bir readResolve yöntemi sağlamalı ve sınıfın tüm örnek alanlarının ilkel veya geçici olduğundan emin olmalısınız.


## Consider serialization proxies instead of serialized instances

- 85 ve 86. maddelerde belirtildiği ve bu bölüm boyunca tartışıldığı gibi, Serializable'ı uygulama kararı, sıradan oluşturucular yerine dil dışı bir mekanizma kullanılarak örneklerin oluşturulmasına izin verdiği için hata ve güvenlik sorunları olasılığını artırır. Ancak, bu riskleri büyük ölçüde azaltan bir teknik vardır. Bu teknik, serileştirme proxy deseni olarak bilinir.
- Serileştirme proxy deseni oldukça basittir. İlk olarak, çevreleyen sınıfın bir örneğinin mantıksal durumunu özlü bir şekilde temsil eden özel bir statik iç içe sınıf tasarlayın. Bu iç içe sınıf, çevreleyen sınıfın serileştirme proxy'si olarak bilinir. Parametre türü çevreleyen sınıf olan tek bir oluşturucuya sahip olmalıdır. Bu oluşturucu yalnızca verileri argümanından kopyalar: herhangi bir tutarlılık denetimi veya savunma kopyalaması yapması gerekmez. Tasarım gereği, serileştirme proxy'sinin varsayılan serileştirilmiş biçimi, çevreleyen sınıfın mükemmel serileştirilmiş biçimidir. Serializable'ı uygulamak için hem çevreleyen sınıf hem de serileştirme proxy'si beyan edilmelidir.
- Serileştirme proxy kalıbının iki sınırlaması vardır. Kullanıcıları tarafından genişletilebilen sınıflarla uyumlu değildir (Öğe 19). Ayrıca, nesne grafikleri dairesellikler içeren bazı sınıflarla uyumlu değildir: Böyle bir nesne üzerinde bir yöntemi, serileştirme proxy'sinin readResolve yöntemi içinden çağırmaya çalışırsanız, henüz nesneye sahip olmadığınız, yalnızca serileştirme proxy'sine sahip olduğunuz için bir ClassCastException alırsınız.
- Özetle, istemcileri tarafından genişletilemeyen bir sınıf üzerinde readObject veya writeObject yöntemi yazmak zorunda kaldığınızda serileştirme proxy desenini göz önünde bulundurun. Bu desen, önemsiz olmayan değişmezlere sahip nesneleri sağlam bir şekilde serileştirmenin belki de en kolay yoludur.