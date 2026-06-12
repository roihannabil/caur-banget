# Panduan Implementasi Server Hardened CIS + Docker Swarm + Mayan EDMS (Kelas C)

## Deskripsi

Panduan ini menjelaskan proses pembangunan infrastruktur server yang mengikuti prinsip **CIS Hardening**, menggunakan **Arch Linux dengan kernel LTS**, tanpa paket `base-devel`, menggunakan **Docker Swarm** untuk deployment **Mayan EDMS**, serta diamankan menggunakan **Nginx Reverse Proxy** dan **Firewalld**.

Metode yang digunakan adalah **CLI (Command Line Interface) murni** agar pengguna memahami fungsi setiap konfigurasi yang diterapkan.

---

# Architecture Overview

## Topologi Jaringan

| Perangkat | IP Address | Fungsi                                                               |
| --------- | ---------- | -------------------------------------------------------------------- |
| Laptop A  | 11.11.11.2 | Arch Linux LTS, Docker Swarm Leader, Mayan EDMS, Nginx Reverse Proxy |
| Laptop B  | 11.11.11.3 | PostgreSQL Database Server                                           |

---

# FASE 1 — Persiapan OS dan Manajemen Proyek

## 1. Inisialisasi GitHub Project

Buat GitHub Project dengan template **Kanban Board** yang berisi:

* [ ] Fase 1: Instalasi Paket Mandiri & Kernel LTS
* [ ] Fase 2: Hardening Kernel & Pemblokiran Modul (CIS)
* [ ] Fase 3: Konfigurasi Firewalld Jaringan Lokal
* [ ] Fase 4: Setup Docker Swarm & Pemisahan Database
* [ ] Fase 5: Konfigurasi Nginx Reverse Proxy & Pengujian

---

## 2. Instalasi Paket Mandiri (Tanpa base-devel)

```bash
sudo pacman -Syu

sudo pacman -S \
git \
cmake \
make \
gcc \
binutils \
patch \
awk \
grep \
sed \
--noconfirm
```

### Alasan Tidak Menggunakan base-devel

Prinsip CIS menganjurkan pengurangan *attack surface*. Paket `base-devel` menginstal banyak utilitas tambahan yang tidak diperlukan pada server produksi sehingga berpotensi meningkatkan risiko keamanan.

---

## 3. Migrasi ke Kernel Linux-LTS

```bash
sudo pacman -S linux-lts linux-lts-headers --noconfirm

sudo grub-mkconfig -o /boot/grub/grub.cfg
```

Reboot sistem lalu pilih kernel **Linux-LTS** pada menu boot.

---

# FASE 2 — Hardening Kernel (CIS Benchmark)

## 1. Blacklist Modul yang Tidak Digunakan

Buat file:

```bash
sudo nano /etc/modprobe.d/cis-blacklist.conf
```

Isi dengan:

```text
# Disable unused filesystems
blacklist cramfs
blacklist freevxfs
blacklist jffs2
blacklist hfs
blacklist hfsplus
blacklist udf

# Disable uncommon network protocols
blacklist dccp
blacklist sctp
blacklist rds
blacklist tipc
```

---

## 2. Hardening Sysctl

Buat file:

```bash
sudo nano /etc/sysctl.d/99-cis-server.conf
```

Isi dengan:

```text
# Disable ICMP redirects
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.default.send_redirects = 0
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.default.accept_redirects = 0

# Enable source route verification
net.ipv4.conf.all.rp_filter = 1
net.ipv4.conf.default.rp_filter = 1

# Disable source routed packets
net.ipv4.conf.all.accept_source_route = 0
net.ipv4.conf.default.accept_source_route = 0

# Log suspicious packets
net.ipv4.conf.all.log_martians = 1
net.ipv4.conf.default.log_martians = 1

# Disable core dumps
fs.suid_dumpable = 0
```

Terapkan konfigurasi:

```bash
sudo sysctl --system
```

---

# FASE 3 — Konfigurasi Firewalld

## Instalasi Firewalld

```bash
sudo pacman -S firewalld --noconfirm

sudo systemctl enable --now firewalld
```

## Membuka Port yang Dibutuhkan

### HTTP dan HTTPS

```bash
sudo firewall-cmd --permanent --add-service=http

sudo firewall-cmd --permanent --add-service=https
```

### Docker Swarm

```bash
sudo firewall-cmd --permanent --add-port=2377/tcp

sudo firewall-cmd --permanent --add-port=7946/tcp

sudo firewall-cmd --permanent --add-port=7946/udp

sudo firewall-cmd --permanent --add-port=4789/udp
```

Reload firewall:

```bash
sudo firewall-cmd --reload
```

---

# FASE 4 — Docker Swarm dan Database Terpisah

## 1. Instalasi Docker

```bash
sudo pacman -S docker --noconfirm

sudo systemctl enable --now docker
```

---

## 2. Inisialisasi Docker Swarm

Laptop A (11.11.11.2):

```bash
sudo docker swarm init --advertise-addr 11.11.11.2
```

---

## 3. Persiapan Database

Laptop B (11.11.11.3):

Pastikan:

* PostgreSQL telah terinstal
* Service PostgreSQL aktif
* `pg_hba.conf` mengizinkan koneksi dari IP Laptop A

---

## 4. Deployment Mayan EDMS

Buat direktori deployment:

```bash
mkdir ~/mayan-swarm

cd ~/mayan-swarm

nano docker-compose.yml
```

Isi file:

```yaml
version: "3.8"

services:
  redis:
    image: redis:7.0-alpine
    networks:
      - mayan_network

  app:
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
      - mayan_network

    ports:
      - "8000:8000"

    volumes:
      - mayan_data:/var/lib/mayan

networks:
  mayan_network:
    driver: overlay

volumes:
  mayan_data:
```

Deploy stack:

```bash
sudo docker stack deploy -c docker-compose.yml mayan_stack
```

---

# FASE 5 — Konfigurasi Nginx Reverse Proxy

## Instalasi Nginx

```bash
sudo pacman -S nginx-mainline --noconfirm
```

Edit konfigurasi:

```bash
sudo nano /etc/nginx/nginx.conf
```

Tambahkan blok:

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

Aktifkan Nginx:

```bash
sudo systemctl enable --now nginx
```

---

# Verifikasi Akhir

## 1. Uji Akses Jaringan

Dari perangkat lain yang terhubung ke jaringan yang sama:

```text
http://11.11.11.2
```

Jika halaman Mayan EDMS muncul, berarti:

* Docker Swarm berhasil
* Firewalld berfungsi
* Nginx Reverse Proxy berfungsi
* Jaringan lokal terhubung

---

## 2. Verifikasi Hardening CIS

```bash
sysctl net.ipv4.conf.all.send_redirects
```

Output yang diharapkan:

```text
net.ipv4.conf.all.send_redirects = 0
```

---

## 3. Verifikasi Kernel LTS

```bash
uname -r
```

Contoh output:

```text
6.x.x-lts
```

---

## 4. Verifikasi base-devel Tidak Terinstal

```bash
pacman -Qi base-devel
```

Output yang diharapkan:

```text
error: package 'base-devel' was not found
```

---

# Kesimpulan

Implementasi ini memenuhi kriteria:

* Menggunakan kernel Linux-LTS
* Tidak menggunakan paket base-devel
* Mengikuti prinsip CIS Hardening
* Menggunakan Firewalld
* Menggunakan Docker Swarm
* Database dipisahkan pada host berbeda
* Menggunakan Nginx Reverse Proxy
* Dapat diakses melalui jaringan lokal
