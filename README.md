# Rumah Laundry — Inventory & Sales Management System
**Integrated Laravel Project — Web Programming 2**

Sistem Manajemen Persediaan & Penjualan Terintegrasi (**Inventory & Sales Management System**) untuk operasional bisnis Rumah Laundry. Dibangun menggunakan framework Laravel 11, PHP 8.2+, MySQL / MariaDB, Eloquent ORM, Blade templating, middleware otorisasi server-side, dan database transactions dengan mekanisme row-locking untuk menjamin integritas stok dan konsistensi data bisnis.

---

## 1. Deskripsi Project
Aplikasi ini dirancang untuk mengatasi permasalahan pencatatan manual pada usaha laundry dan distribusi perlengkapan laundry, seperti ketidaksesuaian stok fisik dengan pembukuan, penjualan barang saat stok habis (*overselling*), riwayat transaksi yang tercecer, serta tidak adanya pemisahan wewenang antara Pemilik (**Admin**) dan Petugas Kasir/Gudang (**Staff**).

Sistem memastikan bahwa:
- Setiap perubahan stok hanya terjadi saat transaksi pembelian atau penjualan berhasil disimpan.
- Harga jual diambil langsung dari database (*server-side source of truth*), bukan dipercaya dari sisi peramban (*client*).
- Transaksi multi-item disimpan secara atomik dalam `DB::transaction`. Jika terjadi kegagalan pada satu item, seluruh transaksi di-rollback secara otomatis.
- Pembatalan transaksi (*cancellation*) memulihkan stok secara aman dan mencegah pembatalan ganda (*double cancel*).

---

## 2. Fitur Aplikasi

### A. Autentikasi & Kontrol Akses Berbasis Peran (RBAC)
- **Login & Logout** dengan password hashing (`bcrypt`), regenerasi session, dan proteksi CSRF.
- **Role Admin / Owner**:
  - Akses penuh ke dashboard metrik bisnis real-time.
  - CRUD master data: Kategori, Produk, dan Pemasok.
  - Manajemen Pengguna (tambah, edit, ubah role Admin/Staff, hapus dengan proteksi akun aktif & riwayat transaksi).
  - Melihat seluruh riwayat transaksi pembelian dan penjualan.
  - Hak eksklusif membatalkan transaksi pembelian dan penjualan dengan pemulihan stok otomatis.
  - Akses laporan ringkasan transaksi dan nilai inventaris.
- **Role Staff**:
  - Melihat katalog dan ketersediaan stok produk persediaan.
  - Membuat transaksi pembelian / stok masuk (*Stock-In*).
  - Membuat transaksi penjualan (*Stock-Out*).
  - Melihat riwayat transaksi yang diizinkan.
  - **Dibatasi secara ketat di sisi server**: Tidak dapat mengelola user, tidak dapat menghapus master data sensitif, dan tidak dapat membatalkan transaksi (akses langsung via URL menghasilkan status `403 Forbidden`).

### B. Modul Master Data
- **Kategori Produk**: Pengelompokan jenis laundry (Deterjen, Pewangi, Pelembut, dll.) dengan nama unik dan proteksi penghapusan jika masih menaungi produk.
- **Produk Persediaan**: Pengelolaan SKU (unik), nama produk, kategori, harga beli, harga jual, stok gudang, dan batas minimum stok. Validasi angka positif di sisi server. Produk yang pernah terlibat dalam transaksi dilindungi dari penghapusan (*integrity protection*).
- **Pemasok (Suppliers)**: Informasi mitra pasokan (nama, telepon, email, alamat). Pemasok yang memiliki riwayat pembelian tidak dapat dihapus sembarangan.

### C. Modul Transaksi & Konsistensi Stok
- **Pembelian Stok (Stock-In)**:
  - Multi-item per transaksi.
  - Subtotal dan total nilai dihitung oleh server.
  - Stok produk bertambah secara otomatis dalam `DB::transaction`.
  - Pembatalan pembelian oleh Admin memverifikasi ketersediaan stok terlebih dahulu untuk mencegah stok menjadi negatif.
