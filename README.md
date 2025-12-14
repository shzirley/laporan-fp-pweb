# Laporan Proyek: Frontend & Backend Development

**Sistem Informasi Florist "Puspa"** dikembangkan menggunakan pendekatan *Native PHP Hybrid*, di mana logika antarmuka (*Frontend*) dan logika server (*Backend*) terintegrasi dalam satu arsitektur aplikasi (*Monolithic*). Berikut adalah rincian teknis implementasinya.

## A. Frontend Development (Antarmuka Pengguna)

Pengembangan sisi klien (*client-side*) berfokus pada pengalaman pengguna yang responsif dan estetis, memastikan website dapat diakses dengan baik melalui perangkat desktop maupun seluler.

### 1\. Teknologi & Komponen yang Digunakan

Berdasarkan struktur file proyek, teknologi utama yang digunakan meliputi:

  * **HTML5 (Hypertext Markup Language):**
      * **Fungsi:** Membangun kerangka dasar semantik halaman web.
      * **Implementasi:** Digunakan di semua file `.php` (seperti `index.php`, `dashboard.php`) untuk menyusun struktur DOM (*Document Object Model*).
  * **CSS3 & Bootstrap 5.3.0:**
      * **Fungsi:** Menangani tata letak (*layout*) responsif dan styling visual secara cepat.
      * **File Pendukung:** `style.css` (kustomisasi landing page) dan `dashboard.css` (layout spesifik halaman admin).
      * **Komponen Bootstrap:** Menggunakan *Grid System* (untuk menyusun katalog produk), *Navbar* (navigasi responsif), *Card* (kontainer produk), dan *Modal* (notifikasi/popup).
  * **Bootstrap Icons:**
      * **Fungsi:** Menyediakan elemen visual vektor yang ringan tanpa membebani *load time*.
      * **Implementasi:** Digunakan pada ikon keranjang (`bi-cart`), profil user (`bi-person`), dan rating bintang (`bi-star-fill`).

### 2\. Implementasi Halaman Utama

Halaman antarmuka dibagi menjadi dua konteks utama berdasarkan status login pengguna:

  * **Public Landing Page (`index.php`):**

      * Merupakan halaman statis yang berfungsi sebagai etalase toko bagi pengunjung umum.
      * **Fitur:** Menampilkan *Hero Section* dengan latar gambar menarik, kategori produk, dan testimoni pelanggan.
      * **Logika Tampilan:** Navbar menggunakan logika kondisional PHP. Jika user belum login, tombol "Login with Google" ditampilkan. Jika sudah login, sistem menampilkan nama user dan foto profil.

  * **User Dashboard (`dashboard.php`):**

      * Halaman dinamis yang hanya dapat diakses setelah otentikasi berhasil.
      * **Fitur Utama:**
          * **Sidebar Tetap:** Navigasi vertikal untuk akses cepat ke Profil, Riwayat Pesanan, dan Review.
          * **Katalog Dinamis:** Menampilkan produk yang diambil secara *real-time* dari database.
          * **Pencarian:** Bilah pencarian di bagian atas untuk memfilter produk.

-----

## B. Backend Development (Logika Server)

Sisi backend bertanggung jawab penuh atas pemrosesan data, manajemen sesi (*session*), keamanan, dan komunikasi dengan pihak ketiga (Google API).

### 1\. File & Logika Backend

Arsitektur backend dibangun secara modular untuk memudahkan pemeliharaan kode:

  * **`config.php`:**
      * Berfungsi sebagai pusat konfigurasi sistem. File ini menginisialisasi koneksi ke database MySQL menggunakan `mysqli` (dengan konfigurasi Host: `127.0.0.1`, Port: `3307` untuk stabilitas koneksi XAMPP) dan mengatur kredensial *Google Client API*.
  * **Dependency Management (`composer.json` & `vendor/`):**
      * Menggunakan **Composer** untuk mengelola library eksternal. Library utama yang diinstal adalah `google/apiclient` yang digunakan untuk menangani protokol otentikasi OAuth 2.0 yang kompleks.

### 2\. Implementasi Autentikasi (Google OAuth 2.0)

Sistem menggunakan fitur "Login with Google" untuk mempermudah akses pengguna tanpa perlu proses registrasi manual yang panjang.

  * **Alur Teknis:**
    1.  User mengklik link login yang digenerate secara dinamis oleh fungsi `$client->createAuthUrl()`.
    2.  Google mengarahkan user kembali ke file **`google_callback.php`** dengan membawa parameter `code`.
    3.  Backend menukar `code` tersebut dengan *Access Token* yang valid.
    4.  Data profil (Email, Nama, Foto) diambil dari layanan Google.
    5.  **Logika Database:** Backend mengecek tabel `users`.
          * Jika email belum ada → Lakukan `INSERT` (User Baru).
          * Jika email sudah ada → Lakukan `UPDATE` (Sinkronisasi data terbaru).
    6.  Status login disimpan dalam variabel global `$_SESSION['user_login']`.

### 3\. Implementasi Katalog Produk (Server-Side Rendering)

