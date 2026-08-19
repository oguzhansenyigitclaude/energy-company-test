# Düzeltmeler (Öncesi / Sonrası)

Bu dosya, bakım botunun bu turda index.html üzerinde yaptığı düzeltmelerin ÖNCE/SONRA kod parçalarını ve gerekçesini listeler. Yerel geliştirici bunları ana projeye taşıyabilir / gözden geçirebilir.

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

İşlenen diğer 9 kayıt: 2'si önceki turda patrona sorulan konularla aynı kök nedene sahip olduğu için yerel geliştirici tarafından zaten çözülmüş bulundu (kod değişikliği gerekmedi), 7'si durum özeti/bilgi amaçlı (aksiyon gerekmedi). Detaylar `bot-notlar/islenen.md`'de.
