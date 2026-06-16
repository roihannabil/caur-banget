Berikut adalah panduan lengkap dan urut dari awal (kondisi server baru selesai instal desktop XFCE & NetworkManager) hingga aplikasi web **ARTERI** siap diakses secara aman sesuai standar pengerasan (*hardening*).
Panduan ini juga dilengkapi dengan **penjelasan logis ("Kenapa ini dilakukan?")** di setiap langkahnya agar kelompok Anda siap menjawab pertanyaan dosen saat asistensi atau ujian praktikum.
## TAHAP 1: Topologi, Pengkabelan, dan Pengaturan IP
Sebelum menginstal aplikasi, jaringan antar-laptop harus terhubung dan memiliki identitas (IP Address) yang jelas.
### Langkah-langkah:
 1. Hubungkan **Laptop Admin** dan **Laptop Server** secara fisik menggunakan kabel LAN.
 2. Hubungkan **Laptop Server** dan **Laptop Client** ke jaringan Wi-Fi/Hotspot yang sama.
 3. Buka NetworkManager di **Laptop Server** (bisa lewat ikon jaringan di pojok kanan bawah XFCE atau via terminal menggunakan perintah nmtui).
 4. Atur IP statis pada interface LAN Server ke 192.168.1.1/24 dan biarkan interface Wi-Fi mendapatkan IP otomatis dari hotspot (misal didapat 192.168.43.10).
 5. Atur IP statis pada **Laptop Admin** (interface LAN) ke 192.168.1.2/24.
> 💡 **Pertanyaan Dosen:** *"Kenapa topologinya dibuat seperti ini? Kenapa Admin pakai kabel LAN dan Client pakai Wi-Fi?"*
> 💬 **Jawaban Anda:** *"Untuk menerapkan prinsip **Segregasi Jaringan (Network Segregation)**. Jalur manajemen/remot oleh Admin dipisahkan secara fisik menggunakan kabel LAN privat agar tidak bisa diintip atau diserang oleh Client umum yang berada di jaringan Wi-Fi."*
> 
## TAHAP 2: Instalasi Komponen Web Server & Database (LAMP Stack)
Aplikasi ARTERI adalah arsip elektronik berbasis web. Agar bisa berjalan, Server membutuhkan Web Server (Apache), Database (MariaDB/MySQL), dan PHP.
### Langkah-langkah:
Jalankan perintah ini di terminal **Laptop Server**:
```bash
# 1. Update repositori sistem
sudo apt update

# 2. Instal Apache, MariaDB, PHP, dan modul pendukungnya
sudo apt install apache2 mariadb-server php php-mysql libapache2-mod-php unzip -y

# 3. Pastikan semua layanan langsung berjalan otomatis saat server menyala
sudo systemctl enable --now apache2 mariadb

```
> 💡 **Pertanyaan Dosen:** *"Apa fungsi dari software yang baru kalian instal ini?"*
> 💬 **Jawaban Anda:** *"Kami menginstal **LAMP Stack**. Apache bertugas sebagai Web Server untuk memproses permintaan halaman web ARTERI, MariaDB sebagai basis data untuk menyimpan data arsip, dan PHP sebagai bahasa pemrograman yang mengeksekusi aplikasi ARTERI tersebut."*
> 
## TAHAP 3: Deployment & Pengerasan Aplikasi ARTERI
Sekarang kita memasukkan source code ARTERI ke server dan langsung melakukan *hardening* pada level web dan database.
### Langkah-langkah:
 1. Pindahkan file *source code* ARTERI (misal berupa .zip) ke direktori web server, lalu ekstrak:
   ```bash
   sudo unzip arteri.zip -d /var/www/html/
   
   ```
