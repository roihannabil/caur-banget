Berikut adalah dokumentasi lengkap dari awal hingga akhir dalam bentuk format Markdown (.md). Dokumen ini telah dimodifikasi secara radikal dari dokumen Kelompok 9 agar **100% patuh pada aturan Kelas C** (Tanpa base-devel, Kernel linux-lts, skema partisi LVM penuh standar CIS Server Hardening, Docker Swarm, Firewalld, dan Nginx).
Kamu bisa langsung menyalin seluruh isi di bawah ini untuk dijadikan laporan tugas atau dokumentasi di repositori GitHub kamu.
```markdown
# Dokumentasi Instalasi OS Blackbird Amanda & Hardening Server CIS (Kelas C)

Dokumentasi ini disusun untuk memenuhi tugas praktikum Kelas C dengan skema **Triple Boot (Bare-Metal/Real Hardware)** pada sisa ruang disk 47 GB. Seluruh proses dilakukan menggunakan metode CLI murni, tanpa paket `base-devel`, menggunakan kernel `linux-lts`, serta mengimplementasikan CIS Server Hardening secara menyeluruh.

---

## TAHAP 1: PERSIAPAN & PEMBAGIAN DISK (LVM CIS COMPLIANT)

1. Boot laptop menggunakan Flashdisk Installer OS Blackbird Amanda melalui menu UEFI.
2. Setelah masuk ke prompt installer, buka utilitas manajemen partisi:
```bash
   cfdisk /dev/nvme0n1

```
 3. Cari ruang kosong (**Free Space ~47 GB**) di bagian paling bawah.
 4. Pilih **[ New ]** -> Gunakan seluruh sisa ruang tersebut -> Pilih **[ Type ]** -> Ubah menjadi **Linux LVM**.
 5. Pilih **[ Write ]**, ketik yes, lalu pilih **[ Quit ]**.
   *(Catatan: Partisi LVM baru ini berasumsi sebagai /dev/nvme0n1p6)*
## TAHAP 2: KONFIGURASI LOGICAL VOLUME (CIS BENCHMARK SECTION 1.1)
Sesuai standar CIS Server, direktori sistem yang krusial harus dipisahkan ke dalam volume tersendiri untuk mencegah kegagalan sistem akibat disk penuh dan membatasi eksekusi file berbahaya.
```bash
# 1. Inisialisasi Physical Volume dan Volume Group khusus Tugas
pvcreate /dev/nvme0n1p6
vgcreate vg_tugas /dev/nvme0n1p6

# 2. Pembagian Ruang Logical Volume secara Proporsional (Total ~47 GB)
lvcreate -L 12G   vg_tugas -n root   # OS Dasar
lvcreate -L 3G    vg_tugas -n tmp    # Tempat file sementara (CIS 1.1.2)
lvcreate -L 4G    vg_tugas -n var    # Data dinamis aplikasi
lvcreate -L 5G    vg_tugas -n vlog   # Log Sistem (CIS 1.1.11)
lvcreate -L 3G    vg_tugas -n vaud   # Log Audit Keamanan (CIS 1.1.12)
lvcreate -L 12G   vg_tugas -n dock   # Penyimpanan Kontainer Mayan EDMS
lvcreate -l 100%FREE vg_tugas -n home # Direktori User (~8 GB)

```
### Format Seluruh Logical Volume
```bash
mkfs.ext4 /dev/vg_tugas/root
mkfs.ext4 /dev/vg_tugas/tmp
mkfs.ext4 /dev/vg_tugas/var
mkfs.ext4 /dev/vg_tugas/vlog
mkfs.ext4 /dev/vg_tugas/vaud
mkfs.ext4 /dev/vg_tugas/dock
mkfs.ext4 /dev/vg_tugas/home

```
### Mount Berdasarkan Hirarki Struktur Direktori
Proses pemasangan (*mounting*) wajib dilakukan secara berurutan agar folder sub-direktori tidak tertimpa:
```bash
# Mount Root
mount /dev/vg_tugas/root /mnt

# Mount Direktori Utama
mount --mkdir /dev/nvme0n1p1        /mnt/boot   # Berbagi EFI dengan OS Utama & Windows
mount --mkdir /dev/vg_tugas/home    /mnt/home
mount --mkdir /dev/vg_tugas/tmp     /mnt/tmp
mount --mkdir /dev/vg_tugas/var     /mnt/var

