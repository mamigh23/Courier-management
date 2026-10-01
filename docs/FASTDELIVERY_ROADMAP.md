# FASTDELIVERY ROADMAP

## Hedef
FastDelivery'yi güvenli, finansal olarak tutarlı, multi-tenant izolasyonu sağlam ve sürdürülebilir bir kurye operasyon SaaS'ına dönüştürmek.

## Faz 0 — Dokümantasyon ve Güvenli Başlangıç
- [x] FASTDELIVERY_ANAYASA.md
- [x] FASTDELIVERY_ROADMAP.md
- [ ] Secret/env yapısının belirlenmesi
- [ ] Production hata gösteriminin kapatılması
- [ ] Mevcut DB backup prosedürünün yazılması

## Faz 1 — Güvenlik Temeli
- [ ] CSRF token altyapısı
- [ ] Tüm state-changing action'larda CSRF doğrulaması
- [ ] Yıkıcı endpoint'lerde POST zorunluluğu
- [ ] Session cookie hardening
- [ ] Login rate limiting / brute-force koruması
- [ ] Yetkisiz IDOR erişimlerinin tam taraması
- [ ] Owner / admin / accountant / courier izin matrisinin netleştirilmesi
- [ ] Güvenlik regresyon testleri

## Faz 2 — Tenant Isolation
- [ ] Tüm restaurant/courier/invoice/payment sorgularının erişim denetimi
- [ ] URL ID manipülasyon testleri
- [ ] Ortak authorization helper/service
- [ ] Yetki kontrolünün controller/action katmanında standardizasyonu
- [ ] Cross-tenant erişim test senaryoları

## Faz 3 — Finans Motoru
- [ ] InvoiceCalculator oluştur
- [ ] Hakediş formülünü tek kaynağa taşı
- [ ] Prim/mesai/ceza/avans hesaplamalarını merkezileştir
- [ ] Tevkifat formülünü doğrula ve açıkça dokümante et
- [ ] UI, Excel ve faturanın aynı hesap motorunu kullanmasını sağla
- [ ] PaymentService oluştur
- [ ] Kısmi ödeme politikasını kesinleştir
- [ ] Finansal regression testleri

## Faz 4 — Dönem ve Fatura Tutarlılığı
- [ ] Ödeme sıklığını restaurant-wide örnek kayıttan alma problemini kaldır
- [ ] Kurye + restoran bazında ödeme dönemi hesaplama
- [ ] Fatura kilitleme akışını standardize et
- [ ] Kilitli finans kayıtlarının değiştirilememesi
- [ ] Fatura numarası/idempotency kontrolleri
- [ ] Transaction güvenliği

## Faz 5 — Veri ve Performans
- [ ] N+1 sorguları tarama
- [ ] Dashboard sorgularını optimize et
- [ ] Analytics sorgularını optimize et
- [ ] Composite index'leri gözden geçir
- [ ] Gereksiz SELECT * kullanımlarını azalt
- [ ] Büyük veri senaryoları için pagination

## Faz 6 — Mimari Refactor
Önerilen yapı:

/config
/controllers veya /actions
/services
/repositories
/helpers
/views
/tests
/docs

Öncelikli servisler:
- InvoiceService
- InvoiceCalculator
- PaymentService
- CourierService
- RestaurantService
- PenaltyService
- AuthorizationService

## Faz 7 — Test Altyapısı
- [ ] Unit test altyapısı
- [ ] Integration test altyapısı
- [ ] Authentication testleri
- [ ] Authorization testleri
- [ ] Tenant isolation testleri
- [ ] Invoice calculation testleri
- [ ] Payment state testleri
- [ ] Validation testleri
- [ ] Security regression testleri

## Faz 8 — Operasyonel Sağlamlık
- [ ] Audit log standardizasyonu
- [ ] Hata loglama
- [ ] Health check
- [ ] Backup/restore dokümantasyonu
- [ ] Database migration stratejisi
- [ ] Production deployment checklist
- [ ] Environment separation

## Faz 9 — SaaS Ürünleştirme
- [ ] Tenant/company onboarding
- [ ] Abonelik planları
- [ ] Plan bazlı özellikler
- [ ] Firma bazlı ayarlar
- [ ] Kullanım limitleri
- [ ] Bildirim altyapısı
- [ ] API katmanı
- [ ] Webhook altyapısı

## Faz 10 — Gelişmiş Operasyon
- [ ] Kurye performans raporları
- [ ] Restoran SLA/operasyon göstergeleri
- [ ] Gelişmiş filtreleme
- [ ] Toplu işlemler
- [ ] Mobil uyumluluk iyileştirmeleri
- [ ] Gerçek zamanlı bildirimler
- [ ] Harita/konum entegrasyonu gerektiğinde

## Geliştirme İlkesi
Her faz tamamlanmadan sonraki faza kritik borç taşınmamalıdır.

Öncelik sırası:
1. Güvenlik
2. Tenant isolation
3. Finansal doğruluk
4. Veri bütünlüğü
5. Performans
6. Mimari sürdürülebilirlik
7. UX ve yeni özellikler

Her tamamlanan iş:
- küçük kapsamlı olmalı
- test edilebilir olmalı
- geri alınabilir olmalı
- açıklayıcı commit ile kaydedilmeli
- anayasa ile uyumlu olmalı
