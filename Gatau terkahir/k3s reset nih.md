Betul sekali! Karena Anda berpindah dari Wi-Fi teman kos ke hotspot HP sendiri, seluruh *subnet* jaringan akan berubah. Otomatis, IP server (K3S_URL) yang lama tidak akan bisa dihubungi lagi oleh Node Data dan Node Internal, sehingga statusnya bisa berubah menjadi NotReady atau terputus.
Agar kluster K3s Anda berpindah dengan mulus ke jaringan hotspot HP Anda yang baru menggunakan IP Statis (192.168.1.x), berikut adalah urutan langkah yang harus Anda lakukan:
## **Langkah 1: Hubungkan Semua Node ke Hotspot HP & Setup IP Statis**
Sebelum menjalankan ulang perintah K3s, kita harus mengunci IP semua mesin di bawah hotspot HP Anda agar tidak berubah menggunakan systemd-networkd.
 1. Nyalakan Hotspot HP Anda.
 2. Hubungkan keempat mesin (Admin, Data, Internal, Public) ke hotspot tersebut.
 3. Di **setiap mesin**, edit berkas konfigurasi network untuk mengunci IP barunya:
   ```bash
   sudo nano /etc/systemd/network/20-wired.network
   
   ```
 4. Sesuaikan konfigurasinya seperti ini (Contoh untuk **Node Admin**):
   ```ini
   [Match]
   Name=wlan0  # Sesuaikan dengan nama interface Wi-Fi Anda (cek via 'ip link')
   
   [Network]
   Address=192.168.1.2/24
   Gateway=192.168.1.1    # IP Gateway hotspot HP biasanya 192.168.1.1 atau cek via 'ip route'
   DNS=8.8.8.8
   
   ```
   * *Lakukan hal yang sama di mesin lain dengan mengganti Address menjadi 192.168.1.3 (Data), 192.168.1.4 (Internal), dan 192.168.1.5 (Public).*
 5. Restart network daemon di semua mesin:
   ```bash
   sudo systemctl restart systemd-networkd
   
   ```
 6. Pastikan IP sudah berubah dengan mengetik perintah ip a.
## **Langkah 2: Bersihkan (Reset) K3s Agent Lama**
Karena sebelumnya Node Data dan Node Internal sudah terlanjur dipasang menggunakan IP Wi-Fi lama, Anda perlu membersihkan sisa konfigurasi agent yang lama agar tidak bentrok.
 1. Masuk ke **Node Data** dan **Node Internal**.
 2. Jalankan perintah *uninstall* bawaan K3s Agent untuk membersihkan *state* lama:
   ```bash
   sudo k3s-agent-uninstall.sh
   
   ```
   *(Perintah ini akan menghapus konfigurasi lama tanpa merusak sistem operasi Anda).*
## **Langkah 3: Ambil Token Baru di Node Admin**
Seringkali saat IP Node Admin berubah, K3s secara otomatis memperbarui sertifikat internalnya.
 1. Buka **Node Admin** (192.168.1.2).
 2. Restart service K3s Server agar membaca IP barunya:
   ```bash
   sudo systemctl restart k3s
   
   ```
 3. Intip kembali token kluster terbaru Anda:
   ```bash
   sudo cat /var/lib/rancher/k3s/server/node-token
   
   ```
   *Salin kode token panjang yang muncul.*
## **Langkah 4: Hubungkan Ulang Agent Menggunakan IP Baru**
Sekarang, Anda tinggal menjalankan ulang perintah curl penggabungan agent seperti kemarin, namun dengan **menggunakan IP Admin yang baru (192.168.1.2)** dan token yang baru saja Anda salin.
 1. Masuk ke **Node Data** (192.168.1.3), jalankan:
   ```bash
   curl -sfL https://get.k3s.io | K3S_URL="https://192.168.1.2:6443" K3S_TOKEN="<TOKEN_BARU_ANDA>" sh -s - agent
   
   ```
 2. Masuk ke **Node Internal** (192.168.1.4), jalankan perintah yang sama:
   ```bash
   curl -sfL https://get.k3s.io | K3S_URL="https://192.168.1.2:6443" K3S_TOKEN="<TOKEN_BARU_ANDA>" sh -s - agent
   
   ```
## **Langkah 5: Verifikasi Akhir**
Kembali ke **Node Admin**, lalu cek apakah semua mesin sudah terikat dengan benar di bawah jaringan hotspot HP Anda:
```bash
kubectl get nodes -o wide

```
Jika berhasil, Anda akan melihat nama node Admin, Data, dan Internal terdaftar dengan kolom **STATUS = Ready** dan kolom **INTERNAL-IP** yang sudah berubah menggunakan subnet baru Anda (192.168.1.x). Setelah ini, Anda bisa langsung melanjutkan ke tahap pengaturan firewalld dan deploy manifest aplikasi.
