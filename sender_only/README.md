# Sender Only

Docker setup untuk menjalankan **Sender** pada desa yang **sudah memiliki OpenSID** dan dihosting secara terpisah.

## Kapan Menggunakan Folder Ini?

Gunakan `sender_only` jika:

- Desa **sudah memiliki** instance OpenSID yang berjalan di server
- OpenSID tidak dikelola oleh repository `warehouse_desa_docker`
- Hanya perlu mengirim data dari OpenSID desa ke Receiver (Warehouse Desa)

## Prasyarat

Pastikan sudah menginstall:

- [Docker](https://www.docker.com/)

Akses:

- Akses ke **database MariaDB/MySQL** OpenSID desa
- IP/hostname **Receiver** (Warehouse Desa)

## Cara Install

### 1. Masuk ke Folder Sender Only

```bash
cd warehouse_desa_docker/sender_only
```

Struktur folder:

```text
sender_only/
├── docker-compose.yml
├── .env.example
└── README.md
```

### 2. Buat File `.env`

```bash
cp .env.example .env
```

### 3. Isi Konfigurasi `.env`

Buka file `.env` dan isi sesuai data desa:

| Variable          | Keterangan                        | Contoh                             |
| ----------------- | --------------------------------- | ---------------------------------- |
| `DB_HOST`         | IP/hostname database OpenSID desa | `192.168.1.100`                    |
| `DB_PORT`         | Port database (default: 3306)     | `3306`                             |
| `DB_USER`         | Username database                 | `opensid`                          |
| `DB_PASSWORD`     | Password database                 | `rahasia123`                       |
| `DB_NAME`         | Nama database OpenSID             | `opensid`                          |
| `DESA_KODE`       | Kode desa dari tabel `config`     | `3518131001`                       |
| `WEBHOOK_TOKEN`   | `app_key` dari tabel `config`     | `abc123xyz`                        |
| `WEBHOOK_URL`     | URL endpoint Receiver             | `http://10.0.0.1:3000/api/webhook` |
| `WEBHOOK_TIMEOUT` | Timeout koneksi (detik)           | `10`                               |
| `MAX_RETRY`       | Jumlah retry maksimal             | `5`                                |
| `POLL_INTERVAL`   | Interval polling (detik)          | `3`                                |

> **Catatan:** `DESA_KODE` dan `WEBHOOK_TOKEN` diambil dari tabel `config` database OpenSID desa.

### 4. Jalankan Docker

```bash
docker-compose up -d
```

### 5. Periksa Status

```bash
docker-compose ps
```

Pastikan container `sender_desa` memiliki status **Up**.

### 6. Lihat Log

```bash
docker-compose logs -f sender
```

Pastikan tidak ada error koneksi ke database.

## Setup Database OpenSID Desa

Sebelum menjalankan sender, **database OpenSID desa** harus sudah dipersiapkan:

### 1. Buat Tabel `webhook_queue`

Jalankan SQL berikut di database OpenSID desa:

```sql
CREATE TABLE webhook_queue (
    id INT AUTO_INCREMENT PRIMARY KEY,
    event_id VARCHAR(100),
    config_id INT,
    event_type VARCHAR(20),
    table_name VARCHAR(100),
    record_id INT,
    payload JSON,
    status ENUM('pending', 'success', 'failed', 'retry'),
    retry_count INT,
    error_message TEXT,
    received_at TIMESTAMP NULL DEFAULT NULL,
    processed_at TIMESTAMP NULL DEFAULT NULL
);
```

### 2. Buat Trigger

Jalankan isi file `sql_setup/setup-trigger-desa.sql` di database OpenSID desa. File ini berisi trigger untuk tabel `config`, `user`, `kategori`, `artikel`, dan `komentar`.

Cara jalankan:

```bash
mysql -u username -p nama_database < sql_setup/setup-trigger-desa.sql
```

Atau via phpMyAdmin → pilih database → menu SQL → paste isi file → klik Go.

### 3. Catat Data dari Tabel `config`

Buka tabel `config` di database OpenSID desa, catat:

| Kolom       | Keterangan                                   |
| ----------- | -------------------------------------------- |
| `kode_desa` | Masukkan ke `DESA_KODE` di `.env`            |
| `app_key`   | Masukkan ke `WEBHOOK_TOKEN` di `.env`        |
| `nama_desa` | Diperlukan untuk registrasi desa di Receiver |

## Daftarkan Desa di Receiver

Setelah sender berjalan, daftarkan desa ke database Receiver:

1. Buka phpMyAdmin Receiver
2. Pilih database `desa_induk`
3. Buka tabel `desa`
4. Klik **Insert**
5. Isi data:

| Kolom       | Nilai                                 |
| ----------- | ------------------------------------- |
| `kode_desa` | dari tabel `config` OpenSID           |
| `nama_desa` | dari tabel `config` OpenSID           |
| `api_token` | `app_key` dari tabel `config` OpenSID |
| `status`    | `ACTIVE`                              |

## Perintah Docker

| Perintah                        | Fungsi                        |
| ------------------------------- | ----------------------------- |
| `docker-compose up -d`          | Jalankan sender di background |
| `docker-compose down`           | Hentikan sender               |
| `docker-compose restart`        | Restart sender                |
| `docker-compose ps`             | Lihat status container        |
| `docker-compose logs -f sender` | Lihat log sender realtime     |
