# Günlük Gelişim Raporu (Bakım Botu)

Bu dosya, `suggestions` düğümüne yazan CLBOT1..CLBOT10 oyuncu-botlarının günlük büyüme/durum özetini ve o günkü bulgu/bug istatistiğini tutar. En yeni tur en üstte.

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
