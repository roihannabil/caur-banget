# Panduan Lengkap Tugas Kelompok 8 — Kelas C
> Instalasi Server Arteri + Nginx + PHP-FPM + CIS Hardening  
> OS: Amanda (Arch-based) | Kernel: linux-hardened | Firewall: Firewalld

---

## PERSIAPAN AWAL

### 1. Download & Flash OS Amanda

1. Download ISO Amanda dari https://distro.yuros.org
2. Flash ke USB menggunakan Rufus (Windows) atau `dd` (Linux):
```bash
# Jika flash dari Linux (ganti /dev/sdX dengan device USB kamu)
dd if=amanda.iso of=/dev/sdX bs=4M status=progress oflag=sync
```
3. Boot dari USB, pilih mode **native/CLI** (bukan GUI installer)

---

## FASE 1: INSTALASI DASAR

### 2. Persiapan Disk — Layout CIS

CIS merekomendasikan partisi terpisah untuk direktori-direktori kritis. Gunakan `cfdisk` atau `fdisk`.

**Layout partisi yang direkomendasikan:**

| Partisi | Mount Point | Ukuran | Tipe |
|---------|-------------|--------|------|
| /dev/sda1 | /boot | 512MB | EFI System |
| /dev/sda2 | / (root) | 20GB | Linux filesystem |
| /dev/sda3 | /tmp | 2GB | Linux filesystem |
| /dev/sda4 | /var | 5GB | Linux filesystem |
| /dev/sda5 | /var/log | 3GB | Linux filesystem |
| /dev/sda6 | /var/log/audit | 2GB | Linux filesystem |
| /dev/sda7 | /home | Sisa | Linux filesystem |

```bash
# Buka tool partisi
cfdisk /dev/sda

# Format setiap partisi
mkfs.fat -F32 /dev/sda1           # boot EFI
mkfs.ext4 /dev/sda2               # root
mkfs.ext4 /dev/sda3               # /tmp
mkfs.ext4 /dev/sda4               # /var
mkfs.ext4 /dev/sda5               # /var/log
mkfs.ext4 /dev/sda6               # /var/log/audit
mkfs.ext4 /dev/sda7               # /home

# Mount partisi
mount /dev/sda2 /mnt
mkdir -p /mnt/{boot,tmp,var,home}
mkdir -p /mnt/var/{log}
mkdir -p /mnt/var/log/audit
mount /dev/sda1 /mnt/boot
mount /dev/sda3 /mnt/tmp
mount /dev/sda4 /mnt/var
mount /dev/sda5 /mnt/var/log
mount /dev/sda6 /mnt/var/log/audit
mount /dev/sda7 /mnt/home
```

### 3. Instalasi Base System

```bash
# Install base system Amanda/Arch
pacstrap /mnt base base-devel linux-hardened linux-hardened-headers linux-firmware

# Generate fstab
genfstab -U /mnt >> /mnt/etc/fstab

# Masuk ke sistem baru
arch-chroot /mnt
```

### 4. Konfigurasi Dasar Sistem

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

### 5. Install Bootloader

```bash
# Install bootloader (gunakan grub atau systemd-boot)
pacman -S grub efibootmgr

grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
grub-mkconfig -o /boot/grub/grub.cfg
```

### 6. Install Package yang Dibutuhkan

```bash
# CATATAN: Dilarang menggunakan package meta base-devel
# Install satu per satu jika ada yang dibutuhkan

pacman -S nginx php-fpm php php-gd php-intl php-mysqli \
          mariadb firewalld openssh git wget curl \
          audit aide
```

---

## FASE 2: SETUP WEBSERVER — NGINX + PHP-FPM

### 7. Konfigurasi PHP-FPM

```bash
# Edit konfigurasi PHP
nano /etc/php/php.ini
```