Fitur ini menggunakan konsep *Server-Side Rendering* (SSR) di mana data diproses di server sebelum halaman dikirim ke browser.

  * **Data Fetching:** Pada file `dashboard.php`, script mengeksekusi query SQL `SELECT * FROM products` untuk mengambil seluruh data.
  * **Dynamic Rendering:** Hasil query diulang menggunakan struktur kontrol `while`. Setiap baris data database (Nama, Harga, Stok) dirender menjadi kartu HTML.
  * **Handling Gambar:** Backend memanggil URL gambar eksternal (Unsplash) yang disimpan dalam kolom `image` (tipe data `TEXT`) di database. Metode ini memastikan gambar tampil berkualitas tinggi tanpa membebani penyimpanan server lokal.

### 4\. Manajemen Keranjang & Transaksi

Logika bisnis untuk transaksi ditangani oleh file-file pendukung:

  * **`keranjang.php`:** Bertugas mengelola penambahan item ke dalam *Session Array* (`$_SESSION['cart']`). Penggunaan session memungkinkan keranjang belanja bertahan selama browser aktif tanpa perlu menulis data ke database secara terus-menerus (mengurangi beban server).
  * **`checkout.php`:** Menangani kalkulasi akhir. Script ini merekapitulasi item yang ada di session keranjang, menghitung subtotal, menambahkan biaya layanan, dan memproses data final sebelum disimpan ke tabel riwayat pesanan.

### 5\. Keamanan & Proteksi Akses

Backend menerapkan lapisan keamanan standar untuk melindungi data dan mencegah akses ilegal:

  * **Session Hijacking Prevention:** Setiap halaman privat (`dashboard.php`, `checkout.php`) diawali dengan pengecekan sesi yang ketat:
    ```php
    if (!isset($_SESSION['user_login'])) {
        header("Location: index.php"); // Tendang ke halaman depan
        exit();
    }
    ```
    Script ini menolak akses langsung (*Direct URL Access*) oleh pengguna yang tidak dikenal.
  * **SQL Injection Protection:** Input dari pengguna (seperti kata kunci pada fitur pencarian) disanitasi menggunakan fungsi `real_escape_string` sebelum dimasukkan ke dalam query database.
  * **Environment Isolation:** Penggunaan port database spesifik (3307) dan IP `127.0.0.1` meminimalkan risiko konflik port dan akses jaringan yang tidak diinginkan pada lingkungan lokal.

  Berikut adalah format Markdown (`.md`) yang rapi dan siap pakai untuk bagian **Database Implementation**. Laporan ini disusun berdasarkan file `database.sql` final yang kamu kirimkan, mencakup analisis struktur tabel, relasi, dan keputusan teknis mendalam.

-----

# 2\. Database Implementation

Implementasi basis data pada sistem **Puspa Florist** dirancang menggunakan **MySQL (MariaDB)**. Desain database menerapkan prinsip **Relational Database Management System (RDBMS)** yang ketat untuk memastikan integritas data, efisiensi penyimpanan, dan skalabilitas fitur e-commerce.

## A. Desain Skema Database (Schema Design)

Database proyek ini diberi nama **`puspa`**. Berdasarkan skema final, sistem terdiri dari **7 entitas tabel** yang saling berelasi, dibagi menjadi dua kategori: *Master Data* dan *Transactional Data*.

### 1\. Master Data (Data Utama)

Tabel-tabel ini menyimpan informasi inti yang jarang berubah namun sering diakses (*read-heavy*).

#### a. Tabel `users` (Manajemen Pengguna)

Menyimpan data otentikasi pengguna yang masuk melalui Google OAuth.

  * **`id` (PK):** *Auto Increment Integer*.
  * **`google_id`:** *Varchar(255)*. Menyimpan ID unik dari API Google sebagai validasi login utama.
  * **`picture`:** *Varchar(255) / TEXT*. Digunakan untuk menyimpan URL foto profil.
  * **Analisis Teknis:** Struktur ini didesain *password-less*, artinya tidak ada kolom password karena otentikasi ditangani sepenuhnya oleh pihak ketiga (Google).

#### b. Tabel `products` (Inventaris Produk)

Pusat data katalog bunga yang menyimpan informasi detail produk.

  * **`price`:** *DECIMAL(10, 0)*. Penggunaan tipe data `DECIMAL` sangat krusial untuk aplikasi toko guna menghindari kesalahan pembulatan (*floating point error*) yang sering terjadi pada tipe `FLOAT`.
  * **`description`:** *TEXT*. Kolom ini menampung deskripsi naratif panjang mengenai filosofi bunga.
  * **`main_image`:** *Varchar*. Menyimpan nama file gambar utama (*thumbnail*) untuk tampilan kartu di dashboard.
  * **`stock` & `sold`:** *Integer*. Digunakan untuk manajemen stok dan pelacakan popularitas produk (*best seller*).

-----

### 2\. Transactional & Feature Data (Data Fitur)

Tabel-tabel ini menangani interaksi pengguna dengan produk. Keunggulan skema ini adalah penggunaan **Foreign Key** dengan kendala **`ON DELETE CASCADE`**.

