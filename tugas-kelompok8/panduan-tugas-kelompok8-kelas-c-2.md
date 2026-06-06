# Panduan Lengkap Tugas Kelompok 8 — Kelas C
> Instalasi Server Arteri + Nginx + PHP-FPM + CIS Hardening + LUKS on LVM
> OS: Amanda (Arch-based) | Kernel: linux-hardened | Firewall: Firewalld

---

## ⚠️ PENTING SEBELUM MULAI

- Rekam **SEMUA percobaan** dengan asciinema — termasuk yang gagal
- Pahami setiap perintah, jangan hanya copy-paste — dosen akan menguji secara acak
- **Deadline: Senin 8 Juni 2026 jam 23.59 WIB**
- Tidak disarankan menunggu hasil presentator — mulai mandiri sekarang

---

## FASE 0: PERSIAPAN

### Download & Flash ISO Amanda

1. Download ISO Amanda dari https://distro.yuros.org
2. Flash ke USB dengan Rufus (Windows) atau `dd` (Linux):

```bash
# Flash dari Linux — ganti /dev/sdX dengan nama USB kamu
dd if=amanda.iso of=/dev/sdX bs=4M status=progress oflag=sync
```

3. Boot dari USB, pilih mode **native / CLI**

---

## FASE 1: PARTISI — LUKS ON LVM (Disk Encryption)

### Konsep LUKS on LVM

```
Disk Fisik (/dev/sda)
 ├── /dev/sda1  → /boot (tidak dienkripsi, biar bisa boot)
 └── /dev/sda2  → LUKS Container (dienkripsi)
                   └── LVM PV → VG → LV-LV (partisi logis)
                        ├── lv_root   → /
                        ├── lv_tmp    → /tmp
                        ├── lv_var    → /var
                        ├── lv_log    → /var/log
                        ├── lv_audit  → /var/log/audit
                        └── lv_home   → /home
```

### Langkah 1: Partisi Disk Fisik

```bash
# Buka tool partisi
fdisk /dev/sda

# Di dalam fdisk:
# g       → buat GPT partition table baru
# n       → partisi baru
# enter   → partition number 1
# enter   → first sector default
# +512M   → ukuran 512MB untuk /boot
# t       → ubah tipe
# 1       → pilih EFI System (untuk boot EFI)

# n       → partisi baru lagi
# enter   → partition number 2
# enter   → first sector default
# enter   → pakai sisa disk semua
# (tipe default Linux filesystem sudah OK)

# w       → write dan keluar

# Verifikasi
fdisk -l /dev/sda
```

### Langkah 2: Format Partisi Boot

```bash
# Format /boot sebagai FAT32 (untuk EFI)
mkfs.fat -F32 /dev/sda1
```

### Langkah 3: Setup LUKS Encryption

```bash
# Enkripsi partisi sda2 dengan LUKS
# Kamu akan diminta masukkan passphrase — INGAT BAIK-BAIK, tidak bisa dipulihkan!
cryptsetup luksFormat /dev/sda2

# Buka (unlock) container LUKS — beri nama "cryptlvm"
cryptsetup open /dev/sda2 cryptlvm

# Verifikasi sudah terbuka
ls /dev/mapper/
# Harus ada: cryptlvm
```

### Langkah 4: Setup LVM di dalam LUKS

```bash
# Buat Physical Volume (PV) di dalam container LUKS
pvcreate /dev/mapper/cryptlvm

# Buat Volume Group (VG) bernama "vg0"
vgcreate vg0 /dev/mapper/cryptlvm

# Buat Logical Volumes (LV) sesuai layout CIS
lvcreate -L 20G   vg0 -n lv_root     # /
lvcreate -L 2G    vg0 -n lv_tmp      # /tmp
lvcreate -L 5G    vg0 -n lv_var      # /var
lvcreate -L 3G    vg0 -n lv_log      # /var/log
lvcreate -L 2G    vg0 -n lv_audit    # /var/log/audit
lvcreate -l 100%FREE vg0 -n lv_home  # /home (sisa semua)

# Verifikasi LV berhasil dibuat
lvdisplay
```

### Langkah 5: Format Logical Volumes

```bash
mkfs.ext4 /dev/vg0/lv_root
mkfs.ext4 /dev/vg0/lv_tmp
mkfs.ext4 /dev/vg0/lv_var
mkfs.ext4 /dev/vg0/lv_log
mkfs.ext4 /dev/vg0/lv_audit
mkfs.ext4 /dev/vg0/lv_home
```

