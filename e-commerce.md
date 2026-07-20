# Blueprint Modul — Project Fullstack E-Commerce Multi-Vendor

> Dokumen konsep (blueprint) sebelum diimplementasikan jadi modul HTML seperti modul lain
> (`modul-*.html`). Isinya: apa yang dibangun, pembagian modul **per kategori fullstack**,
> desain database multi-vendor, alur integrasi **API Ongkir** & **Payment Gateway**, serta
> **wireframe/mockup** tiap halaman kunci.

---

## 1. Apa yang Dibangun

**Marketplace multi-vendor** — bukan toko online satu penjual, tapi platform tempat **banyak
penjual** membuka toko sendiri (seperti Tokopedia/Shopee). Satu pembeli bisa membeli produk
dari **beberapa toko sekaligus dalam satu transaksi**, dengan **ongkir dihitung otomatis per
toko** (karena tiap toko punya kota asal berbeda) dan **satu pembayaran digital** lewat
payment gateway.

### Tiga Aktor (Role)

| Role | Bisa apa |
|------|----------|
| **Buyer** (pembeli) | Jelajah katalog, keranjang, checkout multi-toko, bayar, lacak resi |
| **Seller** (vendor/penjual) | Buka toko, kelola produk & stok, terima pesanan, input resi, lihat saldo |
| **Admin** | Verifikasi toko, kelola kategori, pantau transaksi, proses payout ke vendor |

### Fitur Inti

| Fitur | Deskripsi |
|-------|-----------|
| Multi-toko dalam 1 cart | Keranjang dikelompokkan per toko, subtotal & ongkir per toko |
| Ongkir otomatis | Hitung real-time via API ongkir (asal = kota toko, tujuan = alamat pembeli, berat = total item) |
| Pemecahan order | 1 pembayaran → dipecah jadi beberapa **sub-order** per toko (tiap toko kirim & input resi sendiri) |
| Payment gateway | Bayar sekali untuk seluruh order (Midtrans Snap), status update via webhook |
| Manajemen resi | Tiap vendor input nomor resi → status kirim per sub-order |
| Payout / settlement | Dana diteruskan ke masing-masing vendor setelah pesanan selesai |

---

## 2. Arsitektur Fullstack (High Level)

```
                         ┌───────────────────────────────────────────┐
                         │           FRONTEND (Vue 3 SPA)            │
                         │  Katalog · Cart · Checkout · Dashboard    │
                         │  Vendor · Dashboard Admin                 │
                         └───────────────┬───────────────────────────┘
                                         │  REST API (JSON) + token (Sanctum)
                                         ▼
                         ┌───────────────────────────────────────────┐
                         │           BACKEND (Laravel API)           │
                         │  Auth & Role · Produk · Cart · Checkout   │
                         │  Order · Payment · Ongkir · Settlement    │
                         └───┬───────────────┬───────────────┬───────┘
                             │               │               │
              ┌──────────────▼──┐   ┌────────▼───────┐  ┌────▼──────────────┐
              │  PostgreSQL     │   │  API Ongkir    │  │  Payment Gateway  │
              │  (data utama)   │   │ (RajaOngkir/   │  │  (Midtrans Snap)  │
              │                 │   │  Komerce)      │  │                   │
              └─────────────────┘   └────────────────┘  └───────────────────┘
```

**Stack yang dipakai:**

| Lapisan | Teknologi | Alternatif |
|---------|-----------|-----------|
| Frontend | Vue 3 + Vite + Vue Router + Pinia + Vue Query + Tailwind | React (nanti, opsional) |
| Backend | Laravel 12 (REST API) + Sanctum | — |
| Database | PostgreSQL | MySQL |
| Ongkir | RajaOngkir / Komerce (Komship) | Biteship |
| Payment | Midtrans (Snap) | Xendit |
| Tools | Git · Docker · GitHub Actions | — |

---

## 3. Pembagian Modul per Kategori Fullstack

Project ini sengaja **dipecah per lapisan fullstack** supaya jelas bagian mana yang frontend,
backend, database, integrasi, dan infra. Urutan belajar mengikuti alur "dari dalam ke luar":
**Database → Backend → Integrasi → Frontend → DevOps**.

### KATEGORI A — Database (PostgreSQL)

| Kode | Modul | Fokus |
|------|-------|-------|
| DB-1 | Desain Skema Multi-Vendor & ERD | Model data marketplace, kenapa order dipecah jadi sub-order |
| DB-2 | Migrasi, Relasi, Index & Constraint | Foreign key, unique, index untuk pencarian produk |
| DB-3 | Transaksi & Concurrency | Kunci stok saat checkout, cegah *race condition* & oversell |

