# Toplan — Şartname (SPEC)

> Bu dosya projenin "anayasası". Ne yapılacağı burada yazar. Bir şey değişirse önce burası güncellenir, sonra kod.

## 1. Ürün tek cümlede

**Toplan**, şehirlerdeki etkinliklerin (atölye, konser, meetup, spor buluşması) yayınlandığı ve insanların bu etkinliklere kayıt olduğu bir platform.

## 2. Neden bu proje? (öğrenme hedefi)

Nautelo'daki her önemli kavramın küçük bir kopyası burada var:

| Kavram | Toplan'da | Nautelo'da karşılığı |
|---|---|---|
| Roller | Katılımcı, Organizatör, Moderatör | Buyer, Private seller / Broker, Staff |
| Rol yükseltme | Katılımcı → "Organizatör ol" başvurusu | Buyer → "Satış yapmaya başla" |
| Sahiplik yetkisi | Sadece etkinliği oluşturan düzenler | `IsOwnerOrBrokerEditor` |
| Moderasyon | Moderatör etkinliği onaylar/reddeder | İlan onay akışı |
| Durum makinesi | Taslak → İncelemede → Yayında / Reddedildi / İptal | İlan durumları |
| İş kuralı + eşzamanlılık | Kapasite dolunca bekleme listesi | Kota ve hak (entitlement) kuralları |
| SEO'lu herkese açık sayfa | Etkinlik detay sayfası | Tekne ilan sayfası |
| Çok dillilik | Türkçe / İngilizce | İngilizce / İtalyanca / İspanyolca |
| JWT + HttpOnly çerez | Aynı yapı | `accounts/tokens.py`, `accounts/cookies.py` |

## 3. Kullanıcı tipleri

- **Ziyaretçi** (giriş yapmamış): Yayındaki etkinlikleri görür, arar, filtreler.
- **Katılımcı** (`PARTICIPANT`): Kayıt olan herkes. E-postasını doğruladıysa etkinliklere kayıt olur, kaydını iptal eder.
- **Organizatör** (`ORGANIZER`): Başvurusu moderatörce onaylanmış katılımcı. Etkinlik oluşturur, kendi etkinliklerini yönetir, katılımcı listesini görür.
- **Moderatör** (`STAFF`): Organizatör başvurularını ve etkinlikleri onaylar/reddeder. Django admin'e girer.

## 4. Veri modeli (tablolar)

- **User** (kendi modelimiz, ilk günden): email (giriş için), full_name, role, is_email_verified, preferred_language (tr/en), created_at
- **OrganizerApplication**: user, organization_name, description, status (PENDING / APPROVED / REJECTED), review_note, reviewed_by, reviewed_at
- **Category**: name, slug
- **City**: name, slug
- **Event**: organizer → User, title, slug, description, category, city, venue_name, address, starts_at, ends_at, capacity, cover_image, status (DRAFT / PENDING_REVIEW / PUBLISHED / REJECTED / CANCELLED), rejection_reason, published_at, created_at, updated_at
- **Registration**: event, user, status (CONFIRMED / WAITLISTED / CANCELLED), created_at — aynı kullanıcı aynı etkinliğe bir kez kayıt olur

## 5. İş kuralları

1. Ziyaretçiler ve herkese açık API sadece `PUBLISHED` etkinlikleri görür.
2. Etkinliği sadece `ORGANIZER` oluşturur, sadece sahibi düzenler.
3. Organizatör etkinliği "incelemeye gönderir" → moderatör onaylar (`PUBLISHED`) veya gerekçe yazarak reddeder (`REJECTED`).
4. Yayındaki bir etkinliğin tarihi veya şehri değişirse tekrar incelemeye düşer.
5. Kapasite, onaylı kayıt sayısının altına düşürülemez.
6. Kayıt için: e-posta doğrulanmış, etkinlik yayında ve gelecekte olmalı. Organizatör kendi etkinliğine kayıt olamaz.
7. Kapasite doluysa kayıt `WAITLISTED` olur. Onaylı biri iptal ederse bekleme listesindeki ilk kişi otomatik `CONFIRMED` olur. (Aynı anda iki kişi son yeri almaya çalışırsa bile kapasite aşılmamalı.)
8. Organizatör kendi etkinliğinin katılımcı **isimlerini** görür, e-postalarını görmez (gizlilik).
9. Etkinlik iptal edilirse kayıtlı herkese e-posta gider.

