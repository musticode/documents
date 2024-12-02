
# OCPP 1.6 Operations

## OCPP 1.6 - Operations Initiated by Charge Point

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

- Şarj Noktası, başlatılan bir işlem hakkında bilgi vermek için Merkezi Sisteme bir StartTransaction.req PDU göndermelidir. Bu işlem bir rezervasyonu sonlandırırsa (Şimdi Rezervasyon Yap işlemine bakın), StartTransaction.req, reservationId'yi içermelidir.
Bir StartTransaction.req PDU alındığında, Merkezi Sistem bir StartTransaction.conf PDU ile yanıt vermelidir. Bu yanıt PDU'su bir işlem kimliği ve bir yetkilendirme durumu değeri içermelidir.
- Merkezi Sistem, StartTransaction.req PDU'sundaki tanımlayıcının geçerliliğini doğrulamalıdır çünkü tanımlayıcı, güncel olmayan bilgiler kullanılarak Şarj Noktası tarafından yerel olarak yetkilendirilmiş olabilir. Örneğin, tanımlayıcı, Şarj Noktasının Yetkilendirme Önbelleğine eklendiğinden beri engellenmiş olabilir.
- Charge Point bir Yetkilendirme Önbelleği uyguladıysa, bir StartTransaction.conf PDU alındığında, idTag Yerel Yetkilendirme Listesinde değilse, Yetkilendirme Önbelleği altında açıklandığı gibi yanıttaki IdTagInfo değeriyle Charge Point önbellek girişini GÜNCELLEYECEKTİR.
- Merkezi Sistemin aldığı bir StartTransaction.req'te bulunan verilere mantık kontrolleri uygulaması muhtemeldir. Bu tür mantık kontrollerinin sonucu, Merkezi Sistemin bir StartTransaction.conf ile yanıt vermemesine ASLA neden OLMAMALIDIR. Bir StartTransaction.conf ile yanıt vermemek, Charge Point'in yalnızca İşlemle ilgili mesajlara verilen Hata yanıtlarında belirtildiği gibi aynı mesajı tekrar denemesine neden olur.
  
### Status Notification

- Bir Şarj Noktası, Şarj Noktası içindeki bir durum değişikliği veya hata hakkında Merkezi Sistem'i bilgilendirmek için Merkezi Sistem'e bir bildirim gönderir. Aşağıdaki tablo, bir Şarj Noktası'nın Merkezi Sistem'e bir StatusNotification.req PDU gönderebileceği önceki bir durumdan (sol sütun) yeni bir duruma (üst sıra) geçişleri göstermektedir.
  
> The Occupied state as defined in previous OCPP versions is no longer relevant. The Occupied state is split into five new statuses: Preparing, Charging, SuspendedEV, Suspended EVSE and Finishing.

#### Table Notes

> A Charge Point Connector MAY have any of the 9 statuses as shown in the table above. For ConnectorId 0, only a limited set is applicable, namely: Available, Unavailable and Faulted. The status of ConnectorId 0 has no direct connection to the status of the individual Connectors (>0).

> If charging is suspended both by the EV and the EVSE, status SuspendedEVSE SHALL have precedence over status SuspendedEV.

> When a Charge Point or a Connector is set to status Unavailable by a Change Availability command, the 'Unavailable' status MUST be persistent across reboots. The Charge Point MAY use the Unavailable status internally for other purposes (e.g. while updating firmware or waiting for an initial Accepted RegistrationStatus).