### KATEGORI B — Backend (Laravel REST API)

| Kode | Modul | Fokus |
|------|-------|-------|
| BE-1 | Setup API + Auth + Role | Sanctum, register/login, guard role buyer/seller/admin |
| BE-2 | Toko & Produk (Vendor) | CRUD toko, CRUD produk + stok + gambar + berat |
| BE-3 | Katalog Publik & Pencarian | Endpoint publik, filter kategori/harga, full-text search |
| BE-4 | Cart Multi-Vendor | Tambah/ubah item, pengelompokan per toko |
| BE-5 | Checkout & Pemecahan Order | 1 order → banyak sub-order per toko, validasi stok |
| BE-6 | Integrasi Ongkir | Hitung ongkir per toko (asal berbeda), pilih kurir/layanan |
| BE-7 | Integrasi Payment Gateway | Midtrans Snap token, webhook, verifikasi signature |
| BE-8 | Order Management & Resi | Status kirim per sub-order, input resi vendor, lacak |
| BE-9 | Settlement / Payout Vendor | Saldo vendor, pencairan setelah order selesai |

### KATEGORI C — Frontend (Vue 3 SPA)

| Kode | Modul | Fokus |
|------|-------|-------|
| FE-1 | Setup Vue + Routing + Auth | Vite, Vue Router, Pinia, simpan token, protected route |
| FE-2 | Katalog & Detail Produk | Grid produk, filter, halaman detail + galeri |
| FE-3 | Cart (Grouped per Toko) | Tampilan keranjang dikelompokkan per toko |
| FE-4 | Checkout | Pilih alamat, pilih kurir per toko, ringkasan biaya |
| FE-5 | Pembayaran & Status Order | Snap popup Midtrans, halaman status & lacak resi |
| FE-6 | Dashboard Vendor | Kelola produk, daftar pesanan masuk, input resi, saldo |
| FE-7 | Dashboard Admin | Verifikasi toko, kelola kategori, monitor transaksi |

### KATEGORI D — Integrasi Eksternal

| Kode | Modul | Fokus |
|------|-------|-------|
| INT-1 | API Ongkir (RajaOngkir/Komerce) | Ambil provinsi/kota, hitung tarif, caching wilayah |
| INT-2 | Payment Gateway (Midtrans) | Sandbox, Snap, webhook, skenario gagal/expired |

### KATEGORI E — DevOps & Tools

| Kode | Modul | Fokus |
|------|-------|-------|
| OPS-1 | Git Workflow | Branch per fitur, konvensi commit, PR review |
| OPS-2 | Docker (App + PostgreSQL) | `docker-compose` backend + db + frontend |
| OPS-3 | Testing | Pest (backend), Vitest (frontend), test alur checkout |
| OPS-4 | CI/CD & Deploy | GitHub Actions, deploy backend & frontend |

---

## 4. Desain Database Multi-Vendor (ERD)

Inti perbedaan dengan e-commerce satu penjual: **`order` (level pembayaran) dipecah jadi
`sub_orders` (level per toko)**. Satu pembeli bayar sekali, tapi barangnya dikirim terpisah
per toko dengan ongkir & resi masing-masing.

```
users ──────┐
  id         │ hasMany
  name       ├──────────────► stores
  email      │                  id
  password   │                  user_id (FK)
  role       │                  nama_toko, slug
             │                  kota_id        ← asal pengiriman
             │                  status
             │
             │ hasMany            ▲ belongsTo
             ├──► addresses       │
             │      id            │
             │      user_id       │ hasMany
             │      penerima      └──────────── products
             │      telepon                       id
             │      provinsi_id                   store_id (FK)
             │      kota_id                        category_id (FK)
             │      kecamatan                      nama, slug, harga
             │      detail                         stok, berat_gram
             │                                     status
             │ hasOne                              │ hasMany
             ├──► carts                            └──► product_images
             │      id                                   id, product_id, path, is_primary
             │      user_id
             │        │ hasMany
             │        └──► cart_items
             │               id, cart_id (FK)
             │               product_id (FK)
             │               qty
             │
             │ hasMany
             └──► orders  ◄────────────── LEVEL PEMBAYARAN (1 transaksi)
                    id
                    user_id (FK)
                    kode_invoice
                    address_id (FK)      ← alamat tujuan
                    total_barang
                    total_ongkir
                    total_bayar
                    status_bayar (pending|paid|expired|failed)
                    │
                    │ hasMany
                    ├──► sub_orders  ◄──── LEVEL PER TOKO (dikirim terpisah)
                    │      id
                    │      order_id (FK)
                    │      store_id (FK)
                    │      subtotal
                    │      ongkir
                    │      kurir, layanan       ← contoh: JNE REG
                    │      berat_total_gram
                    │      no_resi
                    │      status_kirim (menunggu|diproses|dikirim|selesai)
                    │      │ hasMany
                    │      └──► order_items
                    │             id, sub_order_id (FK)
                    │             product_id (FK)
                    │             qty
                    │             harga_saat_beli   ← beku harga saat transaksi
                    │             berat_saat_beli
                    │
                    └──► payments  (hasOne)
                           id, order_id (FK)
                           midtrans_order_id
                           snap_token
                           metode
                           status
                           raw_response (jsonb)

settlements  (payout ke vendor)
  id, store_id (FK), sub_order_id (FK), amount, status (tertahan|cair), created_at

categories  (self-reference)
  id, nama, slug, parent_id (nullable FK → categories.id)
```

