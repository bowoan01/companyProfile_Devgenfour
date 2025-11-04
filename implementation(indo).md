# Rencana Implementasi

## 1. Cakupan & Pendekatan

* Menghasilkan situs profil perusahaan berbasis Laravel 12 dengan halaman Home, About, Services, Portfolio, Contact, dan Blog (fase selanjutnya), dilengkapi panel admin khusus untuk mengelola semua konten publik.
* Mengimplementasikan CRUD berbasis AJAX untuk layanan, proyek, anggota tim, postingan, dan manajemen kontak menggunakan jQuery, Yajra DataTables, dan Vanilla JS agar antarmuka tetap responsif tanpa perlu memuat ulang halaman penuh.
* Mengadopsi arsitektur Blade yang terkomponen dengan Bootstrap 5 untuk layout, Vite untuk pengelolaan aset, serta pemisahan ketat antara area publik dan `/admin` menggunakan middleware Spatie Permission.
* Mengikuti alur kerja iteratif: finalisasi aset UI dari desain, bangun model domain dan migrasi, implementasikan halaman publik, lalu lapisi dengan fitur CRUD admin, QA, dan otomatisasi deployment.

---

## 2. Arsitektur Tingkat Tinggi

```
Browser (Publik + Admin)
        │
        ▼
Laravel 12 (Blade, Controllers, Policies, Services)
        │            │
        │            ├── Redis (queues, cache, rate limiting)
        │            └── Mailer (Mailgun/Postmark/SES)
        ▼
MySQL 8+ (konten, autentikasi, role/permission Spatie)
```

* Permintaan publik diarahkan ke controller Blade yang mengambil konten dari cache dan merender template Bootstrap.
* Permintaan admin melewati middleware `auth` + `role/permission`, menggunakan endpoint AJAX untuk operasi CRUD yang diolah oleh DataTables.
* Job antrian menangani pengiriman email, cache warming, dan pemrosesan media; perintah terjadwal menghasilkan sitemap dan menjalankan tugas pemeliharaan.
* Penyimpanan menggunakan disk publik (lokal atau S3) dengan `storage:link` untuk menayangkan file upload.

---

## 3. Lingkungan & Peralatan

| Lingkungan | Tujuan                          | Hosting & Konfigurasi                                                  |
| ---------- | ------------------------------- | ---------------------------------------------------------------------- |
| Lokal      | Pengembangan fitur, QA          | Laravel Sail atau Valet, `.env.local`, Telescope diaktifkan            |
| Staging    | Review stakeholder, uji regresi | Server Forge, HTTPS, diisi data dummy                                  |
| Produksi   | Lalu lintas nyata               | Forge + Envoyer blue/green deploy, SSL otomatis, pekerja antrian aktif |

* Stack inti: PHP 8.3, Laravel 12 LTS, MySQL 8+, Redis, Nginx + PHP-FPM.
* Frontend: Blade, Bootstrap 5, Vite, Vanilla JS, jQuery (untuk integrasi AJAX dan DataTables).
* Paket: spatie/laravel-permission, yajra/laravel-datatables, spatie/laravel-seo, spatie/laravel-sitemap, Laravel Breeze, Laravel Telescope (dev), Sentry (produksi).
* Alat bantu: PestPHP/PHPUnit, PHP-CS-Fixer atau Pint, ESLint (opsional), Prettier, Git hooks via Husky, CI dengan GitHub Actions, dan deployment melalui Forge/Envoyer.

---

## 4. Desain Basis Data

