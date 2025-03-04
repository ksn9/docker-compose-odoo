# Dokumentasi Deployment Odoo di WSL dengan Docker Compose

Dokumentasi ini menjelaskan cara meng-deploy Odoo menggunakan Docker Compose di WSL (Ubuntu 24.04). Materi mencakup pembuatan struktur direktori, konfigurasi Docker Compose, langkah-langkah deployment, serta troubleshooting (termasuk penanganan masalah permission dan reset database).

---

## Struktur Direktori Proyek

Buat struktur direktori berikut di dalam folder proyek (misalnya `~/oddo`):

oddo/ 
├── web-data/ # Data Odoo (dipasang ke /var/lib/odoo)
├── config/ # File konfigurasi Odoo (opsional: misalnya odoo.conf) 
├── custom-addons/ # Modul/addons kustom Odoo 
└── db-data/ # Data PostgreSQL (dipasang ke /var/lib/postgresql/data)


**Langkah Pembuatan Struktur:**

Buka terminal di WSL (Ubuntu 24.04) dan jalankan perintah berikut:
mkdir -p ~/oddo/{web-data,config,custom-addons,db-data}
cd ~/oddo
Konfigurasi Docker Compose
Buat file bernama docker-compose.yml di dalam direktori ~/oddo dengan isi sesuai yang disediakan:
Keterangan:

Service odoo-latest:

Menggunakan image odoo:latest untuk mendapatkan versi terbaru Odoo.
Port 8069 pada host dipetakan ke port 8069 di container.
Volume web-data dipasang ke /var/lib/odoo untuk menyimpan data Odoo.
Volume config dipasang ke /etc/odoo untuk konfigurasi (jika diperlukan).
Volume custom-addons dipasang ke /mnt/extra-addons untuk modul kustom.
Environment variable mengatur parameter koneksi ke database.
Service db-postgres:

Menggunakan image postgres:latest untuk versi terbaru PostgreSQL.
Volume db-data dipasang ke /var/lib/postgresql/data untuk menyimpan data database.
Environment variable mengatur nama database, user, dan password.
Network Custom:

Kedua service terhubung ke network network_odoo dengan driver bridge.
Menjalankan Deployment
Mulai Docker Compose:

Dari direktori ~/oddo, jalankan perintah:
docker-compose up -d
Perintah ini akan:

Mengunduh image terbaru (jika belum tersedia).
Membuat dan menjalankan container odoo-latest dan db-postgres secara background.
Verifikasi Container:
docker-compose ps
Pastikan kedua container menunjukkan status Up.

Akses dan Konfigurasi Awal Odoo
Buka Browser:

Akses Odoo melalui URL:
http://localhost:8069
Setup Database Odoo:

Anda akan diarahkan ke halaman Database Manager.
Buat database baru dengan nama yang diinginkan.
Atur password dan parameter lain sesuai kebutuhan.
(Opsional) Konfigurasi Odoo:

Jika diperlukan, buat file odoo.conf di folder ./config untuk mengkustomisasi pengaturan, misalnya:
[options]
addons_path = /mnt/extra-addons,/usr/lib/python3/dist-packages/odoo/addons
db_host = db-postgres
db_port = 5432
db_user = odoo
db_password = odoo
Pastikan file tersebut telah ter-mount ke /etc/odoo dalam container.

Troubleshooting dan Reset Database
Reset Database
Jika pernah terjadi error akibat penggunaan nama database yang salah (misalnya "tes"), lakukan salah satu dari langkah berikut:

Melalui Web Database Manager:
http://localhost:8069/web/database/manager
Hapus database bermasalah (misalnya "tes") melalui opsi yang tersedia.
Buat database baru dengan konfigurasi yang benar.
Menghapus Data Volume PostgreSQL:

Matikan container:
docker-compose down
Hapus folder data PostgreSQL:
sudo rm -rf ./db-data
Mulai ulang container:
docker-compose up -d
Buat database baru melalui Database Manager.
Troubleshooting Permasalahan Permission
Masalah Permission
Jika muncul error permission (misalnya ketika Odoo tidak dapat membuat folder seperti /var/lib/odoo/.local), kemungkinan penyebabnya adalah folder yang di-bind dari host tidak memiliki izin yang sesuai untuk user Odoo dalam container.

Solusi:
Cek UID/GID User dalam Container: Jalankan perintah berikut untuk memeriksa UID/GID user Odoo:
docker exec -it odoo-latest id odoo
Contoh output:
uid=100(odoo) gid=101(odoo) groups=101(odoo)
Ubah Kepemilikan Folder pada Host: Pastikan folder web-data memiliki kepemilikan sesuai dengan UID dan GID yang diperoleh:
sudo chown -R 100:101 ./web-data
Lakukan langkah serupa untuk folder-folder lain (seperti ./config atau ./custom-addons) jika diperlukan.

Atur Izin Folder (Opsional untuk Testing): Untuk sementara, Anda dapat memberikan izin penuh:
chmod -R 777 ./web-data
Peringatan: Izin 777 tidak disarankan untuk lingkungan produksi karena risiko keamanan.

Restart Container: Setelah mengubah izin, restart container:
docker-compose down
docker-compose up -d
Periksa kembali log untuk memastikan error permission sudah tidak muncul.

Penutup
Dokumentasi ini menjelaskan langkah-langkah mulai dari pembuatan struktur direktori, konfigurasi Docker Compose, deployment, hingga troubleshooting (termasuk penanganan masalah permission dan reset database). Gunakan dokumen ini sebagai panduan dalam meng-deploy Odoo di lingkungan WSL dengan Docker Compose.
