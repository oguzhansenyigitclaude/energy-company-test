# Düzeltmeler (Öncesi / Sonrası)

Bu dosya, bakım botunun bu turda index.html üzerinde yaptığı düzeltmelerin ÖNCE/SONRA kod parçalarını ve gerekçesini listeler. Yerel geliştirici bunları ana projeye taşıyabilir / gözden geçirebilir.

## 2026-08-24 21:13 turu

### Düzeltme 1: `placeAt()` `PENDING` boşken çöküyordu (`TypeError: Cannot destructure property 'kind' of 'PENDING' as it is null`)

**Bulgu:** `-P-meKO5DSvsGxBLnR7i` (CLBOT2), `-P-meKR_0oDgyUVx16YS` (CLBOT3), `-P-meKWUJgJxwzYC7LWw` (CLBOT4), `-P-meKaAFf4vXxlswlyI` (CLBOT5) — 4 bağımsız bot, aynı gün ~07:19 civarı, aynı script hatasını bildirdi:
```
TypeError: Cannot destructure property 'kind' of 'PENDING' as it is null.
    at placeAt (index.html:3237:...)
```

**Kök neden:** `placeAt(c, ll)` (satır ~3236) hiç kontrolsüz `const { kind, t } = PENDING;` ile başlıyordu. Tek gerçek çağrı noktası olan `mapTapAt()` (satır 7030) `if (PENDING) placeAt(...)` ile zaten koruyor, yani **gerçek oyuncu haritaya tıklama akışında bu satıra `PENDING` boşken ulaşılamaz**. Önceki turda aynı ailede tespit edilen `countryAt()` çökmesiyle (bkz. aşağıdaki "2026-08-23 21:08 turu") aynı kalıp: botlar `window.__setSel`/`window.__addUnit` debug hook'larını kullanan kendi `tryBuild()` test fonksiyonları üzerinden — `doBuild()` (PENDING'i dolduran tek yer) çağrılmadan — `placeAt`'e denk gelen bir yola ulaşınca çöküyor. Yine de savunma ucuz ve risksiz; PENDING boşsa artık sessizce moddan çıkıyor.

**ÖNCESİ** (`index.html`, satır 3236-3238):
```js
function placeAt(c, ll) {
  const { kind, t } = PENDING;
  const T = TYPE_OF(kind, t);
```

**SONRASI:**
```js
function placeAt(c, ll) {
  // 🐞 [BOT-FIX] PENDING boşken (gerçek oyuncu akışında ulaşılamaz — yalnız botların kendi
  // test/debug çağrılarında görüldü, bkz. bot-notlar/duzeltmeler.md) çökme yerine sessizce çık.
  if (!PENDING) { setMode(null); return; }
  const { kind, t } = PENDING;
  const T = TYPE_OF(kind, t);
```

**Neden düşük risk / ekonomiyi değiştirmiyor:** Tek eklenen şey bir erken-çıkış koşulu. Gerçek oyuncu akışında `placeAt` her zaman `PENDING` doluyken çağrıldığından (`mapTapAt` zaten kontrol ediyor) bu dal hiç tetiklenmez, davranış birebir aynı kalır. Sadece bot/test çağrılarında (veya ileride oluşabilecek benzer bir durumda) çökme yerine modu temizleyip sessizce çıkıyor — kasa, ekonomi, satın alma mantığına dokunulmadı.

**Doğrulama:** `new Function` ile tek inline `<script>` bloğu sözdizim kontrolünden geçti; `npm i playwright` + mevcut `/opt/pw-browsers` Chromium ile yerel `http://` sunucu üzerinden sayfa açılışında `pageerror`/`console.error` üretmedi.

---

## 2026-08-23 21:08 turu

### Düzeltme 1: `countryAt()` bozuk/eksik koordinatta çöküyordu (`TypeError: ... reading 'toFixed'`)

**Bulgu:** `-P-jQwvsVFpWg9ACOaip` (CLBOT7), `-P-jQwwGGNJ0DoOF9k9W` (CLBOT6), `-P-jQze1vpJTmg9nh2TH` (CLBOT9) — 3 bağımsız bot, aynı gün 16:24 civarı, `page.evaluate` üzerinden kendi `tryBuild()` test fonksiyonlarının debug hook'ları (`window.__setSel`/`window.__addUnit`) aracılığıyla `placeAt()`'i çağırırken aynı hatayı bildirdi:
```
TypeError: Cannot read properties of undefined (reading 'toFixed')
    at countryAt (index.html:830:21)
    at oyunDisiMi (index.html:680:38)
    at placeAt (index.html:3236:7)
```

**Kök neden:** `countryAt(ll, pay)` (satır ~827) yalnız `ll`'nin var olup olmadığını kontrol ediyordu (`!ll`), ama `ll[0]`/`ll[1]`'in gerçekten sayı olup olmadığını değil. Botlar debug/test API'sini (`window.__setSel(c, ll)`) yanlış bir `ll` değeriyle (muhtemelen bir hücre-id ya da eksik koordinat) çağırınca `ll[0].toFixed(2)` doğrudan `undefined.toFixed` şeklinde patlıyordu. **Gerçek oyuncu akışında bu koda ulaşılamaz** — tek çağrı noktası olan `placeAt(SEL.cell, SEL.ll)` (satır 6818), `SEL.ll`'yi ya Leaflet tıklama olayından (`[e.latlng.lat, e.latlng.lng]`, satır 6814) ya da hücre merkezinden (`cellCenter()`, her zaman geçerli sayı çifti döndürür, satır 7646) alır — hiçbir zaman eksik/bozuk olmaz. Yine de test/debug API'sinin sağlamlaştırılması ucuz ve risksiz.

**ÖNCESİ** (`index.html`, satır 827-829):
```js
function countryAt(ll, pay) {
  if (!LAND_BOX || !ll) return null;
  const t = pay === undefined ? 0.12 : pay; // ~12 km
```

**SONRASI:**
```js
function countryAt(ll, pay) {
  // 🐞 [BOT-FIX] ll[0]/ll[1] sayı değilse (bozuk/eksik koordinat) eskiden burada çöküyordu.
  if (!LAND_BOX || !ll || typeof ll[0] !== 'number' || typeof ll[1] !== 'number') return null;
  const t = pay === undefined ? 0.12 : pay; // ~12 km
```

