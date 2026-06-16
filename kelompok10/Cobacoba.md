Baik, mari kita bedah dari nol besar dengan bahasa yang sangat santai dan mudah dipahami. Anggap saja ini panduan "selangkah demi selangkah" di mana tidak ada yang perlu merasa bingung.
Siapkan **1 laptop yang sudah terpasang Blackbird OS** (ini akan jadi **Server**), beberapa laptop anggota kelompok lain (sebagai **Admin**, **Operator**, dan **Client**), serta **kabel LAN** (plus alat bernama *Switch/Hub* colokan LAN jika ingin menghubungkan lebih dari 2 laptop sekaligus).
Berikut adalah panduan lengkapnya dari awal sampai selesai untuk ke-6 anggota:
## 🔌 LANGKAH 1: Menyambungkan Kabel LAN (Fisik)
**Penanggung Jawab: Anggota 1 & Anggota 2**
Kabel LAN berfungsi seperti pipa air. Kita ingin menghubungkan laptop Server (Blackbird OS) ke laptop teman-temanmu agar mereka bisa mengakses perpustakaan SLiMS yang ada di dalam server tersebut tanpa perlu internet.
 1. **Colok Kabel:** Hubungkan kabel LAN dari laptop Server (Blackbird OS) ke *Switch/Hub* LAN. Lalu, colokkan laptop anggota lainnya ke *Switch* tersebut menggunakan kabel LAN masing-masing.
 2. **Cek Koneksi di Server:**
   * Di laptop Blackbird OS, buka aplikasi bernama **Terminal** (bisa dicari di menu aplikasi atau tekan tombol Ctrl + Alt + T).
   * Ketik perintah ini lalu tekan Enter:
     ```bash
     
     ```
ip a
```
   * Lihat daftarnya. Cari kata-kata yang diawali dengan huruf **e** (misalnya `eth0` atau `enp3s0`). Itu adalah nama "lubang" kartu LAN kamu. Catat namanya.
3. **Setel Alamat (IP) Server:**
   * Di pojok kanan bawah desktop Blackbird OS (KDE), klik ikon jaringan/network.
   * Pilih koneksi kabel (*Wired Connection*), lalu klik **Configure / Sifat**.
   * Masuk ke tab **IPv4**. Ubah metodenya dari *Automatic (DHCP)* menjadi **Manual**.
   * Klik **Add (Tambah)**, lalu isi:
     * **Address (IP):** `10.10.1.1`
     * **Netmask:** `255.255.255.0`
     * **Gateway:** (Kosongkan saja)
   * Klik **Save (Simpan)**.
4. **Setel Alamat di Laptop Teman (Admin & Operator):**
   * Minta **Anggota 5 (Laptop Admin)** untuk mengubah pengaturan LAN di Windows/Mac-nya menjadi manual dengan IP: `10.10.1.2` dan Netmask: `255.255.255.0`.
   * Minta **Anggota 6 (Laptop Operator)** untuk mengubah pengaturan LAN menjadi manual dengan IP: `10.10.1.3` dan Netmask: `255.255.255.0`.

---

## 🛡️ LANGKAH 2: Mengunci Gerbang Server (CIS Linux Benchmark)
**Penanggung Jawab: Anggota 3**

Sekarang kabel sudah terhubung, tapi server kita masih "telanjang" dan berbahaya. Sesuai standar keamanan dunia teknologi (CIS Benchmark), kita harus memasang pagar (*Firewall*) agar tidak sembarang orang bisa mengacak-acak server.

1. Buka **Terminal** di Blackbird OS.
2. Ketik perintah ini untuk memasang pagar pengaman bernama UFW (masukkan password laptopmu jika diminta):
   ```bash
   sudo apt update && sudo apt install ufw -y

```
 3. Atur agar semua orang luar **ditolak masuk secara default**:
   ```bash
   sudo ufw default deny incoming
   sudo ufw default allow outgoing
   
   ```
```
4. Sekarang, buat pengecualian pintar sesuai diagram tugas kelompokmu:
   * **Beri izin khusus untuk Laptop Admin (`10.10.1.2`):**
     ```bash
     sudo ufw allow from 10.10.1.2 to any port 3306
     sudo ufw allow from 10.10.1.2 to any port 80

```
 * **Beri izin terbatas untuk Laptop Operator (10.10.1.3):** (Hanya boleh buka web, dilarang intip database port 3306)
   ```bash
   sudo ufw allow from 10.10.1.3 to any port 80
   
   ```
```
5. Nyalakan pagarnya dengan perintah:
   ```bash
   sudo ufw enable

