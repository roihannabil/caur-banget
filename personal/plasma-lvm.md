# i use arch btw

https://g.co/gemini/share/61fb5f4e39a9

Ini adalah ide yang **sangat brilian dan taktis!** Memisahkan OS harian dengan OS untuk tugas kuliah adalah keputusan paling tepat. Kamu jadi bebas mengutak-atik, menghapus, bahkan merusak OS tugas di partisi 49.8 GB tanpa perlu takut data pribadi atau sistem utama kamu di ruang  .

Skema **Triple Boot (Windows + Arch Linux Official + Blackbird Amanda)** sangat bisa dilakukan dan tidak akan saling bentrok, karena ketiganya nanti akan berbagi rumah yang sama di partisi EFI Boot (`/dev/nvme0n1p1` berukuran 260M).

---

### 1. Apakah Perlu Rufus Lagi?

**Ya, kamu perlu flashdisk satu lagi atau menggunakan flashdisk yang sekarang untuk di-flash ulang** menggunakan ISO Arch Linux Official terbaru. Kamu bisa membuatnya lewat Windows menggunakan Rufus (pilih mode *DD Image* agar terbaca lancar saat booting).

---

### 2. Peta Pembagian Disk di cfdisk

Biar rapi dan tidak pusing, silakan atur ruang disk kamu di cfdisk menjadi seperti ini:

1. **Arahkan ke baris Free Space 127G (Atas):**
   * Pilih **New** $\rightarrow$ Ambil semua ruangnya $\rightarrow$ Pilih **Type** $\rightarrow$ Ubah menjadi **Linux LVM**.
   * *Ini akan menjadi wadah Arch Linux OS Utama kamu (KDE Plasma).*
2. **Arahkan ke baris Free Space 49.8G (Bawah):**
   * Biarkan saja dulu dalam bentuk **Free Space** (jangan dibuat partisi sekarang). Nanti saat ada tugas kuliah instalan baru, kamu tinggal arahkan installer Blackbird Amanda untuk memakai ruang kosong terbawah ini.
3. Pilih **Write**, ketik `yes`, lalu **Quit**.

---

### 3. Panduan Eksekusi Arch Linux Official (KDE Plasma - 127 GB)

Setelah kamu membuat installer Arch Linux Official di flashdisk dan melakukan booting masuk ke dalamnya, ketik `lsblk` untuk melihat nomor partisi LVM 127G kamu (misalkan terbaca sebagai `/dev/nvme0n1p5`). 

Ikuti urutan perintah bersih ini di terminal untuk membangun sistem utamamu:

#### A. Setup LVM & Volume Virtual (Di dalam 127 GB)
```bash
pvcreate /dev/nvme0n1p5   # Sesuaikan dengan nomor partisi LVM 127G kamu
vgcreate system /dev/nvme0n1p5

# Potong ruang virtual secara proporsional
lvcreate -L 35G   system -n root     # OS Utama + KDE Desktop + App Harian
lvcreate -L 4G    system -n swap     # SWAP Space untuk RAM virtual kamu
lvcreate -L 25G   system -n home     # Data harian biasa/kuliah
lvcreate -l 100%FREE system -n crab  # Sisa disk (~63 GB) untuk brankas LUKS terenkripsi
