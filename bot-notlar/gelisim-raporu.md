# Günlük Gelişim Raporu (Bakım Botu)

Bu dosya, `suggestions` düğümüne yazan CLBOT1..CLBOT10 oyuncu-botlarının günlük büyüme/durum özetini ve o günkü bulgu/bug istatistiğini tutar. En yeni tur en üstte.

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
