
# OCPP 1.6 - Operations Initiated by Charge Point

**Request and responses :** https://github.com/gennadiygnezdilov/ocpp-1.6J-example-request-response/tree/main

### Authorize

- Bir elektrikli aracın sahibi şarj işlemini başlatabilmeden veya durdurabilmeden önce Şarj Noktasının işlemi yetkilendirmesi gerekir. Şarj Noktası yalnızca yetkilendirmeden sonra enerji sağlamalıdır. Bir İşlemi durdururken Şarj Noktası yalnızca işlemi durdurmak için kullanılan tanımlayıcı, işlemi başlatan tanımlayıcıdan farklı olduğunda bir Authorize.req göndermelidir
- Authorize.req SADECE şarj etmek için bir tanımlayıcının yetkilendirilmesi amacıyla KULLANILMALIDIR.
- Bir Şarj Noktası, Yerel Yetkilendirme Listesi'nde açıklandığı gibi, Merkezi Sistemi dahil etmeden tanımlayıcıyı yerel olarak yetkilendirebilir. Kullanıcı tarafından sunulan bir idTag, Yerel Yetkilendirme Listesi'nde veya Yetkilendirme Önbelleğinde mevcut değilse, Şarj Noktası yetkilendirme talebinde bulunmak için Merkezi Sisteme bir Authorize.req PDU göndermelidir. idTag, Yerel Yetkilendirme Listesi'nde veya Yetkilendirme Önbelleğinde mevcutsa, Şarj Noktası Merkezi Sisteme bir Authorize.req PDU gönderebilir
- Bir Authorize.req PDU alındığında, Merkezi Sistem bir Authorize.conf PDU ile yanıt vermelidir. Bu yanıt PDU'su, idTag'in Merkezi Sistem tarafından kabul edilip edilmediğini göstermelidir. Merkezi Sistem idTag'i kabul ederse, yanıt PDU'su bir parentIdTag içerebilir ve kabulü veya reddetme nedenini belirten bir yetkilendirme durumu değeri içermelidir.
- Charge Point bir Yetkilendirme Önbelleği uyguladıysa, bir Authorize.conf PDU alındığında Charge Point, idTag Yerel Yetkilendirme Listesinde değilse, Yetkilendirme Önbelleği altında açıklandığı gibi yanıttaki IdTagInfo değeriyle önbellek girişini GÜNCELLEYECEKTİR.

Authorize request: CP -> CS

```json
[
    2,
    "01221201194032",
    "Authorize",
    {
        "idTag": "D0431F35"
    }
]
```

Authorize response: CS -> CP

```json
[
    3,
    "01221201194032",
    {
        "idTagInfo": {
            "status ": " Accepted"
        }
    }
]
```

### BootNotification

- Başlatma sonrasında, bir Şarj Noktası, yapılandırması hakkında bilgi (örneğin sürüm, satıcı, vb.) içeren bir talebi Merkezi Sisteme göndermelidir. Merkezi Sistem, Şarj Noktasını kabul edip etmeyeceğini belirtmek için yanıt vermelidir.
- Şarj Noktası her önyükleme veya yeniden başlatmada bir BootNotification.req PDU göndermelidir. Fiziksel güç açma/yeniden başlatma ile Merkezi Sistemin Kabul Edildi veya Beklemede döndürdüğü bir BootNotification'ın başarılı bir şekilde tamamlanması arasında Şarj Noktası Merkezi Sisteme başka bir istek GÖNDERMEMELİDİR. Bu, Şarj Noktasında daha önce hala mevcut olan önbelleğe alınmış mesajları da içerir.
- Merkezi Sistem BootNotification.conf dosyasıyla Kabul Edildi durumuyla yanıt verdiğinde, Şarj Noktası kalp atışı aralığını yanıt PDU'sundan gelen aralığa göre ayarlayacaktır ve dahili saatini sağlanan Merkezi Sistemin geçerli saatiyle senkronize etmesi ÖNERİLİR. Merkezi Sistem Kabul Edildi dışında bir değer döndürürse, aralık alanının değeri bir sonraki BootNotification isteğini göndermeden önce gereken minimum bekleme süresini gösterir. Bu aralık değeri sıfırsa, Şarj Noktası kendi başına bir bekleme aralığı seçer ve bu şekilde Merkezi Sistemin isteklerle dolmasını önler. Bir Şarj Noktası, TriggerMessage.req ile bunu yapması istenmediği sürece daha erken bir BootNotification.req göndermemelidir.
- Merkezi Sistem 'Reddedildi' durumunu döndürürse, Şarj Noktası yukarıda belirtilen yeniden deneme aralığı sona erene kadar Merkezi Sisteme herhangi bir OCPP mesajı GÖNDERMEMELİDİR. Bu aralıkta Şarj Noktasına Merkezi Sistemden ulaşılamayabilir. Örneğin iletişim kanalını kapatabilir veya iletişim donanımını kapatabilir. Ayrıca Merkezi Sistem, örneğin sistem kaynaklarını serbest bırakmak için iletişim kanalını kapatabilir. Reddedilmişken, Şarj Noktası Merkezi Sistem tarafından başlatılan herhangi bir mesaja yanıt VERMEMELİDİR. Merkezi Sistem herhangi birini başlatmamalıdır.
- Merkezi Sistem ayrıca, Şarj Noktası'nı kabul etmeden önce Şarj Noktası'ndaki belirli bilgileri almak veya ayarlamak istediğini belirtmek için Bekleyen kayıt durumunu da döndürebilir. Merkezi Sistem Bekleme durumunu döndürürse, iletişim kanalı Şarj Noktası veya Merkezi Sistem tarafından KAPATILMAMALIDIR. Merkezi Sistem, Şarj Noktası'ndan bilgi almak veya yapılandırmasını değiştirmek için istek mesajları gönderebilir. Şarj Noktası bu mesajlara YANIT VERMELİDİR. Şarj Noktası, Merkezi Sistem tarafından bir TriggerMessage.req isteği ile talimat verilmediği sürece Merkezi Sistem'e istek mesajları GÖNDERMEMELİDİR.


