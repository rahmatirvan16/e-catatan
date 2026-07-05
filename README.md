# E-Catatan &mdash; Modul Belajar Laravel

Aplikasi catatan belajar berisi modul/langkah-langkah belajar dengan tampilan HTML
yang responsif, sehingga memudahkan mencari modul kapan saja &mdash; termasuk dibaca
langsung online lewat GitHub.

## Cara Membaca Online (GitHub Pages)

Karena semua modul berupa file HTML statis (tanpa backend), GitHub tidak bisa
me-render-nya langsung di halaman repo. Aktifkan **GitHub Pages** agar bisa dibuka
seperti website biasa:

1. Push folder ini ke repository GitHub.
2. Buka **Settings &rarr; Pages**.
3. Pilih **Deploy from a branch**, branch `main`, folder `/ (root)`.
4. Setelah aktif, buka `https://<username>.github.io/<repo>/` &mdash; halaman
   [index.html](index.html) akan tampil sebagai menu utama.

Tanpa GitHub Pages, modul tetap bisa dibaca dengan mengunduh/`clone` repo lalu
dibuka langsung di browser, atau dijalankan lokal via XAMPP (`index.php` akan
redirect otomatis ke `index.html`).

## Menu Laravel &mdash; Roadmap Belajar

🎉 **Roadmap 11 minggu sudah lengkap!** Modul disusun mengikuti roadmap berikut.
Baca detailnya juga di [index.html](index.html).

| Minggu | Fokus Utama | Teknologi / Fitur | Status |
|---|---|---|---|
| 1 | Core Laravel & Database | Instalasi (XAMPP/WSL), Eloquent & relasi, Migrations, Query Optimization (N+1, Indexing), CSRF/XSS/SQL Injection, REST API, JWT | ✅ [Tersedia](modul-rest-api-sanctum.html) ([+JWT](modul-rest-api-jwt.html)) |
| 2 | Arsitektur Kode & Otorisasi | Repository/Service Pattern, Middleware & Validation, Roles & Permission (Spatie), Rate Limiting | ✅ [Tersedia](modul-arsitektur-otorisasi.html) |
| 3 | Testing & Optimasi | Pest/PHPUnit, Redis, Laravel Horizon (Queues) | ✅ [Tersedia](modul-optimasi.html) |
| 4 | Observability | Laravel Telescope, Logging (Monolog), Error Tracking (Sentry) | ✅ [Tersedia](modul-observability.html) |
| 5 | API Design & Real-time | OpenAPI/Swagger, API Versioning, WebSocket/Broadcasting (Laravel Reverb) | ✅ [Tersedia](modul-api-design-realtime.html) |
| 6 | Security Deep-dive | OWASP Top 10, Encryption & Secrets Management, API Security Hardening | ✅ [Tersedia](modul-security-deepdive.html) |
| 7 | Capstone Project: E-commerce | API e-commerce lengkap (produk, cart, checkout, payment gateway, order management) | ✅ [Tersedia](modul-capstone-ecommerce.html) |
| 8 | Containerization & Deployment | Docker, CI/CD (GitHub Actions), k6 (Load Testing) | ✅ [Tersedia](modul-containerization-deployment.html) |
| 9 | Cloud & Infrastructure | Nginx/Supervisor, Cloud (AWS/GCP), Infrastructure as Code (Terraform) | ✅ [Tersedia](modul-cloud-infrastructure.html) |
| 10 | Skala Besar | Microservices, Message Brokers (Kafka/RabbitMQ), CQRS & Event Sourcing, Kubernetes | ✅ [Tersedia](modul-skala-besar.html) |
| 11 | System Design | Database Sharding & Replication, Load Balancing, Distributed Systems (CAP Theorem) | ✅ [Tersedia](modul-system-design.html) |

## Struktur File

- `index.html` &mdash; halaman menu utama (roadmap Laravel per minggu)
- `index.php` &mdash; redirect ke `index.html` (untuk akses via XAMPP/Apache)
- `modul-rest-api-sanctum.html` &mdash; Minggu 1: Instalasi (XAMPP/WSL), CRUD, Eloquent & Relasi, Query Optimization, REST API + Sanctum, Keamanan dasar
- `modul-rest-api-jwt.html` &mdash; Minggu 1 (lanjutan): Workshop autentikasi JWT
- `modul-arsitektur-otorisasi.html` &mdash; Minggu 2: Repository/Service Pattern, Middleware & Form Request, Spatie Roles & Permission, Rate Limiting
- `modul-optimasi.html` &mdash; Minggu 3: Redis, Laravel Horizon, Pest/PHPUnit
- `modul-observability.html` &mdash; Minggu 4: Laravel Telescope, Logging &amp; Monolog Channel, Error Tracking dengan Sentry
- `modul-api-design-realtime.html` &mdash; Minggu 5: API Versioning, Dokumentasi OpenAPI/Swagger, Real-time dengan Laravel Reverb
- `modul-security-deepdive.html` &mdash; Minggu 6: OWASP Top 10, Encryption &amp; Secrets Management, API Security Hardening
- `modul-capstone-ecommerce.html` &mdash; Minggu 7: Capstone &mdash; Shopping Cart, Checkout, Payment Gateway (Midtrans), Order Management
- `modul-containerization-deployment.html` &mdash; Minggu 8: Install Docker Desktop, Dockerize Laravel, CI/CD (GitHub Actions), Install &amp; jalankan k6 Load Testing
- `modul-cloud-infrastructure.html` &mdash; Minggu 9: Install &amp; konfigurasi Nginx/Supervisor, Install CLI AWS/GCP, Install Terraform, Deploy ke Cloud
- `modul-skala-besar.html` &mdash; Minggu 10: Microservices, Install RabbitMQ/Kafka, CQRS &amp; Event Sourcing, Install kubectl/Minikube
- `modul-system-design.html` &mdash; Minggu 11 (terakhir): Database Replication &amp; Sharding, Load Balancing, CAP Theorem
- `about.md` &mdash; deskripsi singkat aplikasi
