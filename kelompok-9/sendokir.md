Jika kamu memutuskan untuk menggunakan **hanya 1 laptop saja** tanpa Virtual Machine (VirtualBox/VMware), hal tersebut **sangat bisa dilakukan**.
Namun, **ada perubahan besar dalam alur arsitekturnya**. Karena tidak ada perangkat fisik kedua atau mesin virtual terpisah, kita tidak bisa membagi beban kerja secara Multi-Node (*Manager & Worker*). Sebagai gantinya, arsitekturnya berubah menjadi **Single-Node Swarm Cluster** (Satu laptop bertindak sebagai Manager sekaligus Worker), dan kontainer database PostgreSQL akan dijalankan di dalam Docker internal laptop yang sama.
Meskipun berjalan di 1 laptop, **aturan pengerasan keamanan (CIS Hardening), penggunaan kernel LTS, dan pelarangan base-devel tetap wajib diterapkan** agar tugas dari dosen tetap terpenuhi 100%.
Berikut adalah panduan CLI murni dari awal instalasi hingga *deployment* untuk skenario **1 Laptop**:
## FASE 1: Instalasi OS Blackbird Amanda (Di Partisi 47 GB)
Silakan booting menggunakan flashdisk installer *Blackbird Amanda* kamu.
### 1. Partisi dan Alokasi LVM Modular (CIS Benchmark 1.1)
Buka cfdisk untuk membuat partisi LVM di ruang kosong 47 GB kamu:
```bash
cfdisk /dev/nvme0n1
# Pilih Free Space 47G -> New -> Type: Linux LVM -> Write -> Quit

```
Potong volume virtual secara modular untuk membatasi ruang gerak jika ada eksploitasi (asumsi partisi baru adalah /dev/nvme0n1p6):
```bash
pvcreate /dev/nvme0n1p6
vgcreate amanda /dev/nvme0n1p6

lvcreate -L 20G amanda -n root
lvcreate -L 4G  amanda -n swap
lvcreate -L 5G  amanda -n tmp       # Ruang temporary terisolasi (CIS 1.1.2)
lvcreate -L 8G  amanda -n var       # Ruang log sistem terisolasi (CIS 1.1.6)
lvcreate -l 100%FREE amanda -n home # Dokumen pengguna (CIS 1.1.17)

```
### 2. Formatting & Mounting Sistem Berkebun
```bash
mkfs.ext4 /dev/amanda/root
mkfs.ext4 /dev/amanda/tmp
mkfs.ext4 /dev/amanda/var
mkfs.ext4 /dev/amanda/home
mkswap /dev/amanda/swap

mount /dev/amanda/root /mnt
mount --mkdir /dev/nvme0n1p1 /mnt/boot   # Berbagi rumah EFI (260M)
mount --mkdir /dev/amanda/tmp /mnt/tmp
mount --mkdir /dev/amanda/var /mnt/var
mount --mkdir /dev/amanda/home /mnt/home
swapon /dev/amanda/swap

```
### 3. Pacstrap Mandiri (Tanpa base-devel & Menggunakan Kernel LTS)
Instal paket pembangun secara eceran demi meminimalkan celah keamanan (*Attack Surface Reduction*):
```bash
pacstrap /mnt base linux-lts linux-lts-headers linux-firmware lvm2 networkmanager nano sudo grub efibootmgr git cmake make gcc binutils patch awk grep sed

genfstab -U /mnt >> /mnt/etc/fstab
arch-chroot /mnt

```
### 4. Konfigurasi Dasar & GRUB di Dalam Chroot
```bash
ln -sf /usr/share/zoneinfo/Asia/Jakarta /etc/localtime
hwclock --systohc
echo "en_US.UTF-8 UTF-8" > /etc/locale.gen && locale-gen
echo "LANG=en_US.UTF-8" > /etc/locale.conf
echo "amanda-tunggal" > /etc/hostname

useradd -m -G wheel -s /bin/bash mahasiswa
passwd mahasiswa
passwd root

# Aktifkan akses sudo
EDITOR=nano visudo  # Hapus pagar pada baris %wheel ALL=(ALL:ALL) ALL
systemctl enable NetworkManager

# Generate mkinitcpio (Pastikan HOOKS memuat lvm2 sebelum filesystems)
mkinitcpio -P
grub-mkconfig -o /boot/grub/grub.cfg
exit
umount -R /mnt
reboot

```
*Jangan lupa masuk ke Arch Official KDE Plasma kamu sebentar untuk menjalankan sudo grub-mkconfig -o /boot/grub/grub.cfg agar OS Amanda Tunggal ini muncul di menu triple-boot utama.*
## FASE 2: Implementasi CIS Hardening & Keamanan Kernel
*Boot laptopmu, masuk ke OS **Blackbird Amanda** yang baru.*
### 1. Pengerasan fstab (CIS 1.1)
```bash
sudo nano /etc/fstab

```
Tambahkan opsi nosuid,nodev,noexec pada partisi /tmp:
```text
/dev/mapper/amanda-tmp    /tmp        ext4    rw,nosuid,nodev,noexec,relatime   0   2
/dev/mapper/amanda-var    /var        ext4    rw,nosuid,nodev,relatime          0   2

```
### 2. Blacklist Modul Kernel Pengganggu (CIS 1.1 & 3.4)
```bash
sudo nano /etc/modprobe.d/cis-blacklist.conf

```
```text
blacklist cramfs
blacklist freevxfs
blacklist jffs2
blacklist hfs
blacklist hfsplus
blacklist udf
blacklist dccp
blacklist sctp
blacklist rds
blacklist tipc

```
### 3. Hardening Jaringan via Sysctl (CIS 3.1 & 3.2)
```bash
sudo nano /etc/sysctl.d/99-cis-hardening.conf

```
```text
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.default.send_redirects = 0
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.default.accept_redirects = 0
net.ipv4.conf.all.rp_filter = 1
net.ipv4.conf.default.rp_filter = 1
net.ipv4.conf.all.accept_source_route = 0
net.ipv4.conf.default.accept_source_route = 0
net.ipv4.conf.all.log_martians = 1
net.ipv4.conf.default.log_martians = 1
fs.suid_dumpable = 0

```
Terapkan: sudo sysctl --system
## FASE 3: Konfigurasi Firewalld (Versi 1 Laptop)
Karena aplikasinya berada di laptop yang sama, aturan firewall difokuskan untuk mengamankan akses masuk dari jaringan lokal luar ke arah Nginx, sekaligus mengisolasi port Docker internal.
```bash
sudo pacman -S firewalld --noconfirm
sudo systemctl enable --now firewalld

# Hanya buka port HTTP dan HTTPS untuk akses eksternal jaringan lokal
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https

# Tutup rapat port internal Docker dari luar, ijinkan hanya untuk localhost loopback
sudo firewall-cmd --permanent --add-interface=lo --zone=trusted

sudo firewall-cmd --reload

```
## FASE 4: Deployment Mayan EDMS Berbasis Docker Swarm (1 Device)
### 1. Inisialisasi Single-Node Swarm
```bash
sudo pacman -S docker --noconfirm
sudo systemctl enable --now docker

# Karena hanya 1 laptop, ikat swarm ke alamat IP localhost (127.0.0.1)
sudo docker swarm init --advertise-addr 127.0.0.1

```
### 2. Aturan CIS Docker Daemon
```bash
sudo nano /etc/docker/daemon.json

```
```json
{
  "icc": false,
  "live-restore": true,
  "userland-proxy": false,
  "no-new-privileges": true
}

```
sudo systemctl restart docker
### 3. Pembuatan Docker Compose File (Database & Aplikasi Digabung Bersebelahan)
Karena hanya ada 1 laptop, kontainer PostgreSQL (db) diletakkan di dalam file orkestrasi yang sama, namun komunikasinya diisolasi penuh di dalam jaringan virtual kustom (*Overlay Network*).
```bash
mkdir ~/mayan-single && cd ~/mayan-single
nano docker-compose.yml

```
```yaml
version: '3.8'

services:
  db:
    image: postgres:13-alpine
    environment:
      - POSTGRES_DB=mayan_db
      - POSTGRES_USER=mayan_user
      - POSTGRES_PASSWORD=PasswordCISSekaliPakai123
    volumes:
      - db_data:/var/lib/postgresql/data
    networks:
      - internal_net

  redis:
    image: redis:7.0-alpine
    networks:
      - internal_net

  mayan_app:
    image: mayanid/mayan:latest
    environment:
      - MAYAN_DATABASE_ENGINE=django.db.backends.postgresql
      - MAYAN_DATABASE_HOST=db  # Menembak container db di jaringan lokal internal docker
      - MAYAN_DATABASE_NAME=mayan_db
      - MAYAN_DATABASE_USER=mayan_user
      - MAYAN_DATABASE_PASSWORD=PasswordCISSekaliPakai123
      - MAYAN_CELERY_BROKER_URL=redis://redis:6379/0
      - MAYAN_CELERY_RESULT_BACKEND=redis://redis:6379/0
    networks:
      - internal_net
    ports:
      - "8000:8000"
    volumes:
      - app_data:/var/lib/mayan

networks:
  internal_net:
    driver: overlay

volumes:
  db_data:
  app_data:

```
Deploy stack ke klaster tunggal:
```bash
sudo docker stack deploy -c docker-compose.yml kelompok9_tunggal

```
## FASE 5: Konfigurasi Nginx Reverse Proxy (Akses Jaringan Lokal)
Nginx dipasang di depan sistem operasi untuk menjembatani port internal Docker (8000) ke port standar jaringan lokal (80) yang terikat ke IP LAN laptopmu (misal IP asli wifi laptopmu adalah 11.11.11.2).
```bash
sudo pacman -S nginx-mainline --noconfirm
sudo nano /etc/nginx/nginx.conf

```
Sesuaikan bagian blok server:
```text
server {
    listen 80;
    server_name 11.11.11.2; # Masukkan Alamat IP Jaringan Wifi/LAN laptopmu

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

```
sudo systemctl enable --now nginx
## 💡 Perbedaan yang Harus Kamu Laporkan ke Dosen:
Saat maju presentasi menggunakan metode **1 Laptop**, katakan ini kepada dosenmu agar beliau paham bahwa kamu tahu teorinya:
 1. **Segregasi via Kontainerisasi:** *"Karena keterbatasan perangkat fisik (hanya 1 device), kami menerapkan prinsip **Segregation of Services** secara virtual menggunakan Network Isolation di Docker Swarm. Kontainer database tidak membuka port ke luar sistem operasi laptop, melainkan dikunci rapat di dalam jaringan Overlay internal."*
 2. **Cluster Tunggal:** *"Docker Swarm tetap kami inisialisasi menggunakan mode Single-Node Cluster, sehingga manajemen arsitektur stack deployment-nya tetap identik dengan kondisi multi-node server."*