**Neden düşük risk / ekonomiyi değiştirmiyor:** Yalnızca yeni bir erken-çıkış koşulu eklendi; gerçek oyuncu çağrılarında `ll` her zaman `[sayı, sayı]` olduğundan bu dal hiç tetiklenmez, davranış birebir aynı kalır. Bozuk girdi artık çökme yerine `null` (bilinmeyen ülke) döndürüyor — `oyunDisiMi()` zaten `null` sonucu "oyun dışı değil" olarak ele alıyor (güvenli varsayılan).

**Doğrulama:** `new Function` ile tek inline `<script>` bloğu sözdizim kontrolünden geçti; Playwright (headless Chromium, yerel `http://` sunucu) sayfa açılışında `pageerror`/`console.error` üretmedi; ayrıca `countryAt([41.01])` (eksik ikinci koordinat) ile manuel test edilip artık çökmeden `null` döndürdüğü doğrulandı.

## 2026-08-21 14:58 turu

### Düzeltme 1: `sellAll(frac)` (Pazar paneli "Tüm Depoları Sat"/"Yarısını Sat") bağlı uydu depoyu da sayıyordu

**Bulgu:** `-P-Yv5Tg9gJM972CAFNh` (10:44) — bot, 00:54 build'te `sellAllStores()`'da düzeltilen "bağlı depo filtresi eksik" bug'ının bir kardeş fonksiyonda (`sellAll(frac)`, satır ~2652 — Pazar panelindeki "Tüm Depoları Sat"/"Yarısını Sat" butonlarının kullandığı fonksiyon) hâlâ mevcut olduğunu bildirdi. Repro: aynı karede 2 depo birbirine bağlanır (0 ücret, aynı il), bağlı uydu depoya stok eklenir, "Tüm Depoları Sat" tetiklenir → toast "2 depo boşaltılıyor" der, ama bağlı uydu depo `disch=null` kalır çünkü `startDischarge()` (satır ~2590) onu `st.link` kontrolüyle sessizce reddeder. Kayıp/hatalı gelir YOK (merkez depo grup toplamını zaten doğru satıyor), ama gösterilen "N depo" sayacı gerçek satılan depo sayısından fazla — yanıltıcı toast.

**Kök neden:** `sellAllStores()`'daki (`[BOT-FIX b25679a]`) düzeltme yalnız o fonksiyona uygulanmıştı; aynı desendeki ikinci çağrı noktası `sellAll(frac)` gözden kaçmış.

**ÖNCESİ** (`index.html`, ~satır 2652-2658):
```js
function sellAll(frac) {
  let n = 0;
  for (const st of stores())
    if (!st.disch && st.buildDone <= Date.now() && st.stored * frac >= .005) { startDischarge(st.id, frac, null, null, true); n++; }
  if (n) { toast('⚡ ' + n + ' depo boşaltılıyor — para satış aktıkça hesabına geçer', ''); beep(600, .08); }
  else toast('Boşaltılacak dolu depo yok', 'warn');
}
```

**SONRASI:**
```js
function sellAll(frac) {
  let n = 0;
  // 🐞 [BOT-FIX] sellAllStores()'da (b25679a/f5a9750) düzeltilen "!link filtresi eksik" bug'ının
  // aynısı burada da vardı: BAĞLI (uydu) depo doğrudan seçilince startDischarge() onu sessizce
  // reddediyor (satır ~2590), ama n sayaca girdiği için toast yanlış sayı gösteriyordu.
  // Grup enerjisi zaten MERKEZ (hub) depo üzerinden satılıyor (dischargeTick grup-çekme mantığı) —
  // uyduyu ayrıca seçmeye gerek yok. Artık yalnız bağlı olmayan depolar seçiliyor.
  for (const st of stores())
    if (!st.disch && !st.link && st.buildDone <= Date.now() && st.stored * frac >= .005) { startDischarge(st.id, frac, null, null, true); n++; }
  if (n) { toast('⚡ ' + n + ' depo boşaltılıyor — para satış aktıkça hesabına geçer', ''); beep(600, .08); }
  else toast('Boşaltılacak dolu depo yok', 'warn');
}
```

**Neden düşük risk / ekonomiyi değiştirmiyor:** Tek eklenen koşul `!st.link` — reddedilen bağlı uydu depo zaten `startDischarge()` içinde işlem yapmıyordu (kasa/stored hiç değişmiyordu), bu değişiklik sadece onu döngüden ve sayaçtan çıkarıyor. Merkez (hub) depo `isLinked` olduğunda grup toplamını `dischargeTick`'teki mevcut cross-draw mantığıyla zaten satıyor (satır ~2604-2619, dokunulmadı) — hiçbir satış yolu/miktarı değişmedi. Bota önerilen tek satırlık düzeltme (`&& !st.link`) birebir uygulandı.

**Doğrulama:** `new Function` ile inline `<script>` sözdizim kontrolü geçti; Playwright (headless Chromium, yerel `http://` sunucu üzerinden) sayfa açılışında `pageerror` üretmedi (yalnızca bilinen bir statik kaynak 404 uyarısı var, JS hatası yok).

## 2026-08-21 00:54 turu

### Düzeltme 1: "Hepsini Sat" bağlı (uydu) depo varken hiçbir şey satmıyordu (sessiz başarısızlık)

**Bulgu:** `-P-WLwOAS1WqbHNR6_ZV`, `-P-WLwWINVDojrNq2QeR`, `-P-WLwdOLFWJltkTWDht`, `-P-WM3TUYnd_9Itn_I7O` (22:47-22:48), `-P-WmDhLvA3Z04zFNxsW`, `-P-WmDu3n3nNY2QJffa0` (00:46) — toplam 3 bağımsız devriye turu aynı bug'ı bildirdi. Repro: merkez depoya bağlı bir uydu depo olsun, merkezin kendi tankı ~boşken uydu doluyken "Hepsini Sat" → onay ekranı doğru grup toplamını gösteriyor ("1 depo · tahmini $X"), "Evet" sonrası kasa deltası TAM 0, hiçbir depo boşalmıyor, ama "N depo boşaltılıyor — para akmaya başladı" toast'ı yine de çıkıyor (oyuncu parasının geldiğini sanıyor).

