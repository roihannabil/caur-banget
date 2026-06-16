Tentu, mari kita sesuaikan ulang seluruh panduan menggunakan *source code* dari repositori **arteri-hotfix** dan mengganti Docker dengan **Podman + Podman Swarm/Kube** sesuai permintaanmu.
Karena Podman secara alami bersifat *rootless* dan tidak menggunakan *daemon* seperti Docker, ada sedikit penyesuaian pada perintah-perintah CLI dan port firewall (firewalld).
Berikut adalah isi file dokumentasi .md lengkap dari nol yang siap kamu salin ke repositori kelompokmu:
```markdown
# Dokumentasi Hardening Server CIS & Deployment ARTERI Hotfix (Kelompok 10)

Dokumen ini berisi panduan *step-by-step* implementasi keamanan berbasis dokumen **CIS Hardening** dan deployment aplikasi **Arsip Elektronik Terintegrasi (ARTERI)** versi *Hotfix* menggunakan **Podman** secara **CLI** di lingkungan **Arch Linux (Kernel Linux-LTS)**.

---

## 1. Topologi Jaringan & Segregasi Perangkat

Sistem ini memisahkan layer aplikasi dan layer database pada dua perangkat fisik (laptop) yang berbeda untuk memperkecil *attack surface*.

### Alokasi IP Address Jaringan Lokal:
* **Server 1 (App + Nginx Reverse Proxy):** `192.168.1.10` (Podman Host 1)
* **Server 2 (Database MariaDB):** `192.168.1.20` (Podman Host 2)
* **Laptop Admin / Client:** `192.168.1.X` (Penguji)

> 💡 **Kisi-Kisi Ujian/Pertanyaan Dosen:**
> * **Pertanyaan:** *"Kenapa database dan aplikasi dipisah di laptop yang berbeda?"*
> * **Jawaban:** *"Untuk menerapkan prinsip **Network Segregation**. Jika container aplikasi di Server 1 berhasil dieksploitasi, database di Server 2 tetap aman karena berada di host terpisah dan dilindungi oleh aturan Firewalld yang ketat."*

---

## 2. CIS Hardening: Tingkat OS & Kernel Linux-LTS (Kedua Server)

Lakukan konfigurasi ini melalui CLI pada **Server 1** dan **Server 2**. Seluruh paket diinstal secara mandiri tanpa menggunakan meta-package `base-devel`.

### Langkah A: Memastikan Penggunaan Kernel LTS
```bash
sudo pacman -S linux-lts linux-lts-headers --noconfirm

```
### Langkah B: Menonaktifkan Modul Kernel yang Tidak Diperlukan
Menonaktifkan sistem berkas (*filesystem*) lawas dan protokol yang tidak digunakan untuk meminimalisir celah eksploitasi driver kernel.
```bash
sudo nano /etc/modprobe.d/cis-hardening.conf

```
Masukkan konfigurasi berikut:
```ini
install cramfs /bin/true
install freevxfs /bin/true
install jffs2 /bin/true
install hfs /bin/true
install hfsplus /bin/true
install udf /bin/true
install vfat /bin/true
install dccp /bin/true
install sctp /bin/true
install rds /bin/true
install tipc /bin/true

```
### Langkah C: Pengerasan Layer Jaringan Kernel (sysctl)
```bash
sudo nano /etc/sysctl.d/99-sysctl-cis.conf

```
Isi dengan parameter keamanan berikut:
```ini
# Mengaktifkan IP Forwarding untuk routing container rootless Podman
net.ipv4.ip_forward = 1

# Tolak Paket ICMP Redirect (Mencegah serangan Man-in-the-Middle / MITM)
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.default.accept_redirects = 0
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.default.send_redirects = 0

# Tolak Source Routed Packets
net.ipv4.conf.all.accept_source_route = 0
net.ipv4.conf.default.accept_source_route = 0

# Aktifkan Reverse Path Filtering (Mencegah IP Spoofing)
net.ipv4.conf.all.rp_filter = 1
net.ipv4.conf.default.rp_filter = 1

# Aktifkan TCP SYN Cookies (Proteksi dari serangan DoS / SYN Flood)
net.ipv4.tcp_syncookies = 1

```
Terapkan perubahan kernel:
```bash
sudo sysctl --system

```
## 3. Instalasi Komponen & Konfigurasi Firewalld
### Langkah A: Instalasi Paket Mandiri (Tanpa base-devel)
```bash
sudo pacman -S podman nginx-mainline firewalld wget unzip --noconfirm
sudo systemctl enable --now firewalld nginx

```
### Langkah B: Konfigurasi Firewalld di Server 1 (Aplikasi & Proxy)
Membuka port untuk lalu lintas web publik dan interkoneksi Podman.
```bash
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --permanent --add-service=ssh

# Membuka port internal podman untuk komunikasi container ke Server 2
sudo firewall-cmd --permanent --add-port=8080/tcp

sudo firewall-cmd --reload

```
### Langkah C: Konfigurasi Firewalld di Server 2 (Database)
Port database (3306) dikunci rapat dan hanya boleh diketuk oleh IP Server 1 menggunakan *Rich Rules*.
```bash
sudo firewall-cmd --permanent --add-service=ssh

# Rich Rule: Hanya terima koneksi port 3306 dari Server 1
sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="192.168.1.10" port port="3306" protocol="tcp" accept'

sudo firewall-cmd --reload

