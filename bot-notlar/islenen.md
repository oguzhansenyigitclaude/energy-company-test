# İşlenen Firebase Anahtarları

Bu dosya, `suggestions` düğümünden okunup bakım botu tarafından değerlendirilen kayıtların anahtarlarını listeler. Sadece burada olmayan (yeni) anahtarlar bir sonraki turda işlenir.

| Anahtar | Zaman (UTC) | Özet | Sonuç |
|---|---|---|---|
| `-P-LuNJCmDWvKqE3jQvR` | 2026-08-18 22:06 | ClaudeBot: headless Chromium tüm HTTPS hedeflerinde ERR_CONNECTION_RESET veriyor (proxy üzerinden curl çalışıyor) | Oyun koduyla ilgili değil, bildiren botun kendi test ortamı/proxy sorunu. Kod değişikliği yapılmadı. |
| `-P-LuNULjpqrFq_JQc0M` | 2026-08-18 22:06 | ClaudeBot: `saves/0` boş, kurulum sihirbazı hiç çalışmamış (tarayıcı açılamadığı için) | Oyun kodunda hata değil, bildiren botun hesap durumu bilgisi. Kod değişikliği yapılmadı. |
| `-P-LuV7U_OgxAOnPntbH` | 2026-08-18 22:07 | ClaudeBot: önceki iki kayıtta `fromId` yanlış yazılmış, kendi düzeltme notu | Oyun kodunu ilgilendirmiyor (bildiren botun kendi script hatası). Kod değişikliği yapılmadı. |

Bu turda oyun kodunda (index.html) bir hataya işaret eden bulgu yoktu; üç kayıt da bildiren botun kendi test ortamı/hesap durumu hakkındaydı. Bu yüzden BUILD_TAG/version.json güncellenmedi.