**Kök neden:** Önceki turda (`b25679a`) sadece onay ekranını dolduran `sellAllStoresAsk()` düzeltilmişti — bağlı depoları hariç tutup (`!s.link`) grup toplamına (`isLinked(s) ? groupStored(s) : s.stored`) göre filtreliyor. Ama asıl satışı tetikleyen `sellAllStores()` (satır ~7035) hâlâ eski filtreyi kullanıyordu: `s.stored >= .005`, `!s.link` kontrolü YOK. Böylece onay ekranı merkezi (tankı boş) elip uyduyu (dolu) seçtiği için doğru tutarı gösteriyor, ama gerçek satış fonksiyonu da uyduyu seçip `startDischarge()`'a gönderiyor — `startDischarge()` ise `st.link` kontrolüyle ("🔗 Bu depo bir BAĞLI DEPO" toast'ı, satır ~2590) bağlı depoları sessizce (`quiet:true` olduğu için toast'sız) reddediyor. Sonuç: seçilen depo sayısı > 0 ("N depo boşaltılıyor" yazıyor) ama gerçek satılan depo sayısı 0.

**ÖNCESİ** (`index.html`, ~satır 7035-7040):
```js
window.sellAllStores = () => {
  const sts = stores().filter(s => s.buildDone <= Date.now() && !s.disch && s.stored >= .005);
  sts.forEach(s => startDischarge(s.id, 1, null, null, true)); // quiet: tek tek bildirim yağmuru olmasın
  toast('💰 ' + sts.length + ' depo boşaltılıyor — para akmaya başladı', '', null, 0, true);
  save(); renderNav();
};
```

**SONRASI:**
```js
window.sellAllStores = () => {
  // 🐞 [BOT-FIX] onay ekranı (sellAllStoresAsk) BAĞLI (uydu) depoları hariç tutup grup toplamını
  // sayıyordu, ama asıl satışı yapan burası hâlâ TÜM depoları (uydu dahil) "s.stored" ile seçiyordu —
  // startDischarge() uyduyu sessizce reddettiği için "N depo boşaltılıyor" diyip HİÇ satış olmuyordu.
  // Artık onay ekranıyla AYNI filtre: yalnız bağlı olmayan depolar, grup toplamına göre.
  const sts = stores().filter(s => s.buildDone <= Date.now() && !s.disch && !s.link && (isLinked(s) ? groupStored(s) : s.stored) >= .005);
  sts.forEach(s => startDischarge(s.id, 1, null, null, true)); // quiet: tek tek bildirim yağmuru olmasın
  toast('💰 ' + sts.length + ' depo boşaltılıyor — para akmaya başladı', '', null, 0, true);
  save(); renderNav();
};
```

**Neden düşük risk / ekonomiyi değiştirmiyor:** Filtre, zaten canlıda çalışan ve doğrulanmış `sellAllStoresAsk()` filtresiyle birebir aynı — yeni bir kural eklemiyor, sadece iki fonksiyonu tutarlı hale getiriyor. Satış fiyatı/miktar hesabı (`sellRevenueAfter`, `startDischarge`) değişmedi; yalnızca HANGİ depoların bu toplu işleme dahil edileceği (bağlı olmayanlar) düzeltildi. Tekil "SAT" butonu (satır ~7004, `sellUnit`) zaten etkilenmiyordu, bu turda da dokunulmadı.

**Doğrulama:** `new Function` ile inline `<script>` sözdizim kontrolü geçti; Playwright (headless Chromium, yerel `http://` sunucu üzerinden) sayfa açılışında `pageerror` üretmedi (yalnızca bilinen/önceden not edilmiş statik kaynak 404'leri var, JS hatası yok).

---

## 2026-08-20 21:56 turu

### Düzeltme 1: Hoşgeldin (yönetim raporu) modalında dakika dalı "dakikate" gibi hatalı ek alıyordu

**Bulgu:** `-P-VUYQ1uPTScLdXCXoI` (18:45). <60 dk yoklukta Hoşgeldin modalındaki ledger satırı "Üretilen enerji ... (51 dakikate)" yazıyor; doğrusu "dakikada" olmalı. Saat dalı ("1.5 saatte") zaten doğru. Repro: oyunu kapat, 1-59 dk sonra aç, hoşgeldin tablosundaki "Üretilen enerji" satırına bak.

**Kök neden:** `away` değişkeni "51 dakika" / "1.5 saat" gibi çıplak bir süre metni üretiyor (satır ~6110). Ledger satırında (satır ~6115) bu metne durum eki olarak sabit `'te'` harfi ekleniyordu: `${away.replace(' ',' ')}te)`. "Saat" ünsüzle bitip düzensiz bir kelime olduğundan +te doğru sonuç veriyor ("saatte"), ama "dakika" ünlü ile bittiğinden Türkçe'de +da eki alması gerekiyor ("dakikada") — sabit "te" burada "dakikate" gibi hatalı bir sonuç üretiyordu.

**ÖNCESİ** (`index.html`, `showWelcomeReport` benzeri fonksiyon içinde, ~satır 6110-6115):
```js
const away = gapSec >= 3600 ? (gapSec / 3600).toFixed(1) + ' saat' : Math.round(gapSec / 60) + ' dakika';
$('modal').innerHTML = `
  <h2>👋 Hoşgeldiniz!</h2>
  <p style="text-align:center" class="sub"><b>${G.company}</b> yönetim raporu — ${away} uzaktaydın</p>
  <table class="ledger">
    <tr><td>⚡ Üretilen enerji</td><td><b>${fmtE(produced)} MWh</b> (${away.replace(' ', ' ')}te)</td></tr>
```

**SONRASI:**
```js
const away = gapSec >= 3600 ? (gapSec / 3600).toFixed(1) + ' saat' : Math.round(gapSec / 60) + ' dakika';
// [BOT-FIX] "dakika" ünlüyle bitiyor (-da eki), "saat" ünsüzle biter ve düzensiz (-te eki) — tek "te" eki her ikisine de uygulanınca "51 dakikate" gibi hatalı çıkıyordu.
const awayIn = gapSec >= 3600 ? away + 'te' : away + 'da';
$('modal').innerHTML = `
  <h2>👋 Hoşgeldiniz!</h2>
  <p style="text-align:center" class="sub"><b>${G.company}</b> yönetim raporu — ${away} uzaktaydın</p>
  <table class="ledger">
    <tr><td>⚡ Üretilen enerji</td><td><b>${fmtE(produced)} MWh</b> (${awayIn})</td></tr>
```

**Doğrulama:** Tüm inline `<script>` blokları `new Function()` ile sözdizimi kontrolünden geçti (0 hata). `npx playwright` ile Chromium'da sayfa yerel http sunucu üzerinden açıldı, `pageerror` yok (yalnız oyundan bağımsız, önceden var olan `img/splash.webp` 404'ü görüldü, düzeltmeyle ilgisi yok).

## 2026-08-20 14:59 turu

### Düzeltme 1: "Hepsini Sat" onay penceresi bağlı (uydu) depo stoğunu saymıyordu — tahmin gerçekten ~%20 düşük çıkıyordu

**Bulgu:** `-P-TYIPb85XcfS0O-6qc` (09:42). Depo panelinde "Hepsini Sat" butonu ör. $144.343 (1 depo) yazıyor, ama tıklayınca açılan onay penceresi AYNI işlem için 0,64 MWh / tahmini $111.917 gösteriyor (~%22 fark). Repro: sağ çekmece (navhandle) → DEPOLARIN → Hepsini Sat.

**Kök neden:** Panel butonunu üreten `renderStorePanel()` (satır ~6757) her depo için `dolu(s) = isLinked(s) ? groupStored(s) : s.stored` kullanıyor — yani bağlı MERKEZ depo için tüm grubun (merkez + uydular) stoğunu sayıyor. Gerçek satış `startDischarge()` (satır ~2430) da aynı grup toplamını satar. Ama onay penceresi `sellAllStoresAsk()` sadece `s.stored` (merkezin kendi tankı) kullanıyordu; bağlı uydunun stoğu (ör. id6, ~0,18 MWh) tahmine hiç girmiyordu. Sonuç: panel butonu ve gerçek satış grup toplamını gösterirken, arada duran onay penceresi eksik rakam gösteriyordu — "gördüğün = aldığın" sözü bağlı depolu merkezlerde bozuluyordu.

**ÖNCESİ** (`index.html`, `sellAllStoresAsk` içinde, ~satır 6807):
```js
const kareSatilan = {};
const toplam = sts.reduce((a, s) => {
  const onceki = kareSatilan[s.cell] || 0;
  kareSatilan[s.cell] = onceki + s.stored;
  return a + sellRevenueAfter(s.cell, s.stored, onceki).gain;
}, 0);
const mwh = sts.reduce((a, s) => a + s.stored, 0);
```

**SONRASI:**
```js
// 🐞 BOT DÜZELTMESİ: onay penceresi BAĞLI depoların stoğunu saymıyordu — merkez depo için
// s.stored (yalnız kendi tankı) kullanılıyordu, oysa panel butonu (renderStorePanel) ve
// gerçek satış (startDischarge) grup toplamını (groupStored) satıyor. Sonuç: bağlı uydusu
// olan merkezde onaydaki tahmin gerçekten ~%20 düşük çıkıyordu ("gördüğün = aldığın" bozuluyordu).
// Çözüm: panel/satışla aynı `dolu` (isLinked ? groupStored : stored) değeri kullanılıyor.
const kareSatilan = {};
const dolu = s => isLinked(s) ? groupStored(s) : s.stored;
const toplam = sts.reduce((a, s) => {
  const onceki = kareSatilan[s.cell] || 0;
  kareSatilan[s.cell] = onceki + dolu(s);
  return a + sellRevenueAfter(s.cell, dolu(s), onceki).gain;
}, 0);
const mwh = sts.reduce((a, s) => a + dolu(s), 0);
```

**Değişiklik:** `s.stored` → `dolu(s)` (üç yerde) + `dolu` yardımcı fonksiyonu eklendi. Salt gösterim düzeltmesi — ekonomi/satış mantığı değişmedi (gerçek satış zaten grup toplamını satıyordu); yalnız onay penceresindeki tahmin artık gerçekten alınacak tutarla eşleşiyor.

**Doğrulama:** `new Function` ile 4 inline `<script>` bloğunun tamamı sözdizimi kontrolünden geçti (0 hata). Playwright + headless Chromium (`/opt/pw-browsers/chromium`) ile sayfa `file://` üzerinden açıldı: `pageerror` = 0, `typeof window.sellAllStoresAsk` = "function". Bu tur bağlı uydu deposunun stoğu artık hem panel butonu hem onay penceresinde aynı `dolu` değerinden hesaplanıyor.

---

## 2026-08-19 21:56 turu

### Düzeltme 1: `demolishDo()` bağlı depo koruması eksikti (konsoldan çağrıyla grup kilidi bypass edilebiliyordu)

**Bulgu:** `-P-QJxsC30FPkL3aeHNF` (ClaudeBot, 18:40) ve `-P-QlT1J-JcZyQyEjnGe` (ClaudeBot, 20:44) — aynı kök nedenin iki bağımsız bildirimi. `demolishUnit(id)` (satış onayı akışı, UI butonu) bağlı depoyu doğru şekilde engelliyor, ama gerçek silme/ödeme işlemini yapan `demolishDo(id)` fonksiyonunda bu kontrol yoktu. Konsoldan doğrudan `demolishDo(6)` çağrılınca bağlı (linkli) depo, bağlantı kesilmeden ve hat ücreti ödenmeden sessizce satılıyor, içindeki enerji siliniyor ve iade kasaya geçiyordu.

**Kök neden:** Grup kilidi kontrolü (`u.kind === 'store' && isLinked(u)`) sadece `demolishUnit()` içindeki onay-modalı açan yolda vardı (satır ~7418); asıl mutasyonu yapan `demolishDo()` bu kontrolü tekrarlamıyordu.

**ÖNCESİ** (`index.html`, ~satır 7429):
```js
window.demolishDo = id => {
  const i = G.units.findIndex(x => x.id === id);
  if (i < 0) return;
  const u = G.units[i];
  if (u.kind === 'office' && u.hq) return;
  const r = resaleValue(u);
```

**SONRASI:**
```js
window.demolishDo = id => {
  const i = G.units.findIndex(x => x.id === id);
  if (i < 0) return;
  const u = G.units[i];
  if (u.kind === 'office' && u.hq) return;
  // 🔗 GRUP KİLİDİ: demolishUnit() (satır ~7418) bu kontrolü yapıyor ama demolishDo()
  // konsoldan doğrudan çağrılabildiği için aynı korumaya burada da ihtiyaç var.
  if (u.kind === 'store' && isLinked(u)) return;
  const r = resaleValue(u);
```

**Etki:** Mevcut "bağlı depo tek başına satılamaz" kuralını (ekonomi/dengeyi DEĞİŞTİRMEDEN) zaten var olan UI yolunun dışına da genişletiyor — ekonomiye yeni bir kural eklemiyor, sadece var olan korumadaki bir bypass deliğini kapatıyor. Doğrulama: tüm inline `<script>` blokları `new Function` ile sözdizimi kontrolünden geçti; Playwright (headless Chromium, hem `file://` hem yerel `http://` sunucu üzerinden) sayfa açılışında `pageerror` üretmedi.

## 2026-08-19 14:58 turu

### Düzeltme 1: VERİM DÖKÜMÜ tablosu "yeni şirket takviyesi" (NEWBIE_WIND/NEWBIE_CLOUD) çarpanını göstermiyordu

**Bulgu:** `-P-P3ghTOJp9tTk1ImXv` (ClaudeBot, 12:50) — RES T-200 Onshore #1 için tabloda ×0.53(rüzgar)×0.43(sağlamlık)×2MW=0.45MW gösteriliyor ama gerçek Anlık üretim 0.66MW. Aynı eksikliğin güneş/bulut tarafında (NEWBIE_CLOUD) da olduğu bildirildi.

**Kök neden:** `plantOutput()` canlı hava verisi varken rüzgara `+NEWBIE_WIND*newbieAid()` takviyesi ekliyor, bulutu da `(1-NEWBIE_CLOUD*newbieAid())` ile azaltıyor (yeni şirketlere düşük üretimde tanınan başlangıç desteği). VERİM DÖKÜMÜ tablosu ise bu iki satır için ham `wx2.w` / `wx2.cloud` değerlerini kullanıyordu — takviye tabloya hiç yansımıyordu, "çarpımların sonucu = anlık üretim" iddiası tutmuyordu.

**ÖNCESİ** (`index.html`, VERİM DÖKÜMÜ satırları):
```js
if (wx2.day) rows.push(['☁️ Bulut %' + Math.round(wx2.cloud || 0), '×' + Math.max(.05, 1 - (wx2.cloud || 0) / 100).toFixed(2)]);
} else rows.push(['🛰️ Canlı hava verisi yok — simülasyon çarpanı', '×' + sunFactor(q2).toFixed(2)]);
} else if (T.kind === 'wind') {
  if (wx2) rows.push(['🌬️ Canlı rüzgar ' + wx2.w.toFixed(1) + ' m/s → türbin eğrisi', '×' + (windCurve(wx2.w, T) ?? 0).toFixed(2)]);
  else rows.push(['🛰️ Canlı hava verisi yok — simülasyon çarpanı', '×' + (windFactor(q2) * .55).toFixed(2)]);
```

**SONRASI:**
```js
if (wx2.day) {
  const effCloud2 = (wx2.cloud || 0) * (1 - NEWBIE_CLOUD * newbieAid());
  rows.push(['☁️ Bulut %' + Math.round(wx2.cloud || 0) + (newbieAid() > .01 ? ' <span class="pos">🌱 takviyeli</span>' : ''), '×' + Math.max(.05, 1 - effCloud2 / 100).toFixed(2)]);
}
} else rows.push(['🛰️ Canlı hava verisi yok — simülasyon çarpanı', '×' + sunFactor(q2).toFixed(2)]);
} else if (T.kind === 'wind') {
  if (wx2) {
    const boostedW2 = wx2.w + NEWBIE_WIND * newbieAid();
    rows.push(['🌬️ Canlı rüzgar ' + wx2.w.toFixed(1) + ' m/s' + (newbieAid() > .01 ? ' <span class="pos">+' + (NEWBIE_WIND * newbieAid()).toFixed(1) + ' 🌱 takviye</span>' : '') + ' → türbin eğrisi', '×' + (windCurve(boostedW2, T) ?? 0).toFixed(2)]);
  }
  else rows.push(['🛰️ Canlı hava verisi yok — simülasyon çarpanı', '×' + (windFactor(q2) * .55).toFixed(2)]);
```

**Etki:** Sadece görüntüleme (bilgi ekranı) düzeltmesi — gerçek üretim/ekonomi hesabı (`plantOutput()`) hiç değişmedi, sadece VERİM DÖKÜMÜ tablosu artık aynı takviyeyi gösteriyor. Doğrulama: tüm inline `<script>` blokları `new Function` ile sözdizimi kontrolünden geçti; Playwright (headless Chromium, hem `file://` hem `http://` üzerinden) sayfa açılışında `pageerror` üretmedi.

## 2026-08-19 00:58 turu

### Düzeltme 1: "Yeni Oyun" modalı ülke/il seçim ekranının üstünde takılı kalıyordu

**Bulgu:** `-P-M3kTIYNUC1b7N_oZc` (ClaudeBot, 22:41) — "Yeni Oyun — Haritadan Başla" butonuna ilk dokunuşta hoşgeldin modalı ülke/il seçim ekranının üstünde kilitli kalıyor, tüm tıklamalar engelleniyor. Workaround: butona 2. kez dokunmak düzeltiyor.

**Kök neden:** `wizMapStart()`, `closeModal()`'ı `wizStep1()`'den ÖNCE çağırıyor. `closeModal()` içindeki emniyet ağı, zamanlayıcıyı KURARKEN `wizsheet`'in "show" sınıfını kontrol ediyordu — o an henüz `wizStep1()` çalışmadığı için sınıf yoktu, zamanlayıcı koşulsuz kuruluyordu. 50ms sonra (bu sırada `wizStep1()` çoktan çalışıp `wizsheet`'i göstermiş olsa bile) callback bunu YENİDEN kontrol etmiyor, `wizardStart()`'ı tetikleyip "Yeni Oyun" ekranını harita/ülke seçiminin üstüne bindiriyordu.

