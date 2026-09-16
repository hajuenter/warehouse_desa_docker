# WAREHOUSE DESA — Docker

Panduan menjalankan **WAREHOUSE DESA** yang terdiri dari **Receiver, OpenSID, Sender, dan Monitoring Web Terpusat**.

Proses instalasi terdiri dari beberapa tahap. **Pastikan setiap tahap selesai sebelum melanjutkan ke tahap berikutnya.**

## 1. Clone Repository

Clone repository ini menggunakan Git:

```bash
git clone https://github.com/hajuenter/warehouse_desa_docker.git
```

Masuk ke folder repository:

```bash
cd warehouse_desa_docker
```

Setelah repository berhasil di-clone, struktur folder akan tersedia secara otomatis:

```text
warehouse_desa_docker/

├── README.md
├── receiver/
├── opensid/
├── monitoring_web/
├── sender_only/
└── sql_setup/
```

## 2. Buat Docker Network

Beberapa service pada project ini menggunakan Docker network bersama dengan nama:

```text
wh_shared
```

Network ini digunakan untuk memungkinkan komunikasi antar-service, khususnya:

- Receiver
- OpenSID
- Sender
- Monitoring Web

Buat Docker network tersebut:

```bash
docker network create wh_shared
```

Periksa apakah network berhasil dibuat:

```bash
docker network ls
```

Pastikan terdapat network:

```text
wh_shared
```

> **Catatan:** Network `wh_shared` hanya perlu dibuat satu kali. Jika network tersebut sudah ada, tidak perlu membuatnya kembali.

## 3. Menjalankan Receiver

Jalankan **Receiver (Warehouse Desa)** terlebih dahulu.

Ikuti panduan instalasi dan konfigurasi Receiver pada README berikut:

👉 [Panduan Menjalankan Receiver](./receiver/README.md)

**Penting:** Pada tahap ini proses Receiver **belum perlu diselesaikan seluruhnya**. Setelah Receiver berhasil dijalankan dan dapat diakses, lanjutkan ke tahap berikutnya untuk melakukan instalasi dan konfigurasi OpenSID serta Sender.

## 4. Menjalankan OpenSID dan Sender

Setelah Receiver berhasil dijalankan, lanjutkan dengan instalasi dan konfigurasi **OpenSID dan Sender**.

Ikuti panduan pada README berikut:

👉 [Panduan Menjalankan OpenSID dan Sender](./opensid/README.md)

Pada akhir proses instalasi OpenSID dan Sender, panduan OpenSID akan mengarahkan kembali ke:

**Receiver — Langkah 8: Tambahkan Desa**

👉 [Lanjutkan ke Langkah 8: Tambahkan Desa](./receiver/README.md#8-tambahkan-desa)

Pada tahap tersebut, gunakan data dari tabel `config` OpenSID:

- `kode_desa`
- `app_key`
- `nama_desa`

Data tersebut digunakan untuk mendaftarkan desa pada database Receiver.

## 5. Alternatif: OpenSID Sudah Terpasang dan Terhosting

Jika desa **sudah memiliki** instance OpenSID yang berjalan di server lain (tidak dikelola oleh repository ini), tidak perlu menginstal ulang OpenSID. Cukup pasang **Sender** saja pada server desa tersebut.

Panduan lengkap:

👉 [Panduan Sender Only](./sender_only/README.md)

## 6. Menjalankan Monitoring Web

Setelah:

- Receiver berhasil berjalan.
- OpenSID berhasil di-install dan dikonfigurasi.
- Sender berhasil berjalan.
- Desa berhasil didaftarkan pada Receiver.

Selanjutnya jalankan **Monitoring Web Terpusat**.

Ikuti panduan instalasi dan konfigurasi pada README berikut:

👉 [Panduan Menjalankan Monitoring Web](./monitoring_web/README.md)

---

## License

Copyright (c) 2026 **hajuenter**

This project is licensed under the **MIT License**.

You are free to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of this software, subject to the conditions of the MIT License.

See the [LICENSE](./LICENSE) file for the full license text.