### Kenapa Order Dipecah Jadi Sub-Order?

Karena tiap toko punya **kota asal berbeda** → **ongkir berbeda** → **kurir & resi berbeda**.
Kalau semua digabung dalam satu `order` datar, tidak ada tempat menyimpan resi per toko dan
tidak bisa melacak pengiriman yang datang di waktu berbeda. Level `orders` menyatukan
**pembayaran** (biar pembeli cukup bayar sekali), level `sub_orders` menyatukan
**pengiriman** (tiap toko urus kirimannya sendiri).

### Kenapa `harga_saat_beli` & `berat_saat_beli` Dibekukan?

Harga dan berat produk di master data bisa berubah kapan saja. Nilai di `order_items` membekukan
kondisi **saat transaksi terjadi**, supaya invoice lama & perhitungan ongkir historis tetap akurat
walau produk sumbernya berubah nanti.

---

## 5. Alur Integrasi Ongkir & Payment Gateway

### 5a. Alur Ongkir (saat Checkout)

```
Pembeli pilih alamat tujuan (kota X)
        │
        ▼
Backend kelompokkan cart per toko
        │
        ▼
Untuk TIAP toko:
   asal    = kota_id toko
   tujuan  = kota_id alamat pembeli
   berat   = Σ (qty × berat_gram) item toko itu
        │
        ▼
Panggil API Ongkir  ──►  RajaOngkir/Komerce  ──►  daftar kurir + tarif
        │
        ▼
Frontend tampilkan pilihan kurir PER TOKO
   (Toko A: JNE 12.000 / SiCepat 10.000 …)
   (Toko B: J&T  9.000  / …)
        │
        ▼
Pembeli pilih layanan tiap toko → ongkir terkunci per sub-order
```

### 5b. Alur Payment Gateway (Midtrans Snap)

```
1. Pembeli klik "Bayar"                → OrderController::store()
   → validasi & kunci stok (transaksi DB)
   → buat 1 order + N sub_orders + order_items
   → total_bayar = Σ subtotal + Σ ongkir
   → kosongkan cart

2. Backend minta Snap Token           → PaymentController::createSnapToken()
   → kirim detail order ke Midtrans
   → simpan snap_token & midtrans_order_id

3. Frontend buka Snap popup (bayar)   → snap.pay(token)

4. Midtrans kirim WEBHOOK ke server   → PaymentController::webhook()
   → verifikasi signature (keamanan)
   → status "settlement/capture" → orders.status_bayar = paid
   → sub_orders.status_kirim = diproses
   → (opsional) broadcast notifikasi real-time ke tiap vendor

5. Vendor proses & input resi          → SubOrderController::inputResi()
   → status_kirim = dikirim

6. Pembeli konfirmasi terima            → status_kirim = selesai
   → dana sub-order masuk settlement vendor
```

### Kenapa Status Diubah lewat Webhook, Bukan Redirect?

Redirect setelah bayar **tidak bisa dipercaya** — pembeli bisa menutup browser sebelum kembali,
atau memalsukan URL sukses. **Webhook dari Midtrans** adalah sumber kebenaran status pembayaran,
dan wajib **diverifikasi signature-nya** supaya tidak ada yang bisa memalsukan notifikasi "paid".

---

## 6. Wireframe / Mockup Halaman Kunci

### 6a. Homepage / Katalog Marketplace