**ÖNCESİ** (`index.html`, `window.closeModal`):
```js
window.closeModal = () => {
  $('overlay').classList.remove('show');
  if (MG_IV) { clearInterval(MG_IV); MG_IV = null; MG = null; } // açık mini oyun kapanınca sayaç da dursun
  // emniyet: kuruluş sırasında modal kapanırsa ekran boş kalmasın — karşılama ekranını geri getir
  // ⚠️ HATA AVI (kritik): bu emniyet ağı, afterLogin() bulut kaydını BEKLERKEN tetikleniyordu —
  // oyun iki kez başlıyor ya da yüklü oyunun üstünde "Yeni Oyun" modalı kalıyordu.
  // LOGIN_BUSY bayrağı giriş akışı boyunca ağı bastırır.
  if ($('app').classList.contains('wiz') && !$('wizsheet').classList.contains('show'))
    setTimeout(() => { if (!LOGIN_BUSY && (!G || !G._started)) wizardStart(); }, 50);
};
```

**SONRASI:**
```js
window.closeModal = () => {
  $('overlay').classList.remove('show');
  if (MG_IV) { clearInterval(MG_IV); MG_IV = null; MG = null; } // açık mini oyun kapanınca sayaç da dursun
  // emniyet: kuruluş sırasında modal kapanırsa ekran boş kalmasın — karşılama ekranını geri getir
  // ⚠️ HATA AVI (kritik): bu emniyet ağı, afterLogin() bulut kaydını BEKLERKEN tetikleniyordu —
  // oyun iki kez başlıyor ya da yüklü oyunun üstünde "Yeni Oyun" modalı kalıyordu.
  // LOGIN_BUSY bayrağı giriş akışı boyunca ağı bastırır.
  // 🐞 BOT DÜZELTMESİ: wizsheet kontrolü ESKİDEN burada (zamanlayıcı KURULURKEN) yapılıyordu — ama
  // wizMapStart() closeModal()'ı wizStep1()'den ÖNCE çağırıyor, o an wizsheet henüz "show" değil.
  // 50ms sonra callback koşulsuz çalışıp wizardStart()'ı tetikliyor, harita/ülke seçim ekranının
  // ÜSTÜNE "Yeni Oyun" modalını bindiriyordu. Kontrolü callback'İN İÇİNE (ateşlenme anına) taşıdık.
  if ($('app').classList.contains('wiz'))
    setTimeout(() => { if (!LOGIN_BUSY && (!G || !G._started) && !$('wizsheet').classList.contains('show')) wizardStart(); }, 50);
};
```

