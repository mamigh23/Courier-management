# FASTDELIVERY ANAYASASI

> FastDelivery'nin güvenlik, veri bütünlüğü, yetkilendirme, finansal doğruluk ve sürdürülebilir geliştirme kuralları.

## 1. Amaç
FastDelivery; kurye, restoran, operasyon ve muhasebe süreçlerini tek platformda yöneten çok kiracılı (multi-tenant) bir kurye yönetim/CRM uygulamasıdır.

Bu dosya, sistemi geliştirirken uyulacak temel teknik ve ürün kurallarını tanımlar. Yeni özellikler mevcut iş kurallarını, tenant isolation'ı, yetkilendirmeyi veya finansal doğruluğu bozamaz.

## 2. Değişmez Temel Kurallar
1. Backend authoritative'dir. Güvenlik, yetki, tenant erişimi ve finansal hesaplar istemciye bırakılamaz.
2. Kullanıcı yalnızca yetkili olduğu tenant/restoran verisine erişebilir.
3. URL'den gelen ID hiçbir zaman tek başına erişim yetkisi kanıtı değildir.
4. State değiştiren işlemler uygun HTTP yöntemiyle yapılır ve CSRF koruması uygulanır.
5. SQL sorgularında prepared statement kullanılır; kullanıcı girdisi string birleştirme ile SQL'e katılmaz.
6. Production sırları kaynak koda yazılmaz; environment/config secret kullanılır.
7. Finansal hesaplama tek bir merkezi iş mantığından üretilir; aynı formül farklı sayfalarda yeniden yazılmaz.
8. Kritik çok adımlı işlemler transaction içinde atomik tutulur.
9. Silme işlemlerinde mümkün olduğunca soft-delete yaklaşımı korunur; tarihsel muhasebe/audit kayıtları yok edilmez.
10. Audit log, kritik kullanıcı ve finans işlemlerinde tutulur.
11. Yeni dependency ancak gerçekten gerekli olduğu ve mevcut stack ile gerekçelendirildiği durumda eklenir.
12. Mevcut route, tasarım ve paylaşılan stiller gereksiz yere kırılmaz.
13. Mock/sahte veri veya gerçekte bulunmayan endpoint oluşturulmaz.
14. Hata mesajları production'da hassas bilgi sızdırmaz.
15. Her değişiklik mümkün olduğunca dar kapsamlı, geri alınabilir ve test edilebilir yapılır.

## 3. Roller
Desteklenen temel roller:
- owner
- restaurant_admin
- accountant
- courier

Rol kontrolü backend tarafında yapılır. UI'daki gizleme tek başına güvenlik mekanizması değildir.

## 4. Tenant Isolation
Her sorguda veri kapsamı kullanıcının yetkisiyle sınırlandırılır.

Özellikle:
- courier -> kendisi ve yetkili restoran ilişkileri
- restaurant_admin -> yetkili restoran(lar)
- accountant -> tanımlı muhasebe kapsamı
- owner -> sistem kapsamı

Bir nesnenin ID'sini bilen kullanıcının o nesneyi okuyabilmesi veya değiştirebilmesi garanti değildir.

## 5. Kimlik Doğrulama ve Session
- Parolalar password_hash / password_verify ile yönetilir.
- Başarılı login sonrasında session ID yenilenir.
- Session cookie'lerinde uygun Secure/HttpOnly/SameSite ayarları kullanılır.
- Login denemelerine rate limiting / brute-force koruması eklenir.
- Logout ve kritik session sonlandırma akışları güvenli tutulur.

## 6. CSRF
State-changing endpoint'lerde CSRF token zorunludur.

Kapsam:
- create/update/delete
- ödeme
- fatura kilitleme
- ceza/prim ekleme
- atama değişiklikleri
- admin kullanıcı işlemleri
- dönem kapatma

## 7. Finansal İş Mantığı
Finansal hesaplamalar merkezi servis üzerinden yapılır.

Önerilen çekirdek:
- InvoiceCalculator
- PaymentService
- PenaltyService

Hakediş, prim, mesai, ceza, avans, tevkifat ve net ödeme formülleri tek kaynaktan yönetilir.

