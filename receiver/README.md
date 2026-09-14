# Receiver Docker

Docker setup untuk menjalankan **Receiver** (Warehouse Desa) secara lokal menggunakan Docker Compose.

## Prasyarat

Pastikan sudah menginstall:

- [Docker](https://www.docker.com/)

## Cara Install

### 1. Download file yang diperlukan

Download 2 file berikut:

- `docker-compose.yml`
- `.env.example`

Letakkan kedua file tersebut dalam **satu folder yang sama**.

### 2. Buat file `.env`

Rename:

```text
.env.example
```

menjadi:

```text
.env
```

### 3. Isi konfigurasi `.env`

Buka file `.env` dan isi dengan konfigurasi yang sesuai:

| Variable         | Keterangan                 |
| ---------------- | -------------------------- |
| `DB_PASSWORD`    | Password database MariaDB  |
| `JWT_SECRET`     | Secret key untuk JWT token |
| `ADMIN_PASSWORD` | Password admin             |
| `ADMIN_EMAIL`    | Email admin                |
| `SMTP_USER`      | Username SMTP              |
| `SMTP_PASS`      | Password SMTP              |

### 4. Jalankan Docker

Buka terminal pada folder tersebut, kemudian jalankan:

```bash
docker-compose up -d
```

Tunggu beberapa saat hingga seluruh container berhasil berjalan.

### 5. Buka Receiver

Buka browser dan akses:

**http://localhost:3000**

### 6. Login

Gunakan akun administrator:

| Konfigurasi | Nilai       |
| ----------- | ----------- |
| Username    | `admin`     |
| Password    | `admin123@` |

---

### 7. Tambahkan desa setelah selesai install OPENSID

Ambil konfigurasi dari database opensid yaitu tabel config dan tambahkan ke database desa_induk ke tabel desa agar desa terdeteksi.

## Opsional Install Ulang

Jika ingin melakukan instalasi ulang dari awal, jalankan:

```bash
docker-compose down -v
```

Setelah itu jalankan kembali:

```bash
docker-compose up -d
```

> **Perhatian:** `docker-compose down -v` akan menghapus volume Docker, termasuk database yang tersimpan di dalamnya. Pastikan sudah melakukan backup jika terdapat data penting.

---

## Akses Layanan

| Layanan    | URL                   |
| ---------- | --------------------- |
| Receiver   | http://localhost:3000 |
| phpMyAdmin | http://localhost:8082 |

---

## Login phpMyAdmin

| Konfigurasi | Nilai        |
| ----------- | ------------ |
| Server      | `receiver`   |
| Username    | `root`       |
| Password    | `rahasia123` |

---

## Perintah Docker

| Perintah                 | Fungsi                                               |
| ------------------------ | ---------------------------------------------------- |
| `docker-compose up -d`   | Menjalankan container di background                  |
| `docker-compose down`    | Menghentikan dan menghapus container                 |
| `docker-compose down -v` | Menghentikan container dan menghapus volume/database |
| `docker-compose restart` | Restart container                                    |

---

## Environment Variables

| Variable         | Default      | Keterangan                 |
| ---------------- | ------------ | -------------------------- |
| `DB_PASSWORD`    | `rahasia123` | Password database MariaDB  |
| `JWT_SECRET`     | `-`          | Secret key untuk JWT token |
| `ADMIN_PASSWORD` | `admin123@`  | Password admin             |
| `ADMIN_EMAIL`    | `-`          | Email admin                |
| `SMTP_USER`      | `-`          | Username SMTP              |
| `SMTP_PASS`      | `-`          | Password SMTP              |

---

## Database

Database `desa_induk` akan otomatis dibuat dengan tabel:

| Tabel           | Keterangan                  |
| --------------- | --------------------------- |
| `desa`          | Registry desa               |
| `inbound_queue` | Audit trail event masuk     |
| `dw_config`     | Warehouse: konfigurasi desa |
| `dw_user`       | Warehouse: data user        |
| `dw_kategori`   | Warehouse: kategori artikel |
| `dw_artikel`    | Warehouse: artikel          |
| `dw_komentar`   | Warehouse: komentar artikel |
