# Monitoring Web Docker

Dashboard monitoring untuk memantau aktivitas dan riwayat sinkronisasi data desa secara terpusat.

Monitoring Web dijalankan menggunakan Docker dan terhubung ke service lainnya melalui Docker network:

```text
wh_shared
```

## Prasyarat

Pastikan sudah menginstall:

- [Docker](https://www.docker.com/)

Pastikan juga Docker network `wh_shared` sudah dibuat.

Jika mengikuti panduan dari README utama, network tersebut sudah dibuat pada tahap awal instalasi.

Untuk memastikan network tersedia, jalankan:

```bash
docker network ls
```

Pastikan terdapat:

```text
wh_shared
```

> **Catatan:** Jangan membuat network `wh_shared` kembali jika network tersebut sudah tersedia.

## Cara Install

### 1. Masuk ke Folder Monitoring Web

Setelah repository berhasil di-clone, masuk ke folder `monitoring_web`:

```bash
cd warehouse_desa_docker/monitoring_web
```

Folder `monitoring_web` sudah tersedia di dalam repository sehingga file yang diperlukan tidak perlu didownload secara manual.

Struktur folder:

```text
warehouse_desa_docker/

├── README.md
├── receiver/
├── opensid/
├── monitoring_web/
│   ├── docker-compose.yml
│   └── README.md
└── sql_setup/
```

### 2. Jalankan Docker

Pastikan terminal berada di folder:

```text
warehouse_desa_docker/monitoring_web
```

Kemudian jalankan:

```bash
docker-compose up -d
```

Docker Compose akan menjalankan container Monitoring Web.

Periksa status container:

```bash
docker-compose ps
```

Pastikan container:

```text
monitoring_web
```

memiliki status:

```text
Up
```

### 3. Buka Monitoring Web

Setelah container berhasil berjalan, buka browser dan akses:

**http://localhost:8090**

Monitoring Web akan menampilkan dashboard monitoring data dan aktivitas sinkronisasi desa yang diterima oleh Receiver.

---

## Opsional: Install Ulang

Jika ingin menjalankan ulang container Monitoring Web, jalankan:

```bash
docker-compose down
```

Kemudian jalankan kembali:

```bash
docker-compose up -d
```

> **Catatan:** Perintah `docker-compose down` hanya menghentikan dan menghapus container Monitoring Web. Tidak menghapus database Receiver maupun data desa.

---

## Akses Layanan

| Layanan        | URL                   |
| -------------- | --------------------- |
| Monitoring Web | http://localhost:8090 |

---

### Docker Network `wh_shared` tidak ditemukan

Jika muncul error bahwa network `wh_shared` tidak ditemukan, buat network tersebut dari folder mana saja:

```bash
docker network create wh_shared
```

Kemudian jalankan kembali:

```bash
docker-compose up -d
```

> **Catatan:** Dalam instalasi normal, network `wh_shared` sudah dibuat pada tahap awal melalui [README utama](../README.md).