Bir finansal formül değiştiğinde:
1. Formül dokümante edilir.
2. Merkezi hesaplama güncellenir.
3. Testler güncellenir.
4. Excel/UI/raporlama çıktılarının aynı sonucu verdiği doğrulanır.

## 8. Fatura ve Ödeme Kuralları
- Fatura tutarı backend tarafından hesaplanır.
- İstemciden gelen ödeme tutarı doğrulanmadan kabul edilmez.
- Kısmi ödeme destekleniyorsa kalan bakiye açık şekilde hesaplanır.
- Kilitlenen faturanın tarihsel finansal sonucu rastgele değiştirilemez.
- Fatura/dönem mantığı restoran geneli varsayımlara değil ilgili kurye + restoran kapsamındaki kurallara dayanır.

## 9. Veri Bütünlüğü
Database constraint'leri iş kurallarını desteklemelidir:
- foreign key
- unique constraint
- uygun index
- not null / check uygunluğu

Duplicate kayıtlar yalnızca application-level kontrolle değil gerektiğinde database constraint ile de engellenir.

## 10. Transaction
Aşağıdaki işlemlerde gerektiğinde transaction zorunludur:
- çoklu kayıt oluşturan/silen işlemler
- fatura üretimi ve ilişkili kayıtlar
- ödeme oluşturma
- kullanıcı/kurye silme ve ilişkili durum güncellemeleri
- kritik toplu operasyonlar

## 11. Performans
N+1 sorgular azaltılır.

Özellikle:
- dashboard
- analytics
- invoice listeleri
- payment listeleri
- günlük kayıtlar

için uygun JOIN, aggregate ve composite index kullanılır.

## 12. Mimari
Yeni geliştirmelerde mümkün olduğunca:
- controllers/actions
- services
- repositories/data access
- views

sorumlulukları ayrıştırılır.

View dosyalarının içinde gereksiz business logic büyütülmez.

## 13. Logging ve Audit
Kritik olaylar loglanır:
- login/logout
- password/security işlemleri
- ödeme
- fatura işlemleri
- ceza/prim
- kullanıcı/rol değişiklikleri
- kritik ayar değişiklikleri

Loglarda parola, token, secret veya gereksiz hassas veri tutulmaz.

## 14. Secret Yönetimi
DB parolası, API key, SMTP secret ve benzeri bilgiler:
- repository'ye commit edilmez
- .env / server environment / secret store üzerinden sağlanır
- örnek config dosyasında yalnızca placeholder tutulur

Bir secret yanlışlıkla repoya girdiyse artık güvenli kabul edilmez; rotate edilmelidir.

## 15. Test Politikası
Yeni kritik iş mantığı testsiz bırakılmaz.

Öncelik:
1. auth/authorization
2. tenant isolation
3. invoice calculation
4. payment state
5. period locking
6. validation
7. security regression

## 16. Geliştirme Prosedürü
Her değişiklikte:
1. İlgili anayasa maddesi kontrol edilir.
2. Etkilenen modüller belirlenir.
3. En dar değişiklik yapılır.
4. Güvenlik ve tenant etkisi kontrol edilir.
5. Finans etkisi varsa hesaplama regresyonu yapılır.
6. Test/manuel doğrulama yapılır.
7. Açıklayıcı commit mesajı yazılır.

## 17. Anti-Patternler
Aşağıdakiler kabul edilmez:
- GET /delete_* ile yıkıcı işlem
- CSRF'siz state change
- frontend'den gelen role güvenmek
- frontend'den gelen finans sonucunu authoritative kabul etmek
- SQL string interpolation
- production secret commit etmek
- başka dosyada zaten bulunan finans formülünü kopyalayıp değiştirmek
- erişim kontrolü yapmadan ID ile kayıt okumak
- geçici mock endpoint'i production kodu olarak bırakmak

## 18. Dokümantasyon Kuralı
Yeni kritik davranışların kaynağı kodla birlikte dokümante edilir.

Bu anayasa, roadmap ve teknik karar kayıtları birlikte güncel tutulur.
