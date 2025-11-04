# 📘 Dokumen Kebutuhan Produk (PRD)

## Website Profil Perusahaan Devgenfour

## 1. Latar Belakang

Devgenfour adalah sebuah **software house** yang telah berhasil menangani berbagai proyek untuk klien dari berbagai industri. Namun, saat ini perusahaan belum memiliki **website resmi** untuk merepresentasikan merek, menampilkan karya, dan menarik klien baru.

Website ini akan menjadi **identitas digital Devgenfour** — sebuah platform profesional yang:

* Menyoroti keahlian dan portofolio perusahaan.
* Membangun kredibilitas dengan calon klien dan mitra.
* Mengkomunikasikan nilai dan budaya merek Devgenfour.
* Berfungsi sebagai titik kontak utama untuk pertanyaan dan kolaborasi.

---

## 2. Tujuan Produk

1. **Membangun kredibilitas** dan memposisikan Devgenfour sebagai mitra pengembangan perangkat lunak yang terpercaya.
2. **Menghasilkan prospek (leads)** melalui ajakan bertindak (CTA) dan kanal kontak yang jelas.
3. **Menampilkan portofolio perusahaan** dan ragam layanan yang ditawarkan.
4. **Menonjolkan identitas merek** melalui tampilan antarmuka yang minimalis, modern, dan profesional.

---

## 3. Pengguna Sasaran

* **Calon klien** — startup, perusahaan, atau institusi yang membutuhkan layanan pengembangan perangkat lunak.
* **Calon karyawan** — desainer, developer, dan manajer proyek yang tertarik bergabung dengan Devgenfour.
* **Mitra bisnis atau investor** — organisasi atau individu yang ingin menjajaki peluang kolaborasi.

---

## 4. Desain & Gaya Visual

### Arah Visual

* **Bahasa desain:** Minimalis, halus, dan bersih.
* **Mood:** Profesional, modern, elegan.
* **Warna aksen utama:** Biru muda `#5AB3F1`, dikombinasikan dengan warna putih dan abu muda.
* **Tipografi:**

  * Judul: *Poppins* atau *Inter* (tegas, bersih, modern).
  * Teks isi: *Open Sans* atau *Lato* (mudah dibaca, netral).

### Prinsip Desain

* **Kesederhanaan utama:** Fokus hanya pada konten esensial.
* **Ruang putih yang cukup:** Menjaga kejernihan visual dan keseimbangan ruang.
* **Grid dan jarak konsisten:** Menyatukan tata letak di berbagai ukuran layar.
* **Responsif penuh:** Desain dioptimalkan dengan pendekatan *mobile-first*.

---

## 5. Struktur Website

### 1. Beranda / Halaman Utama

* Bagian hero dengan judul:
  *“Kami membangun produk digital yang memberdayakan bisnis Anda.”*
* CTA: **“Mari Bekerja Sama”** → mengarah ke halaman Kontak.
* Tampilan 3–4 proyek unggulan.
* Testimoni klien (opsional).
* Nilai inti: Kualitas, Kolaborasi, Inovasi.

### 2. Tentang Kami

* Latar belakang dan timeline perusahaan.
* Profil kepemimpinan atau tim.
* Nilai inti dan pernyataan misi.

### 3. Layanan

* Gambaran layanan utama:

  * **Pengembangan Perangkat Lunak Kustom**
  * **Desain UI/UX**
  * **Pengembangan Aplikasi Mobile**
  * **Pengembangan Website**
  * **Integrasi & Pemeliharaan Sistem**
* CTA: **“Diskusikan Proyek Anda.”**

### 4. Portofolio

* Galeri proyek bergaya grid.
* Filter berdasarkan kategori (Web, Mobile, Enterprise, dll).
* Setiap halaman proyek mencakup:

  * Judul, klien, teknologi, deskripsi, dan tangkapan layar.

### 5. Kontak

* Formulir (nama, email, pesan).
* Detail kontak perusahaan dan peta Google Maps tersemat.
* CTA: **“Jadwalkan Panggilan.”**

### 6. Blog (Opsional — Fase 2)

* Artikel tentang teknologi, desain, dan pembaruan perusahaan.
* Tujuan: SEO dan otoritas merek.

---

## 6. Fitur Utama

