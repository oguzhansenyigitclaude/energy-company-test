# Patrona Sorulacaklar / Bilgi Notları

## 2026-08-19 00:58 turu — karar bekleyen 2 konu

### 1) ✅ ÇÖZÜLDÜ (19 Ağu, yerel geliştirici — wizardStart artık yerel kayıt yokken bulutu BEKLİYOR, yerel varken daha yeni bulut kaydında SEÇİM soruyor) — ⚠️ VERİ KAYBI RİSKİ — `wizardStart()` bulut kaydını kontrol etmiyor
Kayıt: `-P-MVJtR47ZJMnSEotVE` (ClaudeBot, 00:51)

Normal açılış akışında (Google yönlendirmesinden dönüş DEĞİLSE, ki çoğu açılış böyle) `wizardStart()` sadece `load()` (yani localStorage) kontrol ediyor; bulut kaydına hiç bakmıyor. Bulut restore SADECE `afterLogin()` içinde (Google login sonrası yönlendirme dönüşünde) çalışıyor.

Sonuç: aynı misafir/Google hesabı temiz bir tarayıcıda veya cihazda açıldığında, localStorage boş olduğu için "Yeni Oyun" ekranı çıkabiliyor. Oyuncu farkında olmadan devam ederse, otomatik kayıt (`save()`/`cloudSave()`) sunucudaki eski bulut kaydının (`saves/uid`) ÜZERİNE YAZABİLİR — geri dönüşü olmayan ilerleme kaybı riski.

**Neden kendim düzeltmedim:** Bu, `wizardStart()`'ı `afterLogin()` gibi async/bulut-bekleyen bir akışa çevirmeyi gerektiriyor — tam da `closeModal()` emniyet ağındaki (bu turda düzelttiğimiz) yarış durumuna benzer yeni bir yarış durumu riski taşıyor. Auth/senkronizasyon akışının merkezinde, misafir/Google/yönlendirme senaryolarının hepsini etkileyen bir değişiklik; "küçük ve düşük riskli" sınırının dışında kalıyor.

**Öneri (karar sizde):** `wizardStart()` başına da `afterLogin()`'deki gibi önce `NET.cloudLoad()` beklenip, `local` ile karşılaştırılan bir kontrol eklenebilir — ama bu, LOGIN_BUSY bayrağı ve mevcut giriş ekranlarıyla nasıl bir arada çalışacağının dikkatlice tasarlanmasını gerektiriyor.

### 2) ✅ ÇÖZÜLDÜ (19 Ağu, yerel geliştirici — tabloya 🛰️ simülasyon çarpanı ve 🎓 eğitim güvencesi satırları eklendi) — 📊 VERİM DÖKÜMÜ tablosu gerçek üretimi yansıtmıyor (kozmetik, ekonomiyi etkilemiyor)
Kayıtlar: `-P-M3kcJvQ7wJpegQxDB` (22:50) ve `-P-MVKEAScAJOZw4rcVT` (00:51) — aynı kök nedenin iki farklı görünümü.

`plantOutput()` gerçek üretimi hesaplarken iki ekstra çarpan/taban uyguluyor ama "VERİM DÖKÜMÜ" bilgi tablosu bunları HİÇ satır olarak göstermiyor:
- Eğitim (tutorial) tabanı: `if (tutActive()) f = Math.max(f, .35)` (satır ~2048) — ilk 1 saatlik eğitim penceresinde üretimi olduğundan çok yükseğe zorluyor ama tabloda görünmüyor (bot örneği: tabloya göre ~0.04MW beklenirken gerçek 0.67MW).
- Rüzgar/güneşte canlı hava verisi yokken uygulanan simülasyon çarpanı (örn. `windFactor(q, atMin) * .55`, satır ~2042) — tabloda bu satır da yok.

Tablonun başlığında "çarpanların sonucu = anlık üretim" yazıyor ama şu an bu doğru değil; oyuncu kafası karışabilir (bot bunu "YANLIŞ hesap" olarak bildirdi, aslında ekonomi/kasa doğru, sadece gösterim eksik).

**Neden kendim düzeltmedim:** Doğru düzeltme, tablo oluşturma kodunun (`index.html` ~6872-6899 arası) `plantOutput()`'taki hesabı BİREBİR aynalaması gerekiyor (özellikle eğitim tabanının hangi durumda devreye girdiğini göstermek için ham `f` değerini tablo kapsamında yeniden hesaplamak gerekecek). Bu, iki yerde aynı mantığın kopyalanması riskini taşıyor ve "küçük, hedefli" değişiklik sınırını aşıyor; ayrıca ekonomi/dengeyle ilgili olmasa da görünür sayılara dokunduğu için sizin onayınızı istiyorum.

**Öneri (karar sizde):** Tabloya eğitim modunda "🎓 Eğitim tabanı (min %35)" ve hava verisi yokken "🌬️ Simülasyon çarpanı (canlı veri yok)" satırları eklenebilir — veya daha basitçe, tablo başlığındaki "çarpımların sonucu = anlık üretim" iddiası bu iki senaryoda kaldırılıp "(eğitim/simülasyon modunda yaklaşık)" notu eklenebilir.

---

## Önceki turlar

- 2026-08-18 22:06-22:07 civarı `suggestions` düğümüne "ClaudeBot" adlı bir oyuncu-botundan 3 kayıt düştü. Üçü de oyun koduyla ilgili bir hata bildirmiyor; bildiren botun kendi headless tarayıcı/proxy sorunu ve hesap kurulum durumu hakkında. Detaylar `bot-notlar/islenen.md` içinde. Aksiyon gerekmiyor, sadece bilginize.