- Occupied durumu beş yeni duruma bölündüğünden (Hazırlanıyor, Şarj Ediliyor, Askıya Alınmış EV, Askıya Alınmış EVSE ve Bitiriliyor), Şarj Noktasından Merkezi Sisteme daha fazla StatusNotification.req PDU gönderilecektir. Örneğin, bir işlem başlatıldığında, Bağlayıcı durumu, muhtemelen birkaç saniye içinde, arada kısa bir Askıya Alınmış EV ve/veya "Askıya Alınmış EVSE" ile sırasıyla Hazırlanıyor'dan Şarj Ediliyor'a değişecektir.
- Geçiş sayısını sınırlamak için, Şarj Noktası, isteğe bağlı yapılandırma anahtarı MinimumStatusDuration'da tanımlanandan daha az süre aktifse bir StatusNotification.req göndermeyi atlayabilir. Bu şekilde, bir Şarj Noktası belirli StatusNotification.req PDU'larını göndermemeyi seçebilir.
- Şarj Noktası, Merkezi Sistem'e arıza koşullarını bildirmek için bir StatusNotification.req PDU gönderebilir. 'status' alanı Arızalı olmadığında, şarj işlemleri hala mümkün olduğundan durum bir uyarı olarak değerlendirilmelidir.
- Bir Şarj Noktası, EV tarafı bağlantısı kesildiğinde StopTransaction false olarak ayarlandığında, bir işlem çalışıyor ve EV StopTransactionOnEVSideDisconnect oluyor, ardından SuspendedEV durumuyla bir StatusNotification.req Merkezi Sisteme gönderilmeli ve 'errorCode' alanı 'NoError' olarak ayarlanmalıdır. Şarj Noktası 'info' alanına ek bilgi eklemeli ve Merkezi Sisteme askıya alma nedenini bildirmelidir: 'EV tarafı bağlantısı kesildi'. Mevcut işlem durdurulmaz.
- Bir Şarj Noktası StopTransactionOnEVSideDisconnect true olarak ayarlandığında, bir işlem çalışıyor ve EV EV tarafında bağlantısı kesiliyorsa, o zaman 'Finishing' durumu olan bir StatusNotification.req Merkezi Sisteme gönderilmeli ve 'errorCode' alanı 'NoError' olarak ayarlanmalıdır. Şarj Noktası 'info' alanına ek bilgi eklemeli ve Merkezi Sisteme durma nedenini bildirmelidir: 'EV tarafı bağlantısı kesildi'. Mevcut işlem durdurulur.
- Bir Şarj Noktası çevrimdışı olduktan sonra Merkezi Sisteme bağlandığında, Merkezi Sistemi aşağıdaki kurallara göre durumu hakkında günceller:
  1. Şarj Noktası, Şarj Noktası çevrimdışıyken durum değişirse, mevcut durumuyla birlikte bir StatusNotification.req PDU göndermelidir.
  2. Şarj Noktası, Şarj Noktası çevrimdışıyken meydana gelen bir hatayı bildirmek için bir StatusNotification.req PDU gönderebilir.
  3. Şarj Noktası, Şarj Noktası çevrimdışıyken meydana gelen ve Şarj Noktası hataları veya Şarj Noktasının mevcut durumu hakkında Merkezi Sistemi bilgilendirmeyen geçmiş durum değişikliği olayları için StatusNotification.req PDU'ları göndermemelidir.
  4. StatusNotification.req mesajları, açıkladıkları olayların meydana geldiği sırayla gönderilmelidir. 
- StatusNotification.req PDU'sunun alınması üzerine, Merkezi Sistem bir StatusNotification.conf PDU'su ile yanıt VERMELİDİR.

### Stop Transaction
- Bir işlem durdurulduğunda, Şarj Noktası, işlemin durdurulduğunu Merkezi Sisteme bildiren bir StopTransaction.req PDU göndermelidir.
- Bir StopTransaction.req PDU, işlem kullanımı hakkında daha fazla ayrıntı sağlamak için isteğe bağlı bir TransactionData öğesi içerebilir. İsteğe bağlı TransactionData öğesi, MeterValues.req PDU'sunun meterValue öğeleriyle aynı veri yapısını kullanan herhangi bir sayıda MeterValues ​​için bir kapsayıcıdır (bkz. MeterValues ​​bölümü)
- Stop Transaction.request PDU'su alındığında, Merkezi Sistem bir Stop Transaction.conf PDU'su ile yanıt VERMELİDİR.

> The Central System cannot prevent a transaction from stopping. It MAY only inform the Charge Point it has received the StopTransaction.req and MAY send information about the idTag used to stop the transaction. This information SHOULD be used to update the Authorization Cache, if implemented.