| Fitur                         | Deskripsi                                                                     |
| ----------------------------- | ----------------------------------------------------------------------------- |
| **Panel Admin Sederhana**     | Antarmuka admin bawaan untuk mengelola konten situs (tanpa CMS pihak ketiga). |
| **Desain Responsif**          | Dioptimalkan untuk semua ukuran layar dan perangkat.                          |
| **Optimasi SEO**              | Pemrosesan cepat, metadata, dan struktur heading yang baik.                   |
| **Integrasi Formulir Kontak** | Sistem email berbasis Laravel dengan validasi.                                |
| **Manajemen Gambar & File**   | Unggah dan tampilkan gambar secara efisien dari penyimpanan.                  |
| **Keamanan**                  | Perlindungan CSRF, autentikasi, dan pencegahan spam.                          |

---

## 7. Spesifikasi Teknis (Lingkungan Laravel) — **Versi Final**

| Kategori                  | Spesifikasi                                                                                                         |
| ------------------------- | ------------------------------------------------------------------------------------------------------------------- |
| **Bahasa**                | PHP 8.3+                                                                                                            |
| **Framework**             | Laravel 12 (LTS preferred)                                                                                          |
| **Frontend**              | **Laravel Blade + Bootstrap 5**                                                                                     |
| **Interaktivitas UI**     | **Vanilla JavaScript** untuk manipulasi DOM ringan (modal, dropdown, validasi, animasi kecil)                       |
| **AJAX CRUD**             | **jQuery AJAX** untuk operasi CRUD tanpa reload halaman                                                             |
| **DataTable Integration** | **Yajra Laravel DataTables** untuk *server-side processing* (pagination, search, sort) agar controller tetap ringan |
| **Role Management**       | **Spatie Laravel Permission** untuk manajemen peran & izin (role & permission) berbasis middleware                  |
| **Database**              | MySQL 8+ atau PostgreSQL 18+ dengan Eloquent ORM                                                                    |
| **Panel Admin**           | Dibangun custom menggunakan Blade templates (tanpa Filament atau Nova)                                              |
| **Autentikasi**           | Laravel Breeze (login/logout/reset password)                                                           |
| **Caching & Performa**    | Redis untuk cache dan queue; route & config cache diaktifkan                                                        |
| **Sistem Email**          | Laravel Mail (Mailgun, Postmark, atau SES)                                                                          |
| **Hosting**               | Nginx + PHP-FPM via Laravel Forge atau Envoyer                                                                      |
| **Penyimpanan**           | Lokal atau AWS S3 dengan Laravel Filesystem                                                                         |
| **Analitik**              | Google Analytics 4 atau Plausible                                                                                   |
| **SEO Tools**             | spatie/laravel-seo dan spatie/laravel-sitemap                                                                       |
| **Keamanan**              | Perlindungan CSRF/XSS, hCaptcha atau reCAPTCHA untuk formulir                                                       |
| **Pengujian**             | PestPHP / PHPUnit untuk unit & feature test                                                                         |
| **Monitoring**            | Laravel Telescope (dev), Sentry (production)                                                                        |
| **Target Performa**       | TTFB < 200ms, LCP < 2.5s, PageSpeed ≥ 90 (desktop)                                                                  |

---

### 🔧 Penjelasan Teknis

1. **Bootstrap 5** digunakan untuk tampilan modern, elegan, dan cepat dikembangkan.
2. **Vanilla JS** untuk manipulasi UI ringan agar performa tetap optimal.
3. **jQuery AJAX** mengelola CRUD secara asinkron tanpa reload halaman.
4. **Yajra DataTables** menghadirkan *pagination, search, sort* dari sisi server (efisien untuk data besar).
5. **Spatie Laravel Permission** memberikan kontrol penuh terhadap peran & izin pengguna:

   * Role default: `Admin`, `Editor`, `Viewer (opsional)`
   * Middleware berbasis role dan permission (`role:Admin`, `permission:edit posts`)
   * Tabel bawaan: `roles`, `permissions`, `model_has_roles`, `role_has_permissions`
   * Integrasi dengan route:

     ```php
     Route::group(['middleware' => ['role:Admin']], function() {
         // Rute untuk admin
     });
     ```
6. Kombinasi **Spatie + Yajra + jQuery AJAX** menciptakan sistem admin yang **ringan, modular, dan scalable**.

---

## 8. Fitur Panel Admin

*(Tetap sama, dengan tambahan Spatie untuk otorisasi)*

---

### 8.1 Akses & Autentikasi

* Login admin dengan email/password
* Lupa kata sandi & reset melalui email
* Sesi berakhir setelah tidak aktif
* **Role Management** dengan Spatie (Admin, Editor, Viewer)

**Prefix route:** `/admin`

---

