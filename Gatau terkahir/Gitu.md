Tentu, mari kita luruskan dulu konsep instalasi aplikasi di dalam arsitektur Kubernetes/Container sebelum masuk ke langkah-langkah teknis lengkapnya.
### **Apakah SLiMS & ATOM tidak pakai wget/curl/git clone?**
Benar, jika kita menggunakan metode modern berbasis **Podman/Container/Kubernetes**, kita **tidak perlu lagi** mengunduh source code manual lewat git clone atau wget, lalu mengonfigurasi Apache/PHP dari nol di sistem operasi.
Semua *source code* dan *environment* aplikasi sudah dibungkus menjadi sebuah **Image Container** (seperti slimsio/slims9). Kita cukup menarik (*pull*) image tersebut, dan container engine akan langsung menjalankannya secara instan.
### **Apakah ATOM bisa diinstal lewat Podman/Container?**
**Bisa sekali.** Paket aplikasi Atom (AtoM: Access to Memory) memiliki image resmi maupun komunitas di Docker Hub/Quay.io. Karena Podman kompatibel dengan standar Docker, Anda bisa menjalankan ATOM langsung menggunakan perintah podman run atau mendefinisikannya ke dalam file YAML Kubernetes agar di-manage oleh K3s di Node Internal.
Berikut adalah **panduan lengkap dari awal sampai akhir** (End-to-End) berdasarkan instruksi, skema jaringan, dan aturan firewall kelompok Anda.
## **Langkah 1: Setup IP Statis di Semua Node (Arch Linux)**
Gunakan systemd-networkd untuk mengunci IP agar tidak berubah dan tidak konflik.
 1. Jalankan ip link di setiap node untuk mengetahui nama interface Wi-Fi/Ethernet Anda (misal: wlan0).
 2. Buat file konfigurasi network:
   ```bash
   sudo nano /etc/systemd/network/20-wired.network
   
   ```
 3. Isi sesuai dengan IP masing-masing node:
   * **Admin (192.168.1.2):**
     ```ini
     [Match]
     Name=wlan0
     
     [Network]
     Address=192.168.1.2/24
     Gateway=192.168.1.1
     DNS=8.8.8.8
     
     ```
   * **Data (192.168.1.3):** Ganti baris Address menjadi 192.168.1.3/24
   * **Internal (192.168.1.4):** Ganti baris Address menjadi 192.168.1.4/24
   * **Public (192.168.1.5):** Ganti baris Address menjadi 192.168.1.5/24
 4. Terapkan konfigurasi:
   ```bash
   sudo systemctl restart systemd-networkd
   
   ```
*(Khusus Node Public, pastikan Anda sudah mengaktifkan Hotspot/AP via iwctl dengan subnet 172.22.1.1/24 menuju Client).*
## **Langkah 2: Disable Module Berbahaya / Tidak Digunakan**
Untuk keamanan ekstra (hardening), beberapa modul kernel dipadamkan (disable) jika tidak digunakan.
 1. Buat file blacklist modul kernel:
   ```bash
   sudo nano /etc/modprobe.d/security-blacklist.conf
   
   ```
 2. Tambahkan modul yang ingin di-disable (contoh standar filesystem tak terpakai atau modul legacy):
   ```text
   blacklist cramfs
   blacklist freevxfs
   blacklist hfs
   blacklist hfsplus
   blacklist jffs2
   blacklist h接觸 # Sesuaikan jika ada modul wifi/bluetooth internal yang ingin dimatikan
   
   ```
## **Langkah 3: Konfigurasi Firewalld (Remove DHCP & Add Port)**
Tugas Anda meminta untuk **menghapus layanan DHCP/DNS bawaan di semua zone kecuali zone public**, serta membuka port spesifik.
### **1. Bersihkan Service DHCP & DNS di Semua Zone Eksternal**
Jalankan perintah ini di **Semua Node** untuk memastikan zone bawaan seperti trusted, home, work, dll., tidak membuka celah DHCP:
```bash
# Hapus dhcpv6-client dan dns dari zone default selain public
sudo firewall-cmd --permanent --zone=home --remove-service=dhcpv6-client
sudo firewall-cmd --permanent --zone=work --remove-service=dhcpv6-client
sudo firewall-cmd --permanent --zone=internal --remove-service=dhcpv6-client

# Pastikan zone public hanya mengizinkan servis krusial
sudo firewall-cmd --permanent --zone=public --add-service=ssh

```
### **2. Tambah Port Spesifik per Node**
 * **Node Admin:** Buka port K3s API Server (6443) agar Agent bisa mendaftar.
   ```bash
   sudo firewall-cmd --permanent --zone=public --add-port=6443/tcp
   sudo firewall-cmd --reload
   
   ```
 * **Node Data (Database MariaDB):** Hanya ijinkan port 3706 diakses dari IP Node Internal (192.168.1.4).
   ```bash
   sudo firewall-cmd --permanent --zone=public --add-rich-rule='rule family="ipv4" source address="192.168.1.4" port port="3706" protocol="tcp" accept'
   sudo firewall-cmd --reload
   
   ```
 * **Node Internal (Aplikasi):** Hanya ijinkan port 8080 diakses dari IP Node Public (192.168.1.5).
   ```bash
   sudo firewall-cmd --permanent --zone=public --add-rich-rule='rule family="ipv4" source address="192.168.1.5" port port="8080" protocol="tcp" accept'
   sudo firewall-cmd --reload
   
   ```
 * **Node Public (Nginx Proxy):** Buka port 80 umum agar Client dari hotspot (172.22.1.0/24) bisa masuk.
   ```bash
   sudo firewall-cmd --permanent --zone=public --add-port=80/tcp
   sudo firewall-cmd --reload
   
   ```