```
2. **Hardening Hak Akses File (Penting!):** Jangan biarkan folder web bisa dimodifikasi bebas (akses 777). Batasi hak aksesnya:
   ```bash
   sudo chown -R www-data:www-data /var/www/html/arteri
   sudo find /var/www/html/arteri -type d -exec chmod 755 {} \;
   sudo find /var/www/html/arteri -type f -exec chmod 644 {} \;

```
 3. **Hardening Informasi Web Server (Sembunyikan Versi Apache):**
   Buka file konfigurasi keamanan Apache:
   ```bash
   sudo nano /etc/apache2/conf-available/security.conf
   
   ```
```
   Ubah parameter berikut menjadi:
   ```ini
   ServerTokens Prod
   ServerSignature Off

```
Simpan (Ctrl+O, Enter, Ctrl+X) lalu restart Apache: sudo systemctl restart apache2.
4. **Hardening Database:** Secure instalasi MariaDB dengan perintah:
```bash
sudo mysql_secure_installation

```
*Pilih **Y** untuk semua opsi (buat password root baru, hapus anonymous user, matikan remote login untuk root, dan hapus test database).*
> 💡 **Pertanyaan Dosen:** *"Kenapa hak akses file diatur ke 755 dan 644? Kenapa versi Apache-nya harus disembunyikan?"*
> 💬 **Jawaban Anda:** *"Akses 755 dan 644 memastikan bahwa file aplikasi hanya bisa ditulis (*write*) oleh pemiliknya (user web server) dan mencegah penyerang mengunggah skrip berbahaya. Menyembunyikan versi Apache (ServerTokens Prod) dilakukan untuk **mencegah Information Leakage**, sehingga penyerang tidak tahu versi persis web server kami dan kesulitan mencari celah eksploitasi (*exploit*) yang spesifik."*
> 
## TAHAP 4: Implementasi CIS Hardening (Sistem & Jaringan)
Tahap ini adalah inti dari tugas kelompok Anda, yaitu menerapkan pengerasan keamanan pada sistem operasi Linux sesuai standar CIS.
### Langkah A: Banner Peringatan (*Login Banners*)
Ubah teks sambutan login agar penyerang secara hukum tahu bahwa sistem ini diawasi.
```bash
sudo echo "Akses Terbatas! Hanya untuk personel berwenang. Semua aktivitas dicatat." > /etc/issue.net
sudo chmod 644 /etc/issue.net

```
### Langkah B: Menonaktifkan Layanan yang Tidak Diperlukan
Layanan bawaan OS yang tidak dipakai untuk web ARTERI harus dimatikan demi mengurangi *attack surface*.
```bash
sudo systemctl disable --now avahi-daemon cups 2>/dev/null

```
### Langkah C: Pengamanan Kernel Jaringan (sysctl)
Buat file konfigurasi keamanan jaringan baru pada kernel:
```bash
sudo nano /etc/sysctl.d/99-hardening.conf

```
Masukkan parameter perlindungan berikut:
```ini
# Matikan IP Forwarding (Server tidak boleh jadi router)
net.ipv4.ip_forward = 0

# Tolak paket ICMP redirect (Mencegah serangan Man-in-the-Middle / MITM)
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.default.accept_redirects = 0

# Aktifkan SYN Cookies (Proteksi dari serangan DoS / SYN Flood)
net.ipv4.tcp_syncookies = 1

```
Terapkan konfigurasi baru: sudo sysctl --system
> 💡 **Pertanyaan Dosen:** *"Apa itu SYN Cookies dan kenapa accept_redirects dimatikan?"*
> 💬 **Jawaban Anda:** *"**SYN Cookies** digunakan untuk melindungi server dari serangan **DoS (Denial of Service) tipe SYN Flood** agar memori server tidak habis saat dibombardir koneksi palsu. Sedangkan **accept_redirects dimatikan** untuk mencegah serangan **MITM (Man-in-the-Middle)** di mana penyerang mencoba membelokkan jalur data server lewat perangkat mereka."*
> 
## TAHAP 5: Konfigurasi Host-Based Firewall (IPTables)
Kita akan mengunci total Server menggunakan IPTables. Server hanya akan merespons SSH dari Laptop Admin, dan merespons Web (HTTP/HTTPS) dari siapapun.
### Langkah-langkah:
 1. Buat skrip firewall otomatis:
   ```bash
   
   ```