### 8.2 Dashboard

* Ringkasan total proyek, layanan, posting, dan pesan
* Tautan cepat untuk “Tambah Baru”
* Menampilkan data dinamis via Yajra DataTables
* Opsional: Ringkasan analitik dari Google Analytics API

---

### 8.3 Modul Manajemen Konten

*(Semua modul CRUD memanfaatkan jQuery AJAX + DataTables)*
### 8.3 Modul Manajemen Konten

#### a. Manajemen Layanan

* CRUD untuk layanan
* Kolom: judul, deskripsi singkat, deskripsi detail, ikon, urutan
* Aksi: Tambah/Edit/Hapus/Atur urutan

#### b. Manajemen Portofolio

* CRUD untuk studi kasus proyek
* Kolom: judul, klien, kategori, teknologi, deskripsi, hasil, gambar utama, galeri
* Tombol publikasi / nonpublikasi

#### c. Manajemen Tim

* CRUD untuk anggota tim
* Kolom: nama, peran, bio, foto, tautan sosial, urutan

#### d. Manajemen Blog (Fase 2)

* CRUD untuk posting blog dengan editor WYSIWYG (Trix atau TinyMCE)
* Kolom: judul, slug, ringkasan, konten, gambar utama, tag, status publikasi

#### e. Pesan Kontak

* Hanya untuk membaca kiriman formulir kontak
* Tandai sebagai dibaca atau arsip
* Hapus entri lama

---

### 8.4 Antarmuka & UX Panel Admin

*(Tetap sama, tapi kini menampilkan data menggunakan DataTables dengan AJAX call)*

* Navigasi sidebar: Dashboard, Layanan, Portofolio, Tim, Blog (opsional), Kontak
* Tampilan tabel dengan pagination, pencarian, dan pengurutan
* Validasi formulir dan notifikasi flash
* Unggah file dengan pratinjau gambar
* Perlindungan CSRF dan sanitasi input
* Tata letak dua kolom (sidebar + area konten) yang bersih

---

### 8.5 Struktur Route

```
/admin
├── /dashboard
├── /services
│   ├── create / edit / delete
├── /portfolio
│   ├── create / edit / delete
├── /team
│   ├── create / edit / delete
├── /blog (opsional)
│   ├── create / edit / delete
├── /contacts
│   ├── view / delete
└── /profile
    ├── change-password
```

---

### 8.6 Ringkasan Teknis

| Komponen        | Deskripsi                                                            |
| --------------- | -------------------------------------------------------------------- |
| **Views**       | Template Blade di `resources/views/admin`                            |
| **Controllers** | Namespace `app/Http/Controllers/Admin`                               |
| **Models**      | Service, Project, Post, TeamMember, ContactMessage, Role, Permission |
| **Middleware**  | `auth`, `verified`, `role`, `permission`                             |
| **Assets**      | Bootstrap 5, jQuery, Vanilla JS                                      |
| **DataTable**   | Yajra DataTables (server-side rendering)                             |
| **Uploads**     | Disimpan di `/storage/app/public/uploads`                            |
| **Editor**      | Trix atau TinyMCE (lokal, tanpa integrasi pihak ketiga)              |

---

## 9. Timeline Proyek (Estimasi)

| Fase                   | Durasi   | Deliverables                    |
| ---------------------- | -------- | ------------------------------- |
| Riset & Sitemap        | 1 minggu | Alur UX dan arsitektur situs    |
| Wireframe & UI Design  | 2 minggu | Mockup fidelity tinggi          |
| Pengembangan (Laravel) | 4 minggu | Website + Panel Admin Sederhana |
| Pengujian & Peluncuran | 1 minggu | Rilis publik                    |

**Total estimasi waktu:** 8 minggu

---

## 10. Metrik Keberhasilan

* Bounce rate < **40%**
* Durasi sesi rata-rata > **1 menit**
* ≥ **10 pertanyaan/bulan** via formulir kontak
* Skor PageSpeed > **90 (desktop)**
* Panel admin sepenuhnya berfungsi dan mudah digunakan tanpa dokumentasi

---

## 11. Catatan Tambahan

* Tema utama tetap **biru muda (#5AB3F1)** sebagai aksen utama.
* Gaya visual **teknologis namun humanis**, hindari tampilan terlalu korporat.
* Tambahkan **mikro-interaksi** untuk CTA dan transisi.
* Pastikan **kompatibilitas lintas browser**.
* Pisahkan **rute admin dan publik** untuk keamanan & skalabilitas.

---