## **Langkah 4: Binding Kubernetes (K3s Server & Agent)**
Sesuai dokumentasi grup Anda (kubernetes-admin.md), kita lakukan binding kluster.
### **1. Jalankan Server di Node Admin**
```bash
curl -sfL https://get.k3s.io | sh -

```
Ambil token autentikasinya:
```bash
sudo cat /var/lib/rancher/k3s/server/node-token

```
### **2. Jalankan Agent di Node Data & Internal**
Masuk ke Node Data dan Node Internal, lalu hubungkan ke Admin:
```bash
curl -sfL https://get.k3s.io | K3S_URL=https://192.168.1.2:6443 K3S_TOKEN=<TOKEN_MANAJER_ADMIN> sh -

```
*Verifikasi di Admin dengan perintah kubectl get nodes untuk memastikan statusnya Ready.*
## **Langkah 5: Deploy Aplikasi Lewat Kubernetes & Podman**
### **1. Deploy MariaDB di Node Data (Lewat K3s)**
Di Node Admin, buat file mariadb-deploy.yaml:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mariadb-data
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mariadb
  template:
    metadata:
      labels:
        app: mariadb
    spec:
      containers:
      - name: mariadb
        image: mariadb:10.6
        env:
        - name: MARIADB_ROOT_PASSWORD
          value: "mypass123"
        - name: MARIADB_DATABASE
          value: "slims_db"
---
apiVersion: v1
kind: Service
metadata:
  name: mariadb-service
spec:
  type: NodePort
  selector:
    app: mariadb
  ports:
    - port: 3306
      targetPort: 3306
      nodePort: 3706

```
Jalankan perintah: kubectl apply -f mariadb-deploy.yaml
### **2. Deploy SLiMS & ATOM di Node Internal (K3s / Podman)**
 * **Opsi SLiMS (Kubernetes K3s):**
   Buat file slims-deploy.yaml di Admin:
   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: slims-internal
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: slims
   template:
     metadata:
       labels:
         app: slims
     spec:
       containers:
       - name: slims
         image: slimsio/slims9:latest
         env:
         - name: DB_HOST
           value: "192.168.1.3"
         - name: DB_PORT
           value: "3706"
   ---
   apiVersion: v1
   kind: Service
   metadata:
     name: slims-service
   spec:
     type: NodePort
     selector:
       app: slims
     ports:
       - port: 80
         targetPort: 80
         nodePort: 8080
   
   ```
   Jalankan perintah: kubectl apply -f slims-deploy.yaml
 * **Opsi ATOM (Menggunakan Podman di Node Internal):**
   Jika ingin menggunakan Podman langsung di OS Node Internal:
   ```bash
   sudo pacman -S podman --noconfirm
   podman run -d --name atom-app -p 8080:80 docker.io/local_atom_image:latest
   
   ```
## **Langkah 6: Akses Client Melalui Nginx (Node Public)**
Agar Client hanya bisa mengakses Node Public, konfigurasikan Nginx di Node Public untuk melemparkan trafik (*Reverse Proxy*) ke Node Internal port 8080.
 1. Buka Node Public, edit file /etc/nginx/nginx.conf:
   ```nginx
   server {
       listen 80;
       server_name 172.22.1.1; # IP Access Point Public
   
       location / {
           proxy_pass http://192.168.1.4:8080; # Dilempar ke Node Internal
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
       }
   }
   
   ```
 2. Jalankan Nginx:
   ```bash
   sudo systemctl enable --now nginx
   
   ```
Sekarang, **Client** tinggal menyambungkan perangkatnya ke Hotspot dari Node Public, membuka browser, dan mengetik http://172.22.1.1. Client akan langsung diarahkan ke SLiMS/ATOM dengan aman tanpa bisa menembus jaringan inti Admin dan Data.