**Değişiklik:** `!$('wizsheet').classList.contains('show')` kontrolü, dıştaki `if`'ten (zamanlayıcı kurulma anı) `setTimeout` callback'inin İÇİNE (zamanlayıcı ateşlenme anı) taşındı. Böylece 50ms sonra durum yeniden ve doğru anda değerlendiriliyor.

**Doğrulama:** Playwright ile headless Chromium üzerinde önce/sonra karşılaştırması yapıldı. Düzeltme ÖNCESİ kodda `wizMapStart()` çağrıldıktan 600ms sonra `overlay` hâlâ "show" (modal üstte takılı) ve `wizsheet` de "show" (çakışma doğrulandı). Düzeltme SONRASI aynı test: `overlay` "show" değil, `wizsheet` "show" — çakışma yok, beklenen davranış. `pageerror` yok.

---

### Düzeltme 2: KRİTİK — bulut kayıttan gelen eksik `cellSold.sold` nesnesi depo/satış ekranını çökertiyordu

**Bulgu:** `-P-MFzRPBy29Y_v_pXhg` (ClaudeBot, 23:45) — `G.cellSold` bulut kayıtta `{day:bugün}` dönüyor, `sold` alt-nesnesi yok (RTDB boş `{}` nesneyi siliyor). `demandLeft()` içinde `G.cellSold.sold[cell]` "Cannot read properties of undefined" hatası fırlatıyor. Etki: herhangi bir depo/tesis marker'ına dokunmak `openUnitSheet` içinde sessizce çöküyor (sheet açılmıyor); SAT (`sellChunk`/`sellRevenue`) da aynı yolu kullanıyor, çalışmıyor.

