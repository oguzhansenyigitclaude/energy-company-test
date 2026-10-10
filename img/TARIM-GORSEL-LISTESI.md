# 🌾 Tarım — Katalog Görsel Listesi

**Nasıl çalışır:** `prototype/img/` klasörüne `<kod>.webp` adıyla at. Oyun otomatik algılar —
kartta küçük, "İncele" sayfasında büyük gösterilir. **Görsel yoksa emoji yedeği görünür**,
yani parça parça eklenebilir, hiçbir aşamada oyun bozulmaz.

Enerji tarafındaki `GORSEL-LISTESI.md` ile **aynı sistem**: `img/<kod>.webp` +
`onerror="this.remove()"`. Yeni kod yazılmadı, mevcut desen kullanılıyor.

**Boyut:** 768×768 kare. PNG üret → WebP'ye çevrilecek (enerji görselleri 40-110 KB bandında).

---

## 🎨 ORTAK STİL CÜMLESİ

Her promptun sonuna **aynen** eklenecek:

> `photorealistic, bright sunny day, blue sky with soft clouds, vibrant colors, Anatolian and Mediterranean farmland, professional agricultural photography, no text, no watermark, no people's faces`

Enerji setiyle aynı ışık ve doygunlukta olsun — oyuncu iki sektör arasında geçerken
**görsel dil değişmemeli**. Enerji cümlesinden tek farkı "energy industry" yerine
"agricultural" ve Anadolu/Akdeniz coğrafyası.

---

## 1 · ÜRÜNLER (5) — stok ve satış panelinde görünür

Ürün görselleri **hasat edilmiş hâli** olacak: tarlada değil, **sandıkta/çuvalda/yığın hâlinde**.
Çünkü bunlar envanter kalemi; tarlanın kendisi ayrı görsel.

| Kod | Ürün | Prompt konusu |
|---|---|---|
| `urun_bugday` ★ | Buğday | pile of golden wheat grain in burlap sacks, close-up, warm light |
| `urun_patates` ★ | Patates | freshly harvested potatoes in wooden crates, soil still on them |
| `urun_aycicek` | Ayçiçeği | sunflower seeds in sacks beside cut sunflower heads |
| `urun_misir` | Mısır | harvested corn cobs piled in crates, golden kernels |
| `urun_domates` | Domates | ripe red tomatoes in harvest crates, glossy |

## 2 · TARLALAR (6) — haritada ve tarla kartında

Tarla görseli **ekili ürüne göre değişir** — oyuncu haritaya bakınca ne ektiğini görsün.

| Kod | Ne | Prompt konusu |
|---|---|---|
| `tarla_bos` ★ | Sürülmüş boş tarla | freshly ploughed empty farmland field, brown soil furrows, aerial three-quarter view |
| `tarla_bugday` ★ | Buğday tarlası | golden wheat field ready for harvest, gentle wind, aerial three-quarter view |
| `tarla_patates` ★ | Patates tarlası | potato field with green leafy rows, aerial three-quarter view |
| `tarla_aycicek` | Ayçiçeği tarlası | field of blooming sunflowers facing the sun, aerial three-quarter view |
| `tarla_misir` | Mısır tarlası | tall green corn field in neat rows, aerial three-quarter view |
| `tarla_domates` | Domates tarlası | tomato field with staked plants and red fruit, aerial three-quarter view |

## 3 · BİNALAR (1 — dilim 1)

| Kod | Ne | Prompt konusu |
|---|---|---|
| `lojman` ★ | İşçi Lojmanı | modest two-storey farm workers housing building beside a field, clean and tidy, aerial three-quarter view |

---

## 🔜 SONRAKİ DİLİMLER — şimdi üretme, sıra gelince

Dilim 1 bitmeden bunlara kota harcama; tasarım değişirse boşa gider.

| Dilim | Kodlar |
|---|---|
| **Hayvancılık** | `ahir_inek`, `agil_koyun`, `kumes_tavuk`, `urun_sut`, `urun_et`, `urun_yumurta`, `urun_yun`, `urun_yem` — **yalnız inek, koyun, tavuk** (patron: at/eşek yok, getirisi yok) |
| **İşleme** | `fabrika_cips`, `fabrika_yag`, `fabrika_un`, `fabrika_sut`, `urun_cips`, `urun_yag`, `urun_un`, `urun_peynir` |
| **Lojistik** | `tir_kucuk`, `tir_orta`, `tir_buyuk`, `transfer_tarim` |
| **Depolama** | `soguk_depo`, `ambar` |

---

## ⚠️ ÜRETİM NOTLARI

1. **Metin yok.** Görselde yazı/logo çıkarsa telif ve dil sorunu olur (21 dil destekleniyor).
2. **Marka yok.** Gerçek bir traktör/tohum markası görünmesin.
3. **İnsan yüzü yok.** Uzaktan silüet olur, portre olmaz.
4. **Aynı açı.** Tarlalar ve binalar hep **havadan üç-çeyrek** açı — enerji setiyle tutarlı.
5. **Tanıdık ürün.** Katalog kuralı: oyuncu adını duyunca ne olduğunu biliyorsa girer (bkz. tasarım belgesi §4c). Nadir/egzotik bitki görseli üretme.
6. **★ işaretliler önce:** oyuncunun ilk saatte göreceği 6 görsel. Gerisi sonra gelebilir.

**Öncelik sırası (★ 6 adet):** `tarla_bos` · `tarla_bugday` · `tarla_patates` ·
`urun_bugday` · `urun_patates` · `lojman`

---

İlgili: `prototype/img/GORSEL-LISTESI.md` (enerji seti, 52 görsel) ·
`docs/tarim-sektoru-tasarim.md` · `docs/tarim-kod-sozlesmesi.md`
