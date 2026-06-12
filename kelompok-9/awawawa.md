# Panduan Lengkap Instalasi Blackbird Amanda & Implementasi Tugas Praktikum Kelas C (Kelompok 9)

Panduan ini mencakup seluruh proses instalasi sistem operasi **Blackbird Amanda** pada partisi kosong 47 GB hingga penyelesaian seluruh **Tugas Praktikum Kelas C (Studi Kasus Kelompok 9)**.

Metode yang digunakan:

* CLI murni (tanpa GUI installer)
* Kernel **Linux-LTS**
* Tanpa paket `base-devel`
* Hardening mengikuti **CIS Linux Benchmark**
* Hardening mengikuti **CIS Docker Benchmark**
* Deployment **Mayan EDMS**
* **Docker Swarm**
* **Nginx Reverse Proxy**
* **Firewalld**

---

# 📐 Arsitektur Jaringan & Topologi

Alamat IP yang digunakan:

| Perangkat | Fungsi                                     | IP         |
| --------- | ------------------------------------------ | ---------- |
| Laptop A  | Aplikasi Mayan EDMS + Docker Swarm + Nginx | 11.11.11.2 |
| Laptop B  | PostgreSQL Database Server                 | 11.11.11.3 |

---

# FASE 1 — Instalasi OS Blackbird Amanda

## 1. Partisi Ruang Kosong 47 GB

Masuk ke lingkungan installer Amanda melalui flashdisk bootable yang dibuat menggunakan Rufus (Mode DD Image).

```bash
cfdisk /dev/nvme0n1
```

Langkah:

1. Pilih **Free Space 47G**
2. Pilih **New**
3. Gunakan seluruh ukuran
4. Pilih **Type**
5. Ubah menjadi **Linux LVM**
6. Pilih **Write**
7. Ketik:

```text
yes
```

8. Pilih **Quit**

---

## 2. Membuat Struktur LVM

Misalkan partisi baru menjadi:

```text
/dev/nvme0n1p6
```

Buat Physical Volume dan Volume Group:

```bash
pvcreate /dev/nvme0n1p6
vgcreate amanda /dev/nvme0n1p6
```

Buat Logical Volume:

```bash
lvcreate -L 20G amanda -n root
lvcreate -L 4G  amanda -n swap
lvcreate -L 5G  amanda -n tmp
lvcreate -L 8G  amanda -n var
lvcreate -l 100%FREE amanda -n home
```

---

## 3. Membuat Filesystem & Mounting

```bash
mkfs.ext4 /dev/amanda/root
mkfs.ext4 /dev/amanda/tmp
mkfs.ext4 /dev/amanda/var
mkfs.ext4 /dev/amanda/home

mkswap /dev/amanda/swap
```

Mounting:

```bash
mount /dev/amanda/root /mnt

mount --mkdir /dev/nvme0n1p1 /mnt/boot
mount --mkdir /dev/amanda/tmp /mnt/tmp
mount --mkdir /dev/amanda/var /mnt/var
mount --mkdir /dev/amanda/home /mnt/home

swapon /dev/amanda/swap
```

---

## 4. Pacstrap (Kernel LTS Tanpa base-devel)

```bash
pacstrap /mnt \
base \
linux-lts \
linux-lts-headers \
linux-firmware \
lvm2 \
networkmanager \
nano \
sudo \
grub \
efibootmgr \
git \
cmake \
make \
gcc \
binutils \
patch \
awk \
grep \
sed
```

Generate fstab:

```bash
genfstab -U /mnt >> /mnt/etc/fstab
```

Masuk ke chroot:

```bash
arch-chroot /mnt
```

---

## 5. Konfigurasi Dasar Sistem

### Timezone & Locale

```bash
ln -sf /usr/share/zoneinfo/Asia/Jakarta /etc/localtime

hwclock --systohc

echo "en_US.UTF-8 UTF-8" > /etc/locale.gen

locale-gen

echo "LANG=en_US.UTF-8" > /etc/locale.conf
```

Hostname:

```bash
echo "amanda-tugas" > /etc/hostname
```

### Membuat User

```bash
useradd -m -G wheel -s /bin/bash mahasiswa

passwd mahasiswa
passwd root
```

### Aktifkan sudo

```bash
EDITOR=nano visudo
```

Hilangkan tanda `#` pada:

```text
%wheel ALL=(ALL:ALL) ALL
```

### Aktifkan NetworkManager

```bash
systemctl enable NetworkManager
```

### Tambahkan Hook LVM

```bash
nano /etc/mkinitcpio.conf
```

Pastikan:

```text
HOOKS=(base udev autodetect modconf block lvm2 filesystems keyboard fsck)
```

Generate ulang initramfs:

