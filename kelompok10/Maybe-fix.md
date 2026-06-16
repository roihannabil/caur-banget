Berikut adalah seluruh panduan lengkap dari awal hingga akhir untuk implementasi pengerasan (*hardening*) keamanan tingkat sistem, jaringan, dan penempatan aplikasi web **ARTERI** menggunakan arsitektur **Docker Swarm**.
Dokumen ini ditulis dalam format Markdown (.md) yang bersih, terstruktur, dan siap Anda salin ke repositori tugas kelompok 10.
```markdown
# Dokumentasi Hardening Server CIS & Deployment Arteri Open Source (Kelompok 10)

Dokumen ini berisi panduan *step-by-step* implementasi keamanan berbasis dokumen **CIS Hardening** dan deployment aplikasi **Arsip Elektronik Terintegrasi (ARTERI)** menggunakan arsitektur **Docker Swarm** (Multi-Node) dengan metode **CLI**.

---

## 1. Topologi Jaringan & Segregasi Perangkat

Sistem ini memisahkan layer aplikasi dan layer database pada dua perangkat fisik yang berbeda untuk memperkecil *attack surface*.

### Alokasi IP Address Jaringan Lokal:
* **Server 1 (App + Reverse Proxy):** `192.168.1.10` (Manager Node)
* **Server 2 (Database):** `192.168.1.20` (Worker Node)
* **Laptop Admin / Client:** `192.168.1.X` (Penguji)

> 💡 **Kisi-Kisi Ujian/Pertanyaan Dosen:**
> * **Pertanyaan:** *"Kenapa database dan aplikasi dipisah di laptop yang berbeda?"*
> * **Jawaban:** *"Untuk menerapkan prinsip **Segregasi Jaringan (Network Segregation)**. Jika container aplikasi di Server 1 berhasil ditembus oleh peretas, database di Server 2 tetap aman karena tidak berada di host yang sama dan dilindungi oleh firewall yang ketat."*

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
# Mengaktifkan IP Forwarding (Diperlukan untuk routing container Docker)
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
### Langkah A: Instalasi Paket Mandiri
```bash
sudo pacman -S docker nginx-mainline firewalld --noconfirm
sudo systemctl enable --now docker firewalld nginx

```
### Langkah B: Konfigurasi Firewalld di Server 1 (Aplikasi & Proxy)
Membuka port untuk lalu lintas web publik dan manajemen cluster Docker Swarm.
```bash
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --permanent --add-service=ssh

# Port Komunikasi internal Docker Swarm (Manager Node)
sudo firewall-cmd --permanent --add-port=2377/tcp
sudo firewall-cmd --permanent --add-port=7946/tcp
sudo firewall-cmd --permanent --add-port=7946/udp
sudo firewall-cmd --permanent --add-port=4789/udp

sudo firewall-cmd --reload

```
### Langkah C: Konfigurasi Firewalld di Server 2 (Database)
Port database (3306) dikunci rapat dan hanya boleh diketuk oleh IP Server 1 menggunakan *Rich Rules*.
```bash
sudo firewall-cmd --permanent --add-service=ssh

# Port Komunikasi internal Docker Swarm (Worker Node)
sudo firewall-cmd --permanent --add-port=7946/tcp
sudo firewall-cmd --permanent --add-port=7946/udp
sudo firewall-cmd --permanent --add-port=4789/udp

# Rich Rule: Hanya terima koneksi port 3306 dari Server 1
sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="192.168.1.10" port port="3306" protocol="tcp" accept'

sudo firewall-cmd --reload

```
## 4. Inisialisasi Docker Swarm Cluster
### Langkah A: Di Server 1 (Manager Node)
```bash
sudo docker swarm init --advertise-addr 192.168.1.10

```
*Salin token join yang muncul di terminal (Contoh: docker swarm join --token SWMTKN-1-XXXX...).*
### Langkah B: Di Server 2 (Worker Node)
Tempelkan token yang didapat dari Server 1 ke terminal Server 2:
```bash
sudo docker swarm join --token SWMTKN-1-XXXXX 192.168.1.10:2377

```
## 5. Deployment Docker Stack (Database & Aplikasi)
Di **Server 1**, buat file manifes orkestrasi:
```bash
nano docker-compose.yml

```
Masukkan konfigurasi penempatan (*placement constraints*) berikut:
```yaml
version: '3.8'

services:
  arteri_app:
    image: nginx:alpine
    volumes:
      - /var/www/html/arteri:/usr/share/nginx/html
    ports:
      - "8080:80"
    networks:
      - arteri_net
    deploy:
      placement:
        constraints:
          - node.role == manager

  arteri_db:
    image: mariadb:10.6
    environment:
      MYSQL_ROOT_PASSWORD: PasswordAmanArteri123!
      MYSQL_DATABASE: arteri_db
    ports:
      - "3306:3306"
    networks:
      - arteri_net
    deploy:
      placement:
        constraints:
          - node.role == worker

networks:
  arteri_net:
    driver: overlay

```
Eksekusi deployment stack:
```bash
sudo docker stack deploy -c docker-compose.yml arteri_stack

```
## 6. Integrasi Source Code ARTERI & Nginx Reverse Proxy
### Langkah A: Memasang Source Code ARTERI (Server 1)
```bash
sudo mkdir -p /var/www/html/arteri
sudo unzip arteri.zip -d /var/www/html/arteri/

# Pengerasan Hak Akses Direktori Web (CIS Standard)
sudo chown -R 101:101 /var/www/html/arteri
sudo find /var/www/html/arteri -type d -exec chmod 755 {} \;
sudo find /var/www/html/arteri -type f -exec chmod 644 {} \;

```
### Langkah B: Menghubungkan Aplikasi ke Database
Buka file konfigurasi internal database milik ARTERI (misal: config.php atau .env), lalu sesuaikan parameternya:
 * **Database Host:** arteri_db *(Menggunakan nama service Docker Swarm DNS)*
 * **Database Name:** arteri_db
 * **Database User:** root
 * **Database Password:** PasswordAmanArteri123!
### Langkah C: Konfigurasi Nginx Reverse Proxy (Server 1)
Mengonfigurasi Nginx lokal sebagai garda terdepan untuk menerima request port 80 dan meneruskannya ke kontainer aplikasi.
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
 1. Sambungkan **Laptop Penguji (Client/Admin)** ke jaringan Wi-Fi/LAN yang sama dengan server.
 2. Buka Web Browser (Chrome/Firefox).
 3. Akses URL: http://192.168.1.10/
 4. **Hasil:** Halaman web **ARTERI Open Source** (Arsip Elektronik Terintegrasi) akan berhasil memuat halaman login berwarna merah dengan aman dan responsif.
```

```
