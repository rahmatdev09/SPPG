# MBG System

Aplikasi web progresif (PWA) untuk mengelola operasional SPPG Program Makan Bergizi Gratis. Aplikasi menggunakan HTML, CSS, JavaScript module, Firebase Authentication, dan Cloud Firestore.

## Menjalankan aplikasi

Proyek ini tidak menggunakan proses build atau package manager. Jalankan berkas melalui server lokal, jangan membuka HTML dengan skema `file://`.

1. Buka folder proyek di Visual Studio Code.
2. Jalankan `index.html` dengan ekstensi **Live Server**, atau gunakan server HTTP lokal lain.
3. Pastikan domain lokal yang digunakan, misalnya `localhost`, sudah terdaftar di **Firebase Authentication > Settings > Authorized domains**.
4. Login menggunakan akun Google yang sudah terdaftar di koleksi `app_users`.

> Aplikasi memuat Firebase SDK dan Tailwind CSS dari CDN, sehingga koneksi internet diperlukan.

## Halaman utama

| Halaman | Kegunaan |
| --- | --- |
| `index.html` | Dashboard ringkasan |
| `barang.html` | Kelola barang, penerimaan, persetujuan, dan RAB |
| `stok.html` | Kelola stok, riwayat transaksi, serta stok opname |
| `supplier.html` | Data supplier dan rekening |
| `pm.html` | Data penerima manfaat, porsi, pagu, lokasi, dan rute |
| `menu.html` | Data menu dan informasi AKG |
| `dokumen.html` | Dokumen yang tersimpan di Google Drive |
| `surat.html` | Buat, simpan, buka kembali, dan hapus surat permintaan pembayaran |
| `user.html` | Pengaturan akun dan role pengguna |
| `setting.html` | Identitas dapur/yayasan, logo, lokasi, dan nama penandatangan |
| `tentang.html` | Informasi aplikasi dan layanan |
| `admin-penerimaan.html` | Antarmuka PWA admin logistik untuk dashboard, penerimaan barang, stok, dan profil |

## Firebase

Konfigurasi Firebase web berada di bagian awal `app.js`. Project yang saat ini digunakan adalah `sppg-abc46`. Pastikan Authentication dan Cloud Firestore aktif pada project tersebut.

### Menyiapkan akun Super Admin pertama

Sebelum login pertama, buat dokumen melalui Firebase Console > Firestore Database dengan ketentuan:

- Koleksi: `app_users`
- ID dokumen: email Google dalam huruf kecil, misalnya `admin@example.com`
- Field `email` (string): email Google tersebut
- Field `role` (string): `super_admin`
- Field `active` (boolean): `true`
- Field `createdAt` (string): tanggal/waktu pembuatan

Setelah Super Admin pertama bisa login, akun berikutnya dapat dikelola dari menu **Pengguna**. Email pengguna dinormalisasi menjadi huruf kecil.

### Firestore Rules

Publikasikan isi berkas [`firestore.rules`](./firestore.rules) ke Firebase Console > Firestore Database > **Rules** setiap kali aturan berubah. Koleksi aplikasi yang diatur di dalamnya meliputi:

- `app_users`: akun dan role; pengelolaannya khusus Super Admin.
- `mbg_items`, `inventory_items`, `suppliers`, `menus`, `barang_arrival_photos`, dan `payment_letters`: hanya dapat diakses akun aktif yang memiliki role `super_admin` atau `admin_logistik`.
- `app_metadata`: penanda migrasi awal katalog stok.
- `pms`: dapat dibaca akun aktif dan hanya dapat diubah Super Admin.
- Dokumen lain ditolak secara default.

> Jika tabel surat menampilkan akses ditolak, pastikan aturan untuk `payment_letters` sudah ikut dipublikasikan ke Firebase, lalu muat ulang halaman.

Panduan pembuatan akun awal juga tersedia di [`AKSES_PENGGUNA.md`](./AKSES_PENGGUNA.md).

### Google Drive

Halaman Dokumen meminta izin Google Drive melalui login Google. Aktifkan **Google Drive API** di Google Cloud project Firebase yang digunakan dan pastikan konfigurasi OAuth mengizinkan origin aplikasi. Login web Firebase tetap terpisah dari sesi izin Drive.

### Data pengaturan dan surat

- Pengaturan dapur, yayasan, logo, lokasi, dan nama penandatangan disimpan di `localStorage` browser/perangkat yang dipakai.
- Surat permintaan pembayaran disimpan di koleksi Firestore `payment_letters`. Surat tersimpan mencakup rincian barang, supplier, penandatangan, salinan kop, dan lampiran yang dikompres. Total ukuran lampiran dibatasi agar dokumen tidak melewati batas Firestore.
- Kelola Barang dan Stok Barang memakai koleksi terpisah: `mbg_items` untuk rencana/penerimaan harian dan `inventory_items` untuk saldo serta riwayat persediaan. Saat pembaruan pertama, saldo lama yang memiliki stok atau riwayat disalin satu kali ke katalog persediaan dan barang dengan nama serta satuan yang sama digabung. Penerimaan PWA berikutnya mencatat transaksi ke persediaan.

## PWA dan cache

Service worker berada di `sw.js`; daftar halaman dan aset offline diatur pada `APP_SHELL`. Versi cache perlu dinaikkan saat mengubah aset agar browser dan PWA mengambil berkas terbaru. Setelah pembaruan, muat ulang aplikasi jika tampilan lama masih tersimpan.