BootNotification request: 

```json
[
    2,
    "15455",
    "BootNotification",
    {
        "chargePointVendor": "vekon",
        "chargePointModel": "",
        "chargePointSerialNumber": "",
        "chargeBoxSerialNumber": "",
        "firmwareVersion": "",
        "meterType": ""
    }
]
```

BootNotification response: 

```json
[
    3,
    " 15455 ",
    {
        "currentTime": "2022-12-02T15:08:43.882Z",
        "interval": 60,
        "status": "Accepted"
    }
]
```

### Transactions before being accepted by a Central System

- Bir Şarj Noktası Operatörü, Şarj Noktasının Merkezi Sistem tarafından kabul edilmesinden önce işlemleri kabul etmesi için bir Şarj Noktasını yapılandırmayı SEÇEBİLİR. Bu tür bir davranışı uygulamak isteyen taraflar, bu işlemlerin Merkezi Sisteme teslim edilip edilemeyeceğinin belirsiz olduğunu fark etmelidir.
- Bir yeniden başlatmadan sonra (örneğin uzaktan sıfırlama komutu, elektrik kesintisi, aygıt yazılımı güncellemesi, yazılım hatası vb. nedeniyle) Şarj Noktası Merkezi Sistemle tekrar iletişime geçmeli ve bir BootNotification isteği GÖNDERMELİDİR. Şarj Noktası, Merkezi Sistemden bir BootNotification.conf alamazsa ve doğru şekilde önceden ayarlanmış yerleşik, kalıcı olmayan gerçek zamanlı saat donanımına sahip değilse, Şarj Noktasının geçerli bir tarih/saat ayarı olmayabilir ve bu da işlemlerin tarihini/saatini daha sonra belirlemeyi imkansız hale getirir.
- Ayrıca (örneğin yapılandırma hatası nedeniyle) Merkezi Sistemin uzun bir süre veya süresiz olarak Kabul Edildi dışında bir durum göstermesi de söz konusu olabilir.
- Daha önce Merkezi Sistem tarafından hiçbir Şarj Noktası Kabul edilmemişse (mevcut bağlantı ayarları, URL, vb. kullanılarak) bir Şarj Noktasındaki tüm şarj hizmetlerinin reddedilmesi genellikle tavsiye edilir; çünkü kullanıcıların kimliği doğrulanamaz ve yürütülen işlemler provizyon süreçleriyle çakışabilir.
  