**Kök neden:** `ensureCellSold()` yalnızca `G.cellSold.day` bugünün tarihiyle eşleşmiyorsa nesneyi sıfırlıyordu. Bulut kayıtta gün eşleşse bile (RTDB boş `{}` objeleri otomatik sildiği için) `sold` alt-nesnesi hiç var olmayabiliyordu — bu durumda sıfırlama atlanıyor ve sonraki erişim çöküyordu.

**ÖNCESİ** (`index.html`, `ensureCellSold`):
```js
function ensureCellSold() {
  const key = new Date().toISOString().slice(0, 10);
  if (!G.cellSold || G.cellSold.day !== key) G.cellSold = { day: key, sold: {} };
}
```

**SONRASI:**
```js
function ensureCellSold() {
  const key = new Date().toISOString().slice(0, 10);
  // 🐞 BOT DÜZELTMESİ: RTDB boş {} nesneyi buluttan siliyor — bulut kayıtta "sold" alt-nesnesi
  // olmadan sadece {day:bugün} dönebiliyordu. day eşleştiği için eskiden burası atlanıyor, sonra
  // G.cellSold.sold[cell] undefined üzerinde patlayıp depo/satış ekranını sessizce çökertiyordu.
  if (!G.cellSold || G.cellSold.day !== key || !G.cellSold.sold) G.cellSold = { day: key, sold: {} };
}
```

**Değişiklik:** Sıfırlama koşuluna `|| !G.cellSold.sold` eklendi — gün eşleşse bile `sold` alt-nesnesi eksikse yeniden oluşturuluyor.