- **Penjualan (Stock-Out)**:
  - Multi-item per transaksi.
  - Harga satuan diambil dari database, bukan dari input form browser.
  - Pencegahan *overselling* saat transaksi bersamaan menggunakan `Product::lockForUpdate()`.
  - Pengecekan kecukupan stok di server. Jika stok tidak mencukupi, transaksi ditolak dan seluruh perubahan di-rollback.
  - Stok produk berkurang secara otomatis hanya setelah transaksi sukses.
  - Pembatalan penjualan oleh Admin mengembalikan stok produk secara konsisten dan mencegah pembatalan berulang.

### D. Usability & Pelaporan
- **Pencarian & Filter**:
  - Produk: Berdasarkan nama atau SKU, filter kategori, dan pagination.
  - Pembelian: Berdasarkan nomor invoice (`q`), pemasok, rentang tanggal, dan status transaksi.
  - Penjualan: Berdasarkan nomor invoice (`q`), kasir/staf, rentang tanggal, dan status transaksi.
- **Peringatan Stok Menipis**: Menu khusus dan alert dashboard untuk produk dengan stok $\le$ minimum stok.
- **Dashboard Ringkas & Laporan**: Menampilkan metrik dinamis berbasis query agregasi database (total produk, total penjualan hari ini & bulan ini, total pembelian, dan nilai persediaan inventaris).

---

## 3. Teknologi yang Digunakan
- **Framework**: Laravel 11.57.0 (MVC Architecture)
- **Bahasa Pemrograman**: PHP 8.2+
- **Database**: MySQL 8.0+ / MariaDB 10.4+
- **ORM**: Eloquent ORM dengan relasi lengkap dan eager loading (`with()`)
- **Template Engine**: Laravel Blade dengan layout reusable dan komponen modular
- **Styling**: Vanilla CSS modern, responsif (desktop, tablet, mobile), berdesain elegan dengan tema warna teal Rumah Laundry
- **Testing**: PHPUnit 11 (Automated Feature Tests)

---

## 4. Persyaratan Sistem
1. PHP $\ge$ 8.2 (ekstensi aktif: `pdo_mysql`, `mbstring`, `openssl`, `tokenizer`, `xml`, `ctype`, `json`, `curl`, `sqlite3` untuk unit testing).
2. Composer $\ge$ 2.5
3. MySQL $\ge$ 8.0 atau MariaDB $\ge$ 10.4 (misalnya via XAMPP)
4. Web Browser modern (Chrome, Edge, Firefox, Safari)

> **Catatan Antarmuka & Dependencies:**
> Aplikasi menggunakan arsitektur Blade & CSS murni sehingga **tidak memerlukan build tools Node.js/NPM** untuk berjalan. Seluruh styling dan font telah terintegrasi langsung.

---

## 5. Panduan Instalasi Langkah Demi Langkah (Windows / XAMPP)

### Langkah 1: Buka Terminal pada Folder Project
Buka PowerShell, Command Prompt, atau Terminal di folder:
```bash
cd d:\xampp\htdocs\Rumah-Laundry
```

### Langkah 2: Instalasi Dependensi PHP
```bash
composer install
```

### Langkah 3: Konfigurasi Environment (`.env`)
Salin file template `.env.example` menjadi `.env`:
```bash
copy .env.example .env
```
Pastikan pengaturan koneksi database pada `.env` telah sesuai:
```ini
APP_NAME="Rumah Laundry"
APP_ENV=local
APP_DEBUG=true
APP_URL=http://127.0.0.1:8000

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=rumah_laundry
DB_USERNAME=root
DB_PASSWORD=

SESSION_DRIVER=database
CACHE_STORE=database
```