| Tabel                                                          | Kolom Kunci                                                                                  | Relasi                                                  | Tujuan                                             |
| -------------------------------------------------------------- | -------------------------------------------------------------------------------------------- | ------------------------------------------------------- | -------------------------------------------------- |
| users                                                          | name, email, password, email_verified_at                                                     | `hasMany` posts, contact_notes                          | Akun terautentikasi untuk akses admin              |
| roles / permissions                                            | name, guard_name                                                                             | Many-to-many dengan users via tabel pivot Spatie        | Role-based access control                          |
| model_has_roles / role_has_permissions / model_has_permissions | model_type, model_id, role_id / permission_id                                                | Pivot Spatie                                            | Menghubungkan user dengan role dan permission      |
| services                                                       | title, slug, short_description, description, icon_path, display_order, is_published          | N/A                                                     | Bagian Layanan publik                              |
| projects                                                       | title, slug, client, category, technology_stack, summary, results, cover_image, is_published | `hasMany` project_images; opsional `belongsToMany` tags | Studi kasus portofolio                             |
| project_images                                                 | project_id, path, caption, display_order                                                     | `belongsTo` projects                                    | Galeri gambar proyek                               |
| teams                                                          | name, role_title, bio, photo_path, linkedin_url, order_index, is_visible                     | N/A                                                     | Profil tim                                         |
| posts                                                          | title, slug, excerpt, body, cover_image, status, published_at, author_id                     | `belongsTo` users; `belongsToMany` tags                 | Blog (fase 2)                                      |
| tags                                                           | name, slug                                                                                   | `belongsToMany` posts/projects                          | Taksonomi konten                                   |
| post_tag / project_tag                                         | post_id/project_id, tag_id                                                                   | Pivot table                                             | Asosiasi tag                                       |
| contact_messages                                               | name, email, company, phone, message, status, handled_by, handled_at                         | `belongsTo` users (handled_by)                          | Data prospek dari formulir kontak                  |
| contact_notes                                                  | contact_message_id, user_id, note                                                            | `belongsTo` contact_messages & users                    | Catatan tindak lanjut internal                     |
| media                                                          | uuid, model_type, model_id, disk, path, collection                                           | Polimorfik                                              | Penanganan aset terpusat (opsional)                |
| settings                                                       | key, value, type, group                                                                      | N/A                                                     | Mengelola metadata situs, teks hero, tautan sosial |

---

## 5. Routing & Controllers

* **Area publik (`routes/web.php`):**

  * Route bernama untuk home, about, services index/show, portfolio index/show, contact (GET/POST), dan blog (aktif pada fase 2).
  * Controller berada di `App\Http\Controllers\Site`, mengembalikan Blade view dengan caching route dan composer view.

* **Area admin (`routes/admin.php`)** dengan prefix `/admin` dan middleware `['web','auth','verified','role:Admin|Editor']`:

  * `DashboardController@index` menampilkan metrik dan total entitas.
  * Controller resource seperti `ServiceController`, `ProjectController` menyediakan endpoint JSON untuk DataTables (`index`, `store`, `update`, `destroy`) serta Blade view untuk form.
  * `UploadController` menangani unggahan media; `ContactMessageController` untuk membaca/arsip; `ProfileController` untuk ubah password.

* **API routes (`routes/api.php`)** hanya untuk endpoint JSON ringan jika dibutuhkan, dilindungi `auth:sanctum` dan permission.

* Middleware tambahan: `EnsureFrontendRequestsAreStateful`, `SetLocale` (jika multi-bahasa), rate limiting untuk form kontak, dan guard `permission:` untuk endpoint CRUD.

---

## 6. Implementasi Frontend

* Struktur layout Blade (`layouts/app.blade.php`, `layouts/admin.blade.php`) dengan partial untuk navigasi, footer, modal, dan notifikasi.
* Gunakan kelas utilitas Bootstrap 5, modul SCSS kustom melalui Vite, dan variabel CSS untuk palet warna utama (`#5AB3F1`).
* Modul Vanilla JS mengatur elemen interaktif: animasi hero, efek scroll, validasi form, dan navigasi responsif.
* jQuery digunakan hanya untuk operasi AJAX CRUD; `$.ajax` mengembalikan JSON dan memperbarui DOM secara dinamis.
* Terapkan progressive enhancement: form dapat dikirim normal, JS mencegat untuk pengiriman XHR jika tersedia.
* Optimasi performa: gambar *lazy-load*, tag `<picture>` responsif, *prefetch* CSS penting, dan script analitik dimuat asinkron.

---

## 7. Detail UX Panel Admin

* Layout admin: sidebar tetap dengan navigasi keyboard, topbar untuk aksi cepat, dan area konten berbasis breadcrumb.
* Setiap modul menggunakan Yajra DataTables untuk pagination, sortir, dan filter kolom server-side; tetap responsif di tablet.
* Form CRUD muncul dalam modal atau halaman tersendiri; hasil validasi dikembalikan dalam JSON dan ditampilkan langsung tanpa reload.
* Gunakan komponen Blade reusable (`<x-admin.form.input>`, `<x-admin.table.actions>`) agar konsisten; notifikasi menggunakan toast JS.
* Spatie Permission mengatur visibilitas menu dan tombol aksi: Editor bisa tambah/edit, Admin bisa hapus/publish.
* Pratinjau unggahan file menggunakan FileReader API dengan validasi ukuran/tipe; drag-and-drop reordering diterapkan untuk urutan layanan/proyek.

---

## 8. Keamanan & Kepatuhan