Ubah/cari baris berikut:
```ini
; Aktifkan ekstensi yang dibutuhkan Arteri
extension=gd
extension=intl
extension=mysqli
extension=pdo_mysql

; Hardening PHP
expose_php = Off
display_errors = Off
log_errors = On
error_log = /var/log/php_errors.log
allow_url_fopen = Off
allow_url_include = Off
session.cookie_httponly = 1
session.cookie_secure = 1
```

```bash
# Konfigurasi PHP-FPM pool
nano /etc/php/php-fpm.d/www.conf
```

Pastikan baris ini ada:
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
# Enable dan start PHP-FPM
systemctl enable php-fpm
systemctl start php-fpm
```

### 8. Konfigurasi Nginx

```bash
# Buat direktori untuk aplikasi Arteri
mkdir -p /srv/http/arteri
chown -R http:http /srv/http/arteri
chmod -R 755 /srv/http/arteri

# Buat konfigurasi nginx untuk Arteri
nano /etc/nginx/sites-available/arteri.conf
```

```nginx
server {
    listen 80;
    server_name localhost;
    root /srv/http/arteri;
    index index.php index.html;

    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header Referrer-Policy "no-referrer-when-downgrade" always;

    # Sembunyikan versi nginx
    server_tokens off;

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

    # Log
    access_log /var/log/nginx/arteri_access.log;
    error_log /var/log/nginx/arteri_error.log;
}
```

```bash
# Edit nginx.conf utama
nano /etc/nginx/nginx.conf
```

Tambahkan di dalam block `http {}`:
```nginx
# Matikan versi di header
server_tokens off;