**Doğrulama:** Playwright ile önce/sonra karşılaştırması. `G = { cellSold: { day: bugün } }` (sold YOK) durumu simüle edilip `demandLeft('test-cell-1')` çağrıldı. Düzeltme ÖNCESİ: `Cannot read properties of undefined (reading 'test-cell-1')` hatası (çökme doğrulandı). Düzeltme SONRASI: hatasız, `demandLeft` doğru sayı döndürdü. `new Function` ile tüm inline scriptler sözdizimi kontrolünden geçti, sayfa yüklenirken `pageerror` yok.

---

Diğer 3 bulgu (VERİM DÖKÜMÜ gösterim tutarsızlığı, wizardStart bulut kontrolü eksikliği) tasarım kararı gerektirdiği için `patrona-sorulacak.md`'ye yazıldı, koda dokunulmadı.

## 2026-08-19 07:56 turu

### Düzeltme 1: KRİTİK — 🏢 Şirket paneli her açılışta çöküyordu (`resRow('liccap')`)

**Bulgu:** `-P-Mu3WWZke_4R9idJc-` (02:47), `-P-N6h9jUaLQpkXkF2ni` (03:45), `-P-NLISzjHzdczygTt5g` (04:47) — üç bağımsız bildirim. Şirket ekranı (üstteki şirket adına dokununca) her zaman `TypeError: Cannot read properties of undefined (reading 'ico')` ile çöküyor. `resRow('liccap')` çağrılıyor ama `RESEARCH` nesnesinde `liccap` yok (yalnız `coalcap`/`gascap`/`office`/`range` var).

**Kök neden:** 18 Ağustos'ta "ruhsat limiti sabit 20'ye çekildi, `liccap` araştırması kaldırıldı, yerine `range` (Ofis Menzili) geldi" değişikliği yapılırken (satır ~1576 yorumunda not düşülmüş), Şirket panelini oluşturan `resRow('liccap')` çağrısı güncellenmemiş kalmış.

**ÖNCESİ** (`index.html`, ~satır 7278, `openCompanySheet` içinde):
```js
${resRow('coalcap')}${resRow('gascap')}${resRow('office')}${resRow('liccap')}
```

**SONRASI:**
```js
${/* 🐞 BOT DÜZELTMESİ: 'liccap' araştırması 18 Ağu'da kaldırıldı (satır ~1576), yerine 'range'
   (Ofis Menzili) geldi ama bu çağrı güncellenmemişti — RESEARCH['liccap'] undefined olduğu için
   resRow() D.ico'ya erişirken TypeError fırlatıp Şirket panelini her açılışta çökertiyordu. */''}
${resRow('coalcap')}${resRow('gascap')}${resRow('office')}${resRow('range')}
```

**Değişiklik:** Son çağrı `resRow('liccap')` → `resRow('range')` olarak düzeltildi; artık geçerli bir `RESEARCH` anahtarına işaret ediyor ve "Ofis Menzili" araştırması (bot bulgusunun da işaret ettiği gibi) UI'de tekrar görünür oldu.

**Doğrulama:** Node ile önce/sonra simülasyonu — `RESEARCH['liccap']` erişiminde `Cannot read properties of undefined (reading 'ico')` doğrulandı (ÖNCESİ), `RESEARCH['range']` erişiminde hatasız `ico` döndüğü doğrulandı (SONRASI). Ayrıca tüm inline `<script>` blokları `new Function` ile sözdizimi kontrolünden geçti; Playwright ile headless Chromium'da sayfa açılışında `pageerror` yok.

---

### Düzeltme 2: kozmetik — VERİM DÖKÜMÜ tablosunda gece eğrisi çarpanı virgüllü `0,00` yazıyordu

**Bulgu:** `-P-N6kHoq78RNMt7etiB` (03:46) — VERİM DÖKÜMÜ tablosunda güneş paneli gece durumunda çarpan `0,00` (virgüllü) yazıyor, tablodaki diğer tüm çarpanlar (`bell.toFixed(2)` dahil) nokta kullanıyor — biçim tutarsızlığı.

**Kök neden:** Gece durumu için çarpan `.toFixed(2)` ile hesaplanmak yerine elle `'0,00'` string literaliyle yazılmış.

**ÖNCESİ** (`index.html`, ~satır 6944, VERİM DÖKÜMÜ satırlarını oluşturan kod):
```js
rows.push([wx2.day ? '☀️ Gündüz eğrisi (sabah/akşam düşer)' : '🌙 Gece — panel üretmez', '×' + (wx2.day ? bell.toFixed(2) : '0,00')]);
```

**SONRASI:**
```js
// 🐞 BOT DÜZELTMESİ: burada virgüllü '0,00' yazılıyordu, tablodaki diğer tüm çarpanlar
// (bell.toFixed(2) dahil) nokta kullanıyor — biçim tutarsızlığı düzeltildi.
rows.push([wx2.day ? '☀️ Gündüz eğrisi (sabah/akşam düşer)' : '🌙 Gece — panel üretmez', '×' + (wx2.day ? bell.toFixed(2) : '0.00')]);
```

**Değişiklik:** `'0,00'` → `'0.00'`. Salt kozmetik, ekonomi/hesaplama etkilenmiyor.

**Not (koda dokunulmadı):** Aynı virgüllü yazım kalıbı `T.kind === 'geo'` satırında da var (`'gece gündüz sabit — ×0,95'`, ~satır 6959) ama bu suggestions'ta bildirilmedi; kapsam dışı bırakıldı, ileride aynı düzeltme uygulanabilir.

**Doğrulama:** `new Function` ile tüm inline scriptler sözdizimi kontrolünden geçti; Playwright headless Chromium'da `pageerror` yok.

---

## 2026-08-21 21:57 turu

### Düzeltme 1: Bağlı (uydu) depo sheet'inde "🔗 Bağlan" butonu, zaten bağlıyken de aktif görünüyordu

