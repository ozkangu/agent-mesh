# Mimari İyileştirme Önerileri

## 1. Çapraz Dosya Transaction Desteği
- **Sorun**: Bir görev tamamlanırken `tasks.json`, `inbox.json` ve `activity-log.json` aynı anda güncelleniyor. Herhangi birinde hata olursa partial failure riski var.
- **Öneri**: Basit bir transaction wrapper yazılabilir — tüm dosya değişikliklerini topla, hepsini sırayla yaz, hata olursa rollback uygula.
- **Öncelik**: Orta

## 2. JSON Dosya Boyutu Ölçeklenme Sorunu
- **Sorun**: Tüm veri tek JSON dosyalarında tutuluyor. Binlerce görev/olay biriktiğinde dosya okuma/yazma yavaşlar.
- **Öneri**: `activity-log.json` ve `tasks-archive.json` gibi büyüyen dosyalar için aylık rotasyon veya pagination-at-file-level stratejisi uygulanabilir. Alternatif olarak SQLite gibi hafif bir embedded DB'ye geçiş düşünülebilir.
- **Öncelik**: Düşük (şu an sorun değil, gelecekte olabilir)

## 3. WebSocket / SSE ile Gerçek Zamanlı Güncelleme
- **Sorun**: Frontend 2-30 saniye aralıklarla polling yapıyor. Daemon bir görevi tamamladığında UI'da gecikme oluyor.
- **Öneri**: Next.js API route'larına SSE (Server-Sent Events) endpoint'i eklenebilir. Polling tamamen kaldırılmasa bile kritik olaylar (görev tamamlama, yeni mesaj) için anlık bildirim sağlar.
- **Öncelik**: Düşük

## 4. Çapraz Dosya Mutex Deadlock Riski
- **Sorun**: Mevcut mutex'ler dosya başına ayrı. Eğer bir işlem birden fazla dosyayı sırayla kilitlerse ve başka bir işlem aynı dosyaları ters sırada kilitlerse deadlock olabilir.
- **Öneri**: Dosya kilitleme sırası (lock ordering) kuralı belirlenebilir veya tüm dosyalar için tek bir global mutex kullanılabilir (performans tradeoff'u ile).
- **Öncelik**: Düşük (şu anki kullanımda gerçekleşme ihtimali çok düşük)

## 5. API Rate Limiting
- **Sorun**: API endpoint'lerinde rate limiting yok. Daemon veya hatalı bir ajan aşırı istek gönderebilir.
- **Öneri**: Basit bir in-memory rate limiter middleware'i eklenebilir (token bucket veya sliding window).
- **Öncelik**: Düşük

## 6. Yapılandırılmış Hata Loglama
- **Sorun**: Hata logları dağınık. Daemon logları ayrı, API hataları ayrı, ajan hataları ayrı.
- **Öneri**: Merkezi bir `errors.json` veya `error-log.json` dosyası ile tüm hataları tek yerden izlenebilir hale getirmek. Hata kategorisi, stack trace, ve tekrar sayısı gibi alanlar eklenebilir.
- **Öncelik**: Orta

## 7. Otomatik Veri Yedekleme (Auto-Checkpoint)
- **Sorun**: Checkpoint'ler manuel oluşturuluyor. Veri kaybı riski var.
- **Öneri**: Daemon'a günlük otomatik checkpoint özelliği eklenebilir (cron schedule ile). Son N checkpoint tutulup eskileri otomatik silinebilir.
- **Öncelik**: Orta

## 8. Görev Bağımlılık Döngüsü Tespiti
- **Sorun**: `blockedBy` alanında döngüsel bağımlılık kontrolü yok. A → B → C → A şeklinde bir döngü oluşursa tüm görevler sonsuza kadar bloklanır.
- **Öneri**: Görev oluşturma/güncelleme API'sinde basit bir cycle detection algoritması (DFS) çalıştırılabilir.
- **Öncelik**: Orta

## 9. Ajan Performans Metrikleri
- **Sorun**: Ajanların başarı oranı, ortalama tamamlama süresi, token maliyeti gibi metrikler toplu olarak görüntülenemiyor.
- **Öneri**: Dashboard'a ajan bazlı performans kartları eklenebilir. `active-runs.json` geçmişinden aggregate edilebilir.
- **Öncelik**: Düşük

## 10. Test Kapsamı
- **Sorun**: Mevcut test kapsamı belirsiz. Kritik veri katmanı (mutex, CRUD) ve daemon logic'i için yeterli test olup olmadığı kontrol edilmeli.
- **Öneri**: `src/lib/data.ts` mutex davranışları, API route validasyonları ve daemon dispatcher logic'i için birim testleri eklenmeli.
- **Öncelik**: Yüksek
