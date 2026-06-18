Repository tersebut (kelompok-10_4C/instal_slims.md) mengimplementasikan langkah-langkah yang **sangat baik, terstruktur, dan benar** untuk kebutuhan praktikum kamu.
Mereka menggunakan pendekatan pembuatan **Containerfile / Dockerfile kustom** untuk membangun (*build*) image SLiMS dari *source code* GitHub, lalu menjalankannya di dalam **Podman Pod**. Pendekatan ini jauh lebih rapi karena semua konfigurasi ekstensi PHP sudah otomatis dibungkus di dalam image baru tersebut.
Berikut adalah ringkasan alur instalasi dari repo tersebut yang sudah disesuaikan dengan konfigurasi **Firewalld Port 22** yang kita bahas sebelumnya.
### FASE 1: Di Laptop Server (Fisik)
**1. Konfigurasi Jaringan LAN Server**
Hubungkan kabel LAN, pasang IP 192.168.1.20/24, dan cek nama interface:
```bash
ip link

```
**2. Aktifkan OpenSSH Server**
```bash
sudo pacman -S openssh --noconfirm
sudo systemctl enable --now sshd

```
**3. Konfigurasi Firewalld (Sesuai Aturan Port 22 Temanmu)**
```bash
sudo pacman -S firewalld --noconfirm
sudo systemctl enable --now firewalld

# Amankan Zona Public (Untuk Client)
sudo firewall-cmd --zone=public --remove-service=ssh --permanent
sudo firewall-cmd --zone=public --add-port=8080/tcp --permanent

# Konfigurasi Zona Work (Untuk Admin)
sudo firewall-cmd --zone=work --add-source=192.168.1.10 --permanent
sudo firewall-cmd --zone=work --add-port=22/tcp --permanent

# Reload agar aktif
sudo firewall-cmd --reload

```
### FASE 2: Di Laptop Admin (Fisik)
*Hubungkan ke LAN (192.168.1.10/24) lalu remot Server:*
```bash
ssh username_server@192.168.1.20

```
*Jalankan langkah-langkah berikut di dalam sesi SSH (seperti di repo tersebut):*
**1. Kloning Source Code & Install Podman**
```bash
sudo pacman -S git podman --noconfirm
cd ~
git clone https://github.com/slims/slims9_bulian.git ~/slims9_bulian
cd ~/slims9_bulian

```
**2. Membuat Containerfile (Kustom Image SLiMS)**
Sesuai taktik di repo kelompok tersebut, kita buat sebuah berkas bernama Containerfile di dalam folder slims9_bulian agar instalasi ekstensi PHP langsung otomatis ter-bake saat image dibuat:
```bash
nano Containerfile

```
*Salin dan tempel kode berikut ke dalam file tersebut:*
```dockerfile
FROM docker.io/library/php:7.4-apache

# Menginstall ekstensi pdo_mysql dan mysqli yang dibutuhkan SLiMS
RUN docker-php-ext-install mysqli pdo pdo_mysql

# Menyalin seluruh source code SLiMS ke direktori web Apache
COPY . /var/www/html/

# Memberikan hak akses ke Apache
RUN chown -R www-data:www-data /var/www/html/

```
*Simpan dengan menekan Ctrl+O, lalu Enter, dan keluar dengan Ctrl+X.*
**3. Melakukan Build Image Kustom**
```bash
podman build -t slims-custom:9 .

```
**4. Deploy Menggunakan Podman Pod**
```bash
# Buat wadah Pod
podman pod create --name slims-pod -p 8080:80

# Jalankan Container Database MariaDB
podman run -d --pod slims-pod \
  --name slims-db \
  -e MYSQL_ROOT_PASSWORD=rahasia_root \
  -e MYSQL_DATABASE=slims \
  -e MYSQL_USER=slims \
  -e MYSQL_PASSWORD=slims \
  docker.io/library/mariadb:10.5

# Jalankan Container Web menggunakan Image Kustom yang baru kita build
podman run -d --pod slims-pod \
  --name slims-web \
  localhost/slims-custom:9

```
### FASE 3: Tahap Akhir & Pengujian
 1. **Setup Database via Browser Admin:**
   * Buka browser di Laptop Admin, akses http://192.168.1.20:8080/
   * Masukkan konfigurasi database:
     * **Database Host:** 127.0.0.1
     * **Database Name:** slims
     * **Database User:** slims
     * **Database Password:** slims
 2. **Pengujian Client (192.168.1.30):**
   * Jalankan ssh username@192.168.1.20 \rightarrow **Harus Gagal/Timeout**.
   * Buka browser ke http://192.168.1.20:8080/ \rightarrow **Harus Berhasil Terbuka**.
Metode dari repo tersebut **sudah sangat benar dan lebih aman** dari sisi penanganan *permission file* di dalam container. Kamu bisa langsung mengikuti cara tersebut!
