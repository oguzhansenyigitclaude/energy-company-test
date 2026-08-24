# Günlük Gelişim Raporu (Bakım Botu)

Bu dosya, `suggestions` düğümüne yazan CLBOT1..CLBOT10 oyuncu-botlarının günlük büyüme/durum özetini ve o günkü bulgu/bug istatistiğini tutar. En yeni tur en üstte.

## 2026-08-24 21:13 turu

**Bulgu istatistiği:** 135 yeni kayıt geldi (2026-08-23 21:24 – 2026-08-24 20:26 UTC). 1'i gerçek/net kod hatasıydı ve düzeltildi (`placeAt()` `PENDING` boşken çökmesi, 4 bağımsız bot — `countryAt()` ailesiyle aynı kalıp, yalnız botların kendi debug-hook çağrılarında tetikleniyor). 1'i yeni bir izlenecek konu olarak patrona soruldu (depo SAT tahmini ↔ gerçekleşen arasında %2-%57 değişken sapma, 7 bağımsız bot — muhtemelen boşaltma süresince fiyat/talep değişimi, tasarım olabilir). ~19 kayıt zaten bilinen ailelerin (`MAX_PER_CELL` sessiz-red, ağ/proxy ortam sorunu) tekrarıydı ya da botların kendilerinin "bug değil" diye işaretlediği kayıtlardı. Kalan 114 kayıt saf GELİŞİM/sıralama özetiydi.

**CLBOT büyümesi (turun ilk → son kaydı):**

| Bot | Kayıt | Sıralama (ilk → son) | Şirket değeri (ilk → son) | Tesis (son) | Not |
|---|---|---|---|---|---|
| CLBOT1 | 14 | #10/23 → #10/43 | $63,54M → $71,3M | 19 | Hedef "Claude Enerji" ($78,79M), fark ~$7,5M'ye düştü |
| CLBOT2 | 14 | #16/23 → #13/43 | $50,05M → $52,1M | 16 | |
| CLBOT3 | 13 | #17/23 → #15/43 | $43,99M → ~$54,8M | 16 | Kasa çok düştü (79,9K), bir sonraki tur yatırım önceliği |
| CLBOT4 | 13 | #15/23 → #16/43 | $42,79M → $48,0M | 20 | |
| CLBOT5 | 13 | #20/23 → #14/43 | $38,68M → ~$54,8M | 17 | |
| CLBOT6 | 14 | #14 → #8/~28 | ~$50,2M (gün içi zirve ~$52,6M) → **$11,66M (RESTART)** | 4 | 72 saatlik devriye doldu, Adana'da sıfırdan yeni şirket kuruldu |
| CLBOT7 | 14 | #18 → #8/~28 | ~$44,8M (gün içi zirve ~$48,3M) → **$11,65M (RESTART)** | 4 | Aynı restart, Konya'da yeniden kuruldu |
| CLBOT8 | 13 | #17 → #8/~28 | ~$47,6M (gün içi zirve ~$49,7M) → **$11,65M (RESTART)** | 4 | Aynı restart, Samsun'da yeniden kuruldu |
| CLBOT9 | 14 | #16 → #8/~28 | ~$47,6M (gün içi zirve ~$50,5M) → **$11,66M (RESTART)** | 4 | Aynı restart, Trabzon'da yeniden kuruldu |
| CLBOT10 | 13 | #19 → #8/~28 | ~$45,0M (gün içi zirve ~$51,9M) → **$11,65M (RESTART)** | 4 | Aynı restart, Gaziantep'te yeniden kuruldu |

Not: CLBOT1-5 (43 katılımcılı havuz) gün boyu istikrarlı büyüdü, hiçbiri gerilemedi. CLBOT6-10 ise gün içinde ~$48-53M'ye kadar büyüdükten sonra, günün son kaydında (20:26) 72 saatlik devriyeleri dolup **aynı şehirlerde ($10M başlangıç kasasıyla) sıfırdan yeni şirket kurdu** — bu, 22 Ağustos'taki CLBOT1-5 restart'ıyla aynı beklenen döngüsel davranış (bkz. "2026-08-22 21:07 turu" notu aşağıda), bug değil. Yeni havuzda (~28 katılımcı) 5 bot da aynı anda #8 sırada başladı, hedef "Solaris Group" (~$90K fark).

Kod değişikliği: `placeAt()` `PENDING` boşken çökme koruması (bkz. `duzeltmeler.md`). BUILD_TAG/version.json `2026-08-24 21:13` olarak güncellendi.

## 2026-08-23 21:08 turu

**Bulgu istatistiği:** 140 yeni kayıt geldi (2026-08-22 21:24 – 2026-08-23 20:25 UTC). 1'i gerçek/net kod hatasıydı ve düzeltildi (`countryAt()` çökmesi, 3 bağımsız bot). 1'i yeni ve ciddi bir konu olarak patrona soruldu (RTDB yazma 401 Unauthorized / veri kaybı riski, 2 bağımsız bot). ~20 kayıt zaten bilinen bug ailelerinin (bulut-kaydı-throttle, vendor/leaflet 404, `placeAt()` sessiz-red) tekrarıydı ya da botların kendi test aracı/kararı sorunuydu (bkz. `islenen.md` ve `patrona-sorulacak.md` için detay). Kalan 109 kayıt saf GELİŞİM/sıralama özetiydi.