```bash
mkinitcpio -P
```

---

## 6. Konfigurasi GRUB Internal

```bash
grub-mkconfig -o /boot/grub/grub.cfg

exit

umount -R /mnt

reboot
```

---

## 7. Aktifkan Triple Boot pada Arch Linux Utama

Masuk ke Arch Linux utama.

Edit:

```bash
sudo nano /etc/default/grub
```

Pastikan:

```text
GRUB_DISABLE_OS_PROBER=false
```

Generate ulang:

```bash
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

---

# FASE 2 — CIS Hardening Linux

## 1. Hardening fstab

Edit:

```bash
sudo nano /etc/fstab
```

Tambahkan:

```text
/dev/mapper/amanda-tmp /tmp ext4 rw,nosuid,nodev,noexec,relatime 0 2
/dev/mapper/amanda-var /var ext4 rw,nosuid,nodev,relatime 0 2
```

---

## 2. Blacklist Modul Kernel

Buat file:

```bash
sudo nano /etc/modprobe.d/cis-blacklist.conf
```

Isi:

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

---

## 3. Hardening Sysctl

Buat file:

```bash
sudo nano /etc/sysctl.d/99-cis-hardening.conf
```

Isi:

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

Terapkan:

```bash
sudo sysctl --system
```

---

# FASE 3 — Firewalld

Aktifkan:

```bash
sudo systemctl enable --now firewalld
```

Buka HTTP dan HTTPS:

```bash
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
```

Port Docker Swarm:

```bash
sudo firewall-cmd --permanent --add-port=2377/tcp

sudo firewall-cmd --permanent --add-port=7946/tcp
sudo firewall-cmd --permanent --add-port=7946/udp

sudo firewall-cmd --permanent --add-port=4789/udp
```

Reload:

```bash
sudo firewall-cmd --reload
```

---

# FASE 4 — Deployment Mayan EDMS

## 1. Docker Swarm

```bash
sudo systemctl enable --now docker
```

Inisialisasi:

```bash
sudo docker swarm init --advertise-addr 11.11.11.2
```

---

## 2. CIS Docker Benchmark

Buat:

```bash
sudo nano /etc/docker/daemon.json
```

Isi:

```json
{
  "icc": false,
  "userns-remap": "default",
  "live-restore": true,
  "userland-proxy": false,
  "no-new-privileges": true
}
```

Restart:

```bash
sudo systemctl restart docker
```

---

## 3. PostgreSQL pada Laptop B

Edit:

```text
/var/lib/postgres/data/pg_hba.conf
```

Tambahkan:

```text
host    mayan_db    mayan_user    11.11.11.2/32    md5
```

---

## 4. Deploy Stack

```bash
mkdir ~/mayan-deployment

cd ~/mayan-deployment

nano docker-compose.yml
```

Isi:

```yaml
version: '3.8'

services:
  redis:
    image: redis:7.0-alpine
    networks:
      - network_app

  mayan_app:
    image: mayanid/mayan:latest
    environment:
      - MAYAN_DATABASE_ENGINE=django.db.backends.postgresql
      - MAYAN_DATABASE_HOST=11.11.11.3
      - MAYAN_DATABASE_NAME=mayan_db
      - MAYAN_DATABASE_USER=mayan_user
      - MAYAN_DATABASE_PASSWORD=PasswordDatabaseCIS123

      - MAYAN_CELERY_BROKER_URL=redis://redis:6379/0
      - MAYAN_CELERY_RESULT_BACKEND=redis://redis:6379/0

    networks:
      - network_app

    ports:
      - "8000:8000"

    volumes:
      - data_app:/var/lib/mayan

networks:
  network_app:
    driver: overlay

volumes:
  data_app:
```

Deploy:

```bash
sudo docker stack deploy -c docker-compose.yml kelompok9_stack
```

---

# FASE 5 — Nginx Reverse Proxy

Install:

```bash
sudo pacman -S nginx-mainline --noconfirm
```

Edit:

```bash
sudo nano /etc/nginx/nginx.conf
```

Tambahkan:

```nginx
server {
    listen 80;

    server_name 11.11.11.2;

    location / {
        proxy_pass http://127.0.0.1:8000;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Aktifkan:

```bash
sudo systemctl enable --now nginx
```

---

# FASE 6 — Penyelesaian Tugas Kelompok 9

## 1. GitHub Project

Pindahkan seluruh kartu tugas:

```text
In Progress → Done
```

untuk:

* Fase 1
* Fase 2
* Fase 3
* Fase 4
* Fase 5

---

## 2. Verifikasi Jaringan Lokal

Dari perangkat lain dalam jaringan yang sama, buka:

```text
http://11.11.11.2
```
