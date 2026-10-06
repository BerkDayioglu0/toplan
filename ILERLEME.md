# Toplan — İlerleme ve Ders Notları

> Her aşama sonunda güncellenir. Yeni bir Claude oturumu buradan nerede kaldığımızı anlar.

## Şu anki durum
- **Sıradaki adım:** Aşama 0 — Hazırlık (araçları kontrol et, git repo başlat)
- **Tamamlanan aşamalar:** yok
- Proje klasöründe şimdilik sadece `SPEC.md`, `CLAUDE.md` ve bu dosya var.

## Bu projeden önce (Cowork sohbetinde) işlenen dersler

### Ders 0 — Mimari nedir, Nautelo neden Django + Next.js?
- Mimari = yazılımın yerleşim planı. Backend (mutfak), frontend (salon), API (garson).
- Nautelo'da Next.js: SEO (ilan sayfaları Google'da çıkmalı), çok dillilik, görsel optimizasyonu.
- Nautelo'da Django: hazır admin paneli, güçlü veri modeli (ORM), hazır güvenlik/kullanıcı sistemi, Python ekosistemi.
- Ayrık frontend/backend: ileride mobil uygulama aynı API'yi kullanabilir.
- Altın kural: ihtiyacın olan en basit mimariyle başla, sıkışınca böl.

### Ders 1 — Django'nun temelleri
- **Model** = tablonun tarifi; **Migration** = tarifi veritabanına uygulamak (`makemigrations` → `migrate`).
- **URL** = adres; **View** = işi yapan; **Serializer** = veriyi JSON'a çeviren ve doğrulayan (DRF).
- **Admin** = hazır yönetim paneli. **App** = konu bazlı modül (Nautelo'da `accounts`, `listings`, `brokers`...).
- `services.py` (veri değiştiren iş kuralları) ve `selectors.py` (veri okuyan sorgular) düzeni.

### Ders 2 — Authentication
- Authentication = "Sen kimsin?"; authorization = "Bunu yapmaya iznin var mı?"
- Session (vestiyer fişi) vs JWT (mühürlü kimlik kartı). Ayrık frontend/backend'de JWT.
- Nautelo: 15 dk **access token** (JS hafızasında) + 7 gün **refresh token** (HttpOnly çerezde), her yenilemede refresh token değişir, eskisi kara listeye girer.
- Ekstra önlemler: çıkışta token iptali, şifre değişince tüm oturumların kapanması (`pwv`), e-posta doğrulama, deneme sınırı (10/dk), admin için iki adımlı doğrulama.
- Kurallar: ilk gün custom User, auth'u kendin icat etme, token'ı localStorage'a koyma, giriş uçlarına throttling, varsayılan izin "giriş zorunlu".

### Ders 2.5 — Claude ile proje nasıl yapılır?
- Ezber gerekmez; kavramı, nedenini ve kodun nerede durduğunu bilmek ve test edebilmek yeter.
- Yol: şartname (SPEC.md) → CLAUDE.md → aşamalar → her aşamada plan → onay → uygula → dene → commit → not.
- Token: önce plan, uzun metni dosyaya yaz, konu değişince yeni oturum, net hata mesajı ver, hata döngüsünde dur.

## Proje içinde işlenecek dersler
- **Ders 3 — Roles & Permissions** → Aşama 3'te
- **Ders 4 — i18n** → Aşama 9'da
- **Ders 5 — Sıfırdan proje kontrol listesi** → proje boyunca, Aşama 12 sonunda özet

## Günlük
<!-- Her aşama sonunda en üste ekle: tarih — aşama — ne yapıldı — öğrenilenler — sıradaki adım -->
- 2026-10-06 — Proje planlandı (Cowork'te). SPEC, CLAUDE.md ve bu dosya oluşturuldu.