## 6. Teknoloji kararları

- **Backend:** Python 3.12+, `uv` (paket yöneticisi), Django 5.2 LTS, Django REST Framework, SimpleJWT, django-cors-headers, django-environ, pytest + pytest-django, ruff
- **Frontend:** Next.js (güncel kararlı sürüm, App Router), TypeScript, Tailwind CSS, npm
- **Veritabanı:** Geliştirmede SQLite → Aşama 12'de PostgreSQL
- **E-posta:** Geliştirmede konsola yazdırılır
- **Klasör yapısı:** tek repo, içinde `backend/` ve `frontend/`

## 7. Aşamalar

Her aşamanın sonunda: testler geçiyor, Berk kendi eliyle denedi, git commit atıldı, `ILERLEME.md` güncellendi.

### Aşama 0 — Hazırlık
- **Öğreneceklerin:** Terminal, git'in mantığı, sanal ortam ve paket yöneticisi
- **Yapılacaklar:** Python, uv, Node.js LTS, Git, VS Code kurulu mu kontrol; git repo başlat; `.gitignore`; GitHub'a ilk push
- **Bitti sayılır:** `python --version`, `uv --version`, `node --version`, `git --version` çalışıyor; repo GitHub'da

### Aşama 1 — İskelet
- **Öğreneceklerin:** Django projesi vs app, settings, ortam değişkenleri (`.env`), CORS, bir isteğin frontend'den backend'e yolculuğu
- **Yapılacaklar:** Django projesi (`config/`, settings `base/dev/test` olarak ayrık), `/api/health/` uç noktası; Next.js uygulaması; ana sayfa backend'den "ok" çekip gösteriyor
- **Bitti sayılır:** İki sunucu birlikte çalışıyor, tarayıcıda backend'den gelen veri görünüyor
- **Nautelo:** `backend/config/settings/`

### Aşama 2 — Kullanıcı ve Authentication
- **Öğreneceklerin:** Custom User, şifre hash'leme, JWT access/refresh, HttpOnly çerez, e-posta doğrulama, throttling
- **Yapılacaklar:** `accounts` app; kayıt, giriş, çıkış, token yenileme, e-posta doğrulama, şifre sıfırlama; frontend'de giriş/kayıt sayfaları ve otomatik token yenileyen API istemcisi
- **Bitti sayılır:** Kayıt ol → konsoldaki linkle doğrula → giriş yap → 15 dk sonra sessizce yenile → çıkış
- **Nautelo:** `accounts/tokens.py`, `accounts/cookies.py`, `frontend/src/lib/api/client.ts`

### Aşama 3 — Roller ve Yetkiler
- **Öğreneceklerin:** Authentication vs authorization, rol tabanlı ve nesne tabanlı yetki, DRF permission sınıfları, yetki testleri
- **Yapılacaklar:** `role` alanı, `permissions.py`, organizatör başvurusu + moderatör onayı, Django admin'de moderatör yetkileri; frontend'de role göre menü
- **Bitti sayılır:** Her rol için "yapabilir / yapamaz" testleri geçiyor
- **Nautelo:** `accounts/enums.py`, `accounts/permissions.py`

### Aşama 4 — Ana modeller ve Admin
- **Öğreneceklerin:** İlişkiler (ForeignKey), slug, durum alanları, migration, admin özelleştirme, seed data
- **Yapılacaklar:** `events` app: Category, City, Event; admin'de filtre/arama; örnek veri yükleyen komut
- **Bitti sayılır:** Admin'den etkinlik eklenip filtrelenebiliyor

