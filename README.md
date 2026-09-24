# Randevu Yönetim API

Berber/kuaför işletmeleri için geliştirilmiş, JWT tabanlı bir **randevu ve hizmet yönetimi REST API**'sidir. Müşteri kaydı ve oturum açma, sunulan hizmetlerin (ad, fiyat, süre) yönetimi ve müşteri randevularının oluşturulup listelenmesi gibi temel iş akışlarını uçtan uca karşılar. Node.js + Express üzerinde çalışır, verileri MongoDB'de Mongoose şemalarıyla saklar ve kimlik doğrulamayı access/refresh token ikilisiyle yürütür. Tüm trafik oran sınırlama (rate limiting) katmanından geçer; şifreler veritabanına asla düz metin olarak yazılmaz.

![Node.js](https://img.shields.io/badge/Node.js-339933?style=for-the-badge&logo=node.js&logoColor=white)
![Express](https://img.shields.io/badge/Express-4.16-000000?style=for-the-badge&logo=express&logoColor=white)
![MongoDB](https://img.shields.io/badge/MongoDB-Mongoose%208-47A248?style=for-the-badge&logo=mongodb&logoColor=white)
![JWT](https://img.shields.io/badge/JWT-Auth-000000?style=for-the-badge&logo=jsonwebtokens&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-blue?style=for-the-badge)

---

## Özellikler

- **JWT kimlik doğrulama** — `jsonwebtoken` ile imzalanan access token (24 saat) ve refresh token (7 gün) üretimi.
- **Refresh token akışı** — Refresh token kullanıcı dokümanında saklanır; `/refresh-token` ile yenilenir, `/logout` ile geçersiz kılınır.
- **Korumalı endpoint'ler** — `middleware/auth.js`, `Authorization: Bearer <token>` header'ını doğrular ve çözümlenmiş kullanıcıyı `req.user` içine yerleştirir.
- **Şifre güvenliği** — `bcryptjs` ile Mongoose `pre("save")` hook'unda otomatik hashleme (10 salt round) ve `comparePassword()` metodu ile güvenli karşılaştırma.
- **İki katmanlı rate limiting** — Tüm API için genel limit, kimlik doğrulama uçları için çok daha sıkı ayrı bir limit.
- **Girdi doğrulama** — `is_js` ile e-posta format kontrolü, zorunlu alan ve minimum şifre uzunluğu (8 karakter) denetimleri.
- **Standart yanıt zarfı** — `lib/Response.js` üzerinden tüm başarı/hata yanıtları tek tip `{ code, data }` / `{ code, error }` yapısında döner.
- **Özel hata sınıfı** — `lib/Error.js` (`CustomError`) ile kod + mesaj + açıklama taşıyan hatalar; MongoDB `E11000` yinelenen kayıt hatası otomatik olarak `409 Conflict` ve anlaşılır bir mesaja çevrilir.
- **HTTP istek loglama** — `morgan` (`dev` formatı) ile her istek konsola loglanır.
- **Singleton veritabanı bağlantısı** — `db/Database.js`, Mongoose bağlantısını tek örnek (singleton) olarak yönetir; bağlantı hatasında süreç kontrollü şekilde sonlanır.
- **Ortam tabanlı yapılandırma** — `dotenv` yalnızca production dışında devreye girer; yapılandırma `config/index.js` içinde toplanmıştır.
- **Merkezî sabitler** — `config/Enum.js` içinde HTTP durum kodları ve log seviyeleri tek yerden yönetilir.

---

## Teknolojiler

| Katman | Teknoloji | Versiyon |
|---|---|---|
| Çalışma ortamı | Node.js | — |
| Web framework | Express | `~4.16.1` |
| Veritabanı | MongoDB / Mongoose | `^8.13.2` |
| Kimlik doğrulama | jsonwebtoken | `^9.0.2` |
| Şifre hash'leme | bcryptjs | `^3.0.2` |
| Oran sınırlama | express-rate-limit | `^7.5.0` |
| Doğrulama | is_js | `^0.9.0` |
| Loglama | morgan | `~1.9.1` |
| Şablon motoru | EJS | `~2.6.1` |
| Yapılandırma | dotenv | `^16.5.0` |
| Yardımcı | cookie-parser `~1.4.4`, http-errors `~1.6.3`, debug `~2.6.9` | — |

---

## API Endpoint'leri

Tüm uçlar `/api` ön eki altında sunulur. Korumalı uçlar `Authorization: Bearer <token>` header'ı bekler.

### Kimlik Doğrulama — `/api/auth`

| Metot | Yol | Açıklama | Auth |
|---|---|---|:---:|
| `POST` | `/api/auth/register` | Yeni kullanıcı kaydı oluşturur; access ve refresh token döner | ❌ |
| `POST` | `/api/auth/login` | E-posta ve şifre ile giriş yapar; access ve refresh token döner | ❌ |
| `POST` | `/api/auth/refresh-token` | Geçerli bir refresh token ile yeni token çifti üretir | ❌ |
| `POST` | `/api/auth/logout` | Refresh token'ı geçersiz kılarak oturumu sonlandırır | ❌ |

### Kullanıcılar — `/api/users`

| Metot | Yol | Açıklama | Auth |
|---|---|---|:---:|
| `GET` | `/api/users` | Tüm kullanıcıları listeler | ✅ |
| `POST` | `/api/users/update` | `_id` ile kullanıcı bilgilerini günceller | ✅ |
| `DELETE` | `/api/users/delete` | `_id` ile kullanıcıyı siler | ✅ |

### Hizmetler — `/api/service`

| Metot | Yol | Açıklama | Auth |
|---|---|---|:---:|
| `GET` | `/api/service` | Tanımlı tüm hizmetleri listeler | ✅ |
| `POST` | `/api/service/add` | Yeni hizmet ekler (`name`, `price`, `duration`, `description`) | ✅ |
| `POST` | `/api/service/update` | `_id` ile hizmet fiyatını günceller | ✅ |
| `DELETE` | `/api/service/delete` | `_id` ile hizmeti siler | ✅ |

### Randevular — `/api/appoinstments`

| Metot | Yol | Açıklama | Auth |
|---|---|---|:---:|
| `GET` | `/api/appoinstments` | Tüm randevuları listeler | ✅ |
| `POST` | `/api/appoinstments/add` | Yeni randevu oluşturur (`customer_name`, `phone_number`, `service`, `date`, `notes`, `is_approved`) | ✅ |

> **Not:** Randevu yolu, kaynak dosyadaki yazımla birebir aynıdır (`appoinstments`). `routes/auditlogs.js` dosyası projede bulunur ancak `app.js` içinde henüz mount edilmediği için dışarıya açık değildir.

### Örnek İstek

```bash
# Kayıt
curl -X POST http://localhost:3000/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email":"ornek@mail.com","password":"GucluSifre123","customer_name":"Deniz Akyol","phone_number":"5551112233"}'

# Korumalı uca erişim
curl http://localhost:3000/api/appoinstments \
  -H "Authorization: Bearer <access_token>"
```

---

## Güvenlik

| Önlem | Uygulama |
|---|---|
| **Şifre hash'leme** | `bcryptjs` ile 10 salt round; hashleme model katmanında `pre("save")` hook'u ile zorunlu kılınır, hiçbir route düz metin şifre yazamaz |
| **JWT access token** | `JWT_SECRET` ile imzalanır, 24 saat geçerlidir |
| **Refresh token** | 7 gün geçerli; kullanıcı kaydında saklanır ve `logout` ile `null` yapılarak iptal edilir |
| **Token doğrulama** | `verifyToken` geçersiz/süresi dolmuş token'ları yakalar; middleware `401 Unauthorized` döner |
| **Genel rate limit** | 15 dakikada IP başına 100 istek (tüm uygulamaya uygulanır) |
| **Auth rate limit** | 1 saatte IP başına 5 istek — kaba kuvvet (brute force) ve otomatik kayıt denemelerine karşı |
| **Standart limit header'ları** | `standardHeaders: true`, `legacyHeaders: false` (RateLimit-* header'ları) |
| **Girdi doğrulama** | E-posta format kontrolü (`is_js`), zorunlu alan kontrolleri, minimum 8 karakter şifre politikası |
| **Benzersiz e-posta** | Şema düzeyinde `unique` kısıtı + kayıt öncesi kontrol; ihlal `409 Conflict` olarak raporlanır |
| **Bilgi sızıntısını önleme** | Hatalı giriş denemelerinde "Geçersiz email veya şifre" gibi ayrım yapmayan mesajlar; hata detayları yalnızca development ortamında render edilir |
| **Secret yönetimi** | Tüm gizli değerler ortam değişkenlerinden okunur; `.env` dosyaları `.gitignore` ile sürüm kontrolünün dışında tutulur |

---

## Ortam Değişkenleri

Proje kökündeki `api/.env.example` dosyasını `api/.env` olarak kopyalayın ve değerleri kendi ortamınıza göre doldurun.

| Değişken | Açıklama | Örnek / Varsayılan |
|---|---|---|
| `CONNECTION_STRING` | MongoDB bağlantı adresi | `mongodb://localhost:27017/veritabani-adi` |
| `LOG_LEVEL` | Log seviyesi: `error \| warn \| info \| debug` | `info` (kod varsayılanı: `debug`) |
| `JWT_SECRET` | JWT imzalama anahtarı — uzun ve rastgele bir değer kullanın | *(zorunlu, boş bırakmayın)* |
| `PORT` | Sunucunun dinleyeceği port (`config/index.js` tarafından okunur) | `3000` |

> `dotenv` yalnızca `NODE_ENV !== "production"` olduğunda yüklenir; production ortamında değişkenleri doğrudan platform üzerinden tanımlayın.

---

## Kurulum

**Gereksinimler:** Node.js, npm ve erişilebilir bir MongoDB örneği (yerel veya MongoDB Atlas).

```bash
# 1. Projeyi klonlayın
git clone <repo-url>
cd my_api

# 2. API klasörüne geçip bağımlılıkları yükleyin
cd api
npm install

# 3. Ortam değişkenlerini hazırlayın
cp .env.example .env
# .env dosyasını açıp CONNECTION_STRING ve JWT_SECRET değerlerini doldurun
```

---

## Çalıştırma

```bash
cd api
npm start
```

Sunucu varsayılan olarak `http://localhost:3000` adresinde ayağa kalkar. Başlangıçta MongoDB bağlantısı kurulur ve konsolda `DB Connected.` mesajı görünür; bağlantı kurulamazsa süreç hata koduyla sonlanır.

Hata ayıklama loglarını görmek için:

```bash
# macOS / Linux
DEBUG=api:* npm start

# Windows (PowerShell)
$env:DEBUG="api:*"; npm start
```

---

## Proje Yapısı

```
my_api/
├── api/
│   ├── bin/
│   │   └── www                   # Sunucu başlatma, port normalizasyonu, DB bağlantısı
│   ├── config/
│   │   ├── Enum.js               # HTTP durum kodları ve log seviyeleri
│   │   ├── index.js              # Ortam değişkeni tabanlı yapılandırma
│   │   └── jwt.js                # Token üretme / yenileme / doğrulama
│   ├── db/
│   │   ├── Database.js           # Singleton Mongoose bağlantı yöneticisi
│   │   └── models/
│   │       ├── Users.js          # Kullanıcı şeması + bcrypt hook & comparePassword
│   │       ├── Service.js        # Hizmet şeması (ad, fiyat, süre, açıklama)
│   │       ├── Appoinstments.js  # Randevu şeması (müşteri, hizmet, tarih, onay)
│   │       └── AuditLogs.js      # Denetim kaydı şeması
│   ├── lib/
│   │   ├── Response.js           # Standart başarı / hata yanıt üreticisi
│   │   ├── Error.js              # CustomError sınıfı
│   │   └── auth.js               # Passport-JWT tabanlı rol/yetki taslağı (henüz aktif değil)
│   ├── middleware/
│   │   └── auth.js               # Bearer token doğrulama middleware'i
│   ├── routes/
│   │   ├── auth.js               # register / login / refresh-token / logout
│   │   ├── users.js              # Kullanıcı CRUD
│   │   ├── service.js            # Hizmet CRUD
│   │   ├── appoinstments.js      # Randevu listeleme / oluşturma
│   │   ├── auditlogs.js          # Denetim kaydı ucu (mount edilmemiş)
│   │   └── index.js              # Otomatik route yükleyici (opsiyonel kullanım)
│   ├── views/                    # EJS şablonları (index, error)
│   ├── public/                   # Statik dosyalar
│   ├── app.js                    # Express uygulaması, middleware ve rate limiter'lar
│   ├── .env.example              # Ortam değişkeni şablonu
│   └── package.json
├── .gitignore
├── LICENSE
└── README.md
```

---

## Lisans

Bu proje **MIT Lisansı** ile lisanslanmıştır. Ayrıntılar için [LICENSE](LICENSE) dosyasına bakınız.

© 2025 Deniz Akyol