- İstek PDU'sundaki idTag, Şarj Noktasının işlemi durdurması gerektiğinde atlanabilir. Örneğin, Şarj Noktasının sıfırlanması istendiğinde.
- Bir işlem normal şekilde sonlandırılırsa (örneğin, EV sürücüsü işlemi durdurmak için kimliğini sunarsa), Neden öğesi atlanabilir ve Neden 'Yerel' olarak kabul edilmelidir. İşlem normal şekilde sonlandırılmazsa, Neden doğru bir değere ayarlanmalıdır. Normal işlem sonlandırma işleminin bir parçası olarak, Şarj Noktası kabloyu (kalıcı olarak bağlı değilse) açmalıdır.
- Şarj Noktası, kablo EV'de bağlantısı kesildiğinde kabloyu (kalıcı olarak bağlı değilse) açabilir. Destekleniyorsa, bu işlevsellik UnlockConnectorOnEVSideDisconnect yapılandırma anahtarı tarafından bildirilir ve kontrol edilir.
- Şarj Noktası, kablo EV'de bağlantısı kesildiğinde çalışan bir işlemi DURDURABİLİR. Destekleniyorsa, bu işlevsellik StopTransactionOnEVSideDisconnect yapılandırma anahtarı tarafından bildirilir ve kontrol edilir.
- StopTransactionOnEVSideDisconnect false olarak ayarlanırsa, kablo EV'den bağlantısı kesildiğinde işlem DURDURULMAMALIDIR. EV yeniden bağlanırsa, enerji transferine tekrar izin verilir. Bu durumda, aynı devam eden işlem sırasında diğer EV'lerin şarj olmasını ve bağlantısını kesmesini önleyecek bir mekanizma yoktur. UnlockConnectorOnEVSideDisconnect false olarak ayarlandığında, kullanıcı tanımlayıcıyı sunana kadar Bağlayıcı Şarj Noktasında kilitli KALIR.
- StopTransactionOnEVSideDisconnect true olarak ayarlandığında, kablo EV'den bağlantısı kesildiğinde işlem DURDURULMALIDIR. EV yeniden bağlanırsa, işlem durdurulana ve yeni bir işlem başlatılana kadar enerji transferine izin verilmez. UnlockConnectorOnEVSideDisconnect true olarak ayarlanırsa, Şarj Noktasındaki Bağlayıcı da kilidi açılır.
> - StopTransactionOnEVSideDisconnect false olarak ayarlanırsa, bu, UnlockConnectorOnEVSideDisconnect'e göre öncelikli OLACAKTIR. Başka bir deyişle: StopTransactionOnEVSideDisconnect false olduğunda, kablolar EV tarafında bağlantısı kesildiğinde her zaman kilitli kalır.
> - StopTransactionOnEVSideDisconnect'i true olarak ayarlamak, EV tarafındaki kilitli olmayan kabloları çıkararak enerji akışını durdurmaya yönelik sabotaj eylemlerini önleyecektir.
- Merkezi Sistemin aldığı bir StopTransaction.req'te bulunan verilere mantık kontrolleri uygulaması muhtemeldir. Bu tür mantık kontrollerinin sonucu ASLA Merkezi Sistemin bir StopTransaction.conf ile yanıt vermemesine neden OLMAMALIDIR. Bir StopTransaction.conf ile yanıt vermemek yalnızca Şarj Noktasının işlemle ilgili mesajlara verilen Hata yanıtlarında belirtildiği gibi aynı mesajı tekrar denemesine neden olur.
- Şarj Noktası bir Yetkilendirme Önbelleği uyguladıysa, bir StopTransaction.conf PDU'su alındığında Şarj Noktası, idTag Yerel Yetkilendirme Listesinde değilse, Yetkilendirme Önbelleği altında açıklandığı gibi yanıttaki IdTagInfo değeriyle önbellek girişini GÜNCELLEYECEKTİR.


## OCPP 1.6 - Operations Initiated by Central System