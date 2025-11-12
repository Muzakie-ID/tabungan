# NabungYuk - Aplikasi Menabung Bersama

NabungYuk adalah aplikasi web *fintech* sederhana yang dirancang untuk membantu pengguna mengelola tujuan tabungan mereka, baik secara pribadi maupun bersama pasangan. Aplikasi ini dibangun dengan PHP native dan MySQL, dengan fokus pada fungsionalitas inti, keamanan, dan pengalaman pengguna yang modern.

Proyek ini mencakup alur autentikasi lengkap, sistem undangan untuk tabungan bersama, dan manajemen transaksi (pemasukan & pengambilan) secara *real-time*.

[Tampilan Dashboard Aplikasi]
*(Sangat disarankan: Ambil screenshot dashboard Anda yang sudah jadi dan letakkan di sini)*

---

## ✨ Fitur Utama

-   **Autentikasi Pengguna:** Pendaftaran dan Login yang aman menggunakan *password hashing* (`password_hash`).
-   **Manajemen Tujuan (CRUD):**
    -   **Create:** Buat tujuan tabungan baru (Pribadi / Bersama).
    -   **Read:** Tampilkan semua tujuan dalam bentuk *card* modern.
    -   **Update:** Edit detail tujuan (Judul, Deskripsi, Target).
    -   **Delete:** Hapus tujuan (hanya bisa dilakukan oleh pembuat).
-   **Sistem Tabungan Bersama:**
    -   Undang pengguna lain melalui email.
    -   Penerima undangan dapat **Menerima** atau **Menolak** undangan.
    -   Dashboard menampilkan undangan yang tertunda.
-   **Sistem Transaksi Lengkap:**
    -   Catat **Pemasukan** (Deposit) ke tujuan.
    -   Catat **Pengambilan** (Withdrawal) dari tujuan.
    -   Validasi saldo (tidak bisa mengambil lebih dari yang ada).
-   **Rincian Kontribusi:** (Fitur Baru) Di halaman detail *goal* bersama, tampilkan rincian total pemasukan dari masing-masing anggota.
-   **Visualisasi Progres:** *Progress bar* otomatis ter-update untuk setiap tujuan.
-   **UX Modern:**
    -   Dibangun dengan **Bootstrap 5** dan CSS kustom untuk *shadow* & *border-radius* yang halus.
    -   **Format Rupiah Otomatis:** Input nominal otomatis diformat (`10000` -> `10.000`) menggunakan JavaScript.
-   **Siap Deploy:** Dilengkapi dengan `Dockerfile` dan `docker-compose.yml` untuk *deployment* yang mudah dan terisolasi.

---

## 💻 Teknologi yang Digunakan

-   **Backend:** PHP 8+ (Native, Prosedural) dengan ekstensi PDO
-   **Database:** MySQL
-   **Frontend:** HTML5, CSS3, Bootstrap 5, JavaScript (ES6+)
-   **Server:** Apache (via Docker)
-   **Deployment:** Docker & Docker Compose

---

## 🚀 Instalasi & Setup

Ada dua cara untuk menjalankan proyek ini: secara lokal (menggunakan XAMPP/Laragon) atau menggunakan Docker (direkomendasikan).

### 1. Instalasi Lokal (XAMPP / Laragon)

1.  **Clone Repository**
    ```bash
    git clone [https://github.com/USERNAME/NAMA_REPO.git](https://github.com/Muzakie_ID/tabungan.git)
    cd tabungan
    ```

2.  **Struktur Folder**
    Letakkan seluruh folder proyek di dalam `htdocs` (XAMPP) atau `www` (Laragon).