# Include sites
include /etc/nginx/sites-enabled/*;
```

```bash
# Aktifkan site
mkdir -p /etc/nginx/sites-enabled
ln -s /etc/nginx/sites-available/arteri.conf /etc/nginx/sites-enabled/

# Test konfigurasi
nginx -t

# Enable dan start nginx
systemctl enable nginx
systemctl start nginx
```

### 9. Setup Database MariaDB

```bash
# Inisialisasi database
mariadb-install-db --user=mysql --basedir=/usr --datadir=/var/lib/mysql

# Enable dan start MariaDB
systemctl enable mariadb
systemctl start mariadb

# Amankan instalasi MariaDB
mariadb-secure-installation
# Jawab: set root password = Y, remove anon users = Y, 
#        disallow root remote = Y, remove test db = Y, reload = Y

# Buat database untuk Arteri
mariadb -u root -p
```

Di dalam MariaDB prompt:
```sql
CREATE DATABASE arteri_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'arteri_user'@'localhost' IDENTIFIED BY 'password_kuat_disini';
GRANT ALL PRIVILEGES ON arteri_db.* TO 'arteri_user'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

### 10. Install Aplikasi Arteri

```bash
# Clone atau download Arteri ke direktori web
cd /srv/http/arteri
# (Sesuaikan dengan cara download Arteri yang diberikan dosen)
# Contoh jika dari git:
git clone https://url-repo-arteri.git .

# Set permission
chown -R http:http /srv/http/arteri
find /srv/http/arteri -type f -exec chmod 644 {} \;
find /srv/http/arteri -type d -exec chmod 755 {} \;
```

---

## FASE 3: CIS HARDENING — KERNEL MODULES

### 11. Disable Filesystem Kernel Modules yang Tidak Dibutuhkan

**Penjelasan:** Modul kernel yang tidak digunakan membuka potensi attack surface. CIS merekomendasikan menonaktifkan filesystem yang tidak diperlukan.

```bash
# Buat file konfigurasi CIS untuk modprobe
nano /etc/modprobe.d/CIS.conf
```

Isi file tersebut:
```
# CIS - Disable unused filesystem modules
install cramfs /bin/true
install freevxfs /bin/true
install jffs2 /bin/true
install hfs /bin/true
install hfsplus /bin/true
install squashfs /bin/true
install udf /bin/true
install vfat /bin/true

# CIS - Disable unused network protocol modules
install dccp /bin/true
install sctp /bin/true
install rds /bin/true
install tipc /bin/true

# CIS - Disable unused hardware modules
install usb-storage /bin/true
install firewire-core /bin/true
```

```bash
# Verifikasi modul sudah diblokir
modprobe -n -v cramfs
# Output yang benar: "install /bin/true"
```

---

## FASE 4: CIS HARDENING — KERNEL PARAMETERS (SYSCTL)

### 12. Konfigurasi Sysctl / Kernel Hardening

**Penjelasan:** Sysctl mengatur parameter kernel saat runtime. Pengaturan ini meningkatkan keamanan jaringan dan sistem.

```bash
nano /etc/sysctl.d/99-cis-hardening.conf
```

Isi lengkap file:
```ini
# ============================================================
# CIS KERNEL HARDENING — sysctl parameters
# ============================================================

# --- 1. Filesystem ---
# Cegah core dump dari program SUID (mencegah bocornya data sensitif)
fs.suid_dumpable = 0

# --- 2. Kernel ---
# ASLR: Acak alamat memori untuk mencegah exploit buffer overflow
kernel.randomize_va_space = 2

# Batasi akses ke dmesg (log kernel) hanya untuk root
kernel.dmesg_restrict = 1

# Sembunyikan pointer kernel dari user biasa
kernel.kptr_restrict = 2

# Matikan SysRq key (mencegah aksi berbahaya langsung ke kernel)
kernel.sysrq = 0

# --- 3. Network — IPv4 ---
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

# Log paket mencurigaan (martian packets)
net.ipv4.conf.all.log_martians = 1
net.ipv4.conf.default.log_martians = 1

# Abaikan broadcast ICMP ping (mencegah Smurf attack)
net.ipv4.icmp_echo_ignore_broadcasts = 1

# Abaikan ICMP error palsu
net.ipv4.icmp_ignore_bogus_error_responses = 1

# Aktifkan Reverse Path Filtering
net.ipv4.conf.all.rp_filter = 1
net.ipv4.conf.default.rp_filter = 1

# Aktifkan TCP SYN cookies (mencegah SYN flood attack)
net.ipv4.tcp_syncookies = 1

# --- 4. Network — IPv6 ---
# Tolak Router Advertisement IPv6
net.ipv6.conf.all.accept_ra = 0
net.ipv6.conf.default.accept_ra = 0

# Tolak IPv6 redirect
net.ipv6.conf.all.accept_redirects = 0
net.ipv6.conf.default.accept_redirects = 0
```

```bash
# Terapkan semua pengaturan sysctl tanpa reboot
sysctl -p /etc/sysctl.d/99-cis-hardening.conf

# Verifikasi salah satu parameter
sysctl kernel.randomize_va_space
# Output: kernel.randomize_va_space = 2
```

---

## FASE 5: CIS HARDENING — SSH

### 13. Konfigurasi SSH Hardening

**Penjelasan:** SSH harus dikonfigurasi ketat untuk mencegah akses tidak sah.

```bash
# Backup config asli dulu
cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak

# Set permission file config SSH
chown root:root /etc/ssh/sshd_config
chmod 600 /etc/ssh/sshd_config

# Edit konfigurasi SSH
nano /etc/ssh/sshd_config
```

Tambahkan/ubah baris berikut:
```
# Protocol versi 2 saja (versi 1 rentan)
Protocol 2

# Logging
LogLevel INFO

# Larang login langsung sebagai root
PermitRootLogin no

# Maksimal percobaan login: 4 kali
MaxAuthTries 4

# Timeout session idle: 5 menit
ClientAliveInterval 300
ClientAliveCountMax 0

# Waktu grace login: 1 menit
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

# Banner peringatan
Banner /etc/issue.net

# Algoritma MAC yang aman
MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com,hmac-sha2-512,hmac-sha2-256
```

```bash
# Buat banner peringatan
cat > /etc/issue.net << 'EOF'
*************************************************************
*  AUTHORIZED ACCESS ONLY                                   *
*  Unauthorized access is strictly prohibited and subject  *
*  to legal action.                                         *
*************************************************************
EOF

# Restart SSH
systemctl restart sshd

# Verifikasi
grep "^PermitRootLogin" /etc/ssh/sshd_config
grep "^MaxAuthTries" /etc/ssh/sshd_config
```

---

## FASE 6: KONFIGURASI FIREWALLD

### 14. Setup Firewalld

**Penjelasan:** Firewalld mengatur lalu lintas jaringan yang diizinkan masuk dan keluar server.

```bash
# Enable dan start firewalld
systemctl enable firewalld
systemctl start firewalld

# Cek status
firewall-cmd --state

# Lihat zone aktif
firewall-cmd --get-active-zones

# Set default zone ke public
firewall-cmd --set-default-zone=public

# Hapus service yang tidak dibutuhkan dari zone default
firewall-cmd --permanent --remove-service=dhcpv6-client
firewall-cmd --permanent --remove-service=cockpit

# Izinkan hanya layanan yang diperlukan:
# HTTP (port 80) untuk Arteri
firewall-cmd --permanent --add-service=http

# SSH (port 22) untuk administrasi
firewall-cmd --permanent --add-service=ssh

# Konfigurasikan loopback traffic (CIS requirement)
firewall-cmd --permanent --zone=trusted --add-interface=lo

# Terapkan semua perubahan
firewall-cmd --reload

# Verifikasi konfigurasi
firewall-cmd --list-all
```

Output yang diharapkan:
```
public (active)
  target: default
  interfaces: eth0
  services: http ssh
  ports:
  ...
```

---

## FASE 7: AUDIT & LOGGING

### 15. Setup Auditd

**Penjelasan:** Auditd merekam semua aktivitas penting di sistem untuk keperluan forensik dan monitoring.

```bash
# Install auditd jika belum ada
pacman -S audit

# Enable dan start
systemctl enable auditd
systemctl start auditd

# Tambahkan aturan audit CIS
nano /etc/audit/audit.rules
```

Tambahkan baris berikut:
```
# Hapus semua rules yang ada
-D

# Buffer size
-b 8192

# Monitor perubahan file user/group
-w /etc/group -p wa -k identity
-w /etc/passwd -p wa -k identity
-w /etc/shadow -p wa -k identity
-w /etc/security/opasswd -p wa -k identity

# Monitor penggunaan perintah berbahaya
-w /sbin/insmod -p x -k modules
-w /sbin/rmmod -p x -k modules
-w /sbin/modprobe -p x -k modules

# Monitor login
-w /var/log/lastlog -p wa -k logins
-w /var/run/faillock/ -p wa -k logins

# Monitor perubahan network
-w /etc/hosts -p wa -k system-locale
-w /etc/issue -p wa -k system-locale

# Immutable: lock rules (harus reboot untuk ubah)
-e 2
```

```bash
# Restart auditd
systemctl restart auditd

# Verifikasi rules aktif
auditctl -l
```

---

## FASE 8: KONFIGURASI TAMBAHAN CIS

### 16. Mount Options — Hardening Partisi

**Penjelasan:** Opsi mount `nodev`, `nosuid`, `noexec` mencegah eksploitasi melalui partisi yang tidak tepat.

```bash
# Edit fstab
nano /etc/fstab
```

Pastikan baris partisi /tmp, /var/tmp, /home, /dev/shm memiliki opsi keamanan:
```
# /tmp — tidak boleh ada device, suid, atau eksekusi
UUID=xxx  /tmp          ext4  defaults,nodev,nosuid,noexec  0 2

# /var/log — tidak boleh ada device atau suid
UUID=xxx  /var/log      ext4  defaults,nodev,nosuid         0 2

# /home — tidak boleh ada device
UUID=xxx  /home         ext4  defaults,nodev                0 2

# /dev/shm — tmpfs hardened
tmpfs     /dev/shm      tmpfs defaults,nodev,nosuid,noexec  0 0
```

```bash
# Remount tanpa reboot untuk test
mount -o remount /tmp
mount -o remount /dev/shm

# Verifikasi
mount | grep /tmp
# Output: ... nodev,nosuid,noexec ...
```

### 17. Disable Automounting

```bash
# Jika autofs terpasang, disable
systemctl disable autofs 2>/dev/null || echo "autofs tidak terpasang"
```

### 18. Core Dump Restriction

```bash
# Buat file limits untuk cegah core dump
echo "* hard core 0" >> /etc/security/limits.conf

# Verifikasi sysctl fs.suid_dumpable sudah 0 (dari langkah 12)
sysctl fs.suid_dumpable
```

---

## FASE 9: ASCIINEMA — PEREKAMAN

### 19. Cara Menggunakan Asciinema

**Penjelasan:** Asciinema merekam sesi terminal sebagai file `.cast` yang bisa diputar ulang.

```bash
# Mulai rekaman — nama file sesuai tahap yang dikerjakan
asciinema rec tahap1-instalasi-dasar.cast

# ... lakukan semua perintah instalasi di dalam sesi rekaman ...

# Hentikan rekaman
exit
# atau tekan Ctrl+D
```

**Tips penting:**
- Buat file cast **per tahap**, jangan satu file untuk semua
- Rekam **semua percobaan** termasuk yang gagal — dosen meminta semua file cast
- Contoh penamaan file:
  - `01-partisi.cast`
  - `02-install-base.cast`
  - `03-nginx-phpfpm.cast`
  - `04-cis-kernel-modules.cast`
  - `05-cis-sysctl.cast`
  - `06-cis-ssh.cast`
  - `07-firewalld.cast`
  - `08-audit.cast`
  - `09-install-arteri.cast`

```bash
# Putar ulang rekaman untuk cek
asciinema play namafile.cast

# List semua file cast
ls -la *.cast
```

---

## CHECKLIST VERIFIKASI AKHIR

Jalankan semua perintah ini untuk memastikan konfigurasi sudah benar:

```bash
echo "=== CEK NGINX ===" 
systemctl status nginx

echo "=== CEK PHP-FPM ===" 
systemctl status php-fpm

echo "=== CEK MARIADB ===" 
systemctl status mariadb

echo "=== CEK FIREWALLD ===" 
firewall-cmd --list-all

echo "=== CEK KERNEL MODULES DISABLED ===" 
modprobe -n -v cramfs
modprobe -n -v dccp

echo "=== CEK SYSCTL ===" 
sysctl kernel.randomize_va_space
sysctl net.ipv4.ip_forward
sysctl net.ipv4.tcp_syncookies

echo "=== CEK SSH ===" 
grep "^PermitRootLogin\|^MaxAuthTries\|^X11Forwarding" /etc/ssh/sshd_config

echo "=== CEK MOUNT OPTIONS ===" 
mount | grep -E "/tmp|/home|/dev/shm"

echo "=== CEK AUDITD ===" 
systemctl status auditd
auditctl -l | head -10

echo "=== CEK AKSES LOKAL ===" 
curl http://localhost
```

---

## CATATAN PENTING

1. **Rekam audio** penjelasan untuk setiap file `.cast` — ini wajib dikumpulkan
2. **GitHub Project** — minta admin kelas untuk buat project dengan model iteratif
3. **Deadline** — Senin 8 Juni 2026 jam 23.59 WIB
4. **Pahami setiap langkah** — dosen akan menguji secara acak di pertemuan berikutnya
5. **Jangan tunggu presentator** — mulai eksplorasi mandiri dari sekarang

---

*Referensi: CIS Distribution Independent Linux Benchmark v1.1.0 | CIS AlmaLinux OS 10 Benchmark v1.0.0*
