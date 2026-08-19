# İşlenen Firebase Anahtarları

Bu dosya, `suggestions` düğümünden okunup bakım botu tarafından değerlendirilen kayıtların anahtarlarını listeler. Sadece burada olmayan (yeni) anahtarlar bir sonraki turda işlenir.

| Anahtar | Zaman (UTC) | Özet | Sonuç |
|---|---|---|---|
| `-P-LuNJCmDWvKqE3jQvR` | 2026-08-18 22:06 | ClaudeBot: headless Chromium tüm HTTPS hedeflerinde ERR_CONNECTION_RESET veriyor (proxy üzerinden curl çalışıyor) | Oyun koduyla ilgili değil, bildiren botun kendi test ortamı/proxy sorunu. Kod değişikliği yapılmadı. |
| `-P-LuNULjpqrFq_JQc0M` | 2026-08-18 22:06 | ClaudeBot: `saves/0` boş, kurulum sihirbazı hiç çalışmamış (tarayıcı açılamadığı için) | Oyun kodunda hata değil, bildiren botun hesap durumu bilgisi. Kod değişikliği yapılmadı. |
| `-P-LuV7U_OgxAOnPntbH` | 2026-08-18 22:07 | ClaudeBot: önceki iki kayıtta `fromId` yanlış yazılmış, kendi düzeltme notu | Oyun kodunu ilgilendirmiyor (bildiren botun kendi script hatası). Kod değişikliği yapılmadı. |
| `-P-M3kTIYNUC1b7N_oZc` | 2026-08-18 22:41 | ClaudeBot: BUG — "Yeni Oyun — Haritadan Başla" butonuna ilk dokunuşta hoşgeldin modalı ülke/il seçim ekranının üstünde takılı kalıyor | **DÜZELTİLDİ.** `closeModal()` içindeki emniyet ağı kontrolü zamanlayıcı KURULURKEN değil, 50ms sonra ateşlenme ANINDA yeniden kontrol edecek şekilde taşındı. Detay: `bot-notlar/duzeltmeler.md`. |
| `-P-M3kcJvQ7wJpegQxDB` | 2026-08-18 22:50 | ClaudeBot: BUG — VERİM DÖKÜMÜ tablosu eğitim (tutorial) tabanı `f=Math.max(f,.35)` satırını göstermiyor, tablo çarpımı gerçek üretimle uyuşmuyor (~17x fark) | Kod değiştirilmedi — patrona soruldu (bkz. `patrona-sorulacak.md`), gösterim mantığını `plantOutput()` ile birebir tutmak ek tasarım kararı gerektiriyor. |
| `-P-MFzRPBy29Y_v_pXhg` | 2026-08-18 23:45 | ClaudeBot: KRİTİK — bulut kayıtta `G.cellSold={day:...}` iken `sold` alt-nesnesi RTDB tarafından siliniyor, `demandLeft()` çöküyor, depo/tesis ekranı ve SAT işlemi sessizce çalışmıyor | **DÜZELTİLDİ.** `ensureCellSold()` artık `sold` alt-nesnesinin var olup olmadığını da kontrol ediyor. Detay: `bot-notlar/duzeltmeler.md`. |
| `-P-MHU3LJAS-H2PAsVOM` | 2026-08-18 23:51 | ClaudeBot: durum özeti (bug değil) — GES paneli alımı, bağlı depo kısıtları, kasa hesapları hepsi doğru çalışıyor | Bilgi amaçlı, aksiyon gerekmiyor. |
| `-P-MVD4LWSKYY20TxTHX` | 2026-08-19 00:51 | "Ekopark otomasyon" (farklı kullanıcı, `[BOT` önekli değil) — "Deneme" metni | Oyuncu-botun bulgusu değil (önek yok, farklı fromId); muhtemelen başka bir oyuncunun öneri kutusunu test etmesi. Aksiyon gerekmiyor. |
| `-P-MVJtR47ZJMnSEotVE` | 2026-08-19 00:51 | ClaudeBot: BUG — `wizardStart()` normal açılışta bulut kaydını kontrol etmiyor, sadece localStorage'a bakıyor; temiz tarayıcıda "Yeni Oyun" ekranı çıkabiliyor ve devam edilirse bulut kaydının (`saves/uid`) üzerine yazılabiliyor (VERİ KAYBI RİSKİ) | Kod değiştirilmedi — patrona soruldu (bkz. `patrona-sorulacak.md`). Auth/senkronizasyon akışını etkileyen geniş kapsamlı bir değişiklik, dikkatli tasarım kararı gerektiriyor. |
| `-P-MVKEAScAJOZw4rcVT` | 2026-08-19 00:51 | ClaudeBot: BUG (VERİM DÖKÜMÜ, #2 bulgusunun devamı) — rüzgar/güneş santralinde canlı hava verisi yokken uygulanan simülasyon çarpanı (`windFactor(q)*.55` vb.) tabloda hiç görünmüyor | Kod değiştirilmedi — yukarıdaki `-P-M3kcJvQ7wJpegQxDB` ile aynı kök nedene (VERİM DÖKÜMÜ tablosu eksik satırlar) bağlı, birlikte patrona soruldu. |
| `-P-MVKZ_hdsVYATsd4jt` | 2026-08-19 00:51 | ClaudeBot: durum özeti (bug değil) — SAT/sök/müzayede kısıtları doğru, kasa hesapları doğru, cellSold çökmesi bu turda tetiklenmedi | Bilgi amaçlı, aksiyon gerekmiyor. |

Bu turda 2 net/düşük riskli kod hatası bulundu ve düzeltildi (modal yarış durumu + cellSold çökmesi); detaylar `duzeltmeler.md`'de. 2 bulgu (VERİM DÖKÜMÜ gösterim tutarsızlığı, wizardStart bulut kontrolü eksikliği) tasarım kararı gerektirdiği için patrona bırakıldı. BUILD_TAG ve version.json `2026-08-19 00:58` olarak güncellendi.

## 2026-08-19 07:56 turu

| Anahtar | Zaman (UTC) | Özet | Sonuç |
|---|---|---|---|
| `-P-MhtiodoBI0jYlmwfW` | 2026-08-19 01:51 | ClaudeBot: BUG(high, veri kaybı riski) — boot resume yolu misafir auth önbellekteyken ama yerel kayıt boşken bulutu hiç geri yüklemiyor | Zaten çözülmüş. Bu, önceki turda patrona sorulan `wizardStart bulut kontrolü` konusuyla aynı kök neden — yerel geliştirici 19 Ağu'da `wizardStart()`'a bulut bekleme mantığını (satır ~7499-7530) eklemiş. Kod değişikliği gerekmedi. |
| `-P-Mhtt2gf8HTa6Sc6_o` | 2026-08-19 01:51 | ClaudeBot: BUG(kozmetik) — VERİM DÖKÜMÜ tablosu rüzgar için canlı hava verisi yokken uygulanan simülasyon çarpanı satırını göstermiyordu | Zaten çözülmüş. Yerel geliştirici 19 Ağu'da "🛰️ Canlı hava verisi yok — simülasyon çarpanı" satırını eklemiş (satır ~6949). Kod değişikliği gerekmedi. |
| `-P-Mu3WWZke_4R9idJc-` | 2026-08-19 02:47 | ClaudeBot: KRİTİK — 🏢 Şirket paneli her açılışta çöküyor, `resRow('liccap')` çağrısı RESEARCH nesnesinde artık olmayan bir anahtara erişiyor | **DÜZELTİLDİ.** Detay: `bot-notlar/duzeltmeler.md`. |
| `-P-Mu3dHLLun9HUF0Y71` | 2026-08-19 02:47 | ClaudeBot: giriş akışı — yerel auth var ama yerel kayıt yoksa splash direkt "Yeni Oyun" gösteriyor, bulut kaydı üzerine yazılabilir riski | Zaten çözülmüş (bkz. `-P-MhtiodoBI0jYlmwfW` satırı yukarıda) — aynı `wizardStart()` düzeltmesi bunu da kapsıyor. Kod değişikliği gerekmedi. |
| `-P-N6h9jUaLQpkXkF2ni` | 2026-08-19 03:45 | ClaudeBot: KRİTİK — aynı `resRow('liccap')` çökmesi, ikinci bağımsız bildirim, satır numarası ve önerilen düzeltme (`liccap`→`range`) dahil | **DÜZELTİLDİ** (yukarıdaki `-P-Mu3WWZke_4R9idJc-` ile birlikte, tek düzeltme). |
| `-P-N6kHoq78RNMt7etiB` | 2026-08-19 03:46 | ClaudeBot: durum özeti + kozmetik — VERİM DÖKÜMÜ gece eğrisi çarpanı virgüllü `0,00` yazıyor, diğerleri nokta kullanıyor | **DÜZELTİLDİ.** Detay: `bot-notlar/duzeltmeler.md`. |
| `-P-NLISzjHzdczygTt5g` | 2026-08-19 04:47 | ClaudeBot: KRİTİK — aynı `resRow('liccap')` çökmesi, üçüncü bağımsız bildirim | **DÜZELTİLDİ** (yukarıdaki `-P-Mu3WWZke_4R9idJc-` ile birlikte, tek düzeltme). |
| `-P-NLIchcCkza7Voazxz` | 2026-08-19 04:47 | ClaudeBot: durum özeti (bug değil) — ruhsat alımı, MERKEZ depodan satış, depo grubu doğru çalışıyor | Bilgi amaçlı, aksiyon gerekmiyor. |
| `-P-NYowJjUVB0y7UwbQT` | 2026-08-19 05:33 | ClaudeBot: durum özeti (bug değil) — bulut kayıt yüklendi, depo SAT tahmini UI ile eşleşti | Bilgi amaçlı, aksiyon gerekmiyor. |
| `-P-NjSU5JEdC9IsIUEMo` | 2026-08-19 06:38 | ClaudeBot: durum özeti (bug değil) — bildiren botun kendi tarayıcı/ağ ortamı sorunu, REST-only fallback ile veri bütünlüğü doğrulandı | Oyun koduyla ilgili değil, bildiren botun kendi test ortamı sorunu. Kod değişikliği yapılmadı. |
| `-P-Nz6X3sG1teI4WuzP7` | 2026-08-19 07:46 | ClaudeBot: durum özeti (bug değil) — MERKEZ depo SAT testi, komşu oyuncu tesis kartı doğru açıldı | Bilgi amaçlı, aksiyon gerekmiyor. |

Bu turda 1 kök nedenden 2 KRİTİK çökme bildirimi (Şirket paneli, `liccap`→`range`) ve 1 kozmetik biçim hatası (`0,00`→`0.00`) düzeltildi — toplam 2 kod değişikliği (kritik çökme tek düzeltme sayılır çünkü aynı satırdaki tek çağrıyı düzeltiyor). 2 bulgu önceki turda patrona sorulan konularla aynı kök nedene sahip olduğu için zaten yerel geliştirici tarafından çözülmüş bulundu, ek aksiyon gerekmedi. BUILD_TAG ve version.json `2026-08-19 07:56` olarak güncellendi.
