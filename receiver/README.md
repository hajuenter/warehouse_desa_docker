# Receiver

Docker setup untuk menjalankan **Receiver (Warehouse Desa)** secara lokal menggunakan Docker Compose.

## Prasyarat

Pastikan sudah menginstall:

- [Docker](https://www.docker.com/)

## Cara Install

### 1. Masuk ke Folder Receiver

Setelah repository berhasil di-clone, masuk ke folder `receiver`:

```bash
cd warehouse_desa_docker/receiver
```

Struktur folder:

```text
warehouse_desa_docker/
├── README.md
├── receiver/
│   ├── docker-compose.yml
│   ├── .env.example
│   └── README.md
├── opensid/
├── monitoring_web/
└── sql_setup/
```

### 2. Buat File `.env`

Di dalam folder `receiver`, terdapat file:

```text
.env.example
```

Salin file tersebut menjadi:

```text
.env
```

Sehingga di dalam folder `receiver` terdapat:

```text
receiver/
├── docker-compose.yml
├── .env.example
├── .env
└── README.md
```

### 3. Isi Konfigurasi `.env`

Buka file `.env` dan isi konfigurasi sesuai kebutuhan.

| Variable         | Keterangan                 |
| ---------------- | -------------------------- |
| `DB_PASSWORD`    | Password database MariaDB  |
| `JWT_SECRET`     | Secret key untuk JWT token |
| `ADMIN_PASSWORD` | Password administrator     |
| `ADMIN_EMAIL`    | Email administrator        |
| `SMTP_USER`      | Username SMTP              |
| `SMTP_PASS`      | Password SMTP              |

> **Catatan:** Untuk keamanan, gunakan password dan secret key yang berbeda dari nilai default ketika digunakan pada lingkungan production.

### 4. Jalankan Docker

Pastikan terminal berada di dalam folder:

```text
warehouse_desa_docker/receiver
```

Kemudian jalankan:

```bash
docker-compose up -d
```

Docker Compose akan membuat dan menjalankan seluruh container yang dibutuhkan oleh Receiver.

Tunggu beberapa saat hingga proses selesai.

Untuk memeriksa status container:

```bash
docker-compose ps
```

Pastikan container yang dibutuhkan memiliki status **Up**.

Database `desa_induk` akan otomatis dibuat dengan tabel berikut:

| Tabel           | Keterangan                  |
| --------------- | --------------------------- |
| `desa`          | Registry desa               |
| `inbound_queue` | Audit trail event masuk     |
| `dw_config`     | Warehouse: konfigurasi desa |
| `dw_user`       | Warehouse: data user        |
| `dw_kategori`   | Warehouse: kategori artikel |
| `dw_artikel`    | Warehouse: artikel          |
| `dw_komentar`   | Warehouse: komentar artikel |

### 5. Buka Receiver

Setelah seluruh container berhasil berjalan, Receiver dapat diakses melalui:

**http://localhost:3000**

### 6. Test Login Receiver

Untuk melakukan pengujian login pada API Receiver, gunakan endpoint:

```text
POST http://localhost:3000/api/auth/login
```

Endpoint tersebut dapat diuji menggunakan berbagai HTTP client seperti **cURL, Postman, Insomnia**, atau tools lainnya.

Gunakan akun administrator yang telah dikonfigurasi pada file `.env`.

Jika menggunakan konfigurasi default:

| Konfigurasi | Nilai       |
| ----------- | ----------- |
| Username    | `admin`     |
| Password    | `Admin123@` |

#### Request Body

Gunakan format JSON berikut:

```json
{
  "username": "admin",
  "password": "Admin123@"
}
```

Contoh menggunakan cURL:

```bash
curl -X POST http://localhost:3000/api/auth/login \
  -H "Content-Type: application/json" \
  -d "{\"username\":\"admin\",\"password\":\"admin123@\"}"
```

Jika login berhasil, API akan mengembalikan response yang berisi informasi token yang dapat digunakan untuk mengakses endpoint Receiver yang membutuhkan autentikasi.

> **Catatan:** Password di atas merupakan password default. Untuk keamanan, sebaiknya gunakan password yang berbeda pada lingkungan production.

### 7. Install OpenSID dan Sender

Sebelum melanjutkan ke proses pendaftaran desa, **install dan konfigurasi OpenSID serta Sender terlebih dahulu**.

Ikuti panduan instalasi dan konfigurasi OpenSID dan Sender pada README berikut:

👉 [Panduan Menjalankan OpenSID dan Sender](../opensid/README.md)

Setelah OpenSID dan Sender berhasil dijalankan, kembali ke langkah berikutnya untuk mendaftarkan desa pada Receiver.

### 8. Tambahkan Desa

Setelah **OpenSID selesai di-install, dikonfigurasi, dan Sender berhasil dijalankan**, daftarkan desa ke database Receiver agar desa dapat dikenali dan menerima data dari Sender.

Data desa yang diperlukan sebelumnya telah dicatat dari tabel:

```text
opensid.config
```

yaitu:

- `kode_desa`
- `app_key`
- `nama_desa`

Data tersebut kemudian digunakan untuk membuat **data desa baru** pada database Receiver:

```text
desa_induk
```

pada tabel:

```text
desa
```

#### Mapping Data OpenSID ke Receiver

Gunakan mapping berikut:

| OpenSID (`config`) | Receiver (`desa`) | Keterangan                                           |
| ------------------ | ----------------- | ---------------------------------------------------- |
| `kode_desa`        | `kode_desa`       | Kode unik desa                                       |
| `nama_desa`        | `nama_desa`       | Nama desa                                            |
| `app_key`          | `api_token`       | Token autentikasi untuk komunikasi Sender → Receiver |
| —                  | `status`          | Isi dengan `ACTIVE`                                  |

Kolom `id` tidak perlu diisi karena menggunakan `AUTO_INCREMENT`, sedangkan `created_at` akan diisi otomatis oleh database.

#### Menambahkan Data Desa

Buka phpMyAdmin Receiver:

**http://localhost:8082**

Kemudian:

1. Login ke phpMyAdmin menggunakan akun database Receiver.

| Konfigurasi | Nilai        |
| ----------- | ------------ |
| Server      | `receiver`   |
| Username    | `root`       |
| Password    | `rahasia123` |

> **Catatan:** Nilai password mengikuti konfigurasi pada file `.env`.

2. Pilih database:

```text
desa_induk
```

3. Buka tabel:

```text
desa
```

4. Pilih menu **Insert**.
5. Tambahkan data desa baru berdasarkan konfigurasi yang telah dicatat dari OpenSID.

Contoh:

| Kolom       | Nilai              |
| ----------- | ------------------ |
| `kode_desa` | `3518131001`       |
| `nama_desa` | `Desa Contoh`      |
| `api_token` | `xxxxxxxxxxxxxxxx` |
| `status`    | `ACTIVE`           |

Nilai `api_token` harus diisi menggunakan nilai **`app_key` dari tabel `config` OpenSID**.

Setelah data berhasil ditambahkan, desa tersebut akan terdaftar pada database Receiver dan dapat digunakan untuk proses komunikasi antara **Sender OpenSID** dan **Receiver**.

---

## Opsional: Install Ulang

Jika ingin melakukan instalasi ulang dari awal, jalankan:

```bash
docker-compose down -v
```

Kemudian jalankan kembali:

```bash
docker-compose up -d
```

> ⚠️ **Perhatian:**
> Perintah `docker-compose down -v` akan menghapus volume Docker, termasuk database yang tersimpan di dalamnya. Pastikan sudah melakukan backup jika terdapat data penting.

---

## Akses Layanan

| Layanan    | URL                   |
| ---------- | --------------------- |
| Receiver   | http://localhost:3000 |
| phpMyAdmin | http://localhost:8082 |

---

## Perintah Docker

| Perintah                 | Fungsi                                               |
| ------------------------ | ---------------------------------------------------- |
| `docker-compose up -d`   | Menjalankan container di background                  |
| `docker-compose down`    | Menghentikan dan menghapus container                 |
| `docker-compose down -v` | Menghentikan container dan menghapus volume/database |
| `docker-compose restart` | Restart container                                    |
| `docker-compose ps`      | Melihat status container                             |
| `docker-compose logs`    | Melihat log container                                |

---