### Langkah 4: Generate Application Key
```bash
php artisan key:generate
```

### Langkah 5: Buat Database di MySQL
Buka phpMyAdmin (`http://localhost/phpmyadmin`) atau jalankan perintah MySQL:
```sql
CREATE DATABASE IF NOT EXISTS `rumah_laundry` CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

### Langkah 6: Jalankan Migrasi dan Seeder
Jalankan migrasi seluruh tabel dan isi data contoh awal:
```bash
php artisan migrate --seed
```

> **PERINGATAN:** Jangan menjalankan `php artisan migrate:fresh --seed` pada database produksi atau database yang menyimpan data transaksi penting karena perintah tersebut akan menghapus seluruh data tabel.

---

## 6. Akun Demo Sistem

Aplikasi telah dilengkapi seeder akun demo siap pakai:

| Peran (Role) | Alamat Email | Password | Akses & Wewenang |
| :--- | :--- | :--- | :--- |
| **Admin / Owner** | `admin@laundry.test` | `Laundry123!` | Akses Penuh: Dashboard, CRUD Master Data, Manajemen Pengguna, Pembatalan Transaksi, Laporan |
| **Staff Kasir/Gudang** | `staff@laundry.test` | `Laundry123!` | Akses Operasional: Input Pembelian, Input Penjualan, Cek Stok Produk |

---

## 7. Cara Menjalankan Aplikasi
Jalankan local development server Laravel:
```bash
php artisan serve
```
Buka peramban (*browser*) dan akses tautan:
```
http://127.0.0.1:8000
```
Sistem akan otomatis mengarahkan ke halaman login jika Anda belum terautentikasi.

---

## 8. Struktur Database & Relasi Eloquent (ERD)

### Entitas Utama:
1. **`users`**: Akun pengguna sistem dengan atribut `id`, `name`, `email` (unik), `password` (hashed), `role` (`admin` / `staff`).
2. **`categories`**: Klasifikasi produk dengan `id`, `name` (unik).
3. **`products`**: Item persediaan dengan `id`, `category_id` (FK), `sku` (unik), `name`, `purchase_price`, `selling_price`, `stock`, `minimum_stock`.
4. **`suppliers`**: Pemasok bahan laundry dengan `id`, `name`, `phone`, `email`, `address`.
5. **`purchases`**: Header transaksi pembelian stok dengan `id`, `supplier_id` (FK), `user_id` (FK), `purchase_number` (unik), `total`, `status`, `purchased_at`.
6. **`purchase_items`**: Detail item pembelian dengan `id`, `purchase_id` (FK), `product_id` (FK), `quantity`, `unit_cost`, `subtotal`.
7. **`sales`**: Header transaksi penjualan dengan `id`, `user_id` (FK), `sale_number` (unik), `total`, `status`, `sold_at`.
8. **`sale_items`**: Detail item penjualan dengan `id`, `sale_id` (FK), `product_id` (FK), `quantity`, `unit_price`, `subtotal`.

### Relasi Eloquent ORM:
- `Category hasMany Product` | `Product belongsTo Category`
- `Supplier hasMany Purchase` | `Purchase belongsTo Supplier`
- `User hasMany Purchase` | `Purchase belongsTo User`
- `Purchase hasMany PurchaseItem` | `PurchaseItem belongsTo Purchase`
- `Product hasMany PurchaseItem` | `PurchaseItem belongsTo Product`
- `User hasMany Sale` | `Sale belongsTo User`
- `Sale hasMany SaleItem` | `SaleItem belongsTo Sale`
- `Product hasMany SaleItem` | `SaleItem belongsTo Product`

> Dokumentasi diagram visual dan penjelasan rinci relasi dapat dilihat pada [docs/ERD.md](file:///d:/xampp/htdocs/Rumah-Laundry/docs/ERD.md) dan gambar diagram [docs/erd.svg](file:///d:/xampp/htdocs/Rumah-Laundry/docs/erd.svg).

---

## 9. Daftar Route Utama Aplikasi

| Metode | URI | Nama Route | Middleware / Proteksi | Deskripsi |
| :--- | :--- | :--- | :--- | :--- |
| `GET` | `/login` | `login` | `guest` | Form login |
| `POST` | `/login` | `login.post` | `guest` | Autentikasi user |
| `POST` | `/logout` | `logout` | `auth` | Keluar dari sistem |
| `GET` | `/dashboard` | `dashboard` | `auth` | Dashboard metrik bisnis |
| `GET` | `/products` | `products.index` | `auth` | Daftar produk, search & filter |
| `GET` | `/products/low-stock` | `products.low-stock` | `auth` | Daftar produk stok menipis |
| `GET` | `/products/{product}` | `products.show` | `auth` | Detail data produk |
| `GET` | `/products/create` | `products.create` | `auth`, `role:admin` | Form tambah produk (Admin) |
| `POST` | `/products` | `products.store` | `auth`, `role:admin` | Simpan produk baru |
| `GET` | `/products/{product}/edit` | `products.edit` | `auth`, `role:admin` | Form edit produk |
| `PUT` | `/products/{product}` | `products.update` | `auth`, `role:admin` | Perbarui data produk |
| `DELETE` | `/products/{product}` | `products.destroy` | `auth`, `role:admin` | Hapus produk (terproteksi transaksi) |
| `GET` | `/categories` | `categories.index` | `auth` | Daftar kategori |
| `GET` | `/categories/{category}` | `categories.show` | `auth` | Detail kategori & produk terkait |
| `GET` | `/categories/create` | `categories.create` | `auth`, `role:admin` | Form tambah kategori |
| `POST` | `/categories` | `categories.store` | `auth`, `role:admin` | Simpan kategori (nama unik) |
| `GET` | `/suppliers` | `suppliers.index` | `auth` | Daftar pemasok |
| `GET` | `/suppliers/{supplier}` | `suppliers.show` | `auth` | Detail pemasok & riwayat PO |
| `GET` | `/purchases` | `purchases.index` | `auth` | Riwayat pembelian stok |
| `GET` | `/purchases/create` | `purchases.create` | `auth` | Form input pembelian multi-item |
| `POST` | `/purchases` | `purchases.store` | `auth` | Simpan pembelian & tambah stok |
| `GET` | `/purchases/{purchase}` | `purchases.show` | `auth` | Detail nota pembelian |
| `POST` | `/purchases/{purchase}/cancel` | `purchases.cancel` | `auth`, `role:admin` | Batalkan pembelian (kurangi stok) |
| `GET` | `/sales` | `sales.index` | `auth` | Riwayat penjualan |
| `GET` | `/sales/create` | `sales.create` | `auth` | Form input penjualan multi-item |
| `POST` | `/sales` | `sales.store` | `auth` | Simpan penjualan & kurangi stok |
| `GET` | `/sales/{sale}` | `sales.show` | `auth` | Detail invoice penjualan |
| `POST` | `/sales/{sale}/cancel` | `sales.cancel` | `auth`, `role:admin` | Batalkan penjualan (kembalikan stok) |
| `GET` | `/reports` | `reports.index` | `auth` | Laporan transaksi & nilai stok |
| `RESOURCE`| `/users` | `users.*` | `auth`, `role:admin` | CRUD Pengguna & Role (Admin only) |

---

## 10. Skenario Pengujian & Hasil Pengujian Aktual

Seluruh pengujian fungsional dan aturan bisnis telah diuji menggunakan PHPUnit (`tests/Feature/InventoryAndSalesTest.php`):

```bash
php vendor/phpunit/phpunit/phpunit
```

### Hasil Eksekusi Uji Aktual:
```
PHPUnit 11.5.57 by Sebastian Bergmann and contributors.
Runtime: PHP 8.2.12
Configuration: D:\xampp\htdocs\Rumah-Laundry\phpunit.xml