### Langkah 6: Mount Semua Partisi

```bash
# Mount root dulu
mount /dev/vg0/lv_root /mnt

# Buat direktori mount
mkdir -p /mnt/{boot,tmp,var,home}
mkdir -p /mnt/var/log/audit

# Mount semuanya
mount /dev/sda1            /mnt/boot
mount /dev/vg0/lv_tmp      /mnt/tmp
mount /dev/vg0/lv_var      /mnt/var
mount /dev/vg0/lv_log      /mnt/var/log
mount /dev/vg0/lv_audit    /mnt/var/log/audit
mount /dev/vg0/lv_home     /mnt/home

# Verifikasi semua sudah ter-mount
lsblk
```

---

## FASE 2: INSTALASI BASE SYSTEM

### Langkah 7: Install Base System

```bash
# Install base system Amanda/Arch dengan kernel linux-hardened
pacstrap /mnt base linux-hardened linux-hardened-headers linux-firmware lvm2

# Generate fstab otomatis berdasarkan mount saat ini
genfstab -U /mnt >> /mnt/etc/fstab

# Cek fstab — pastikan semua partisi ada
cat /mnt/etc/fstab
```

### Langkah 8: Masuk ke Sistem Baru

```bash
arch-chroot /mnt
```

### Langkah 9: Konfigurasi Dasar

```bash
# Timezone
ln -sf /usr/share/zoneinfo/Asia/Jakarta /etc/localtime
hwclock --systohc

# Locale
echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen
echo "id_ID.UTF-8 UTF-8" >> /etc/locale.gen
locale-gen
echo "LANG=en_US.UTF-8" > /etc/locale.conf

# Hostname
echo "library-server" > /etc/hostname

# Hosts file
cat > /etc/hosts << 'EOF'
127.0.0.1   localhost
::1         localhost
127.0.1.1   library-server.localdomain library-server
EOF

# Set password root
passwd

# Buat user non-root
useradd -m -G wheel -s /bin/bash namauser
passwd namauser

# Aktifkan sudo untuk grup wheel
EDITOR=nano visudo
# Uncomment baris: %wheel ALL=(ALL:ALL) ALL
```

### Langkah 10: Konfigurasi mkinitcpio untuk LUKS + LVM

Ini **sangat penting** — tanpa ini sistem tidak bisa boot karena tidak tahu cara buka enkripsi.

```bash
nano /etc/mkinitcpio.conf
```

Cari baris HOOKS dan ubah menjadi:
```
HOOKS=(base udev autodetect keyboard keymap consolefont modconf block encrypt lvm2 filesystems fsck)
```

**Penjelasan hook:** `encrypt` untuk buka LUKS saat boot, `lvm2` untuk deteksi LVM di dalamnya.

```bash
# Regenerate initramfs
mkinitcpio -P
```

### Langkah 11: Install Bootloader

```bash
pacman -S grub efibootmgr

# Install GRUB ke EFI
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
```

Edit konfigurasi GRUB untuk menyertakan parameter LUKS:

```bash
nano /etc/default/grub
```

Cari baris `GRUB_CMDLINE_LINUX` dan ubah:
```
GRUB_CMDLINE_LINUX="cryptdevice=UUID=XXXX:cryptlvm root=/dev/vg0/lv_root"
```

Ganti `XXXX` dengan UUID dari `/dev/sda2`:
```bash
# Cek UUID /dev/sda2
blkid /dev/sda2
# Salin UUID yang muncul, lalu paste ke baris GRUB_CMDLINE_LINUX
```

```bash
# Generate grub config
grub-mkconfig -o /boot/grub/grub.cfg
```

---

## FASE 3: INSTALL PACKAGE & PERSIAPAN SERVER

### Langkah 12: Install Package yang Dibutuhkan

```bash
# CATATAN: Dilarang menggunakan package meta base-devel
# Install satu per satu sesuai kebutuhan

pacman -S nginx php-fpm php php-gd php-intl mariadb \
          firewalld openssh git wget curl audit unzip
```

### Langkah 13: Aktifkan Service Dasar

```bash
systemctl enable sshd
systemctl enable firewalld
systemctl enable auditd
```

---

## FASE 4: INSTALL APLIKASI ARTERI

Arteri adalah aplikasi pengelolaan arsip elektronik berbasis web, dibangun dengan PHP dan CodeIgniter.

### Langkah 14: Setup Database MariaDB