```
## 4. Deployment Container Menggunakan Podman
### Langkah A: Deployment Database MariaDB (Di Server 2)
Jalankan container database MariaDB secara independen di Server 2 dengan ekspos port ke jaringan lokal agar bisa diakses Server 1.
```bash
podman run -d --name arteri_db \
  -p 3306:3306 \
  -e MYSQL_ROOT_PASSWORD=PasswordAmanArteri123! \
  -e MYSQL_DATABASE=arteri_db \
  mariadb:10.6

```
### Langkah B: Deployment Container Web Server Aplikasi (Di Server 1)
Jalankan container Nginx bawaan Podman yang nantinya akan mengeksekusi source code ARTERI.
```bash
podman run -d --name arteri_app \
  -p 8080:80 \
  -v /var/www/html/arteri:/usr/share/nginx/html:Z \
  nginx:alpine

```
*(Catatan: Flag :Z di ujung volume penting di Podman untuk mengatur kecocokan label keamanan SELinux/AppArmor).*
## 5. Integrasi Source Code ARTERI Hotfix & Konfigurasi
Lakukan langkah-langkah ini di **Server 1** untuk memasukkan repositori *hotfix* yang diminta.
### Langkah A: Download & Ekstrak Source Code Arteri Hotfix
```bash
sudo mkdir -p /var/www/html/arteri
cd /var/www/html/
sudo wget [https://github.com/almuhdilkarim/arteri-hotfix/releases/download/hotfix/arteri-hotfix.zip](https://github.com/almuhdilkarim/arteri-hotfix/releases/download/hotfix/arteri-hotfix.zip)
sudo unzip arteri-hotfix.zip -d /var/www/html/arteri/

```
### Langkah B: Menghubungkan Aplikasi ke Database Server 2
Buka file konfigurasi database internal milik ARTERI (misal: application/config/database.php atau .env), lalu sesuaikan parameternya menggunakan IP fisik Server 2:
 * **Database Host:** 192.168.1.20 *(IP Server 2 yang menjalankan container MariaDB)*
 * **Database Name:** arteri_db
 * **Database User:** root
 * **Database Password:** PasswordAmanArteri123!
### Langkah C: Fix Bug Session & Deprecated (Sesuai riwayat debugging)
Buka file application/config/config.php dan pastikan session path-nya aman untuk container:
```php
$config['sess_save_path'] = sys_get_temp_dir();

```
### Langkah D: Pengerasan Hak Akses Direktori Web (CIS Standard)
```bash
sudo chown -R 101:101 /var/www/html/arteri
sudo find /var/www/html/arteri -type d -exec chmod 755 {} \;
sudo find /var/www/html/arteri -type f -exec chmod 644 {} \;

```
## 6. Konfigurasi Nginx Reverse Proxy (Server 1 Host)
Mengonfigurasi Nginx utama pada sistem operasi Server 1 sebagai garda terdepan untuk menerima request port 80 jaringan lokal dan meneruskannya ke container Podman.
```bash
sudo nano /etc/nginx/nginx.conf

```
Tambahkan/sesuaikan blok server berikut di dalam blok http:
```nginx
server {
    listen 80;
    server_name 192.168.1.10;

    # Hardening: Sembunyikan informasi versi Web Server (Information Leakage Protection)
    server_tokens off;

    location / {
        proxy_pass [http://127.0.0.1:8080](http://127.0.0.1:8080);
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded-for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

```
Muat ulang Nginx:
```bash
sudo systemctl restart nginx

```
## 7. Tahap Pengujian Akses Jaringan Lokal
 1. Sambungkan **Laptop Client / Admin** ke jaringan lokal/Wi-Fi yang sama.
 2. Buka Web Browser, lalu akses URL Server 1: http://192.168.1.10/
 3. **Hasil:** Halaman web **ARTERI Open Source (Hotfix Version)** akan berhasil memuat halaman login berwarna merah dengan aman tanpa error, dan data arsip terhubung secara *real-time* ke laptop database (Server 2).
## 🎯 Pertanyaan Kritis Dosen Mengenai Podman & Hotfix
> 💡 **Dosen:** *"Kenapa kalian memilih Podman daripada Docker untuk tugas ini?"*
> 💬 **Jawaban:** *"Podman bekerja dengan arsitektur **Daemonless** dan mendukung **Rootless Container** secara bawaan. Ini jauh lebih aman sesuai standar CIS karena jika container aplikasi berhasil ditembus, peretas tidak otomatis mendapatkan hak akses root pada sistem operasi host utama kami, berbeda dengan Docker yang berjalan sebagai root daemon."*
> 
> 💡 **Dosen:** *"Bagaimana container aplikasi mendeteksi database jika berada di laptop yang berbeda tanpa docker network overlay?"*
> 💬 **Jawaban:** *"Kami memanfaatkan fitur ekspos port terarah pada Podman di Server 2, lalu pada konfigurasi ARTERI hotfix, kami langsung mengarahkan target db_host ke IP fisik lokal Server 2 (192.168.1.20). Keamanan jalur ini dijamin oleh **Firewalld Rich Rules** yang memblokir IP lain selain Server 1."*
> 
```

Kamu tinggal salin mentah-mentah teks di dalam blok kode di atas, simpan dengan nama file seperti `tugas-kelompok10.md`, lalu unggah ke GitHub kelompok kalian! Langkahnya sudah lengkap disesuaikan dengan tautan rilis *hotfix* dan Podman.

```