...........................                                       27 / 27 (100%)

Time: 00:01.828, Memory: 44.00 MB
OK (27 tests, 104 assertions)
```

### Rincian 27 Skenario yang Terverifikasi:
1. `test_guest_redirected_to_login_from_dashboard`: Pengguna anonim ditolak masuk dashboard dan diarahkan ke login.
2. `test_login_fails_with_invalid_credentials`: Password keliru ditolak dengan pesan kesalahan.
3. `test_admin_and_staff_can_login`: Akun Admin dan Staff berhasil login dan diarahkan ke dashboard.
4. `test_duplicate_sku_is_rejected`: Validasi database dan form menolak penambahan produk dengan SKU yang sudah ada.
5. `test_negative_price_and_negative_minimum_stock_are_rejected`: Harga dan stok bernilai negatif ditolak di server.
6. `test_valid_product_can_be_created`: Produk dengan data valid berhasil tersimpan.
7. `test_purchase_success_increments_stock`: Pembelian stok bertambah otomatis sesuai kuantitas dan total dihitung di server.
8. `test_sale_success_decrements_stock_and_uses_database_price`: Penjualan mengurangi stok dan mengambil harga resmi dari database.
9. `test_sale_with_insufficient_stock_is_rejected`: Penjualan melebihi stok ditolak dan data tidak berubah sama sekali.
10. `test_multi_item_sale_atomicity_and_rollback`: Jika salah satu item gagal (stok kurang), seluruh transaksi di-rollback tanpa sisa data.
11. `test_staff_cannot_access_admin_user_management`: Staff diblokir (403 Forbidden) saat mencoba membuka halaman user atau membuat produk.
12. `test_staff_cannot_cancel_transactions`: Staff diblokir (403 Forbidden) saat mencoba membatalkan transaksi.
13. `test_search_and_filter_products`: Pencarian produk berdasarkan nama dan SKU menghasilkan data yang akurat.
14. `test_product_in_transaction_cannot_be_deleted`: Produk dengan riwayat transaksi dicegah dari penghapusan.
15. `test_sale_cancellation_restores_stock_and_prevents_double_cancel`: Pembatalan penjualan mengembalikan stok dan mencegah pembatalan ganda.
16. `test_category_with_products_cannot_be_deleted`: Kategori yang memiliki produk tidak dapat dihapus.
17. `test_supplier_with_purchases_cannot_be_deleted`: Pemasok yang memiliki riwayat transaksi tidak dapat dihapus.
18. `test_purchase_cancellation_adjusts_stock_and_prevents_negative`: Pembatalan pembelian menyesuaikan stok produk.
19. `test_transaction_search_and_filters`: Filter tanggal dan status pada transaksi berfungsi dengan benar.
20. `test_category_crud_and_show_detail`: Validasi nama kategori unik dan halaman detail kategori.
21. `test_purchase_cancellation_rejected_when_stock_insufficient`: Pembatalan pembelian ditolak jika stok yang tersisa tidak cukup untuk dikurangkan kembali (mencegah stok negatif).
22. `test_dashboard_metrics_exclude_cancelled_transactions`: Nilai penjualan pada dashboard mengabaikan transaksi yang berstatus batal (*cancelled*).
23. `test_user_cannot_delete_self_or_user_with_transactions`: Admin dicegah menghapus akunnya sendiri atau pengguna yang memiliki riwayat transaksi.
24. `test_duplicate_product_in_same_transaction_is_rejected`: Item produk ganda dalam satu invoice ditolak oleh validasi array `distinct`.
25. `test_reports_page_accessible_and_shows_metrics`: Laporan ringkasan penjualan dan pengadaan stok dapat dibuka dan menampilkan total akurat.
26. `test_low_stock_list_shows_only_low_stock_items`: Halaman stok menipis memfilter produk sesuai batas minimum stok.
27. `test_transaction_search_by_number_and_name`: Pencarian invoice pembelian dan penjualan berdasarkan nomor faktur (`q`) berfungsi akurat.

---

## 11. Keterbatasan yang Masih Ada (Known Limitations)
1. **Multi-Warehouse**: Aplikasi mengasumsikan satu gudang sentral persediaan Rumah Laundry (belum mendukung mutasi antar cabang).
2. **Payment Gateway**: Pencatatan pembayaran saat ini disiapkan untuk kas tunai (*Cash*) dan transfer manual (belum terhubung ke payment gateway online otomatis seperti Midtrans/Xendit).
3. **Akuntansi Lanjut**: Sistem berfokus pada manajemen persediaan dan pencatatan transaksi penjualan/pembelian, belum mencakup jurnal akuntansi debit/kredit penuh (*General Ledger*).

---

## 12. Panduan Wawancara Kepemilikan Kode (Code Interview)

| Pertanyaan Inti | Jawaban & Lokasi Kode |
| :--- | :--- |
| **Mengapa `Sale` memiliki relasi `hasMany` ke `SaleItem`?** | Satu transaksi penjualan dapat memuat lebih dari satu jenis barang yang dibeli pelanggan. Relasi 1-ke-banyak memisahkan data header invoice (nomor, total, tanggal, kasir) dengan rincian item barang yang dibeli ([Sale.php](file:///d:/xampp/htdocs/Rumah-Laundry/app/Models/Sale.php#L22-L24)). |
| **Di mana sistem memastikan stok tidak menjadi negatif?** | Di sisi backend pada [SaleController.php](file:///d:/xampp/htdocs/Rumah-Laundry/app/Http/Controllers/SaleController.php#L63-L70), sistem memeriksa `$product->stock < $qty`. Jika tidak mencukupi, sistem melempar `ValidationException` sehingga stok tidak berkurang. Selain itu, kolom `stock` didefinisikan sebagai `unsignedInteger` di database. |
| **Mengapa total penjualan dihitung di server?** | Mengandalkan harga atau subtotal dari browser sangat berbahaya karena pengguna dapat memanipulasi payload HTTP. Sistem mengambil harga resmi dari database (`$product->selling_price`) dan menghitung perkalian di server ([SaleController.php](file:///d:/xampp/htdocs/Rumah-Laundry/app/Http/Controllers/SaleController.php#L72-L86)). |
| **Apa risiko jika pengurangan stok dilakukan di luar `DB::transaction`?** | Terjadi inkonsistensi data parsial (*partial failure*). Misalnya header penjualan tersimpan tetapi sistem mengalami crash saat menyimpan item ke-2, atau stok berkurang tetapi header penjualan gagal tercatat. Dengan `DB::transaction`, seluruh proses berjalan secara atomik (*All or Nothing*). |
| **Bagaimana mencegah Staff mengakses fungsi Admin melalui URL langsung?** | Route admin dilindungi middleware `role:admin` ([routes/web.php](file:///d:/xampp/htdocs/Rumah-Laundry/routes/web.php#L81-L110)). Server memeriksa `$request->user()->role` di [RoleMiddleware.php](file:///d:/xampp/htdocs/Rumah-Laundry/app/Http/Middleware/RoleMiddleware.php). Jika bukan admin, server langsung menghentikan request dengan respons `403 Forbidden`. |
| **Apa yang terjadi jika dua transaksi memproses stok produk yang sama bersamaan?** | Sistem menggunakan [Product::lockForUpdate()](file:///d:/xampp/htdocs/Rumah-Laundry/app/Http/Controllers/SaleController.php#L62) di dalam transaksi database. Query ini mengunci baris produk di tingkat database (*pessimistic locking*), sehingga transaksi kedua harus mengantre sampai transaksi pertama selesai membaca dan memperbarui stok. Ini mencegah kondisi *race condition* dan *overselling*. |
