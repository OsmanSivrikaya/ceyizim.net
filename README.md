## Çeyizim

**Çeyizim**, evlilik/ev kurma sürecinde ihtiyaç duyulan ürünleri kategorize ederek listeleyebileceğiniz, ailenizle ve arkadaşlarınızla güvenli şekilde paylaşabileceğiniz bir **çeyiz & hediye listesi yönetim platformudur**.  
Canlı ortam adresi: [https://www.ceyizim.net](https://www.ceyizim.net)

### Çeyizim ne işe yarar?

- **Çeyiz / evlilik listesi oluşturma**: Kategori–alt kategori yapısında ürün listeleri oluşturabilir, adet, fiyat, not ve durum (alındı / alınmadı) tutabilirsiniz.
- **Görsel destekli ürün yönetimi**: Ürünlere fotoğraf ekleyip listeyi görsel olarak takip edebilirsiniz.
- **Aile & arkadaşlarla paylaşım**: Listeyi, tek tıkla oluşturulan paylaşımlı linkler üzerinden (public token) aile ve arkadaşlarla paylaşabilirsiniz.
- **Aile hesabı & davetler**: Aile üyelerini davet edip, aynı liste üzerinde birlikte çalışabilirsiniz.
- **Güvenli üyelik & oturum**: E-posta doğrulama, şifre sıfırlama, davet onay akışları ile uçtan uca güvenli bir hesap yapısı sunar.
- **Blog ve bilgilendirici içerikler**: Çeyiz hazırlığına yönelik blog yazıları, ipuçları, sık sorulan sorular ve kampanya sayfaları.

---

## Mimari Genel Bakış

Proje, **.NET 9** tabanlı katmanlı bir backend ile **Vue 3 + Vite** tabanlı bir SPA frontend’inden ve **RabbitMQ üzerinde çalışan Mail Consumer** servisinden oluşur.

- **Çözüm yapısı**
  - **`Ceyizim.API`**: ASP.NET Core Web API katmanı
  - **`Ceyizim.Application`**: Uygulama servisleri, iş kuralları, servis arayüzleri
  - **`Ceyizim.Domain`**: Entity’ler, Request/Response modelleri, enum’lar
  - **`Ceyizim.Infrastructure`**: EF Core context ve migration’lar, repository ve unit of work implementasyonları, teknik altyapı
  - **`Ceyizim.Shared`**: Ortak DTO’lar, helper sınıfları, ayar modelleri
  - **`Ceyizim.VUE`**: Vue 3 SPA frontend projesi
  - **`Ceyizim.Consumers/MailConsumer`**: RabbitMQ üzerinden e-posta bildirimlerini dinleyen ve gönderen consumer servis

---

## Backend (API Katmanı)

- **Teknolojiler**
  - **.NET 9 (ASP.NET Core Web API)**
  - **Entity Framework Core 9**
  - **JWT tabanlı kimlik doğrulama**
  - **Serilog + Seq** ile merkezî loglama
  - **Swagger/OpenAPI** ile otomatik API dokümantasyonu

- **Katmanlar (yüksek seviye)**
  - **`Ceyizim.Domain`**: Domain entity’leri, istek/cevap modelleri, enum’lar.
  - **`Ceyizim.Application`**: Use-case’ler, application servisleri, servis ve repository arayüzleri.
  - **`Ceyizim.Infrastructure`**: EF Core context, migration’lar, repository ve unit of work implementasyonları, altyapı konfigürasyonları.
  - **`Ceyizim.Shared`**: Ortak DTO, helper ve ayar sınıfları.
  - **`Ceyizim.API`**: HTTP endpoint’lerini barındıran Web API projesi (controller ve kod detayları bu dokümantasyonda özellikle paylaşılmamaktadır).

---

## Frontend (Vue SPA)

- **Teknolojiler**
  - **Vue 3** (Composition API)
  - **Vite** (geliştirme ve build aracı)
  - **Pinia** + `pinia-plugin-persistedstate` (state yönetimi)
  - **Vue Router 4**
  - **TailwindCSS 4**, **Bootstrap 5**, **DaisyUI**
  - **Axios**, **SweetAlert2**, **NProgress**, **Font Awesome**

- **Yapı (yüksek seviye)**
  - `src/`
    - `views/`: Auth, profil, ürün/listeler, aile, promosyon ve blog sayfaları.
    - `stores/`: Auth, kullanıcı, ürün, liste, paylaşım, aile ve iletişim gibi global state’ler.
    - `components/`: Layout bileşenleri, kartlar, modal’lar, paylaşımlı link paneli vb. reusable UI parçaları.
    - `api/`, `utils/`, `assets/`, `config/`: API client, yardımcı fonksiyonlar, stiller ve konfigürasyon dosyaları.
  - `public/`: SEO dosyaları (ör. `robots.txt`, `sitemap.xml`) ve statik içerikler.
  - `Dockerfile`, `deploy/nginx.conf`: SPA dağıtımı için Docker ve Nginx konfigürasyonu.

---

## Mail Consumer (Background Servis)

- **Amaç**
  - Kayıt, şifre sıfırlama, e-posta doğrulama, aile daveti vb. işlemler için **asenkron e-posta gönderimi**.

- **Teknolojiler**
  - .NET tabanlı worker uygulaması
  - **RabbitMQ** + **MassTransit**

- **Yapı (yüksek seviye)**
  - `Program.cs` ve konfigürasyon dosyaları: Ortam ayarları ve queue bağlantısı.
  - `Configs/`, `Dtos/`, `Producers/`: Mesajlaşma ve e-posta gönderiminde kullanılan temel bileşenler.
  - `Dockerfile`: Mail consumer için container imajı.

---

## CI/CD ve Dağıtım

- **Kaynak kodu** GitHub üzerinde tutulmaktadır ve:
  - `.github/workflows/deploy-all.yml` ile **GitHub Actions** üzerinden backend, frontend ve consumer bileşenlerinin build/publish/deploy süreçleri otomatikleştirilebilmektedir.
- Her projede yer alan **Dockerfile**’lar sayesinde:
  - API (backend),
  - SPA (frontend),
  - Mail Consumer
  için ayrı container imajları üretilebilmekte ve bu imajlar istenen orkestrasyon ortamına (Docker Compose, Kubernetes vb.) publish edilebilmektedir.

---

## Genel Dosya Mimarisi (Çözüm Seviyesi)

```text
ceyizim/
├─ ceyizim.sln
├─ Ceyizim.API/             # Web API projesi
├─ Ceyizim.Application/     # Application katmanı
├─ Ceyizim.Domain/          # Domain modelleri
├─ Ceyizim.Infrastructure/  # EF Core, repository, migrations
├─ Ceyizim.Shared/          # Ortak DTO & helper katmanı
├─ Ceyizim.VUE/             # Vue 3 SPA projesi
└─ Ceyizim.Consumers/
   └─ MailConsumer/         # RabbitMQ tabanlı mail consumer servisi
```

---

## Üretim Ortamı ve Güvenlik Mimarisi

- Proje, **Ubuntu tabanlı bir sunucu** üzerinde çalışmaktadır.
- Dış dünyaya yalnızca:
  - **Frontend (Vue SPA)** için HTTP(S) trafiği,
  - Gerekli olduğu ölçüde **API endpoint’leri**
  açılmakta; **Mail Consumer**, **SQL veritabanı**, **RabbitMQ** gibi bileşenler tamamen **iç ağa kapalı** tutulmaktadır.
- Sunucuya **doğrudan SSH erişimi internete kapalıdır**; yönetim ve bakım işlemleri yalnızca **Tailscale tabanlı VPN** üzerinden, yetkilendirilmiş cihazlarla gerçekleştirilmektedir.

---

## Özet

**Çeyizim**, gerçek dünyada aktif olarak kullanılan ([ceyizim.net](https://www.ceyizim.net)) bir projedir ve:

- Modern bir **.NET 9 + Vue 3** yığını üzerinde,
- Katmanlı mimari ve temiz domain ayrımı ile,
- Asenkron e-posta altyapısı (RabbitMQ + MailConsumer),
- Zengin kullanıcı deneyimi, SEO ve blog içeriği

sunarak çeyiz hazırlığını daha planlı, şeffaf ve paylaşılabilir hale getirir.