3.  **Database**
    -   Buka `phpMyAdmin` Anda.
    -   Buat database baru (misal: `tabungan_app`).
    -   Impor skema database. (Lihat bagian [Struktur Database](#sturktur-database) di bawah dan jalankan SQL-nya).

4.  **Konfigurasi**
    -   Buka file `src/config/database.php`.
    -   Pastikan variabel *fallback* (setelah `?:`) sesuai dengan pengaturan lokal Anda.
    ```php
    // Atur BASE_URL ke folder lokal Anda
    define('BASE_URL', getenv('BASE_URL') ?: 'http://localhost/tabungan-bootstrap');
    
    // Atur kredensial DB Anda
    $db_host = getenv('DB_HOST') ?: 'localhost';
    $db_name = getenv('DB_NAME') ?: 'tabungan_app'; // <-- Nama DB Anda
    $db_user = getenv('DB_USER') ?: 'root';         // <-- User DB Anda
    $db_pass = getenv('DB_PASS') ?: '';             // <-- Pass DB Anda
    ```
5.  **Jalankan**
    Akses `http://localhost/tabungan-bootstrap/views/auth/login.php` di browser Anda.

### 2. Instalasi dengan Docker (Rekomendasi)

Proyek ini dikonfigurasi untuk terhubung ke *container* database eksternal bernama `db_mysql`.

1.  **Prasyarat**
    -   Pastikan Docker & Docker Compose terinstal.
    -   Pastikan Anda memiliki *container* MySQL yang berjalan dengan nama `db_mysql`.

2.  **Buat Jaringan Bersama**
    Kita perlu membuat jaringan agar *container* `web` dan `db_mysql` bisa berkomunikasi.
    ```bash
    # Buat jaringan baru
    docker network create tabungan_network
    
    # Hubungkan container DB Anda yang sudah ada
    docker network connect tabungan_network db_mysql
    ```

3.  **Konfigurasi `docker-compose.yml`**
    Sesuaikan file `docker-compose.yml` dengan lingkungan Anda.
    ```yaml
    services:
      web:
        # ... (biarkan)
        ports:
          - "8086:80" # Akses via port 8086 di host
        # ...
        environment:
          - BASE_URL=http://IP_ANDA:8086 # <-- Ganti IP/Domain
          - DB_HOST=db_mysql             # <-- Biarkan, ini adalah nama service
          - DB_NAME=nama_database_anda   # <-- Ganti
          - DB_USER=user_database_anda   # <-- Ganti
          - DB_PASS=pass_database_anda   # <-- Ganti
    # ... (biarkan sisa file)
    ```

4.  **Bangun dan Jalankan**
    Dari *root* folder proyek (tempat `docker-compose.yml` berada):
    ```bash
    docker-compose up -d --build
    ```

5.  **Akses Aplikasi**
    Buka `http://IP_ANDA:8086/views/auth/login.php`

---

## 🗃️ Struktur Database



Berikut adalah skema SQL lengkap untuk membuat semua tabel yang diperlukan.

```sql
-- Tabel untuk menyimpan data pengguna
CREATE TABLE `users` (
  `id` int(11) NOT NULL AUTO_INCREMENT PRIMARY KEY,
  `name` varchar(100) NOT NULL,
  `email` varchar(100) NOT NULL UNIQUE,
  `password_hash` varchar(255) NOT NULL,
  `created_at` timestamp NOT NULL DEFAULT current_timestamp()
) ENGINE=InnoDB;

-- Tabel untuk menyimpan tujuan menabung
CREATE TABLE `saving_goals` (
  `id` int(11) NOT NULL AUTO_INCREMENT PRIMARY KEY,
  `creator_id` int(11) NOT NULL,
  `title` varchar(255) NOT NULL,
  `description` text DEFAULT NULL,
  `target_amount` decimal(15,2) DEFAULT NULL,
  `current_amount` decimal(15,2) NOT NULL DEFAULT 0.00,
  `type` enum('private','shared') NOT NULL DEFAULT 'private',
  `created_at` timestamp NOT NULL DEFAULT current_timestamp(),
  FOREIGN KEY (`creator_id`) REFERENCES `users` (`id`) ON DELETE CASCADE
) ENGINE=InnoDB;

-- Tabel untuk mencatat siapa saja yang ada di satu tujuan tabungan
CREATE TABLE `goal_members` (
  `id` int(11) NOT NULL AUTO_INCREMENT PRIMARY KEY,
  `goal_id` int(11) NOT NULL,
  `user_id` int(11) NOT NULL,
  `status` enum('pending','accepted','rejected') NOT NULL DEFAULT 'pending',
  `invited_at` timestamp NOT NULL DEFAULT current_timestamp(),
  FOREIGN KEY (`goal_id`) REFERENCES `saving_goals` (`id`) ON DELETE CASCADE,
  FOREIGN KEY (`user_id`) REFERENCES `users` (`id`) ON DELETE CASCADE,
  UNIQUE KEY `unique_member` (`goal_id`,`user_id`)
) ENGINE=InnoDB;

-- Tabel untuk mencatat semua riwayat transaksi
CREATE TABLE `transactions` (
  `id` int(11) NOT NULL AUTO_INCREMENT PRIMARY KEY,
  `goal_id` int(11) NOT NULL,
  `user_id` int(11) NOT NULL,
  `amount` decimal(15,2) NOT NULL,
  `note` varchar(255) DEFAULT NULL,
  `transaction_date` timestamp NOT NULL DEFAULT current_timestamp(),
  FOREIGN KEY (`goal_id`) REFERENCES `saving_goals` (`id`) ON DELETE CASCADE,
  FOREIGN KEY (`user_id`) REFERENCES `users` (`id`)
) ENGINE=InnoDB;

## 📝 Alur Pengguna (Contoh)

1.  **Pengguna A** dan **Pengguna B** mendaftarkan akun.
2.  **Pengguna A** login, lalu "Buat Tujuan Baru".
3.  Dia memilih tipe "Bersama" dan memasukkan email **Pengguna B**.
4.  **Pengguna B** login ke akunnya. Di dashboard, dia melihat "Undangan Tabungan Bersama" dari **Pengguna A**.
5.  **Pengguna B** menekan "Terima".
6.  Sekarang, *goal* tersebut muncul di dashboard kedua pengguna.
7.  **Pengguna A** menambah "Pemasukan" Rp 50.000.
8.  **Pengguna B** menambah "Pemasukan" Rp 30.000.
9.  Keduanya bisa melihat total tabungan (Rp 80.000) dan rincian kontribusi.
10. **Pengguna A** (sebagai pembuat) dapat mengedit atau menghapus *goal* tersebut.


## 📄 Lisensi

Proyek ini dilisensikan di bawah [MIT License](https://www.google.com/search?q=LICENSE).

```
```
