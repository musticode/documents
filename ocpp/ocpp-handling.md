
# OCPP 1.6 - Operations Initiated by Charge Point

**Request and responses :** https://github.com/gennadiygnezdilov/ocpp-1.6J-example-request-response/tree/main

## Authorize

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

## BootNotification

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

- 