```bash
# Inisialisasi database
mariadb-install-db --user=mysql --basedir=/usr --datadir=/var/lib/mysql

systemctl enable mariadb
systemctl start mariadb

# Amankan instalasi
mariadb-secure-installation
# set root password = Y
# remove anonymous users = Y
# disallow root login remotely = Y
# remove test database = Y
# reload privilege tables = Y

# Buat database untuk Arteri
mariadb -u root -p
```

Di dalam prompt MariaDB:
```sql
CREATE DATABASE arteri_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'arteri_user'@'localhost' IDENTIFIED BY 'password_kuat_disini';
GRANT ALL PRIVILEGES ON arteri_db.* TO 'arteri_user'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

### Langkah 15: Download & Install Arteri

Source code Arteri dapat diunduh dari http://arteri.sainsinformasi.org. Setelah diunduh, extract file zip ke dalam folder webserver, lalu import file `arteri.sql` ke database yang sudah dibuat, kemudian edit file `application/config/database.php` sesuai konfigurasi database.

```bash
# Buat direktori web
mkdir -p /srv/http/arteri
cd /srv/http/arteri

# Download Arteri (dari releases github)
wget https://github.com/dicarve/arteri/releases/download/1.2.5/arteri_web.zip

# Extract
unzip arteri_web.zip
# Jika hasil extract ada subfolder, pindahkan isinya
# mv arteri_web/* . 2>/dev/null || true

# Import database Arteri
mariadb -u arteri_user -p arteri_db < sql/arteri.sql

# Edit konfigurasi database Arteri
nano application/config/database.php
```

Ubah bagian ini di `database.php`:
```php
$db['default'] = array(
    'hostname' => 'localhost',
    'username' => 'arteri_user',
    'password' => 'password_kuat_disini',  // sesuai yang dibuat tadi
    'database' => 'arteri_db',
    'dbdriver' => 'mysqli',
    // ...
);
```

```bash
# Set permission yang benar
chown -R http:http /srv/http/arteri
find /srv/http/arteri -type f -exec chmod 644 {} \;
find /srv/http/arteri -type d -exec chmod 755 {} \;

# Folder files harus writable (untuk upload)
chmod -R 775 /srv/http/arteri/files
```

---

## FASE 5: KONFIGURASI NGINX + PHP-FPM

### Langkah 16: Konfigurasi PHP

```bash
nano /etc/php/php.ini
```

Cari dan ubah baris berikut:
```ini
; Aktifkan ekstensi yang dibutuhkan Arteri (CodeIgniter)
extension=gd
extension=intl
extension=mysqli
extension=pdo_mysql

; Hardening PHP (CIS)
expose_php = Off
display_errors = Off
log_errors = On
error_log = /var/log/php_errors.log
allow_url_fopen = Off
allow_url_include = Off
session.cookie_httponly = 1
session.cookie_secure = 0
```

### Langkah 17: Konfigurasi PHP-FPM

```bash
nano /etc/php/php-fpm.d/www.conf
```

Pastikan baris berikut ada/sesuai:
```ini
user = http
group = http
listen = /run/php-fpm/php-fpm.sock
listen.owner = http
listen.group = http
listen.mode = 0660
pm = dynamic
pm.max_children = 5
pm.start_servers = 2
pm.min_spare_servers = 1
pm.max_spare_servers = 3
```

```bash
systemctl enable php-fpm
systemctl start php-fpm
```

### Langkah 18: Konfigurasi Nginx

```bash
# Buat file konfigurasi site Arteri
mkdir -p /etc/nginx/sites-available /etc/nginx/sites-enabled

