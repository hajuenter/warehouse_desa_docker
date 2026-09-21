# Spesifikasi

## Sistem yang Dikembangkan

**Warehouse Desa** — Sistem integrasi data terpusat yang mengagregasi data dari beberapa aplikasi desa (OpenSID) ke dalam basis data gudang (data warehouse) dengan dashboard monitoring real-time.

---

## Komponen Sistem

| Komponen           | Deskripsi                                         |
| ------------------ | ------------------------------------------------- |
| **Receiver**       | API server penerima data webhook dari sender      |
| **Sender OpenSID** | Agen polling perubahan data dari database OpenSID |
| **OpenSID**        | Aplikasi administrasi desa sumber data            |
| **Monitoring Web** | Dashboard monitoring berbasis SPA                 |

---

## Backend

| Teknologi         | Versi                           | Keterangan                            |
| ----------------- | ------------------------------- | ------------------------------------- |
| PHP               | 8.3.8 (ZTS Visual C++ 2019 x64) | Bahasa pemrograman utama backend      |
| Composer          | 2.8.5                           | Manajer dependensi PHP                |
| PSR-4 Autoloading | -                               | Standard autoloading namespace `App\` |

### PHP Extensions

| Extension   | Keterangan                            |
| ----------- | ------------------------------------- |
| `curl`      | HTTP request                          |
| `mbstring`  | Multibyte string handling             |
| `openssl`   | Enkripsi & SSL/TLS                    |
| `pdo_mysql` | Driver koneksi database MySQL/MariaDB |
| `zip`       | Penanganan file ZIP                   |

### Dependensi PHP — Receiver

| Package               | Versi                  | Kegunaan                                             |
| --------------------- | ---------------------- | ---------------------------------------------------- |
| `vlucas/phpdotenv`    | ^5.6                   | Memuat variabel lingkungan dari file `.env`          |
| `firebase/php-jwt`    | ^6.10 (locked: 6.11.1) | Autentikasi JWT (HS256, expired 8 jam)               |
| `phpmailer/phpmailer` | ^6.9                   | Pengiriman email via SMTP (Gmail, port 587/STARTTLS) |
| `phpunit/phpunit`     | ^10.5                  | Unit testing                                         |

### Dependensi PHP — Sender OpenSID

| Package             | Versi | Kegunaan                                     |
| ------------------- | ----- | -------------------------------------------- |
| `vlucas/phpdotenv`  | ^5.6  | Memuat variabel lingkungan dari file `.env`  |
| `guzzlehttp/guzzle` | ^7.9  | HTTP client untuk pengiriman payload webhook |
| `phpunit/phpunit`   | 10.5  | Unit testing                                 |

---

## Frontend — Monitoring Web

| Teknologi        | Versi   | Kegunaan                             |
| ---------------- | ------- | ------------------------------------ |
| React            | ^19.2.7 | UI library                           |
| React DOM        | ^19.2.7 | DOM rendering                        |
| React Router DOM | ^7.18.1 | Client-side routing                  |
| Tailwind CSS     | ^4.3.2  | CSS utility-first framework          |
| Headless UI      | ^2.2.10 | Komponen UI tanpa style (accessible) |
| Lucide React     | ^1.24.0 | Library ikon                         |
| Axios            | ^1.18.1 | HTTP client untuk panggilan API      |
| React Hot Toast  | ^2.6.0  | Notifikasi toast                     |

### Build Tools

| Teknologi            | Versi   | Kegunaan                         |
| -------------------- | ------- | -------------------------------- |
| Vite                 | ^8.1.1  | Build tool & dev server          |
| @vitejs/plugin-react | ^6.0.3  | Plugin React untuk Vite          |
| ESLint               | ^10.6.0 | Linting kode JavaScript          |
| Node.js              | 22.16.0 | Runtime JavaScript (build stage) |
| npm                  | 10.9.2  | Package manager JavaScript       |

---

## Basis Data

| Teknologi         | Versi  | Kegunaan                          |
| ----------------- | ------ | --------------------------------- |
| MariaDB           | 10.11  | Database utama (MySQL-compatible) |
| PDO (`pdo_mysql`) | -      | Driver koneksi database PHP       |
| phpMyAdmin        | latest | Database management web interface |

### Database dan Tabel

| Database        | Lokasi              | Tabel                                                                                       |
| --------------- | ------------------- | ------------------------------------------------------------------------------------------- |
| `desa_induk`    | Receiver (embedded) | `desa`, `inbound_queue`, `dw_config`, `dw_user`, `dw_kategori`, `dw_artikel`, `dw_komentar` |
| `opensid`       | Shared ke receiver  | Tabel bawaan OpenSID (`config`, `user`, `kategori`, `artikel`, `komentar`)                  |
| `webhook_queue` | Receiver            | Antrian perubahan data dari trigger MySQL                                                   |

---

## OpenSID

| Komponen             | Detail                                                                   |
| -------------------- | ------------------------------------------------------------------------ |
| Versi                | **2607.0.1**                                                             |
| Docker Image         | `hajuenter/opensid:latest`                                               |
| Port                 | 8080 (mapped dari port 80 container)                                     |
| Database             | MariaDB 10.11 (shared dengan receiver)                                   |
| Kode Desa            |                                                                          |
| Tabel yang Dimonitor | `config`, `user`, `kategori`, `artikel`, `komentar`                      |
| Mekanisme            | MySQL trigger → `webhook_queue` → Sender polling → HTTP POST ke Receiver |

---

## Web Server

| Komponen                     | Web Server                      | Port | Keterangan                 |
| ---------------------------- | ------------------------------- | ---- | -------------------------- |
| Receiver                     | PHP Built-in Development Server | 3000 | Managed oleh supervisord   |
| OpenSID                      | Apache                          | 8080 | Bawaan dari image Docker   |
| Monitoring Web (Production)  | Nginx Alpine                    | 8090 | SPA fallback (`try_files`) |
| Monitoring Web (Development) | Vite Dev Server                 | 5173 | `npm run dev`              |
| phpMyAdmin                   | Apache                          | 8082 | Database web interface     |

---

## Containerisasi

| Teknologi      | Versi  | Kegunaan                                                           |
| -------------- | ------ | ------------------------------------------------------------------ |
| Docker         | 29.4.1 | Containerisasi semua komponen                                      |
| Docker Compose | 5.1.3  | Orkestrasi multi-container                                         |
| Supervisord    | -      | Process manager di dalam container Receiver (MariaDB + PHP server) |

### Jaringan Docker

| Nama Jaringan | Tipe     | Keterangan                      |
| ------------- | -------- | ------------------------------- |
| `wh_shared`   | External | Jaringan bersama antar komponen |

### Port Mapping

| Service        | Container Port | Host Port |
| -------------- | -------------- | --------- |
| Receiver API   | 3000           | 3000      |
| MariaDB        | 3306           | 3307      |
| phpMyAdmin     | 80             | 8082      |
| OpenSID        | 80             | 8080      |
| Monitoring Web | 80             | 8090      |

---

## Keamanan

| Fitur             | Implementasi                                                       |
| ----------------- | ------------------------------------------------------------------ |
| Autentikasi Admin | JWT (HS256, expired 8 jam)                                         |
| Hashing Password  | BCrypt (cost factor 10)                                            |
| Rate Limiting     | Maksimal 5 percobaan login, lockout 3 menit                        |
| Reset Password    | Token via email, expired 15 menit, disimpan di `reset_tokens.json` |
| Webhook Auth      | Bearer token (shared secret)                                       |
| CORS              | Header `Access-Control-*` dikonfigurasi di receiver                |

---

## Arsitektur Sistem

```
OpenSID App (Apache:8080)
        |
        | [MySQL trigger → webhook_queue]
        v