```
┌────────────────────────────────────────────────────────────────┐
│  🛍 MultiMart      [ Cari produk…            🔍 ]   🛒(3)  👤   │
├────────────────────────────────────────────────────────────────┤
│  Kategori:  Elektronik · Fashion · Rumah · Hobi · Olahraga  ▾   │
├────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐         │
│  │ [ foto ] │  │ [ foto ] │  │ [ foto ] │  │ [ foto ] │         │
│  │ Produk A │  │ Produk B │  │ Produk C │  │ Produk D │         │
│  │ Rp120.000│  │ Rp 89.000│  │ Rp250.000│  │ Rp 45.000│         │
│  │ ⭐4.8 Toko│  │ ⭐4.5 Toko│  │ ⭐4.9 Toko│  │ ⭐4.7 Toko│         │
│  │  Sinar    │  │  Maju     │  │  Jaya     │  │  Berkah   │         │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘         │
│                                                                  │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐         │
│  │   …      │  │   …      │  │   …      │  │   …      │         │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘         │
│                     ◄  1  2  3  …  ►                             │
└────────────────────────────────────────────────────────────────┘
```

### 6b. Detail Produk

```
┌────────────────────────────────────────────────────────────────┐
│  ‹ Kembali                                                       │
├───────────────────────────┬────────────────────────────────────┤
│                           │  Sepatu Lari XYZ                     │
│      ┌───────────────┐    │  ⭐ 4.8 (120 ulasan) · 340 terjual   │
│      │               │    │                                      │
│      │   [ foto      │    │  Rp 250.000                          │
│      │     utama ]   │    │                                      │
│      │               │    │  🏪 Toko Jaya  · Bandung             │
│      └───────────────┘    │  Berat: 800 gram                     │
│      □  □  □  □            │                                      │
│    (thumbnail galeri)     │  Jumlah:  [ − ] 1 [ + ]   Stok: 25   │
│                           │                                      │
│                           │  [  + Keranjang  ]  [  Beli Sekarang]│
├───────────────────────────┴────────────────────────────────────┤
│  Deskripsi                                                       │
│  Lorem ipsum dolor sit amet …                                    │
└────────────────────────────────────────────────────────────────┘
```

### 6c. Keranjang (Dikelompokkan per Toko)

```
┌────────────────────────────────────────────────────────────────┐
│  Keranjang Saya                                                  │
├────────────────────────────────────────────────────────────────┤
│  🏪 Toko Jaya (Bandung)                                    [☑]  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ [img] Sepatu Lari XYZ     Rp250.000   [−] 1 [+]   🗑        │ │
│  │ [img] Kaos Kaki Sport     Rp 25.000   [−] 2 [+]   🗑        │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                        Subtotal toko: Rp300.000  │
├────────────────────────────────────────────────────────────────┤
│  🏪 Toko Sinar (Jakarta)                                   [☑]  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ [img] Powerbank 10.000mAh Rp120.000   [−] 1 [+]   🗑        │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                        Subtotal toko: Rp120.000  │
├────────────────────────────────────────────────────────────────┤
│                     Total Barang: Rp420.000  [  Checkout (3)  ]  │
└────────────────────────────────────────────────────────────────┘
```

### 6d. Checkout (Ongkir per Toko)

```
┌────────────────────────────────────────────────────────────────┐
│  Checkout                                                        │
├────────────────────────────────────────────────────────────────┤
│  📍 Alamat Pengiriman                                           │
│     Rahmat · 0812xxxx · Jl. Merdeka No.10, Depok, Jawa Barat    │
│                                                     [ Ubah ]     │
├────────────────────────────────────────────────────────────────┤
│  🏪 Toko Jaya (Bandung)                                         │
│     Sepatu Lari XYZ ×1, Kaos Kaki ×2         Rp300.000          │
│     Berat total: 1.100 gram                                     │
│     Kurir:  ( ) SiCepat REG   Rp10.000                          │
│             (•) JNE REG       Rp12.000  (est. 2-3 hari)         │
│             ( ) J&T EZ        Rp11.000                          │
├────────────────────────────────────────────────────────────────┤
│  🏪 Toko Sinar (Jakarta)                                        │
│     Powerbank ×1                             Rp120.000          │
│     Kurir:  (•) J&T EZ        Rp 9.000                          │
│             ( ) JNE REG       Rp10.000                          │
├────────────────────────────────────────────────────────────────┤
│  Ringkasan                                                      │
│     Total Barang ............ Rp420.000                        │
│     Total Ongkir ............ Rp 21.000                        │
│     ─────────────────────────────────────                      │
│     Total Bayar ............. Rp441.000     [  Bayar Sekarang ] │
└────────────────────────────────────────────────────────────────┘
```

### 6e. Dashboard Vendor — Pesanan Masuk