```
*(Jika ada pertanyaan, ketik y lalu Enter)*.
## 🐳 LANGKAH 3: Membuat Kamar Terisolasi (CIS Docker Benchmark)
**Penanggung Jawab: Anggota 4**
Dosen meminta sistem SLiMS ini aman. Jika web SLiMS suatu saat diretas, peretas tidak boleh sampai bisa merusak seluruh isi laptop Blackbird OS kamu. Caranya? Kita buat "kotak terisolasi" menggunakan teknologi bernama **Docker**.
 1. Pasang Docker di Terminal Blackbird OS:
   ```bash
   sudo apt install docker.io docker-compose -y
   sudo systemctl enable --now docker
   
   ```
```
2. Buat folder khusus untuk tugas ini di komputer:
   ```bash
   mkdir ~/tugas-slims && cd ~/tugas-slims

```
 3. Buat file konfigurasi bernama docker-compose.yml. Ketik perintah ini untuk membuka kertas catatan kosong di terminal:
   ```bash
   nano docker-compose.yml
   
   ```
```
4. Salin (*Copy*) dan tempel (*Paste*) kode di bawah ini ke dalam terminal tersebut:
   ```yaml
   version: '3.8'

   networks:
     net-internal:
       ipam:
         config:
           - subnet: 172.26.1.0/24

   services:
     # Kamar Khusus Database SLiMS
     slims-db:
       image: mariadb:10.6
       container_name: slims-data-container
       environment:
         MYSQL_ROOT_PASSWORD: password_aman_kelompok10
         MYSQL_DATABASE: db_slims
         MYSQL_USER: user_slims
         MYSQL_PASSWORD: password_rahasia_slims
       networks:
         net-internal:
           ipv4_address: 172.26.1.2

     # Kamar Khusus Aplikasi Web SLiMS
     slims-app:
       image: dspace/dspace:latest # Atau image SLiMS resmi pilihanmu
       container_name: slims-service-container
       ports:
         - "80:80"
       networks:
         net-internal:
           ipv4_address: 172.26.1.3
       # Sesuai aturan CIS Docker: Jalankan sebagai user biasa (bukan bos/root)
       user: "www-data" 

```
 5. Simpan file tersebut dengan menekan tombol **Ctrl + O**, lalu tekan **Enter**. Untuk keluar, tekan **Ctrl + X**.
## 📚 LANGKAH 4: Menghidupkan & Memasang Aplikasi SLiMS
**Penanggung Jawab: Anggota 5**
Sekarang kita racik dan hidupkan perpustakaan digitalnya.
 1. Di dalam terminal, pastikan kamu masih berada di folder ~/tugas-slims, lalu ketik perintah ini untuk menyalakan semua kotak Docker tadi:
   ```bash
   sudo docker-compose up -d
   
   ```
```
   *(Tunggu proses unduhannya selesai sampai tertulis status "Started" atau "Done")*
2. Periksa apakah layanannya sudah berjalan dengan mengetik:
   ```bash
   sudo docker ps

```
Jika muncul tabel berisi slims-data-container dan slims-service-container, selamat! Mesin perpustakaanmu sudah hidup di latar belakang server.
## 🧪 LANGKAH 5: Uji Coba Keamanan Sistem (Uji Penetrasi)
**Penanggung Jawab: Anggota 6**
Langkah terakhir adalah pembuktian untuk dosen bahwa hasil kerja Kelompok 10 benar-benar aman sesuai skenario.
 1. **Uji Coba dari Laptop Operator (IP 10.10.1.3):**
   * Buka browser (Chrome/Firefox) di laptop operator.
   * Di kotak alamat web, ketik IP Server: [http://10.10.1.1](http://10.10.1.1)
   * **Hasil yang diharapkan:** Halaman perpustakaan SLiMS **terbuka dengan lancar**.
 2. **Uji Coba Pembatasan Sebelah (Kunci CIS):**
   * Masih di laptop Operator, coba instal aplikasi remote database atau buka terminal, lalu coba tembak port database server: 10.10.1.1 port 3306.
   * **Hasil yang diharapkan:** Koneksi **gagal total atau loading terus menerus (Blocked)**. Ini membuktikan aturan firewall buatan Anggota 3 berhasil melindungi database dari orang luar!
 3. **Uji Coba dari Laptop Admin (IP 10.10.1.2):**
   * Lakukan hal yang sama. Laptop admin harusnya **bisa mengakses keduanya** (halaman web SLiMS dan port database 3306) tanpa hambatan karena sudah diberi kartu pas masuk (Jalur Merah).
Selesai! Sekarang server SLiMS kelompokmu sudah berdiri tegak di dalam Blackbird OS, jaringannya terhubung rapi via kabel LAN, dan yang paling penting: sistemnya sangat aman karena sudah mengikuti aturan ketat **CIS Benchmarks** yang diminta dosen.
Apakah ada langkah di atas yang komputer kalian mengalami kendala saat memproses perintahnya?