Sender OpenSID (PHP CLI, polling setiap 3 detik)
        |
        | [HTTP POST dengan Bearer token]
        v
Receiver (PHP built-in server:3000 + MariaDB embedded)
        |
        | [Penyimpanan di database desa_induk]
        v
Monitoring Web (Nginx:8090 → React SPA)
        |
        | [REST API polling setiap 5 detik]
        v
Dashboard Monitoring (real-time)
```

---

## Lingkungan

| Komponen            | Nilai                       |
| ------------------- | --------------------------- |
| Timezone            | `Asia/Jakarta` (WIB, UTC+7) |
| Encoding            | UTF-8 (`charset=utf8mb4`)   |
| Zona Waktu Database | Server timezone             |

---

## Port Service

| Service              | URL Akses                   |
| -------------------- | --------------------------- |
| Receiver API         | `http://localhost:3000/api` |
| Monitoring Web       | `http://localhost:8090`     |
| Monitoring Web (Dev) | `http://localhost:5173`     |
| OpenSID              | `http://localhost:8080`     |
| phpMyAdmin           | `http://localhost:8082`     |
| MariaDB              | `localhost:3307`            |

---

## Tools Pendukung

| Tool | Versi  | Keterangan             |
| ---- | ------ | ---------------------- |
| Git  | 2.45.1 | Version control system |