nano /etc/nginx/sites-available/arteri.conf
```

```nginx
server {
    listen 80;
    server_name localhost;
    root /srv/http/arteri;
    index index.php index.html;

    # Sembunyikan versi nginx
    server_tokens off;

    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header X-Content-Type-Options "nosniff" always;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        fastcgi_pass unix:/run/php-fpm/php-fpm.sock;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }

    # Blokir akses ke file sensitif
    location ~ /\.(ht|git|env) {
        deny all;
    }

    access_log /var/log/nginx/arteri_access.log;
    error_log /var/log/nginx/arteri_error.log;
}
```

```bash
# Edit nginx.conf utama — tambahkan include
nano /etc/nginx/nginx.conf
```

Di dalam block `http { }` tambahkan:
```nginx
server_tokens off;
include /etc/nginx/sites-enabled/*;
```

```bash
# Aktifkan site Arteri
ln -s /etc/nginx/sites-available/arteri.conf /etc/nginx/sites-enabled/

# Test konfigurasi
nginx -t

# Enable dan start
systemctl enable nginx
systemctl start nginx
```

---

## FASE 6: CIS HARDENING — KERNEL MODULES

### Langkah 19: Disable Kernel Modules yang Tidak Dibutuhkan

**Penjelasan:** Modul kernel yang tidak dipakai bisa dijadikan attack vector. CIS mewajibkan menonaktifkan filesystem dan protokol jaringan yang tidak diperlukan server.

```bash
nano /etc/modprobe.d/CIS.conf
```

```
# ============================================================
# CIS — Disable unused filesystem kernel modules
# ============================================================
install cramfs /bin/true
install freevxfs /bin/true
install jffs2 /bin/true
install hfs /bin/true
install hfsplus /bin/true
install squashfs /bin/true
install udf /bin/true
install vfat /bin/true

# ============================================================
# CIS — Disable unused network protocol modules
# ============================================================
install dccp /bin/true
install sctp /bin/true
install rds /bin/true
install tipc /bin/true

# ============================================================
# CIS — Disable unused hardware modules
# ============================================================
install usb-storage /bin/true
install firewire-core /bin/true
```

```bash
# Verifikasi modul sudah diblokir
modprobe -n -v cramfs
# Output yang benar: "install /bin/true"

modprobe -n -v dccp
# Output yang benar: "install /bin/true"
```

---

## FASE 7: CIS HARDENING — KERNEL PARAMETERS (SYSCTL)

### Langkah 20: Konfigurasi Sysctl

**Penjelasan:** Sysctl mengatur parameter kernel. Ini adalah inti dari kernel hardening — mengamankan perilaku jaringan, memori, dan proses di level kernel.

```bash
nano /etc/sysctl.d/99-cis-hardening.conf
```

```ini
# ============================================================
# CIS KERNEL HARDENING — sysctl parameters
# Referensi: CIS Distribution Independent Linux Benchmark v1.1.0
# ============================================================

# --- FILESYSTEM ---
# Cegah core dump dari program SUID (mencegah bocornya data sensitif)
fs.suid_dumpable = 0

# --- KERNEL ---
# ASLR: Acak alamat memori, mencegah exploit buffer overflow
kernel.randomize_va_space = 2

# Batasi akses ke dmesg hanya untuk root
kernel.dmesg_restrict = 1

# Sembunyikan pointer kernel dari user biasa
kernel.kptr_restrict = 2

# Matikan SysRq key (mencegah aksi berbahaya langsung ke kernel)
kernel.sysrq = 0

# --- NETWORK IPv4 ---
# Nonaktifkan IP forwarding (server bukan router)
net.ipv4.ip_forward = 0

# Nonaktifkan pengiriman redirect packet
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.default.send_redirects = 0

# Tolak source routed packets
net.ipv4.conf.all.accept_source_route = 0
net.ipv4.conf.default.accept_source_route = 0

# Tolak ICMP redirect
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.default.accept_redirects = 0

# Tolak secure ICMP redirect
net.ipv4.conf.all.secure_redirects = 0
net.ipv4.conf.default.secure_redirects = 0

# Log paket mencurigakan (martian packets)
net.ipv4.conf.all.log_martians = 1
net.ipv4.conf.default.log_martians = 1

# Abaikan broadcast ICMP ping (mencegah Smurf attack)
net.ipv4.icmp_echo_ignore_broadcasts = 1

# Abaikan ICMP error palsu
net.ipv4.icmp_ignore_bogus_error_responses = 1

# Aktifkan Reverse Path Filtering (mencegah IP spoofing)
net.ipv4.conf.all.rp_filter = 1
net.ipv4.conf.default.rp_filter = 1

# Aktifkan TCP SYN cookies (mencegah SYN flood attack)
net.ipv4.tcp_syncookies = 1

# --- NETWORK IPv6 ---
# Tolak Router Advertisement IPv6
net.ipv6.conf.all.accept_ra = 0
net.ipv6.conf.default.accept_ra = 0

# Tolak IPv6 redirect
net.ipv6.conf.all.accept_redirects = 0
net.ipv6.conf.default.accept_redirects = 0
```

```bash
# Terapkan semua pengaturan langsung tanpa reboot
sysctl -p /etc/sysctl.d/99-cis-hardening.conf

# Verifikasi beberapa parameter
sysctl kernel.randomize_va_space   # harus = 2
sysctl net.ipv4.ip_forward         # harus = 0
sysctl net.ipv4.tcp_syncookies     # harus = 1
```

---

## FASE 8: CIS HARDENING — SSH

### Langkah 21: Hardening SSH

**Penjelasan:** SSH adalah pintu masuk utama ke server. Harus dikonfigurasi ketat untuk mencegah akses tidak sah.

```bash
# Backup konfigurasi asli
cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak

# Set permission ketat pada file config SSH
chown root:root /etc/ssh/sshd_config
chmod 600 /etc/ssh/sshd_config

nano /etc/ssh/sshd_config
```

Tambahkan/ubah baris-baris ini:
```
# Hanya SSH Protocol 2 (versi 1 rentan)
Protocol 2

# Logging level INFO
LogLevel INFO

# Larang login langsung sebagai root
PermitRootLogin no

# Maksimal 4 kali percobaan login
MaxAuthTries 4

# Timeout session idle 5 menit
ClientAliveInterval 300
ClientAliveCountMax 0

# Waktu grace login: 60 detik
LoginGraceTime 60

# Nonaktifkan rhosts authentication
IgnoreRhosts yes

# Nonaktifkan host-based authentication
HostbasedAuthentication no

# Larang login dengan password kosong
PermitEmptyPasswords no

# Larang user set environment variable
PermitUserEnvironment no

# Nonaktifkan X11 forwarding (server tidak butuh GUI)
X11Forwarding no

# Banner peringatan sebelum login
Banner /etc/issue.net

# Algoritma MAC yang aman
MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com,hmac-sha2-512,hmac-sha2-256
```

```bash
# Buat banner peringatan
cat > /etc/issue.net << 'EOF'
*************************************************************
*  AUTHORIZED ACCESS ONLY                                   *
*  Akses tidak sah dilarang dan dapat dikenai sanksi hukum  *
*************************************************************
EOF

# Restart SSH
systemctl restart sshd

# Verifikasi
grep "^PermitRootLogin\|^MaxAuthTries\|^X11Forwarding\|^PermitEmptyPasswords" /etc/ssh/sshd_config
```

---

## FASE 9: KONFIGURASI FIREWALLD

### Langkah 22: Setup Firewalld

**Penjelasan:** Firewalld mengontrol lalu lintas jaringan. Hanya layanan yang benar-benar dibutuhkan yang diizinkan masuk.

```bash
systemctl enable firewalld
systemctl start firewalld

# Cek status
firewall-cmd --state

# Lihat konfigurasi saat ini
firewall-cmd --list-all

# Set default zone
firewall-cmd --set-default-zone=public

# Hapus service yang tidak dibutuhkan
firewall-cmd --permanent --remove-service=dhcpv6-client
firewall-cmd --permanent --remove-service=cockpit

# Izinkan hanya layanan yang dibutuhkan:
# HTTP untuk Arteri
firewall-cmd --permanent --add-service=http

# SSH untuk administrasi
firewall-cmd --permanent --add-service=ssh

# Pastikan loopback interface di zone trusted
firewall-cmd --permanent --zone=trusted --add-interface=lo

# Terapkan semua perubahan
firewall-cmd --reload

# Verifikasi hasil akhir
firewall-cmd --list-all
```

Output yang diharapkan:
```
public (active)
  target: default
  interfaces: eth0
  services: http ssh
  ...
```

---

## FASE 10: AUDIT & LOGGING

### Langkah 23: Setup Auditd

**Penjelasan:** Auditd mencatat semua aktivitas penting di sistem, berguna untuk forensik jika terjadi insiden keamanan.

```bash
systemctl enable auditd
systemctl start auditd

nano /etc/audit/audit.rules
```

```
# Hapus semua rules yang ada
-D

# Buffer size
-b 8192

# Monitor perubahan file identitas user/group
-w /etc/group -p wa -k identity
-w /etc/passwd -p wa -k identity
-w /etc/shadow -p wa -k identity
-w /etc/security/opasswd -p wa -k identity

# Monitor penggunaan perintah kernel module
-w /sbin/insmod -p x -k modules
-w /sbin/rmmod -p x -k modules
-w /sbin/modprobe -p x -k modules

# Monitor login
-w /var/log/lastlog -p wa -k logins
-w /var/run/faillock/ -p wa -k logins

# Monitor perubahan network config
-w /etc/hosts -p wa -k system-locale
-w /etc/issue -p wa -k system-locale

# Immutable: kunci rules (butuh reboot untuk ubah)
-e 2
```

```bash
systemctl restart auditd

# Verifikasi rules aktif
auditctl -l
```

---

## FASE 11: MOUNT OPTIONS HARDENING

### Langkah 24: Hardening Opsi Mount di fstab

**Penjelasan:** Opsi `nodev`, `nosuid`, `noexec` di fstab mencegah eksploitasi melalui partisi seperti /tmp yang sering dijadikan target.

```bash
nano /etc/fstab
```

Pastikan baris-baris partisi punya opsi keamanan:
```
# /tmp — tidak boleh ada device, suid, atau eksekusi
UUID=xxx  /tmp           ext4  defaults,nodev,nosuid,noexec  0 2

# /var/log — tidak boleh ada device atau suid
UUID=xxx  /var/log       ext4  defaults,nodev,nosuid         0 2

# /var/log/audit
UUID=xxx  /var/log/audit ext4  defaults,nodev,nosuid         0 2

# /home — tidak boleh ada device
UUID=xxx  /home          ext4  defaults,nodev                0 2

# /dev/shm — tmpfs hardened
tmpfs     /dev/shm       tmpfs defaults,nodev,nosuid,noexec  0 0
```

```bash
# Remount tanpa reboot untuk langsung aktif
mount -o remount /tmp
mount -o remount /dev/shm

# Verifikasi opsi sudah aktif
mount | grep "/tmp\|/dev/shm"
```

---

## FASE 12: ASCIINEMA — PEREKAMAN

### Langkah 25: Cara Rekam dengan Asciinema

Amanda sudah include asciinema bawaan, jadi langsung bisa dipakai.

```bash
# Mulai rekaman — beri nama sesuai tahap
asciinema rec 01-partisi-luks-lvm.cast

# ... kerjakan semua perintah di tahap itu ...

# Hentikan rekaman
exit
# atau Ctrl+D
```

**Rekomendasi penamaan file cast:**
```
01-persiapan-disk.cast
02-luks-setup.cast
03-lvm-setup.cast
04-install-base.cast
05-mkinitcpio-grub.cast
06-konfigurasi-sistem.cast
07-install-nginx-phpfpm.cast
08-install-mariadb.cast
09-install-arteri.cast
10-cis-kernel-modules.cast
11-cis-sysctl.cast
12-cis-ssh.cast
13-firewalld.cast
14-auditd.cast
15-verifikasi-akhir.cast
```

```bash
# Putar ulang rekaman untuk cek
asciinema play 01-partisi-luks-lvm.cast

# List semua file cast
ls -lh *.cast
```

---

## VERIFIKASI AKHIR — CHECKLIST

Jalankan semua ini untuk memastikan konfigurasi benar sebelum dikumpulkan:

```bash
echo "=== 1. LUKS & LVM ==="
lsblk -f
lvdisplay | grep "LV Name\|LV Size"

echo "=== 2. KERNEL ==="
uname -r
# Harus menampilkan: linux-hardened

echo "=== 3. NGINX ==="
systemctl status nginx
nginx -t

echo "=== 4. PHP-FPM ==="
systemctl status php-fpm
php -v

echo "=== 5. MARIADB ==="
systemctl status mariadb

echo "=== 6. ARTERI ==="
curl -s http://localhost | head -20

echo "=== 7. FIREWALLD ==="
firewall-cmd --list-all

echo "=== 8. KERNEL MODULES DISABLED ==="
modprobe -n -v cramfs
modprobe -n -v dccp

echo "=== 9. SYSCTL ==="
sysctl kernel.randomize_va_space
sysctl net.ipv4.ip_forward
sysctl net.ipv4.tcp_syncookies

echo "=== 10. SSH ==="
grep "^PermitRootLogin\|^MaxAuthTries\|^X11Forwarding" /etc/ssh/sshd_config

echo "=== 11. MOUNT OPTIONS ==="
mount | grep -E " /tmp | /home | /dev/shm "

echo "=== 12. AUDITD ==="
systemctl status auditd
auditctl -l | head -10
```

---

## REFERENSI

- Arteri: https://github.com/dicarve/arteri
- CIS Distribution Independent Linux Benchmark v1.1.0
- CIS AlmaLinux OS 10 Benchmark v1.0.0
- Amanda OS: https://distro.yuros.org