#### c. Tabel `carts` (Keranjang Belanja)

  * **Fungsi:** Menyimpan status belanjaan pengguna sebelum proses *checkout*.
  * **Relasi:** *Many-to-Many* (User ↔ Product).
  * **Logika Bisnis:** Setiap baris merepresentasikan satu jenis item dalam keranjang user. Jika User atau Produk dihapus oleh admin, data keranjang terkait akan otomatis hilang (*Cascade*), mencegah error transaksi pada barang yang sudah tidak ada.

#### d. Tabel `favorites` (Wishlist)

  * **Fungsi:** Fitur personalisasi "Simpan untuk nanti".
  * **Relasi:** Menghubungkan `user_id` dan `product_id`.
  * **Analisis:** Memungkinkan analisis preferensi pengguna tanpa harus melihat riwayat transaksi.

#### e. Tabel `reviews` (Ulasan & Rating)

  * **Fungsi:** *Social Proof* untuk meningkatkan kepercayaan pembeli.
  * **Struktur Unik:** Tabel ini menyimpan `user_name` dan `user_avatar` secara manual (*snapshot*), bukan hanya `user_id`.
      * *Alasan Teknis:* Jika user mengganti nama profilnya di masa depan, nama pada ulasan lama tetap otentik sesuai saat ulasan tersebut ditulis.
  * **Relasi:** Terikat pada `products`. Jika produk dihapus, semua ulasannya ikut terhapus.

#### f. Tabel `product_images` (Galeri Multi-Gambar)

  * **Fungsi:** Mendukung fitur galeri (*Carousel*), di mana satu jenis bunga bisa memiliki banyak sudut foto.
  * **Relasi:** *One-to-Many* (Satu Produk punya Banyak Gambar).
  * **Implementasi:** Memisahkan gambar tambahan dari tabel utama `products` adalah praktik *Normalisasi Database* untuk menjaga tabel utama tetap ramping dan cepat diakses.

#### g. Tabel `notifications` (Sistem Notifikasi)

  * **Fungsi:** Memberikan umpan balik asinkron kepada user.
  * **Kolom Kunci:** `is_read` bertipe **`TINYINT(1)`** (sebagai Boolean 0/1) untuk membedakan notifikasi baru dan lama.

-----

## B. Analisis Relasi & Integritas Data (ERD)

Kekuatan utama database **Puspa Florist** terletak pada **Referential Integrity**. Berikut adalah peta relasi teknisnya:

1.  **User ➡️ Carts (1:N):** Satu user bisa memiliki banyak baris di keranjang (banyak jenis bunga).
2.  **Product ➡️ Carts (1:N):** Satu jenis bunga bisa ada di keranjang banyak user berbeda.
3.  **User ➡️ Favorites (1:N):** User bisa menyukai banyak bunga.
4.  **Product ➡️ Reviews (1:N):** Satu bunga memiliki sekumpulan ulasan.
5.  **Product ➡️ Product\_Images (1:N):** Satu bunga memiliki galeri foto detail.

### Implementasi `ON DELETE CASCADE`

Dalam file SQL, semua *Foreign Key* menggunakan aturan `ON DELETE CASCADE`.

```sql
FOREIGN KEY (product_id) REFERENCES products(id) ON DELETE CASCADE
```

**Dampak Teknis:** Ini adalah fitur *"Self-Cleaning"*. Admin tidak perlu menghapus data secara manual satu per satu. Jika admin menghapus produk "Mawar Merah", maka secara otomatis:

  * Foto-foto galeri mawar merah terhapus.
  * Ulasan mawar merah terhapus.
  * Mawar merah di keranjang belanja user akan hilang.
  * **Hasil:** Database selalu bersih dari sampah data (*Orphan Records*).

-----

## C. Konfigurasi Koneksi & Keamanan

Koneksi database diatur dalam file `config.php` dengan parameter teknis sebagai berikut:

1.  **Driver `mysqli` (MySQL Improved):** Menggunakan antarmuka *Object-Oriented* yang mendukung *Prepared Statements*. Ini adalah lapisan keamanan pertama untuk mencegah serangan **SQL Injection**.
2.  **Port Spesifik (3307):** Dikarenakan lingkungan pengembangan menggunakan XAMPP yang sering mengalami konflik port default (3306), konfigurasi diset secara eksplisit ke port **3307**.
3.  **Protokol TCP/IP (`127.0.0.1`):** Penggunaan IP Loopback `127.0.0.1` alih-alih `localhost` memaksa koneksi menggunakan protokol jaringan TCP/IP, yang terbukti lebih stabil pada sistem operasi Windows.

## D. Seeding Data (Data Awal)

Untuk keperluan pengembangan dan demonstrasi, database telah diisi (*seeded*) dengan data dummy yang komprehensif:

  * **6 Produk Utama:** Lengkap dengan harga, stok, deskripsi, dan nama file gambar.
  * **Galeri:** Setiap produk memiliki 3 foto tambahan di tabel `product_images`.
  * **Ulasan:** Setiap produk memiliki 2-3 ulasan dengan variasi rating bintang 4 dan 5 untuk simulasi tampilan frontend yang realistis.