**CLBOT büyümesi (turun ilk → son kaydı):**

| Bot | Kayıt | Sıralama (son) | Şirket değeri (ilk → son) | Tesis (son) |
|---|---|---|---|---|
| CLBOT1 | 16 | #10/43 | ~$16,88M → $60.057.838 | 9+2depo |
| CLBOT2 | 14 | #17/43 | ~$14,20M → $42.756.054 | 7+2depo |
| CLBOT3 | 14 | #18/43 | ~$13,62M → $37.848.659 | 5+3depo |
| CLBOT4 | 16 | — | ~$14,22M → $41,99M | 10 depo dahil |
| CLBOT5 | 13 | #20/43 | ~$15,65M → $33.910.558 | 8+1depo |
| CLBOT6 | 17 | #18/23 | Rank #5 (22Ağu) → $42.280.904 | 9 |
| CLBOT7 | 13 | #14/23 | Rank #13 (22Ağu) → $47.102.899 | 10 |
| CLBOT8 | 12 | #12/23 | Rank #13 (22Ağu) → $52.137.513 | 9 |
| CLBOT9 | 13 | #13/23 | Rank #13 (22Ağu) → $48.046.490 | 12 |
| CLBOT10 | 12 | #19/23 | Rank #13 (22Ağu) → $39.644.021 | 9 |

Not: CLBOT1-5, CLBOT6-10'dan farklı bir yarış/sıralama havuzunda (43 vs 23 katılımcı) — aralarındaki sıralama numaraları doğrudan karşılaştırılamaz. Tüm botlar bu 23 saatlik pencerede şirket değerini büyütmeye devam etti; hiçbiri gerilemedi (bazı turlarda kasa yüksek yatırım yüzünden düştü, beklenen davranış).

Kod değişikliği: `countryAt()` çökme koruması (bkz. `duzeltmeler.md`). BUILD_TAG/version.json `2026-08-23 21:08` olarak güncellendi.

## 2026-08-22 21:07 turu

**Bulgu istatistiği:** 21 yeni kayıt geldi, 0'ı gerçek kod hatasıydı (hepsi GELİŞİM özeti / devriye durumu; bkz. `islenen.md`). 1 tekrarlayan kozmetik gözlem (leaflet 404'leri) zaten bilinen konuya ek kanıt oldu, koda dokunulmadı.

**CLBOT büyümesi:**

| Bot | Şehir | Tur | Kasa (önce → sonra) | Şirket değeri | Sıralama | Tesis |
|---|---|---|---|---|---|---|
| CLBOT1 | Kocaeli | 19 (yeni kuruldu) | — → 10.077.223 TL | — | — | 3 (kuruluş hediyesi) |
| CLBOT2 | İzmir | 19 (yeni kuruldu) | — → 10.015.146 TL | — | — | 3 (kuruluş hediyesi) |
| CLBOT3 | Ankara | 19 (yeni kuruldu) | — → 10.011.116 TL | — | — | 3 (kuruluş hediyesi) |
| CLBOT4 | Bursa | 19 (yeni kuruldu) | — → 10.014.015 TL | — | — | 3 (kuruluş hediyesi) |
| CLBOT5 | Antalya | 19 (yeni kuruldu) | — → 10.025.009 TL | — | — | 3 (kuruluş hediyesi) |
| CLBOT6 | Adana | 18→20 | 10.026.180 → 1.090.683 TL | 12.500.683 TL | 11/22 | 4 (büyük santral alımı) |
| CLBOT7 | Konya | 18→20 | 10.011.629 → 111.629 TL | 11.661.629 TL | 22/22 | 4 |
| CLBOT8 | Samsun | 18→20 | 10.016.048 → 116.048 TL | 11.666.048 TL | 21/22 | 4 |
| CLBOT9 | Trabzon | 18→20 | 10.016.579 → 116.579 TL | 11.666.579 TL | 20/22 | 4 |
| CLBOT10 | Gaziantep | 18→20 | 10.022.553 → 122.553 TL | 11.672.553 TL | 19/22 | 4 |

Not: CLBOT1-5'in önceki 72 saatlik devriyesi bu gün içinde bitti (`[BOT FINAL] 72 saat doldu`) ve yeni şirketlerle sıfırdan başladılar; CLBOT6-10 ise aynı devriyeye devam edip bu turda büyük tesis yatırımı yaparak kasalarını şirket değerine dönüştürdü (kasa düştü, toplam değer arttı — beklenen davranış).

Kod değişikliği yapılmadı, BUILD_TAG/version.json güncellenmedi.
