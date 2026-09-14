# OpenSID Docker

Docker setup untuk menjalankan **OpenSID** secara lokal menggunakan Docker Compose.

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

### 3. Jalankan Docker

Buka terminal pada folder tersebut, kemudian jalankan:

```bash
docker-compose up -d
```

Tunggu beberapa saat hingga seluruh container berhasil berjalan.

### 4. Buka halaman instalasi OpenSID

Buka browser dan akses:

**http://localhost:8080/install**

### 5. Isi konfigurasi database

Gunakan konfigurasi berikut:

| Konfigurasi   | Nilai            |
| ------------- | ---------------- |
| Database Host | `db`             |
| Database Name | `opensid`        |
| Username      | `opensid`        |
| Password      | `opensid_secret` |

### 6. Buat akun administrator

Ikuti proses instalasi OpenSID dan buat akun admin baru.

### 7. Import SQL

Setelah instalasi OpenSID selesai, import 2 file SQL berikut ke database:

```bash
docker exec -i opensid_db mariadb -u opensid -popensid_secret opensid < setup-tabel-queue.sql
docker exec -i opensid_db mariadb -u opensid -popensid_secret opensid < setup-trigger-desa.sql
```

Atau melalui phpMyAdmin:

- Buka http://localhost:8081
- Login dengan `opensid` / `opensid_secret`
- Buka tab SQL, jalankan isi dari `setup-tabel-queue.sql` lalu `setup-trigger-desa.sql`

---

## Mengubah Konfigurasi Sender

Mengubah `DESA_KODE` atau `WEBHOOK_TOKEN` di file `.env` sesuaikan dengan konfigurasi dengan tabel config yang sudah terbuat otomatis dari install OPENSID:

### 1. Edit file `.env`

```bash
DESA_KODE=
WEBHOOK_TOKEN=
```

### 2. Recreate container sender

```bash
docker-compose up -d sender
```

> **Penting:** Gunakan `docker-compose restart sender` supaya container di-recreate dengan nilai .env yang baru.

## Opsional Install Ulang

Jika ingin melakukan instalasi ulang dari awal, jalankan:

```bash
docker-compose down -v
```

Setelah itu jalankan kembali:

```bash
docker-compose up -d
```

Kemudian buka:

**http://localhost:8080/install**

> **Perhatian:** `docker-compose down -v` akan menghapus volume Docker, termasuk database yang tersimpan di dalamnya. Pastikan sudah melakukan backup jika terdapat data penting.

---

## Akses Layanan

| Layanan    | URL                   |
| ---------- | --------------------- |
| OpenSID    | http://localhost:8080 |
| phpMyAdmin | http://localhost:8081 |

---

## Perintah Docker

| Perintah                      | Fungsi                                               |
| ----------------------------- | ---------------------------------------------------- |
| `docker-compose up -d`        | Menjalankan container di background                  |
| `docker-compose down`         | Menghentikan dan menghapus container                 |
| `docker-compose down -v`      | Menghentikan container dan menghapus volume/database |
| `docker-compose restart`      | Restart container                                    |
| `docker-compose up -d sender` | Recreate container sender (apply perubahan .env)     |
