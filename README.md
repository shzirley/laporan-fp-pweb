# Laporan Proyek: Implementasi Teknis Sistem Florist

**Sistem Informasi Florist "Puspa"** dikembangkan menggunakan pendekatan *Native PHP Hybrid*, di mana logika antarmuka (*Frontend*) dan logika server (*Backend*) terintegrasi dalam satu arsitektur aplikasi (*Monolithic*). Berikut adalah rincian teknis implementasinya.

## 1\. Frontend & Backend Development

Pengembangan sistem dibagi menjadi dua sisi utama: sisi klien (*client-side*) yang berfokus pada estetika dan responsivitas, serta sisi server (*server-side*) yang menangani logika bisnis.

### A. Frontend Development (Antarmuka Pengguna)

Fokus utama adalah memastikan website dapat diakses dengan baik melalui perangkat desktop maupun seluler.

#### 1\. Teknologi & Komponen yang Digunakan

Berdasarkan struktur file proyek, teknologi utama meliputi:

  * **HTML5 (Hypertext Markup Language)**
      * **Fungsi:** Membangun kerangka dasar semantik halaman web.
      * **Implementasi:** Digunakan di semua file `.php` (seperti `index.php`, `dashboard.php`) untuk menyusun struktur DOM (*Document Object Model*).
  * **CSS3 & Bootstrap 5.3.0**
      * **Fungsi:** Menangani tata letak (*layout*) responsif dan *styling* visual secara cepat.
      * **File Pendukung:** `style.css` (kustomisasi landing page) dan `dashboard.css` (layout spesifik halaman admin).
      * **Komponen Bootstrap:** Menggunakan *Grid System* (untuk menyusun katalog produk), *Navbar* (navigasi responsif), *Card* (kontainer produk), dan *Modal* (notifikasi/popup).
  * **Bootstrap Icons**
      * **Fungsi:** Menyediakan elemen visual vektor yang ringan tanpa membebani *load time*.
      * **Implementasi:** Digunakan pada ikon keranjang (`bi-cart`), profil user (`bi-person`), dan rating bintang (`bi-star-fill`).

#### 2\. Implementasi Halaman Utama

Halaman antarmuka dibagi menjadi dua konteks utama berdasarkan status login pengguna:

  * **Public Landing Page (`index.php`)**
      * Merupakan halaman statis yang berfungsi sebagai etalase toko bagi pengunjung umum.
      * **Fitur:** Menampilkan *Hero Section* dengan latar gambar menarik, kategori produk, dan testimoni pelanggan.
      * **Logika Tampilan:** Navbar menggunakan logika kondisional PHP. Jika user belum login, tombol "Login with Google" ditampilkan. Jika sudah login, sistem menampilkan nama user dan foto profil.
  * **User Dashboard (`dashboard.php`)**
      * Halaman dinamis yang hanya dapat diakses setelah otentikasi berhasil.
      * **Fitur Utama:**
          * **Sidebar Tetap:** Navigasi vertikal untuk akses cepat ke Profil, Riwayat Pesanan, dan Review.
          * **Katalog Dinamis:** Menampilkan produk yang diambil secara *real-time* dari database.
          * **Pencarian:** Bilah pencarian di bagian atas untuk memfilter produk.

### B. Backend Development (Logika Server)

Sisi backend bertanggung jawab penuh atas pemrosesan data, manajemen sesi (*session*), keamanan, dan komunikasi dengan pihak ketiga (Google API).

#### 1\. File & Logika Backend

Arsitektur backend dibangun secara modular untuk memudahkan pemeliharaan kode:

  * **`config.php`:** Berfungsi sebagai pusat konfigurasi sistem. File ini menginisialisasi koneksi ke database MySQL menggunakan `mysqli` (dengan konfigurasi Host: `127.0.0.1`, Port: `3307` untuk stabilitas koneksi XAMPP) dan mengatur kredensial *Google Client API*.
  * **Dependency Management (`composer.json` & `vendor/`):** Menggunakan **Composer** untuk mengelola library eksternal. Library utama yang diinstal adalah `google/apiclient` yang digunakan untuk menangani protokol otentikasi OAuth 2.0 yang kompleks.

