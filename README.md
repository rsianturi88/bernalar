# Bernalar Digital Products — Django MVP

Website katalog dan pemesanan produk digital Bernalar, dibangun dengan Django 5.2, SQLite, template server-rendered, dan CSS responsif. Logo Bernalar yang diberikan disertakan sebagai aset situs.

## Menjalankan dengan sekali klik (Windows)

1. Instal Python 3.11+ dari https://www.python.org/downloads/ dan centang **Add Python to PATH**.
2. Ekstrak ZIP ke folder yang dapat ditulis.
3. Klik dua kali `run_bernalar.bat`.
4. Tunggu pemasangan dependensi dan inisialisasi selesai. Browser akan dibuka ke `http://127.0.0.1:8000/`.

Pertama kali berjalan, skrip membuat virtual environment `.venv`, memasang dependensi, migrasi database, membuat role, akun admin demo, dan mengisi katalog contoh. Koneksi internet dibutuhkan pada proses pertama untuk mengunduh paket Python. Setelah paket terpasang, proses startup berikutnya jauh lebih singkat.

## Login demo lokal

- URL admin: `http://127.0.0.1:8000/admin/`
- URL dashboard internal: `http://127.0.0.1:8000/dashboard/`
- Username: `admin`
- Password awal: `BernalarLocal2026!`

Akun hanya dibuat jika belum ada. Untuk mengganti kredensial admin sebelum bootstrap pertama, set environment variable `BERNALAR_ADMIN_USER`, `BERNALAR_ADMIN_PASSWORD`, dan `BERNALAR_ADMIN_EMAIL`. **Ganti password demo segera dan jangan gunakan kredensial demo di server publik.**

## macOS / Linux

Pastikan Python 3.11+ tersedia, kemudian jalankan `./run_bernalar.sh` dari terminal. Jika diperlukan, beri izin eksekusi: `chmod +x run_bernalar.sh`.

## Fitur yang tersedia

- Landing page Bernalar dengan logo, positioning, area kompetensi, kategori, produk unggulan, dan CTA konsultasi.
- Katalog publik dengan pencarian teks, filter kategori dan format, halaman detail, produk terkait, harga, durasi, metode, tingkat, dan instruktur.
- Ragam format produk: E-Book, Bootcamp, Audio Book, Webinar, Pelatihan/Kursus, Konsultasi, Template/Toolkit, Produk Fisik.
- CRUD produk melalui dashboard staf: tambah, lihat, ubah, hapus (penghapusan produk yang sudah terkait pesanan dilindungi oleh foreign key); draft/terbit/arsip; produk unggulan; upload cover.
- Pemesanan oleh pengguna terdaftar, catatan kebutuhan, riwayat pesanan pelanggan, dan pembaruan status oleh staf.
- Form kontak tersimpan di database.
- Django Admin untuk kategori, pengguna, produk, pesanan, dan pesan kontak.
- Role awal `Admin`, `Editor`, dan `Customer`; pengguna biasa tidak bisa mengakses dashboard staf. Pengaturan grup untuk staf dilakukan melalui Django Admin.
- Responsif desktop/mobile, validasi formulir, CSRF protection, password hashing bawaan Django, dan ORM.

## Alur role

| Role | Hak akses utama |
|---|---|
| Admin / superuser | Dashboard, CRUD katalog, pengelolaan pesanan, Django Admin, pengguna dan kategori |
| Editor (anggota grup `Editor`, dengan `is_staff=True`) | Dashboard, CRUD katalog dan pengelolaan pesanan; admin lanjutan sesuai izin Django yang diberikan |
| Customer | Registrasi/login, melihat katalog, memesan, melihat pesanan miliknya, mengirim pesan kontak |
| Pengunjung | Menjelajah katalog, melihat detail produk, membaca informasi, mengirim pesan kontak |

Agar pengguna Editor dapat masuk ke dashboard, buat atau pilih pengguna di `/admin/auth/user/`, aktifkan **Staff status**, lalu tambahkan ke grup `Editor`. Berikan izin model Django sesuai kebutuhan prinsip least privilege.

## Catatan penting tentang transaksi

Ini merupakan fondasi e-commerce/pemesanan, bukan sistem pembayaran otomatis. Status pembayaran dan pemenuhan layanan dikelola staf secara manual. Belum ada integrasi payment gateway, pengiriman email, unduhan file berbayar, kupon, invoice pajak, integrasi LMS, atau sistem stok lanjutan. Jangan mengklaim pembayaran berhasil hanya karena order dibuat.

## Menjalankan perintah manual

```bash
python -m venv .venv
# Windows: .venv\Scripts\activate
# macOS/Linux: source .venv/bin/activate
pip install -r requirements.txt
python manage.py makemigrations catalog
python manage.py migrate
python manage.py bootstrap
python manage.py seed_demo
python manage.py runserver
```

## Deploy produksi — wajib dikerjakan sebelum publikasi

Aplikasi dikonfigurasi untuk **pengembangan lokal** (`DEBUG=1`, SQLite, SECRET_KEY fallback). Sebelum deploy:
1. Atur `DJANGO_DEBUG=0`, `DJANGO_SECRET_KEY` rahasia yang kuat, dan `DJANGO_ALLOWED_HOSTS`.
2. Gunakan database produksi (misalnya PostgreSQL), HTTPS, cookie aman, dan penyimpanan media yang sesuai.
3. Jalankan `python manage.py check --deploy`, konfigurasi `collectstatic`, server WSGI/ASGI, backup, monitoring, logging, dan kebijakan retensi data.
4. Terapkan izin per-model untuk Editor, rate limiting, proteksi spam form kontak, serta prosedur pengelolaan data pribadi.
5. Integrasikan payment gateway hanya setelah proses bisnis, kebijakan refund, rekonsiliasi, dan pengamanan webhook dirancang.

## Struktur

- `bernalar/` — konfigurasi proyek, URL, WSGI/ASGI
- `catalog/` — model, form, view, admin, management command
- `templates/` — halaman publik dan dashboard
- `static/` — CSS, JavaScript, logo
- `media/` — cover yang diunggah administrator
- `run_bernalar.bat` / `run_bernalar.sh` — launcher

## Deploy ke GitHub + Neon + Vercel

Lihat panduan rinci `DEPLOY_GITHUB_NEON_VERCEL.md`. Proyek sudah disiapkan untuk membaca `DATABASE_URL` PostgreSQL (Neon), konfigurasi environment produksi, dan static files Vercel. Jangan lupa menyimpan file migrasi katalog di GitHub dan menyiapkan object storage eksternal untuk upload yang harus persisten.
