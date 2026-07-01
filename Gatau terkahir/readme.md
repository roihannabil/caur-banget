Berikut adalah panduan langkah demi langkah yang sistematis untuk membangun seluruh infrastruktur Anda, mulai dari konfigurasi jaringan, firewall, instalasi K3s, hingga deployment aplikasi sesuai dengan referensi dokumen kelompok Anda.
## **Fase 1: Konfigurasi Jaringan & IP Statis**
Langkah pertama adalah mengunci IP Address pada setiap node (menggunakan jaringan Wi-Fi utama) agar tidak berubah-ubah dan mengonfigurasi Access Point pada Node Public.
### **1. Setup IP Statis di Setiap Node**
Lakukan konfigurasi IP statis pada masing-masing mesin (Admin, Data, Internal, Public) melalui network manager yang digunakan (misal NetworkManager atau systemd-networkd).
 * **Node Admin:** 192.168.1.2
 * **Node Data:** 192.168.1.3
 * **Node Internal:** 192.168.1.4
 * **Node Public:** 192.168.1.5 (Interface Wi-Fi Utama)
### **2. Setup Access Point di Node Public**
Buka Node Public, lalu ubah mode stasiun Wi-Fi menjadi Access Point (AP) menggunakan iwd (iwctl) atau nyalakan Hotspot HP khusus Client.
 * **Interface AP:** Berikan IP lokal baru, misalnya 172.22.1.1 dengan subnet /24.
 * **DHCP Server:** Aktifkan dnsmasq atau sejenisnya pada interface AP ini agar **Client** mendapatkan IP secara otomatis (misal 172.22.1.10) saat terhubung ke hotspot Node Public.
### **3. Setup Akses SSH Tanpa Password (di Node Admin)**
Buka terminal di **Node Admin**, lalu generate SSH key dan kirimkan ke ketiga node lainnya agar Admin bisa melakukan remote management dengan mudah:
```bash
ssh-keygen -t rsa
ssh-copy-id root@192.168.1.3
ssh-copy-id root@192.168.1.4
ssh-copy-id root@192.168.1.5

```
## **Fase 2: Konfigurasi Firewall (Remove & Add Port)**
Langkah ini dilakukan untuk memastikan port yang dibutuhkan terbuka dan port yang tidak digunakan tertutup demi keamanan antar-region.
### **1. Bersihkan Aturan Firewall Lama**
Jalankan perintah berikut di setiap node untuk membersihkan (*flush*) aturan iptables lama jika ada yang konflik:
```bash
iptables -F
iptables -X
iptables -t nat -F
iptables -t nat -X

```
### **2. Buka Port Spesifik pada Masing-Masing Node**
 * **Node Admin (K3s Server):**
   Wajib membuka port 6443 untuk API Server Kubernetes agar Agent bisa join.
   ```bash
   iptables -A INPUT -p tcp --dport 6443 -j ACCEPT
   
   ```
 * **Node Data (MariaDB):**
   Buka port 3706 khusus untuk diakses oleh Node Internal (192.168.1.4).
   ```bash
   iptables -A INPUT -p tcp -s 192.168.1.4 --dport 3706 -j ACCEPT
   iptables -A INPUT -p tcp --dport 3706 -j DROP
   
   ```
 * **Node Internal (Atom & SLiMS):**
   Buka port 8080 khusus untuk menerima trafik dari Node Public (192.168.1.5).
   ```bash
   iptables -A INPUT -p tcp -s 192.168.1.5 --dport 8080 -j ACCEPT
   iptables -A INPUT -p tcp --dport 8080 -j DROP
   
   ```
 * **Node Public (Nginx):**
   Buka port 80 (HTTP) agar bisa diakses oleh Client dari jaringan hotspot (172.22.1.0/24).
   ```bash
   iptables -A INPUT -p tcp -s 172.22.1.0/24 --dport 80 -j ACCEPT
   
   ```