#### 2\. Implementasi Autentikasi (Google OAuth 2.0)

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

#### 3\. Implementasi Katalog Produk (Server-Side Rendering)

Fitur ini menggunakan konsep *Server-Side Rendering* (SSR) di mana data diproses di server sebelum halaman dikirim ke browser.

  * **Data Fetching:** Pada file `dashboard.php`, script mengeksekusi query SQL `SELECT * FROM products` untuk mengambil seluruh data.
  * **Dynamic Rendering:** Hasil query diulang menggunakan struktur kontrol `while`. Setiap baris data database (Nama, Harga, Stok) dirender menjadi kartu HTML.
  * **Handling Gambar:** Backend memanggil URL gambar eksternal (Unsplash) yang disimpan dalam kolom `image` (tipe data `TEXT`) di database. Metode ini memastikan gambar tampil berkualitas tinggi tanpa membebani penyimpanan server lokal.

#### 4\. Manajemen Keranjang & Transaksi

Logika bisnis untuk transaksi ditangani oleh file-file pendukung:

  * **`keranjang.php`:** Bertugas mengelola penambahan item ke dalam *Session Array* (`$_SESSION['cart']`). Penggunaan session memungkinkan keranjang belanja bertahan selama browser aktif tanpa perlu menulis data ke database secara terus-menerus (mengurangi beban server).
  * **`checkout.php`:** Menangani kalkulasi akhir. Script ini merekapitulasi item yang ada di session keranjang, menghitung subtotal, menambahkan biaya layanan, dan memproses data final sebelum disimpan ke tabel riwayat pesanan.

#### 5\. Keamanan & Proteksi Akses

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

-----

## 2\. Database Implementation

Implementasi basis data pada sistem **Puspa Florist** dirancang menggunakan **MySQL (MariaDB)**. Desain database menerapkan prinsip **Relational Database Management System (RDBMS)** yang ketat untuk memastikan integritas data, efisiensi penyimpanan, dan skalabilitas fitur e-commerce.

### A. Desain Skema Database (Schema Design)

Database proyek ini diberi nama **`puspa`**. Berdasarkan skema final, sistem terdiri dari **7 entitas tabel** yang saling berelasi, dibagi menjadi dua kategori:

#### 1\. Master Data (Data Utama)

Tabel-tabel ini menyimpan informasi inti yang jarang berubah namun sering diakses (*read-heavy*).

  * **Tabel `users` (Manajemen Pengguna)**
      * **`id` (PK):** *Auto Increment Integer*.
      * **`google_id`:** *Varchar(255)*. Menyimpan ID unik dari API Google sebagai validasi login utama.
      * **`picture`:** *Varchar(255) / TEXT*. Digunakan untuk menyimpan URL foto profil.
      * **Analisis Teknis:** Struktur ini didesain *password-less*, artinya tidak ada kolom password karena otentikasi ditangani sepenuhnya oleh pihak ketiga (Google).
  * **Tabel `products` (Inventaris Produk)**
      * **`price`:** *DECIMAL(10, 0)*. Penggunaan tipe data `DECIMAL` sangat krusial untuk aplikasi toko guna menghindari kesalahan pembulatan (*floating point error*) yang sering terjadi pada tipe `FLOAT`.
      * **`description`:** *TEXT*. Kolom ini menampung deskripsi naratif panjang mengenai filosofi bunga.
      * **`main_image`:** *Varchar*. Menyimpan nama file gambar utama (*thumbnail*) untuk tampilan kartu di dashboard.
      * **`stock` & `sold`:** *Integer*. Digunakan untuk manajemen stok dan pelacakan popularitas produk (*best seller*).