### Aşama 5 — Etkinlik API'si ve Organizatör Paneli
- **Öğreneceklerin:** Serializer, ViewSet, services/selectors düzeni, durum makinesi, moderasyon akışı
- **Yapılacaklar:** Organizatör CRUD + "incelemeye gönder"; moderatör onay/red; frontend'de organizatör paneli (liste, form, durum rozetleri)
- **Bitti sayılır:** Organizatör etkinlik oluşturup gönderiyor, moderatör onaylıyor, etkinlik yayına çıkıyor

### Aşama 6 — Kayıt Sistemi
- **Öğreneceklerin:** Veritabanı transaction'ı, `select_for_update`, yarış durumu (race condition), iş kuralını tek yerde toplamak
- **Yapılacaklar:** `registrations` app; kayıt ol / iptal et; bekleme listesi ve otomatik terfi; "kayıtlarım" sayfası
- **Bitti sayılır:** Kapasite asla aşılmıyor (test ile kanıtlı), bekleme listesi doğru çalışıyor

### Aşama 7 — Herkese Açık Sayfalar ve SEO
- **Öğreneceklerin:** Server Component vs Client Component, SSR, metadata, sitemap, filtreleme, sayfalama
- **Yapılacaklar:** Etkinlik listesi (şehir/kategori/tarih filtresi, arama, sayfalama), etkinlik detay sayfası, `sitemap.xml`, `robots.txt`
- **Bitti sayılır:** Sayfa kaynağını görüntüle'de içerik HTML olarak var
- **Nautelo:** `frontend/src/app/boats/`, `sitemap.ts`

### Aşama 8 — Görsel Yükleme
- **Öğreneceklerin:** Dosya yükleme, doğrulama (tür, boyut), media dosyaları, Next.js görsel optimizasyonu
- **Yapılacaklar:** Etkinlik kapak görseli yükleme ve gösterme
- **Bitti sayılır:** 5 MB üstü veya resim olmayan dosya reddediliyor

### Aşama 9 — Çok Dillilik (i18n)
- **Öğreneceklerin:** i18n nedir, çeviri anahtarları, dil algılama, URL'de dil ön eki, backend mesajlarının çevirisi
- **Yapılacaklar:** Arayüz TR/EN; `/en/...` ön eki; dil tercihi çerezde ve kullanıcı profilinde; Django hata mesajları ve e-postalar iki dilde
- **Bitti sayılır:** Tarayıcı dili İngilizce olan biri siteyi İngilizce görüyor, dil değiştirici çalışıyor
- **Nautelo:** `frontend/src/proxy.ts`, `frontend/src/i18n/`, `backend/uitext/`

### Aşama 10 — E-postalar ve Arka Plan İşleri
- **Öğreneceklerin:** Neden bazı işler arka planda yapılır, Celery + Redis'in mantığı
- **Yapılacaklar:** Kayıt onayı, bekleme listesinden terfi ve iptal e-postaları
- **Bitti sayılır:** E-posta gönderimi isteği yavaşlatmıyor

### Aşama 11 — Kalite: Test, Güvenlik, Performans
- **Öğreneceklerin:** N+1 sorgu sorunu, `select_related`, güvenlik incelemesi, test kapsamı
- **Yapılacaklar:** Eksik testler, güvenlik kontrol listesi, yavaş sorguların düzeltilmesi
- **Bitti sayılır:** Güvenlik kontrol listesi temiz, ana sayfalar az sorguyla açılıyor

### Aşama 12 — PostgreSQL, Docker, CI ve Yayına Alma
- **Öğreneceklerin:** Neden canlıda SQLite kullanılmaz, Docker, GitHub Actions, ortam değişkenleriyle güvenli yayın
- **Yapılacaklar:** PostgreSQL'e geçiş, `docker-compose.yml`, her push'ta testleri çalıştıran CI, canlı ortama yayın
- **Bitti sayılır:** Site internette açık, iş başvurusunda link olarak verilebilir
- **Nautelo:** `docker-compose.yml`, `.github/workflows/ci.yml`

## 8. Kapsam dışı (bilerek yapmıyoruz)

Ödeme, mesajlaşma, mobil uygulama, gerçek zamanlı bildirim. Bunlar sonraki projenin konusu.