sudo nano /etc/iptables-rules.sh
```
2. Isikan skrip berikut (sesuaikan IP Admin Anda):
   ```bash
   #!/bin/bash
   # 1. Bersihkan semua aturan lama
   iptables -F

   # 2. Blokir semua lalu lintas data secara default (Whitelist System)
   iptables -P INPUT DROP
   iptables -P OUTPUT DROP
   iptables -P FORWARD DROP

   # 3. Izinkan komunikasi internal server (Loopback)
   iptables -A INPUT -i lo -j ACCEPT
   iptables -A OUTPUT -o lo -j ACCEPT

   # 4. KUNCI SSH: Hanya izinkan masuk dari IP Laptop Admin (192.168.1.2)
   iptables -A INPUT -p tcp -s 192.168.1.2 --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
   iptables -A OUTPUT -p tcp --sport 22 -d 192.168.1.2 -m state --state ESTABLISHED -j ACCEPT

   # 5. Buka Akses Web ARTERI (Port 80 HTTP) untuk semua Laptop
   iptables -A INPUT -p tcp --dport 80 -m state --state NEW,ESTABLISHED -j ACCEPT
   iptables -A OUTPUT -p tcp --sport 80 -m state --state ESTABLISHED -j ACCEPT

   # 6. Izinkan server merespons koneksi yang sudah terjadi (Established)
   iptables -A OUTPUT -p tcp -m state --state NEW,ESTABLISHED -j ACCEPT
   iptables -A INPUT -p tcp -m state --state ESTABLISHED -j ACCEPT

```
 3. Beri izin eksekusi dan jalankan skripnya:
   ```bash
   sudo chmod +x /etc/iptables-rules.sh
   sudo /etc/iptables-rules.sh
   
   ```
```

> 💡 **Pertanyaan Dosen:** *"Bagaimana prinsip kerja firewall yang kalian buat?"*
> 💬 **Jawaban Anda:** *"Kami menggunakan metode **Default DROP (Whitelist)**. Kami menutup seluruh akses ke server terlebih dahulu, kemudian secara selektif hanya membuka port 80 untuk web ARTERI agar bisa diakses semua user, serta port 22 (SSH) yang **dikunci khusus** hanya untuk IP Laptop Admin demi keamanan manajemen."*

---

## TAHAP 6: Demo Pengujian di Hadapan Dosen

Untuk membuktikan konfigurasi Anda sukses, tunjukkan skenario pengujian ini langsung menggunakan 3 laptop:

1. **Uji Coba Akses Web (Sukses di Kedua Laptop):**
   * Di **Laptop Admin** dan **Laptop Client**, buka browser lalu ketik IP Server (`[http://192.168.43.10/arteri](http://192.168.43.10/arteri)` atau `[http://192.168.1.1/arteri](http://192.168.1.1/arteri)`).
   * Tunjukkan halaman login ARTERI berhasil terbuka sempurna di kedua laptop.
2. **Uji Coba Remote SSH Admin (Sukses):**
   * Di **Laptop Admin**, buka terminal lalu ketik `ssh user@192.168.1.1`.
   * Anda akan berhasil masuk dan banner peringatan yang dibuat di Tahap 4 akan muncul di layar.
3. **Uji Coba Remote SSH Client (Gagal/Diblokir):**
   * Di **Laptop Client**, coba ketik perintah SSH yang sama ke Server.
   * Koneksi akan macet (*hang*) atau *Timed Out*. Ini membuktikan firewall IPTables Anda bekerja dengan sangat baik dalam melindungi port krusial server dari pihak luar.

```