#### 2\. Transactional & Feature Data (Data Fitur)

Tabel-tabel ini menangani interaksi pengguna dengan produk. Keunggulan skema ini adalah penggunaan **Foreign Key** dengan kendala **`ON DELETE CASCADE`**.

  * **Tabel `carts` (Keranjang Belanja)**
      * **Fungsi:** Menyimpan status belanjaan pengguna sebelum proses *checkout*.
      * **Relasi:** *Many-to-Many* (User ↔ Product).
      * **Logika Bisnis:** Setiap baris merepresentasikan satu jenis item dalam keranjang user. Jika User atau Produk dihapus oleh admin, data keranjang terkait akan otomatis hilang (*Cascade*), mencegah error transaksi pada barang yang sudah tidak ada.
  * **Tabel `favorites` (Wishlist)**
      * **Fungsi:** Fitur personalisasi "Simpan untuk nanti".
      * **Relasi:** Menghubungkan `user_id` dan `product_id`.
      * **Analisis:** Memungkinkan analisis preferensi pengguna tanpa harus melihat riwayat transaksi.
  * **Tabel `reviews` (Ulasan & Rating)**
      * **Fungsi:** *Social Proof* untuk meningkatkan kepercayaan pembeli.
      * **Struktur Unik:** Tabel ini menyimpan `user_name` dan `user_avatar` secara manual (*snapshot*), bukan hanya `user_id`.
      * **Relasi:** Terikat pada `products`. Jika produk dihapus, semua ulasannya ikut terhapus.
  * **Tabel `product_images` (Galeri Multi-Gambar)**
      * **Fungsi:** Mendukung fitur galeri (*Carousel*), di mana satu jenis bunga bisa memiliki banyak sudut foto.
      * **Relasi:** *One-to-Many* (Satu Produk punya Banyak Gambar).
      * **Implementasi:** Memisahkan gambar tambahan dari tabel utama `products` adalah praktik *Normalisasi Database* untuk menjaga tabel utama tetap ramping dan cepat diakses.
  * **Tabel `notifications` (Sistem Notifikasi)**
      * **Fungsi:** Memberikan umpan balik asinkron kepada user.
      * **Kolom Kunci:** `is_read` bertipe **`TINYINT(1)`** (sebagai Boolean 0/1) untuk membedakan notifikasi baru dan lama.

### B. Analisis Relasi & Integritas Data (ERD)

Kekuatan utama database **Puspa Florist** terletak pada **Referential Integrity**. Berikut adalah peta relasi teknisnya:

1.  **User ➡️ Carts (1:N):** Satu user bisa memiliki banyak baris di keranjang (banyak jenis bunga).
2.  **Product ➡️ Carts (1:N):** Satu jenis bunga bisa ada di keranjang banyak user berbeda.
3.  **User ➡️ Favorites (1:N):** User bisa menyukai banyak bunga.
4.  **Product ➡️ Reviews (1:N):** Satu bunga memiliki sekumpulan ulasan.
5.  **Product ➡️ Product\_Images (1:N):** Satu bunga memiliki galeri foto detail.

#### Implementasi `ON DELETE CASCADE`

Dalam file SQL, semua *Foreign Key* menggunakan aturan `ON DELETE CASCADE`.

```sql
FOREIGN KEY (product_id) REFERENCES products(id) ON DELETE CASCADE
```

**Dampak Teknis:** Ini adalah fitur *"Self-Cleaning"*. Admin tidak perlu menghapus data secara manual satu per satu. Jika admin menghapus produk "Mawar Merah", maka secara otomatis semua data terkait (foto galeri, ulasan, item di keranjang user) akan ikut terhapus. Hal ini menjaga database tetap bersih dari sampah data (*Orphan Records*).

-----

## 3\. Integrasi Application Programming Interface (API)

