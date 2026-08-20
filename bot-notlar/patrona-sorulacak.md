# Patrona Sorulacaklar / Bilgi Notları

## 2026-08-20 21:56 turu — yeni karar bekleyen konu yok, 2 mevcut konu yeniden doğrulandı

14 yeni `suggestions` kaydı işlendi, 1 net/düşük riskli kod hatası düzeltildi (hoşgeldin modalı "dakikate" yazım hatası — bkz. `duzeltmeler.md`, `islenen.md`). Aşağıdaki 2 konu zaten karar bekliyordu, bu turda ek bağımsız kanıtla yeniden doğrulandı — yeni madde açmadım, sadece kanıt sayısını güncelliyorum:

- ☀️ **VERİM DÖKÜMÜ ↔ anlık üretim tutarsızlığı** (aşağıdaki "2026-08-20 14:59 turu" konusu): bu turda **2 yeni** bağımsız kayıt (`-P-V3Q24xfQe9Hf2FCN8` 16:47, rüzgar #1; `-P-VhU30oqqQ9wJvNJFV` 19:46, canlı rüzgar verisiyle repro — RES T-200, 2MW×0.14×0.91×1.00=0.254 MW hesaplanıyor ama ekranda "0.3 MW" yazıyor, 1 ondalık yuvarlamadan ~%18 görsel sapma) aynı kök nedeni (aşağıdaki 3 olası neden — önbellek/kelepçe/yuvarlama) doğruladı. Artık rüzgar santrallerinde de gözlemlendi (önceki kayıtlar sadece güneşte bildirmişti) — küçük ondalık yuvarlama sorunu güneşe özgü değil, genel görünüyor. Karar hâlâ sizde.
- 💾 **Bulut kaydı `visibilitychange`/`pagehide` zorunlu flush** (aşağıda "2026-08-19 21:56 turu #1" ve sonraki turlarda tekrarlanan konu): `-P-VH2X6nAzDqxnQWz9t` (17:46) somut bir repro ekledi — santral onarımı yap, 20sn throttle penceresi içinde uygulamayı arkaplana al/kapat, değişiklik buluta hiç düşmüyor (yalnızca `NET.cloudSave(true)` zorlanınca düşüyor). Bu, mevcut "dikkatli tasarım kararı gerektiriyor" değerlendirmesini değiştirmiyor, sadece riskin gerçek senaryoda tetiklendiğini somut olarak gösteriyor.

Kalan 12 kayıt durum özeti/doğrulama, aksiyon gerekmiyor (detaylar `islenen.md`'de).

---

## 2026-08-20 14:59 turu — 1 yeni tasarım/karar konusu (VERİM DÖKÜMÜ ↔ anlık üretim tutarsızlığı)

Bu turda 13 kayıt işlendi, 1 net kod hatası düzeltildi ("Hepsini Sat" onay penceresi — bkz. `duzeltmeler.md`). Aşağıdaki konu ise verim/üretim **hesabıyla ilgili ve kök nedeni belirsiz** olduğu için koda dokunulmadı, kararınızı bekliyor:

### ☀️ Solar "VERİM DÖKÜMÜ" tablosundaki çarpanların çarpımı, gösterilen "anlık üretim" ile birebir tutmuyor

**Bulgu (2 bağımsız bot kaydı, `-P-UDR7FELbVfIVyWhuM` 12:50 ve `-P-UdHzkDm3Mgi2OVhiY` 14:48):**
Santral bilgi kartındaki VERİM DÖKÜMÜ tablosu başlığında "çarpanların sonucu = anlık üretim" yazıyor, ama bazı **güneş** santrallerinde tablodaki çarpanların çarpımı gösterilen anlık üretimden farklı çıkıyor:
- Santral #7 (solar_1__yelkora): tablo 0.9×0.78(bulut)×1.09(saha)×0.25 → **0.172 MW** gösteriyor ama gerçek `plantOutput` = **0.158 MW** (fark tam ×1.09).
- Santral #11 (solar_1__ortaç): tablo 0.268 vs gerçek 0.244 (yine ×1.09) — ya da başka kayıtta "📍 Saha kalitesi: keşif bekleniyor —" (sayı yok) gösterilirken üretim yine de farklı.
- Rüzgar santrallerinde birebir tutuyor; sorun güneşe özgü görünüyor.

**Neden koda dokunmadım (belirsizlik — üç olası kök neden var, hangisinin geçerli olduğu emin değil):**
1. **`siteFactor` 60 sn önbelleği** (`index.html` ~satır 883-889): `plantOutput` önbelleğe alınmış `u._sf`'i kullanır; tablo ise `siteParts(u)`'ı **taze** hesaplar. Kare yeni keşfedildiyse (survey tamamlandıysa) tablo yeni saha çarpanını (×1.09) gösterirken üretim hâlâ eski önbellekli 1.0'ı kullanıyor olabilir → ~60 sn'lik geçici tutarsızlık.
2. **`clampF(.7, 1.3, ...)` kelepçesi** (siteFactor içinde): tablo ham çarpanları çarpıp gösterirken siteFactor toplam çarpımı [0.7,1.3]'e kelepçeliyor; kelepçe devredeyse tablo bunu göstermiyor.
3. **Keşif durumu (`SV(cell)`)**: tablo saha çarpanını 1.09 gösterirken üretimin siteFactor'ü 1.0 uyguluyor olabilir (bot bu yorumu yaptı) — ama bu, önbellek/keşif zamanlamasına bağlı, deterministik değil.

Bu bir **gösterim/UX** hatası (ekonomiyi değiştirmiyor — üretim rakamı doğru, sadece tablo açıklaması tutmuyor), ama düzeltmenin doğru yeri kök nedene göre değişiyor: önbellek meselesiyse tabloyu önbellekli `siteFactor` ile hizalamak, kelepçeyse tabloya kelepçe satırı eklemek gerekir. Yanlış düzeltme oyuncuya farklı bir yanlış sayı gösterir. **Karar:** hangi davranış "doğru" — (a) tablo taze siteParts'ı mı göstermeli (üretim önbelleğini kırıp anında güncelleyerek), yoksa (b) tablo üretimin fiilen kullandığı kelepçeli/önbellekli değeri mi göstermeli? Yerel geliştirici siteFactor önbellek/kelepçe akışını netleştirip tek satırlık hizalama yapabilir.

---

## 2026-08-20 07:58 turu — yeni karar bekleyen konu yok, mevcut konular yeniden doğrulandı

17 yeni `suggestions` kaydı işlendi, net/düşük riskli yeni bir kod hatası bulunmadı. Detaylar `bot-notlar/islenen.md`'de. Kod değişikliği yapılmadı, BUILD_TAG/version.json güncellenmedi.

2 kayıt, aşağıda önceki turlarda size sorulmuş bulut-kayıt-güvenilirliği (throttle/flush) konusunu somut rakamlarla/ek detayla yeniden doğruladı — yeni bilgi olarak not ediyorum, henüz kod değişikliği yapmadım:
- `-P-RpCC3Yi0lpy5ONdew` (01:41): "Tüm Depoları Sat"/MERKEZ depo SAT sonrası ~8sn içinde sekme kapatılınca **823.337,70** kasa artışı buluta yazılmadan kayboldu — aşağıdaki "ara tick kaydı" konusunun (2026-08-19 21:56 turu #1) somut rakamlarla üçüncü bağımsız doğrulaması.
- `-P-SWRRxsOUzmnp3hOtU` (04:53): Ek detay — `save()` içindeki `NET.cloudSave()` çağrısı `await` edilmiyor ve throttle 20sn; açılıştan sonraki ilk 20sn'lik pencerede yapılan aksiyonlar (araştırma başlatma, satış) sekme o an kapatılırsa buluta hiç yazılmıyor. Bu, aşağıdaki hem "ara tick kaydı" hem "pagehide/beforeunload flush" konularıyla aynı kök nedene (`NET.cloudSave` throttle'ının garantili/senkron flush'ı yok) işaret ediyor.

Bu ikisi de mevcut açık konularla aynı kök nedende olduğu için ayrı yeni madde açmadım; aşağıdaki 2 açık konu hâlâ yanıt bekliyor ve artık toplam 4 bağımsız bot bulgusuyla destekleniyor (2026-08-19 18:40, 21:56, ve bugünkü 2 kayıt). Öneri: üçü de aynı "bulut kaydı güvenilirliği" ailesi olduğu için tek bir karar/tasarım turu olarak birlikte değerlendirilmesi en tutarlısı olur.

Aşağıdaki önceki turlardan karar bekleyen konular hâlâ açık (yanıt beklemede):
- 💾 "Tüm Depoları Sat" sonrası erken sekme kapatmada satış kayboluyor (ara tick kaydı) — bkz. 2026-08-19 21:56 turu #1 (şimdi `-P-RpCC3Yi0lpy5ONdew` ile 3. kez doğrulandı)
- 💾 Bulut kaydı `pagehide`/`beforeunload` flush eklensin mi — bkz. 2026-08-19 14:58 turu #1 (şimdi `-P-SFTYSlb_gtgSH0Ctc` ile tekrar bildirildi, `-P-SWRRxsOUzmnp3hOtU` ek detay ekledi: `cloudSave` zaten `await` edilmiyor)

---

## 2026-08-20 00:53 turu — yeni karar bekleyen konu yok

4 yeni `suggestions` kaydı işlendi, hepsi durum özeti/bilgi amaçlıydı. Detaylar `bot-notlar/islenen.md`'de. Kod değişikliği yapılmadı, dolayısıyla BUILD_TAG/version.json güncellenmedi.

Aşağıdaki önceki turlardan karar bekleyen konular hâlâ açık (yanıt beklemede):
- 💾 "Tüm Depoları Sat" sonrası erken sekme kapatmada satış kayboluyor (ara tick kaydı) — bkz. 2026-08-19 21:56 turu #1
- 💾 Bulut kaydı `pagehide`/`beforeunload` flush eklensin mi — bkz. 2026-08-19 14:58 turu #1

---

## 2026-08-19 21:56 turu — karar bekleyen 1 konu, 2 bilgi notu

### 1) 💾 "Tüm Depoları Sat" sonrası erken sekme kapatmada satış kayboluyor — ara tick kaydı eklensin mi?
Kayıt: `-P-QY_wAaRpycmWfhdxV` (ClaudeBot, 19:45)

Bot, "Tüm Depoları Sat" tetikleyip ~2sn sonra sekmeyi kapattığında, istemci tarafında kasa artışını (57.78M→58.44M) gördüğünü ama sunucudaki kaydın (`save/<uid>`) bu artışı hiç yansıtmadığını (kasa aynı kaldı, `disch=null`) doğruladı. Kök neden: `startDischarge()` (satır ~2415-2464) sadece boşaltma BAŞLARKEN ve TAMAMLANDIĞINDA `save()` çağırıyor; boşaltma süresince (saniyeler-dakikalar) ara tick'lerde kaydetmiyor. Süreç tamamlanmadan sekme kapatılırsa (veya tarayıcı/işlem sonlandırılırsa) o satışın geliri sessizce kayboluyor.

**Neden kendim düzeltmedim:** Bu, boşaltma/kayıt akışının merkezine dokunuyor — ne sıklıkta ara `save()` çağrılacağı (her tick mi, throttle'lı mı?), bunun mevcut 20sn'lik `cloudSave` throttle'ıyla nasıl bir arada çalışacağı ve performans/ağ trafiği etkisi dikkatli bir tasarım kararı gerektiriyor. Ayrıca önceki turda size sorulan `pagehide`/`beforeunload` flush konusuyla (aşağıda "Önceki turlar") aynı kayıt-güvenilirliği ailesinden — birlikte değerlendirilmesi daha tutarlı olabilir.

**Öneri (karar sizde):** En basit çözüm, `startDischarge()` içindeki periyodik tick döngüsüne (varsa) düşük sıklıklı bir `save()` eklemek (örn. her ~10-15sn'de bir) olabilir; daha sağlam bir çözüm, boşaltmanın ilerleyişini de kaydeden bir "devam eden işlem" durumu tutup sayfa yeniden açıldığında/`afterLogin`'de bunu tamamlamak olabilir — ama bu daha büyük bir değişiklik.

---

### Bilgi notu 1: `vendor/leaflet.css`'in referans verdiği varsayılan Leaflet ikonları repo'da yok (404, kozmetik)
Kayıt: `-P-PWx-P3ajqJy5amNvz` (ClaudeBot, 14:57)

`vendor/leaflet.css` her sayfa yüklemesinde `vendor/images/layers.png`, `marker-icon.png`, `marker-icon-2x.png`, `marker-shadow.png` gibi dosyalara `background-image: url(...)` ile referans veriyor ama bu dosyalar repo'da (ve canlı sitede) yok — her yükleme 4 adet 404 üretiyor. Oyun özel zoom butonları ve kendi marker ikonlarını kullandığı için görünür bir UI etkisi yok, sadece konsolu kirletiyor.

Kod değiştirmedim çünkü gerçek düzeltme ya eksik görsel dosyaları eklemek ya da `vendor/leaflet.css`'i (bir `.css` dosyası, `index.html` değil) düzenlemek gerektiriyor — bu turun "index.html'de küçük hedefli değişiklik" kapsamının dışında. İsterseniz bir sonraki turda (görsel dosya eklemeye veya CSS'e küçük bir dokunuşa onay verirseniz) hallederiz.

---

### Bilgi notu 2: MERKEZ depoda `stored` okuması ile gerçek satılan miktar arasında ~%43 fark gözlemi (doğrulanmamış)
Kayıt: `-P-QLZjFn21-vrMKgFJO` (ClaudeBot, 18:55)

Bot, SAT'a basmadan hemen önce okunan `stored=10.65 MWh` değeri ile `startDischarge` tetiklendiğinde gerçekte satılan `15.25 MWh` arasında fark olduğunu bildirdi; olası neden olarak "X dk uzaktaydın" offline-catchup simülasyonunun SAT anında arka planda hâlâ dolum yapıyor olabileceğini öne sürdü. Botun kendi ifadesiyle bu "kesin değil, doğrulama ister" bir gözlem — tek seferlik, tekrarlanmadı. Bu turda kod değişikliği yapmadım; eğer gelecek turlarda benzer bir fark tekrar bildirilirse (ideal olarak repro adımlarıyla) önceliklendirip inceleyeceğim.

---

## 2026-08-19 14:58 turu — karar bekleyen 1 konu

### 1) 💾 Bulut kaydı: `pagehide`/`beforeunload` olayında da zorunlu flush eklensin mi?
Kayıt: `-P-OqJokbSRgZGOisfRZ` (ClaudeBot, 11:47 #2)

Şu an `NET.cloudSave(true)` zorunlu flush'ı SADECE `visibilitychange` olayında `document.hidden` olduğunda tetikleniyor (satır ~7918). Bot, mobilde sekme/uygulamanın aniden kapatılması (process kill, swipe-away) gibi senaryolarda `visibilitychange`'in her zaman tetiklenmeyebileceğini, bu durumda throttle'lı (20sn) `cloudSave`'in henüz göndermediği son ilerlemenin kaybolabileceğini bildirdi. Botun kendi değerlendirmesi: "düşük öncelik, veri kaybı riski var ama nadir senaryo."

**Neden kendim düzeltmedim:** Teknik olarak küçük bir ekleme (aynı `save(); if (NET.on()) NET.cloudSave(true);` çağrısını `pagehide`/`beforeunload` olayına da bağlamak), ama bulut senkronizasyon akışına dokunuyor — bu alan önceki turlarda "dikkatli tasarım kararı gerektiriyor" diye size bırakılmıştı (bkz. aşağıdaki 00:58 turu #1). `beforeunload`/`pagehide` bazı tarayıcılarda sayfa zaten kapanırken senkron olmayan ağ isteklerini kesebiliyor (`cloudSave`'in fetch/XHR'ının tamamlanacağı garanti değil) — `navigator.sendBeacon` gibi farklı bir taşıma gerekebilir, bu da mevcut `cloudSave()` implementasyonunun değiştirilmesini gerektirir. Kendi başına karar vermek yerine onayınızı istiyorum.

**Öneri (karar sizde):** `document.addEventListener('pagehide', ...)` ile aynı flush'ı ek güvenlik olarak eklemek düşük risk taşır; gerçekten garantili teslim isteniyorsa `NET.cloudSave`'in `sendBeacon` kullanan ayrı bir "acil" moda sahip olması daha sağlam olur ama bu daha büyük bir değişiklik.

---

## 2026-08-19 07:56 turu — yeni karar bekleyen konu yok

Bu turda `suggestions`'tan gelen 11 yeni kayıt işlendi. 2'si (Şirket paneli çökmesi `resRow('liccap')`, VERİM DÖKÜMÜ `0,00` biçim hatası) net/düşük riskli olduğu için doğrudan düzeltildi. 2'si (boot resume bulut kontrolü, wizardStart giriş akışı) aşağıdaki önceki turun maddesi #1 ile aynı kök nedene sahipti ve zaten sizin tarafınızdan çözülmüş bulundu. Kalan 7'si durum özeti/bilgi amaçlıydı, aksiyon gerekmedi. Detaylar `bot-notlar/islenen.md` ve `bot-notlar/duzeltmeler.md`'de.

**Küçük bilgi notu (aksiyon gerekmiyor, isterseniz):** VERİM DÖKÜMÜ tablosunda jeotermal santral satırında da (`T.kind === 'geo'`) aynı virgüllü yazım kalıbı var — `'gece gündüz sabit — ×0,95'`. Bu suggestions'ta bildirilmediği için bu turda dokunulmadı, ama düzelttiğimiz `0,00`'la aynı kozmetik hata. Dilerseniz bir sonraki turda temizleyebiliriz.

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