## **Fase 3: Instalasi dan Inisialisasi Kluster K3s**
Proses penggabungan Node Admin, Data, dan Internal ke dalam satu manajemen kluster Kubernetes menggunakan K3s.
### **1. Jalankan K3s Server di Node Admin**
Buka Node Admin dan jalankan script instalasi K3s:
```bash
curl -sfL https://get.k3s.io | sh -

```
Tunggu proses selesai, lalu ambil token kluster Anda:
```bash
sudo cat /var/lib/rancher/k3s/server/node-token

```
*Salin token panjang yang muncul (misalnya: K100cc99bf...::server:94c6b...).*
### **2. Jalankan K3s Agent di Node Data dan Node Internal**
Masuk ke **Node Data** (192.168.1.3) dan **Node Internal** (192.168.1.4), lalu jalankan perintah instalasi agent dengan memasukkan IP Admin dan token tadi:
```bash
curl -sfL https://get.k3s.io | K3S_URL=https://192.168.1.2:6443 K3S_TOKEN=<PASTE_TOKEN_DI_SINI> sh -

```
### **3. Verifikasi Kluster (di Node Admin)**
Kembali ke Node Admin, cek apakah semua node sudah berhasil terhubung dan berstatus Ready:
```bash
kubectl get nodes

```
*(Pastikan semua node berstatus Ready. Jika ada yang NotReady seperti pada file 1000240633.jpg teman Anda, cek log service-nya menggunakan journalctl -u k3s-agent -f)*.
## **Fase 4: Deployment Aplikasi & Database**
### **1. Node Data: Deploy MariaDB**
Anda bisa menginstalnya langsung di OS Node Data atau menggunakan manifes pod Kubernetes yang diarahkan ke port 3706. Pastikan konfigurasi bind-address di /etc/my.cnf atau konfigurasi container disetel ke 0.0.0.0 agar bisa menerima koneksi eksternal dari Node Internal.
### **2. Node Internal: Deploy Atom dan SLiMS**
 * **Aplikasi Atom:** Gunakan podman untuk menjalankan container Atom secara lokal atau deploy melalui objek manifest Kubernetes agar berjalan di port 8080.
 * **Aplikasi SLiMS:** Buat file manifes deployment SLiMS (slims-web-deployment.yaml) di Node Admin, lalu jalankan:
   ```bash
   kubectl apply -f slims-web-deployment.yaml
   
   ```
   Pastikan konfigurasi *database host* di dalam SLiMS diarahkan ke IP Node Data yaitu 192.168.1.3:3706. SLiMS disetel untuk mengekspos layanannya ke port 8080.
## **Fase 5: Konfigurasi Nginx Reverse Proxy (Node Public)**
Node Public bertindak sebagai jembatan tunggal agar Client tidak bisa melihat infrastruktur backend Anda secara langsung.
 1. Buka **Node Public**, install Nginx:
   ```bash
   sudo pacman -S nginx  # Jika menggunakan Arch Linux
   
   ```
 2. Edit file konfigurasi Nginx (misal di /etc/nginx/nginx.conf):
   ```nginx
   http {
       server {
           listen 80;
           server_name _; # Menerima dari semua domain/IP hotspot
   
           location / {
               # Teruskan trafik dari client ke Node Internal port 8080
               proxy_pass http://192.168.1.4:8080;
   
               proxy_set_header Host $host;
               proxy_set_header X-Real-IP $remote_addr;
               proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
               proxy_set_header X-Forwarded-Proto $scheme;
           }
       }
   }
   
   ```
 3. Jalankan dan aktifkan layanan Nginx:
   ```bash
   sudo systemctl enable --now nginx
   
   ```
## **Fase 6: Pengujian Akhir**
 1. Hubungkan perangkat **Client** ke hotspot/AP yang dipancarkan oleh **Node Public**.
 2. Buka browser di perangkat Client, lalu akses IP Node Public di alamat [http://172.22.1.1](http://172.22.1.1).
 3. Jika berhasil, Client akan langsung melihat tampilan aplikasi SLiMS atau Atom yang berada di Node Internal tanpa bisa melakukan ping atau SSH ke Node Admin maupun Node Data.
