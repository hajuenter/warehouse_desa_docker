# OpenSID Docker

Docker setup untuk menjalankan **OpenSID dan Sender** secara lokal menggunakan Docker Compose.

## Prasyarat

Pastikan sudah menginstall:

- [Docker](https://www.docker.com/)

## Cara Install

### 1. Masuk ke Folder OpenSID

Setelah repository berhasil di-clone, masuk ke folder `opensid`:

```bash
cd warehouse_desa_docker/opensid
```

Folder `opensid` sudah tersedia di dalam repository sehingga file yang diperlukan tidak perlu didownload secara manual.

Struktur folder:

```text
warehouse_desa_docker/
├── receiver/
├── opensid/
│   ├── docker-compose.yml
│   ├── .env.example
│   └── README.md
├── monitoring_web/
├── sender_only/
└── sql_setup/
```

### 2. Buat File `.env`

Di dalam folder `opensid`, terdapat file:

```text
.env.example
```

Salin file tersebut menjadi:

```text
.env
```

Isi file `.env` sesuai dengan konfigurasi yang diperlukan.

Contoh konfigurasi:

```env
# Database
DB_HOST=db
DB_PORT=3306
DB_DATABASE=opensid
DB_USERNAME=opensid
DB_PASSWORD=opensid_secret

# MySQL root password
MYSQL_ROOT_PASSWORD=root_secret

# Sender Config
DESA_KODE=
WEBHOOK_TOKEN=
WEBHOOK_URL=http://receiver:3000/api/webhook
WEBHOOK_TIMEOUT=10
MAX_RETRY=5
POLL_INTERVAL=3
```

> **Catatan:** `DESA_KODE` dan `WEBHOOK_TOKEN` akan dikonfigurasi setelah proses instalasi OpenSID dan setup database selesai.

### 3. Jalankan Docker

Pastikan terminal berada di folder:

```text
warehouse_desa_docker/opensid
```

Kemudian jalankan:

```bash
docker-compose up -d
```

Docker Compose akan membuat dan menjalankan container:

- OpenSID
- MariaDB
- phpMyAdmin
- Sender

Periksa status container:

```bash
docker-compose ps
```

Pastikan container yang dibutuhkan memiliki status **Up**.

### 4. Periksa Database OpenSID

Sebelum melakukan instalasi OpenSID, buka phpMyAdmin:

**http://localhost:8081**

Login menggunakan akun database yang telah dikonfigurasi pada file `.env`.

| Konfigurasi | Nilai            |
| ----------- | ---------------- |
| Server      | `db`             |
| Username    | `opensid`        |
| Password    | `opensid_secret` |

Pastikan database dengan nama:

```text
opensid
```

sudah berhasil dibuat.

> **Catatan:** Pada tahap ini database `opensid` dapat masih dalam keadaan kosong. Tabel-tabel OpenSID akan dibuat secara otomatis selama proses instalasi dan migrasi OpenSID.

### 5. Install OpenSID

Buka halaman instalasi OpenSID:

**http://localhost:8080/install**

Ikuti proses instalasi OpenSID hingga selesai.

Jika diminta mengisi konfigurasi database, gunakan:

| Konfigurasi   | Nilai            |
| ------------- | ---------------- |
| Database Host | `db`             |
| Database Name | `opensid`        |
| Username      | `opensid`        |
| Password      | `opensid_secret` |

Kemudian lanjutkan proses instalasi.

### 6. Login ke Web Desa OpenSID

Setelah proses instalasi selesai, login ke halaman administrator OpenSID menggunakan akun admin yang telah dibuat pada proses instalasi.

Setelah berhasil login, lakukan konfigurasi **Informasi Desa**.

Pastikan informasi desa sudah diatur sesuai dengan desa yang akan digunakan, terutama:

- Nama desa
- Kode desa
- Informasi identitas desa lainnya

Pastikan konfigurasi desa sudah sesuai sebelum melanjutkan ke tahap berikutnya.

### 7. Pastikan Database OpenSID Sudah Terisi

Setelah proses instalasi dan konfigurasi OpenSID selesai, buka kembali:

**http://localhost:8081**

Kemudian pilih database:

```text
opensid
```

Pastikan tabel-tabel OpenSID sudah berhasil dibuat.

Salah satu tabel yang akan digunakan pada proses berikutnya adalah:

```text
config
```

### 8. Jalankan SQL Setup

Setelah tabel-tabel OpenSID berhasil dibuat, jalankan query SQL yang terdapat pada folder:

```text
sql_setup/
```

Struktur repository:

```text
warehouse_desa_docker/
├── opensid/
└── sql_setup/
    ├── setup-tabel-queue.sql
    └── setup-trigger-desa.sql
```

Jalankan SQL tersebut melalui phpMyAdmin.

#### Melalui phpMyAdmin

1. Buka **http://localhost:8081**
2. Pilih database `opensid`
3. Buka menu **SQL**
4. Buka file SQL dari folder `sql_setup`
5. Salin isi file SQL
6. Paste ke editor SQL phpMyAdmin
7. Jalankan query
8. Ulangi untuk file SQL berikutnya

Pastikan seluruh query berhasil dijalankan tanpa error.

### 9. Catat Konfigurasi Desa dari Tabel `config`

Setelah proses instalasi OpenSID dan SQL setup selesai, buka database:

```text
opensid
```

Kemudian buka tabel:

```text
config
```

Pada tabel tersebut terdapat konfigurasi desa yang akan digunakan pada tahap berikutnya.

Cari dan **catat tiga informasi berikut**:

| Kolom       | Keterangan                | Digunakan Untuk                                                      |
| ----------- | ------------------------- | -------------------------------------------------------------------- |
| `kode_desa` | Kode unik desa            | Konfigurasi `DESA_KODE` pada Sender dan pendaftaran desa di Receiver |
| `app_key`   | App key untuk autentikasi | Konfigurasi `WEBHOOK_TOKEN` pada Sender                              |
| `nama_desa` | Nama desa                 | Pendaftaran desa pada database Receiver                              |

Contoh:

| Kolom       | Nilai              |
| ----------- | ------------------ |
| `kode_desa` | `3518131001`       |
| `app_key`   | `xxxxxxxxxxxxxxxx` |
| `nama_desa` | `Desa Contoh`      |

> ⚠️ **Penting:** Pastikan ketiga nilai tersebut dicatat dengan benar karena akan digunakan pada tahap konfigurasi Sender dan pendaftaran desa pada Receiver.

### 10. Konfigurasi Sender

Buka file `.env` pada folder `opensid`.

Isi konfigurasi Sender berdasarkan data yang diperoleh dari tabel `config`.

```env
DESA_KODE=
WEBHOOK_TOKEN=
WEBHOOK_URL=http://receiver:3000/api/webhook
```

Masukkan nilai:

- `DESA_KODE` → isi dengan nilai `kode_desa` dari tabel `config`.
- `WEBHOOK_TOKEN` → isi dengan nilai `app_key` dari tabel `config`.
- `WEBHOOK_URL` → gunakan URL webhook Receiver yang telah tersedia.

Contoh:

```env
DESA_KODE=3518131001
WEBHOOK_TOKEN=xxxxxxxxxxxxxxxx
WEBHOOK_URL=http://receiver:3000/api/webhook
```

> **Catatan:** `nama_desa` tidak dimasukkan ke dalam `.env`. Nilai `nama_desa` harus tetap dicatat karena akan digunakan pada proses pendaftaran desa di database Receiver.

### 11. Recreate Container Sender

Setelah file `.env` diperbarui, recreate container Sender agar perubahan environment variable diterapkan:

```bash
docker-compose up -d --force-recreate sender
```

Kemudian periksa status container:

```bash
docker-compose ps
```

Pastikan container:

```text
opensid_sender
```

memiliki status **Up**.

Untuk melihat log Sender:

```bash
docker-compose logs -f sender
```

### 12. Lanjutkan ke Receiver

Setelah OpenSID dan Sender berhasil dijalankan, lanjutkan kembali ke panduan **Receiver** pada **Langkah 8 — Tambahkan Desa**.

👉 [Lanjutkan ke Langkah 8: Tambahkan Desa](../receiver/README.md#8-tambahkan-desa)

Pada tahap berikutnya, gunakan tiga data yang telah dicatat dari tabel `config` OpenSID:

- `kode_desa`
- `app_key`
- `nama_desa`

Data tersebut akan digunakan untuk mendaftarkan desa ke database Receiver, khususnya pada database:

```text
desa_induk
```

tabel:

```text
desa
```

Pastikan data desa yang dimasukkan ke Receiver sesuai dengan konfigurasi pada database OpenSID.

## Opsional: Install Ulang

Jika ingin melakukan instalasi ulang OpenSID dari awal, jalankan:

```bash
docker-compose down -v
```

Kemudian jalankan kembali:

```bash
docker-compose up -d
```

Setelah container berjalan, buka kembali:

**http://localhost:8080/install**

> ⚠️ **Perhatian:**
> Perintah `docker-compose down -v` akan menghapus volume Docker, termasuk database yang tersimpan di dalamnya. Pastikan sudah melakukan backup jika terdapat data penting.

---

## Akses Layanan

| Layanan    | URL                   |
| ---------- | --------------------- |
| OpenSID    | http://localhost:8080 |
| phpMyAdmin | http://localhost:8081 |

---

## Perintah Docker

| Perintah                                       | Fungsi                                                    |
| ---------------------------------------------- | --------------------------------------------------------- |
| `docker-compose up -d`                         | Menjalankan seluruh container di background               |
| `docker-compose down`                          | Menghentikan dan menghapus container                      |
| `docker-compose down -v`                       | Menghentikan container dan menghapus volume/database      |
| `docker-compose restart`                       | Restart container                                         |
| `docker-compose ps`                            | Melihat status container                                  |
| `docker-compose logs`                          | Melihat log seluruh container                             |
| `docker-compose logs -f sender`                | Melihat log Sender secara realtime                        |
| `docker-compose up -d --force-recreate sender` | Recreate container Sender dan menerapkan perubahan `.env` |