```
┌────────────────────────────────────────────────────────────────┐
│  🏪 Toko Jaya          Produk · Pesanan · Saldo · Pengaturan    │
├────────────────────────────────────────────────────────────────┤
│  Pesanan Masuk                          Filter: [ Perlu Kirim ▾]│
├──────────┬───────────────┬───────────┬──────────┬──────────────┤
│ Invoice  │ Produk        │ Pembeli   │ Status   │ Aksi         │
├──────────┼───────────────┼───────────┼──────────┼──────────────┤
│ INV-0012 │ Sepatu ×1 +1  │ Rahmat    │ Diproses │ [Input Resi] │
│ INV-0011 │ Kaos Kaki ×3  │ Dewi      │ Dikirim  │ [Lihat]      │
│ INV-0009 │ Sandal ×2     │ Budi      │ Selesai  │ [Lihat]      │
└──────────┴───────────────┴───────────┴──────────┴──────────────┘
   Saldo tertahan: Rp1.250.000   ·   Saldo cair: Rp3.400.000
```

### 6f. Dashboard Admin — Ringkasan

```
┌────────────────────────────────────────────────────────────────┐
│  ⚙ Admin       Toko · Kategori · Transaksi · Payout            │
├────────────────────────────────────────────────────────────────┤
│  ┌───────────┐ ┌───────────┐ ┌───────────┐ ┌───────────┐        │
│  │ Toko      │ │ Produk    │ │ Order hari │ │ Omzet bln │        │
│  │   128     │ │  4.512    │ │   ini 87   │ │ Rp210 jt  │        │
│  └───────────┘ └───────────┘ └───────────┘ └───────────┘        │
├────────────────────────────────────────────────────────────────┤
│  Toko Menunggu Verifikasi                                       │
│   • Toko Makmur   (Surabaya)   [ Setujui ] [ Tolak ]           │
│   • Toko Sentosa  (Medan)      [ Setujui ] [ Tolak ]           │
├────────────────────────────────────────────────────────────────┤
│  Payout Menunggu Proses                                        │
│   • Toko Jaya    Rp1.250.000   [ Cairkan ]                     │
└────────────────────────────────────────────────────────────────┘
```

---

## 7. Urutan Pengerjaan yang Disarankan

```
Fase 1  Database        DB-1 → DB-2 → DB-3
Fase 2  Backend inti    BE-1 → BE-2 → BE-3 → BE-4 → BE-5
Fase 3  Integrasi       INT-1 (ongkir) → BE-6 → INT-2 (payment) → BE-7 → BE-8
Fase 4  Frontend        FE-1 → FE-2 → FE-3 → FE-4 → FE-5 → FE-6 → FE-7
Fase 5  Payout & Admin  BE-9 (+ dashboard admin)
Fase 6  DevOps          OPS-1 → OPS-2 → OPS-3 → OPS-4
```

**Kenapa Database dulu, Frontend belakangan?** Frontend butuh endpoint API yang stabil untuk
dipanggil. Dengan menyelesaikan database → backend → integrasi lebih dulu, saat masuk frontend
semua data & alur sudah bisa diuji lewat Postman, sehingga frontend tinggal "menghidupkan"
tampilan tanpa menebak-nebak bentuk respons API.

---

## 8. Rencana Implementasi HTML

Tiap kode modul di atas (DB-1, BE-1, FE-1, …) akan jadi **satu halaman HTML** dengan gaya sama
seperti modul lain di repo ini (topbar + sidebar + TOC kanan, callout "Kenapa…", blok
**"Cara Membuktikan Tahap Ini Berhasil"**). Usulan penempatan file & menu navbar:

```
e-commerce/
  modul-db-skema.html          (Database)
  modul-db-migrasi.html
  modul-db-transaksi.html
  modul-be-auth.html           (Backend)
  modul-be-produk.html
  … dst …
  modul-fe-setup.html          (Frontend)
  … dst …
  modul-int-ongkir.html        (Integrasi)
  modul-int-payment.html
  modul-ops-docker.html        (DevOps)
```

Di **navbar besar**, project ini masuk kategori **Project → E-Commerce Multi-Vendor**, dengan
sub-menu dikelompokkan per lapisan fullstack (Database · Backend · Frontend · Integrasi · DevOps)
— konsisten dengan pengelompokan kategori yang sudah dibuat di `index.html`.

---

> **Status:** Blueprint konsep. Langkah berikutnya = implementasi modul HTML per kode di atas,
> mulai dari **DB-1 (Desain Skema Multi-Vendor)**.