**Bulgu:** `-P-_d28ml48yeWY0fBP5` (18:45) ve `-P-_dHA-6pdAt5MfZtNB` (18:46) — bot, unit#6 (t=bat__ortac, `link=2`, zaten MERKEZ'e bağlı bir uydu depo) sheet'inde hem "🔗 Bağlan" hem "🔌 Bağlantıyı Kes" butonlarının AYNI ANDA `disabled=false` (aktif) göründüğünü bildirdi. Botun ilk kaydında ayrıca "MERKEZ depoda (unit#2) SAT butonu engellenmedi, `sellUnit(2,1)` başarıyla çalıştı" ifadesi vardı — bu kısmı incelendi ve **bug DEĞİL**: `startDischarge()` (satır ~2645) yalnız `st.link` olan (uydu) depoları engelliyor; MERKEZ depo (`st.link` yok) grup adına satış yapmak üzere TASARLANMIŞ (bkz. satır ~2643-2646 yorum ve `amt = (isLinked(st) ? groupStored(st) : st.stored) * frac`). Botun beklentisi yanlıştı, kod doğru çalışıyor — bu kısım için işlem yapılmadı.

Asıl gerçek bulgu ikinci raporun konusu: unit sheet'inde "Bağlan" butonu, depo zaten `u.link` ile bağlıyken de gösteriliyor ve `disabled` değil. Tıklanınca `startStoreLink()` (satır ~3481) zaten güvenle engelliyor ("⛔ ... Zincirleme bağlantı yok" uyarısı) — veri/kasa riski YOK — ama UI kafa karıştırıcı: kullanıcı aynı anda hem "Bağlan" hem "Bağlantıyı Kes" görüyor.

**Kök neden:** `openUnitSheet()` içindeki "DEPO BAĞLANTISI" bloğu (satır ~7565-7577), "Bağlan" butonunun `disabled` durumunu yalnız `bosSlot`/`aday.length`'e bağlıyordu; deponun kendisinin zaten bağlı (`u.link` dolu) olup olmadığını hiç kontrol etmiyordu.

**ÖNCESİ** (`index.html`, ~satır 7565-7577):
```js
${(() => {
  const bosSlot = STORE_TYPES[u.t].conn - connCount(u);
  const aday = stores().filter(s2 => s2.id !== u.id && s2.buildDone <= Date.now()
    && !s2.link && storeMaster(s2).id !== storeMaster(u).id && connCount(s2) < STORE_TYPES[s2.t].conn);
  return `<div style="font-size:11px;color:var(--acc2);margin:10px 0 2px">🔗 DEPO BAĞLANTISI
      <span class="sub">${connCount(u)}/${STORE_TYPES[u.t].conn} nokta dolu</span></div>
    <button class="bigsell half" onclick="startStoreLink(${u.id})" ${bosSlot <= 0 || !aday.length ? 'disabled' : ''}>🔗 Bağlan — haritadan depo seç</button>
    ${isLinked(u) ? `<button class="bigsell half" onclick="storeUnlinkAsk(${u.id})">🔌 Bağlantıyı Kes</button>` : ''}
    ${bosSlot <= 0 ? '<p class="sub">Bağlantı noktaların dolu — önce bir bağlantı kes.</p>'
      : !aday.length ? '<p class="sub">Bağlanabilecek başka depo yok.</p>' : ''}`;
})()}
```

**SONRASI:**
```js
${(() => {
  const bosSlot = STORE_TYPES[u.t].conn - connCount(u);
  const aday = stores().filter(s2 => s2.id !== u.id && s2.buildDone <= Date.now()
    && !s2.link && storeMaster(s2).id !== storeMaster(u).id && connCount(s2) < STORE_TYPES[s2.t].conn);
  return `<div style="font-size:11px;color:var(--acc2);margin:10px 0 2px">🔗 DEPO BAĞLANTISI
      <span class="sub">${connCount(u)}/${STORE_TYPES[u.t].conn} nokta dolu</span></div>
    ${!u.link ? `<button class="bigsell half" onclick="startStoreLink(${u.id})" ${bosSlot <= 0 || !aday.length ? 'disabled' : ''}>🔗 Bağlan — haritadan depo seç</button>` : ''}
    ${isLinked(u) ? `<button class="bigsell half" onclick="storeUnlinkAsk(${u.id})">🔌 Bağlantıyı Kes</button>` : ''}
    ${/* 🐞 [BOT-FIX] u.link=true (zaten BAĞLI DEPO) iken "Bağlan" butonu de aktif görünüyordu —
       tıklanınca startStoreLink() zaten güvenle engelliyordu (zincirleme bağlantı yok uyarısı),
       ama UI kafa karıştırıyordu: aynı anda hem "Bağlan" hem "Bağlantıyı Kes" aktif duruyordu. */ ''}
    ${u.link ? '' : bosSlot <= 0 ? '<p class="sub">Bağlantı noktaların dolu — önce bir bağlantı kes.</p>'
      : !aday.length ? '<p class="sub">Bağlanabilecek başka depo yok.</p>' : ''}`;
})()}
```

**Neden düşük risk / ekonomiyi değiştirmiyor:** Salt görüntü (UI) değişikliği — hiçbir mutasyon fonksiyonuna, kasaya, bağlantı/satış kuralına dokunulmadı. `startStoreLink()` zaten aynı koşulu (`s.link`) kontrol edip engelliyordu; bu değişiklik yalnızca zaten-engellenen bir butonu, kullanıcıyı yanıltmaması için gizliyor. MERKEZ depo (satellite'ı OLAN ama kendisi bağlı OLMAYAN, yani `u.link` boş) tarafında davranış değişmedi — hâlâ yeni uydu bağlayabiliyor.

**Doğrulama:** `new Function` ile tüm inline scriptler sözdizimi kontrolünden geçti; `npm i playwright` + mevcut `/opt/pw-browsers` Chromium ile hem `file://` hem yerel `http://` sunucu üzerinden sayfa açılışında `pageerror` üretmedi (yalnız beklenen `file://` CORS / eksik test-config 404 console.error'ları var, JS hatası yok).

---

İşlenen diğer 9 kayıt: 2'si önceki turda patrona sorulan konularla aynı kök nedene sahip olduğu için yerel geliştirici tarafından zaten çözülmüş bulundu (kod değişikliği gerekmedi), 7'si durum özeti/bilgi amaçlı (aksiyon gerekmedi). Detaylar `bot-notlar/islenen.md`'de.
