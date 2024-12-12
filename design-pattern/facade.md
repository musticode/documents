# Facade Design Pattern

- Facade Türkçe ’ye cephe olarak çevriliyor. Bu tasarım kalıbı adından da anlaşılacağı üzere yazılımımız için yeni cephe(arayüz) oluşturuyor diyebiliriz.
- Bu tasarım kalıbı bir veya birden fazla sınıftaki karmaşayı bir cephenin ardına gizler.
- Karmaşık alt sistemleri olan bir yapıyı ; tek , makul bir arayüz sağlayan Facade sınıfı oluşturarak basitleştirebiliriz.
- Facade basit bir arayüz sağlar ve alt sistemleri bu arayüze dahil eder.
- Yazacağımız bu arayüzde , alt sınıflar Facade sınıfımızdan bağımsız da çalışabilmeliler, Facade sadece bir kullanım kolaylığı sağlayan arayüz olacak tasarımımızda.