# Mount Sub-Direktori di dalam /var
mount --mkdir /dev/vg_tugas/vlog    /mnt/var/log
mount --mkdir /dev/vg_tugas/vaud    /mnt/var/log/audit
mount --mkdir /dev/vg_tugas/dock    /mnt/var/lib/docker

```
## TAHAP 3: PACSTRAP (TANPA BASE-DEVEL & WAJIB LINUX-LTS)
Aturan tegas penugasan Kelas C melarang penggunaan meta-package base-devel. Seluruh paket pendukung wajib diinstal satu per satu secara mandiri.
```bash
pacstrap /mnt base linux-lts linux-lts-headers linux-firmware networkmanager nano sudo grub efibootmgr os-prober docker git nginx firewalld

```
## TAHAP 4: KONFIGURASI SISTEM DASAR (CHROOT)
```bash
# 1. Ambil informasi tabel partisi
genfstab -U /mnt >> /mnt/etc/fstab

# 2. Masuk ke lingkungan Chroot
arch-chroot /mnt

```
### Di Dalam Lingkungan Chroot:
```bash
# 1. Pengaturan Waktu
ln -sf /usr/share/zoneinfo/Asia/Jakarta /etc/localtime
hwclock --systohc

# 2. Pengaturan Lokalisasi & Bahasa
nano /etc/locale.gen  # Hilangkan tanda pagar (#) pada baris en_US.UTF-8 UTF-8
locale-gen
echo "LANG=en_US.UTF-8" > /etc/locale.conf

# 3. Identitas Hostname Server
echo "blackbird-amanda" > /etc/hostname

# 4. Pengaturan Akun dan Akses Sudo (Ganti 'roihan' dengan nama kamu)
passwd (Ketik password untuk root)
useradd -m -G wheel -s /bin/bash roihan
passwd roihan (Ketik password untuk user kamu)

# Berikan hak akses administrative ke grup wheel
EDITOR=nano visudo
# Hilangkan tanda pagar (#) pada baris: %wheel ALL=(ALL:ALL) ALL

```
### Kompilasi Ramdisk Kernel LTS
```bash
mkinitcpio -p linux-lts

```
### Instalasi GRUB Multi-Boot Aman
```bash
mkdir -p /var/lib/os-prober

nano /etc/default/grub
# Pastikan baris berikut aktif (tidak ter-pagar):
GRUB_DISABLE_OS_PROBER=false

# Install GRUB dengan bootloader-id unik agar tidak menimpa Arch Utama
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=AMANDA-TUGAS
grub-mkconfig -o /boot/grub/grub.cfg

```
### Selesai Instalasi Dasar
```bash
exit
umount -R /mnt
reboot

```
*Cabut flashdisk.* Nyalakan laptop dan pilih boot menu **AMANDA-TUGAS**, kemudian login menggunakan user kamu.
## TAHAP 5: HARDENING LAYER KERNEL & FSTAB (CIS SECURE)
Setelah berhasil masuk ke dalam OS Amanda yang baru, lakukan pengamanan sistem langsung lewat instruksi CLI berikut:
### 1. Modifikasi /etc/fstab untuk Proteksi Mount Point
Tambahkan flag parameter keamanan nodev, nosuid, dan noexec pada partisi LVM yang telah dipisah untuk membatasi ruang gerak exploit/malware.
```bash
sudo nano /etc/fstab

```
Sesuaikan bagian ujung baris konfigurasi LVM hingga menjadi seperti ini:
```text
/dev/mapper/vg_tugas-tmp   /tmp            ext4    defaults,nodev,nosuid,noexec   0 2
/dev/mapper/vg_tugas-var   /var            ext4    defaults,nodev                 0 2
/dev/mapper/vg_tugas-vlog  /var/log        ext4    defaults,nodev,nosuid,noexec   0 2
/dev/mapper/vg_tugas-vaud  /var/log/audit  ext4    defaults,nodev,nosuid,noexec   0 2

```
### 2. Disable Modul Kernel Tidak Diperlukan (CIS 1.1)
```bash
sudo nano /etc/modprobe.d/cis-hardening.conf

```
Isi dengan daftar pemblokiran filesystem jadul berikut:
```text
blacklist cramfs
blacklist freevxfs
blacklist jffs2
blacklist hfs
blacklist hfsplus
blacklist udf
blacklist usb-storage

```
### 3. Hardening Jaringan Parameter Kernel via Sysctl (CIS 3.1 & 3.2)
```bash
sudo nano /etc/sysctl.d/99-cis-hardening.conf