### Data Transfer
- Bir Şarj Noktasının OCPP tarafından desteklenmeyen bir işlev için Merkezi Sisteme bilgi göndermesi gerekiyorsa, DataTransfer.req PDU'sunu KULLANMALIDIR.
- İstekteki vendorId, Merkezi Sistem tarafından bilinmeli ve satıcıya özgü uygulamayı benzersiz bir şekilde tanımlamalıdır. VendorId, tersine çevrildiğinde adın en üst katmanlarının Satıcı kuruluşunun genel olarak kayıtlı birincil DNS adına karşılık gelmesi gereken ters DNS ad alanından bir değer OLMALIDIR.
- İsteğe bağlı olarak, istek PDU'sundaki messageId belirli bir mesajı veya uygulamayı belirtmek için kullanılabilir.
- Hem istek hem de yanıt PDU'sundaki verilerin uzunluğu tanımlanmamıştır ve ilgili tüm taraflarca kabul edilmelidir.
- Eğer isteğin alıcısı belirli vendorId için bir uygulamaya sahip değilse, bir 'UnknownVendor' durumu döndürmelidir ve veri öğesi mevcut OLMAMALIDIR. Bir messageId uyumsuzluğu durumunda (kullanılıyorsa) alıcı 'UnknownMessageId' durumunu döndürmelidir. Diğer tüm durumlarda 'Accepted' veya 'Rejected' durumunun kullanımı ve veri öğesi ilgili taraflar arasındaki satıcıya özgü anlaşmanın bir parçasıdır.


Request : 
```json

```


Response : 
```json

```

### Diagnostics Status Notification
- Şarj Noktası, Merkezi Sistem'e bir tanılama yüklemesinin durumu hakkında bilgi vermek için bir bildirim gönderir. Şarj Noktası, Merkezi Sistem'e tanılama yüklemesinin meşgul olduğunu veya başarıyla tamamlandığını veya başarısız olduğunu bildirmek için bir DiagnosticsStatusNotification.req PDU göndermelidir. Şarj Noktası, yalnızca tanılama yüklemesiyle meşgul olmadığında, bir Tanılama Durumu Bildirimi için bir Tetikleme Mesajı aldıktan sonra Boşta durumunu göndermelidir.
- Bir DiagnosticsStatusNotification.req PDU alındığında, Merkezi Sistem bir DiagnosticsStatusNotification.conf ile yanıt vermelidir.
  
Request : 
```json

```


Response : 
```json

```


### Firmware Status Notification
- Şarj Noktası, Merkezi Sistem'e aygıt yazılımı güncellemesinin ilerlemesi hakkında bilgi vermek için bildirimler gönderir. Şarj Noktası, Merkezi Sistem'e bir aygıt yazılımı güncellemesinin indirilmesi ve kurulumunun ilerlemesi hakkında bilgi vermek için bir FirmwareStatusNotification.req PDU göndermelidir. Şarj Noktası, yalnızca aygıt yazılımı indirme/kurulumla meşgul olmadığında, bir Aygıt Yazılımı Durum Bildirimi için bir Tetikleme Mesajı aldıktan sonra Boşta durumunu göndermelidir.
- Bir FirmwareStatusNotification.req PDU alındığında, Merkezi Sistem bir FirmwareStatusNotification.conf ile yanıt vermelidir.
- FirmwareStatusNotification.req PDU'ları, Merkezi Sistem tarafından bir FirmwareUpdate.req PDU ile başlatılan güncelleme işleminin durumuyla Merkezi Sistem'i güncel tutmak için gönderilmelidir.

Request : 
```json

```


Response : 
```json

```

### Heartbeat
- Merkezi Sistemin bir Şarj Noktasının hala bağlı olduğunu bilmesini sağlamak için, Şarj Noktası yapılandırılabilir bir zaman aralığından sonra bir Heartbeat gönderir.
- Şarj Noktası, Merkezi Sistemin bir Şarj Noktasının hala aktif olduğunu bilmesini sağlamak için bir Heartbeat.req PDU göndermelidir.
- Heartbeat.req PDU alındığında, Merkezi Sistem bir Heartbeat.conf ile yanıt VERMELİDİR. Yanıt PDU'su, Şarj Noktasının dahili saatini senkronize etmek için kullanılması ÖNERİLEN Merkezi Sistemin geçerli saatini içermelidir.
- Şarj Noktası, yapılandırılmış kalp atışı aralığı içinde Merkezi Sisteme başka bir PDU gönderildiğinde bir Heartbeat.req PDU göndermeyi atlayabilir. Bu, bir Merkezi Sistemin, bir Heartbeat.req PDU aldığında olduğu gibi, bir PDU alındığında bir Şarj Noktasının kullanılabilirliğini varsayması GEREKTİĞİ anlamına gelir.
  
Request : 
```json

```


Response : 
```json

```

