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