* Wajib HTTPS di staging/produksi dengan HSTS; atur proxy tepercaya untuk load balancer Forge.
* Token CSRF di semua form; sanitasi input via `request->validated()` dan HTML Purifier untuk konten WYSIWYG.
* Rate limiting pada login & contact form (`ThrottleRequests`); tambah hCaptcha/reCAPTCHA v3.
* Hash password menggunakan Argon2id, terapkan aturan password kuat, aktifkan 2FA (opsional via Fortify/Breeze).
* Terapkan header keamanan: CSP, X-Frame-Options, Referrer-Policy.
* Job terjadwal menghapus data kontak lama sesuai kebijakan privasi; akses data dilacak via Telescope/Sentry untuk audit.

---

## 9. Alur Manajemen Konten

1. **Layanan:** Admin melihat tabel layanan via DataTables, membuka modal untuk tambah/edit, urutan diubah lewat drag handle menuju endpoint `PUT /services/{id}/reorder`.
2. **Portofolio:** Proyek dikelola via modal multi-step; unggahan ke endpoint media melalui AJAX; toggle publish via `PATCH /projects/{id}/status`.
3. **Tim:** CRUD sederhana dengan tautan sosial opsional; toggle visibilitas kontrol tampil di halaman publik.
4. **Blog (Fase 2):** Editor WYSIWYG Trix/TinyMCE; slug otomatis, tag dipilih dengan input token; alur publikasi mengikuti permission Editor/Admin.
5. **Kontak:** DataTable menampilkan pesan belum dibaca; klik baris membuka panel pesan dengan opsi tandai selesai, beri catatan internal, arsip, atau ekspor CSV via job antrean.

---

## 10. Strategi Pengujian

* **Unit Test:** Menguji model Eloquent (accessor, scope), service class, dan gate permission.
* **Feature Test:** Memastikan halaman publik tampil benar, form kontak memvalidasi dan melindungi dari spam, endpoint admin mematuhi izin dan mengembalikan JSON sesuai.
* **Integration Test:** Mensimulasikan alur CRUD AJAX menggunakan `actingAs` dengan role tertentu; memverifikasi respons DataTables, upload file, dan dispatch antrian email.
* **Browser Test (Dusk/Playwright):** Menguji alur penting seperti form kontak dan CRUD admin di berbagai ukuran layar.
* CI otomatis via GitHub Actions: menjalankan `php artisan test`, `npm run build`, analisis statis (Larastan/Psalm), dan format code (Pint).

---

## 11. Checklist Deployment

1. Konfigurasi server Forge (PHP 8.3, Nginx, Redis, Supervisor worker) dan hubungkan repository.
2. Sediakan database, Redis, dan bucket storage; atur environment dan kredensial mail.
3. Jalankan hook Envoyer: `composer install`, `php artisan migrate --force`, `php artisan db:seed`, `npm ci && npm run build`, `php artisan storage:link`.
4. Aktifkan cache (`config:cache`, `route:cache`, `view:cache`, `event:cache`); restart queue worker dan Horizon.
5. Cek status (HTTP 200, dashboard Horizon, status antrean), rotasi log, aktifkan notifikasi (Sentry, Forge).
6. Tambahkan cron `* * * * * php artisan schedule:run` untuk tugas terjadwal seperti sitemap dan pembersihan data kontak.

---

## 12. Penyesuaian Timeline

| Fase PRD                           | Durasi   | Fokus Implementasi                                                                                      |
| ---------------------------------- | -------- | ------------------------------------------------------------------------------------------------------- |
| Riset & Sitemap (Minggu 1)         | 1 minggu | Validasi persona, konfirmasi sitemap, inventaris konten, persetujuan wireframe                          |
| Wireframe & Desain UI (Minggu 2–3) | 2 minggu | Desain mockup responsif, library komponen, blueprint UX admin                                           |
| Pengembangan (Minggu 4–7)          | 4 minggu | Setup Laravel, buat migrasi database, implementasi halaman publik, CRUD admin, integrasi SEO & analitik |
| Pengujian & Peluncuran (Minggu 8)  | 1 minggu | Uji regresi & aksesibilitas, optimasi performa, isi konten final, rilis publik                          |

---

## 13. Pertanyaan Terbuka / Keputusan Diperlukan

1. Konfirmasi preferensi hosting (VPS Forge vs platform terkelola) dan anggaran untuk Redis/S3/Sentry.
2. Tentukan stack analitik (Google Analytics 4 vs Plausible) serta kebutuhan cookie consent.
3. Pastikan waktu peluncuran blog dan alur konten (draft/approve) untuk penyesuaian scope UI.
4. Pilih penyedia layanan email (Mailgun, Postmark, SES) serta domain pengirim notifikasi.
5. Sediakan salinan teks, gambar, dan aset brand final sesuai jadwal milestone pengembangan.

---