### Meter Values
- Bir Şarj Noktası, sayaç değerleri hakkında ek bilgi sağlamak için elektrik sayacını veya diğer sensör/transdüser donanımını örnekleyebilir. Şarj Noktası, sayaç değerlerini ne zaman göndereceğine karar verir. Bu, veri toplama aralıkları için ChangeConfiguration.req mesajı kullanılarak yapılandırılabilir ve toplanacak ve raporlanacak verileri belirtebilir.
- Şarj Noktası, sayaç değerlerini boşaltmak için bir MeterValues.req PDU göndermelidir. İstek PDU'su her örnek için şunları içermelidir:
  1. Örneklerin alındığı Bağlayıcının kimliği. BağlayıcıKimliği 0 ise, tüm Şarj Noktası ile ilişkilidir. BağlayıcıKimliği 0 ise ve Ölçüm Değeri enerjiyle ilgiliyse, örnek ana enerji sayacından alınmalıdır.

  2. Bu değerlerin ilişkili olduğu işlemin transactionId'si, geçerliyse. Devam eden bir işlem yoksa veya değerler ana sayaçtan alınıyorsa, işlem kimliği atlanabilir.

  3. Her biri belirli bir zaman noktasında alınan bir veya daha fazla veri değerini temsil eden MeterValue türünde bir veya daha fazla meterValue öğesi.
- Her MeterValue öğesi bir zaman damgası ve aynı zaman noktasında yakalanan bir veya daha fazla ayrı örneklenmiş değer öğesi kümesi içerir. Her örneklenmiş değer öğesi tek bir değer verisi içerir.
Her örneklenmiş değerin niteliği isteğe bağlı ölçülen büyüklük, bağlam, konum, birim, faz ve biçim alanları tarafından belirlenir.
- `measurand`İsteğe bağlı ölçülen büyüklük alanı ölçülen/raporlanan değerin türünü belirtir. İsteğe bağlı bağlam alanı okumayı tetikleyen nedeni/olayı belirtir.
- `context` İsteğe bağlı konum alanı ölçümün nerede yapıldığını belirtir (ör. Giriş, Çıkış).
- `location` İsteğe bağlı faz alanı değerin elektrik tesisatının hangi fazına veya fazlarına uygulandığını belirtir. Şarj Noktası, elektrik sayacı (veya yokken şebeke bağlantısı) bakış açısından tüm faz numarasına bağlı değerleri bildirmelidir.
- Bireysel konnektör faz rotasyonu bilgileri için, Merkezi Sistem GetConfiguration aracılığıyla Şarj Noktasındaki ConnectorPhaseRotation yapılandırma anahtarını sorgulayabilir. Şarj Noktası faz rotasyonunu şebeke bağlantısına göre bildirmelidir. Konnektör başına olası değerler şunlardır: NotApplicable, Unknown, RST, RTS, SRT, STR, TRS ve TSR. Daha fazla bilgi için Standart Yapılandırma Anahtar Adları ve Değerleri bölümüne bakın.
- DENEYSEL isteğe bağlı biçim alanı, verilerin normal (varsayılan) biçimde basit bir sayısal değer ("Ham") olarak mı yoksa onaltılık veri olarak gösterilen opak bir dijital olarak imzalanmış ikili veri bloğu olan "SignedData" olarak mı temsil edildiğini belirtir. Bu deneysel alan, daha olgun bir çözüm alternatifi sağlandığında kullanımdan kaldırılabilir ve daha sonraki sürümlerde kaldırılabilir.
- Geriye dönük uyumluluğu korumak için, sampledValue öğesindeki tüm isteğe bağlı alanların varsayılan değerleri, herhangi bir ek alanı olmayan bir değerin, Wh (Watt-saat) birimlerindeki etkin içe aktarma enerjisinin bir kayıt okuması olarak yorumlanacağı şekildedir.
- Bir MeterValues.req PDU alındığında, Merkezi Sistem bir MeterValues.conf ile yanıt VERMELİDİR.
- Merkezi Sistemin aldığı bir MeterValues.req'te bulunan verilere mantık kontrolleri uygulaması muhtemeldir. Bu tür mantık kontrollerinin sonucu, Merkezi Sistemin bir MeterValues.conf ile yanıt vermemesine ASLA neden OLMAMALIDIR. Bir MeterValues.conf ile yanıt vermemek, Şarj Noktasının yalnızca İşlemle ilgili mesajlara verilen Hata yanıtlarında belirtildiği gibi aynı mesajı tekrar denemesine neden olur.

Request : 
```json

```


Response : 
```json

```

### Start Transaction
- 