Dalam pengembangan sistem informasi **Puspa Florist**, integrasi API (*Application Programming Interface*) diterapkan untuk menangani dua fungsi krusial: autentikasi pengguna dan pemrosesan pembayaran. Sistem ini bertindak sebagai *client* yang mengonsumsi layanan dari penyedia pihak ketiga, yaitu Google dan Midtrans, guna meningkatkan efisiensi pengembangan dan keamanan data.

### A. Google OAuth 2.0 untuk Autentikasi

Sistem menggunakan layanan **Google Identity Services** dengan protokol OAuth 2.0 untuk fitur login. Implementasi ini memanfaatkan pustaka `google/apiclient` yang dikonfigurasi pada file `config.php` menggunakan *Client ID* dan *Client Secret* yang terdaftar di Google Cloud Console.

**Mekanisme Alur (*Authorization Code Flow*):**

1.  **Request:** Saat pengguna memilih masuk dengan Google, sistem mengarahkan pengguna ke halaman persetujuan Google.
2.  **Callback:** Setelah disetujui, Google mengembalikan *authorization code* ke endpoint **`google_callback.php`**.
3.  **Token Exchange:** Sistem kemudian menukarkan kode tersebut dengan *Access Token* untuk mendapatkan informasi profil pengguna (email, nama, dan foto).
4.  **Session:** Data profil disimpan ke dalam basis data lokal (`users`) sebagai sesi login aktif.

### B. Midtrans Snap untuk Gerbang Pembayaran

Untuk memfasilitasi transaksi non-tunai, sistem terintegrasi dengan **Midtrans Payment Gateway** menggunakan tipe integrasi *Snap*. Integrasi ini memungkinkan jendela pembayaran muncul secara *pop-up* (overlay) di halaman checkout tanpa mengalihkan pengguna keluar dari website.

**Alur Teknis:**

1.  **Backend (`checkout.php`):** Sistem mengalkulasi total belanja dan biaya tambahan, lalu mengirimkan data tersebut dalam bentuk *payload JSON* ke server Midtrans.
2.  **Frontend:** Respon dari API Midtrans berupa **Snap Token** kemudian digunakan oleh fungsi JavaScript `window.snap.pay()` di sisi frontend untuk memunculkan antarmuka pembayaran yang aman dan mendukung berbagai metode pembayaran (GoPay, Virtual Account, Kartu Kredit).

-----

## 4\. Pembagian Jobdesk (Tim Pengembang)

Untuk memastikan kelancaran integrasi sistem antara antarmuka dan logika server, tim pengembang membagi tanggung jawab berdasarkan peran utama (*Role*) sebagai berikut:

| Nama Anggota & NRP | Role Utama | Rincian Tanggung Jawab (Jobdesk) |
| :--- | :--- | :--- |
| **Maleka Ghaniya**<br>(5025241189) | Frontend Designer & UI/UX | • Pengembangan *Landing Page* (`index.php`)<br>• *Styling* Global (`style.css`) & *Branding* (Logo/Aset Visual)<br>• Penyusunan Laporan Proyek & Dokumentasi Awal |
| **Angela Vania S.**<br>(5025241226) | Frontend Developer | • Pengembangan *Dashboard User* (`dashboard.php`)<br>• Implementasi UI Flow Transaksi (Keranjang, Checkout, Detail Produk)<br>• Pengembangan Halaman Fitur User (Review, Favorit, History) |
| **Jorell Ramos S.**<br>(5025241202) | Fullstack Engineer | • **Backend:** Desain Database, Konfigurasi Server (`config.php`), & Integrasi API (*Google/Midtrans*)<br>• **Frontend Support:** Membantu integrasi logika PHP ke dalam tampilan *Landing Page* (Maleka) & *Dashboard* (Angela)<br>• **System Integration:** *Debugging* isu tampilan (*responsive layout*) dan konektivitas data antar halaman |