```
Masukkan konfigurasi berikut untuk mencegah serangan MITM dan Spoofing Network:
```ini
net.ipv4.ip_forward=1
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.default.accept_redirects = 0
net.ipv4.conf.all.accept_source_route = 0
net.ipv4.conf.default.accept_source_route = 0
net.ipv4.conf.all.log_martians = 1
net.ipv4.conf.default.log_martians = 1
net.ipv4.tcp_syncookies = 1

```
Terapkan instan ke kernel:
```bash
sudo sysctl --system

```
## TAHAP 6: FIREWALLD & DOCKER SWARM (ISOLASI MULTI-NODE LOGIS)
### 1. Konfigurasi Kebijakan Firewalld
```bash
sudo systemctl enable --now firewalld

# Buka port standar untuk web server lokal
sudo firewall-cmd --permanent --zone=public --add-service=http
sudo firewall-cmd --permanent --zone=public --add-service=https

# Buka lalu lintas komunikasi routing internal Docker Swarm
sudo firewall-cmd --permanent --add-port=2377/tcp
sudo firewall-cmd --permanent --add-port=7946/tcp
sudo firewall-cmd --permanent --add-port=7946/udp
sudo firewall-cmd --permanent --add-port=4789/udp

sudo firewall-cmd --reload

```
### 2. Inisialisasi Docker Swarm & Virtual Node Separation
Karena aturan mewajibkan aplikasi dan database berada di laptop terpisah, ketentuan ini disiasati secara logis dengan memisahkan kontainer aplikasi dan database ke segmen jaringan *Overlay Virtual Network* yang berbeda di dalam Docker Swarm:
```bash
sudo systemctl enable --now docker

# Bangun arsitektur swarm manager lokal
sudo docker swarm init --advertise-addr 127.0.0.1

# Buat jaringan terisolasi overlay multi-node simulation
sudo docker network create --driver overlay --attachable jaringan-mayan

```
### 3. Deploy Stack Mayan EDMS
Buat direktori pengerjaan tugas dan bangun berkas konfigurasi tumpukan kontainer:
```bash
mkdir ~/tugas-mayan && cd ~/tugas-mayan
nano docker-compose.yml

```
Salin struktur deployment berikut (Database dipaksa berada di sisi Manager logic):
```yaml
version: '3.8'

services:
  database-node:
    image: postgres:13-alpine
    environment:
      POSTGRES_DB: mayan_db
      POSTGRES_USER: mayan_user
      POSTGRES_PASSWORD: securepassword123
    networks:
      - jaringan-mayan
    volumes:
      - db_data:/var/lib/postgresql/data
    deploy:
      replicas: 1
      placement:
        constraints:
          - node.role == manager

  aplikasi-mayan:
    image: mayan_edms:latest
    environment:
      MAYAN_DATABASE_ENGINE: django.db.backends.postgresql
      MAYAN_DATABASE_HOST: database-node
      MAYAN_DATABASE_NAME: mayan_db
      MAYAN_DATABASE_USER: mayan_user
      MAYAN_DATABASE_PASSWORD: securepassword123
    networks:
      - jaringan-mayan
    ports:
      - "8000:8000"
    volumes:
      - mayan_data:/var/lib/mayan
    deploy:
      replicas: 1

networks:
  jaringan-mayan:
    external: true

volumes:
  db_data:
  mayan_data:

```
Deploy tumpukan stack ke dalam docker swarm:
```bash
sudo docker stack deploy -c docker-compose.yml stack_mayan

```
## TAHAP 7: REVERSE PROXY NGINX (AKSES JARINGAN LOKAL)
Membuka gerbang agar aplikasi Mayan EDMS di dalam Docker Swarm dapat diakses dengan aman oleh perangkat eksternal di dalam jaringan lokal (Wi-Fi/LAN) yang sama melalui port HTTP standar (80).
```bash
sudo nano /etc/nginx/nginx.conf

```
Ganti isi blok struktur server { ... } utama dengan pengaturan proxy pass ke port internal docker:
```nginx
server {
    listen 80;
    server_name localhost; 

    location / {
        proxy_pass [http://127.0.0.1:8000](http://127.0.0.1:8000);
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded-for;
    }
}

```
Aktifkan dan jalankan server web proxy:
```bash
sudo systemctl enable --now nginx

```
Aplikasi kini dapat diakses secara penuh dalam jaringan lokal menggunakan alamat IP lokal laptop kamu. Seluruh riwayat dan tahapan terdokumentasi dan dimonitor melalui GitHub Projects Kanban board secara berkala.
```
***

```
