# Panduan Cara Memasang Splash Screen (Layar Loading) via Dolphin (Tanpa Root)

Panduan ini dibuat khusus untuk memasang tema Splash Screen (**FalloutVault-Loading-Plasma6** atau **FalloutPipBoy-Loading-Plasma6**) secara visual lewat File Manager (Dolphin) tanpa memerlukan akses Terminal atau Root.

---

## Kenapa Cara Ini Dipakai?
Pada OS kustom yang dikunci ketat, menjalankan Dolphin sebagai administrator via `pkexec` sengaja diblokir demi keamanan sistem. Solusinya, kita memanfaatkan folder **Lokal User** (`~/.local/share/`) tempat di mana Anda bebas memodifikasi file tanpa memerlukan izin administrator (*root*).

---

## Langkah-Langkah Pemasangan

### Langkah 1: Memunculkan Folder Rahasia di Dolphin
1. Buka aplikasi **Dolphin** (File Manager).
2. Secara default, folder konfigurasi lokal disembunyikan oleh Linux. Untuk memunculkannya, tekan tombol kombinasi **Ctrl + H** pada keyboard Anda.
3. Anda akan melihat banyak folder baru bermunculan yang diawali dengan tanda titik (misal: `.config`, `.local`).

---

## Langkah 2: Masuk ke Jalur Direktori Tema Lokal
Gunakan jendela Dolphin Anda untuk masuk ke dalam folder-folder berikut secara berurutan:
1. Masuk ke folder **`.local`**
2. Masuk ke folder **`share`**
3. Masuk ke folder **`plasma`**
4. Cari folder bernama **`look-and-feel`**

> 💡 **PENTING (Jika folder tidak ada):** > Jika di dalam folder `plasma` Anda tidak menemukan folder bernama `look-and-feel`, buat sendiri secara manual.  
> Klik kanan di area kosong ➔ **Create New** ➔ **Folder** ➔ Beri nama **`look-and-feel`** (pastikan menggunakan huruf kecil semua dan menggunakan tanda strip). Masuk ke folder kosong tersebut.

---

## Langkah 3: Menyalin Folder Tema Fallout
1. Buka jendela Dolphin baru (atau tab baru) lalu pergi ke tempat Anda menyimpan unduhan tema tadi: `/home/oi/Downloads/splash/`.
2. Klik kanan pada folder tema yang ingin Anda gunakan (contoh: **`FalloutVault-Loading-Plasma6`** atau **`FalloutPipBoy-Loading-Plasma6`**), lalu pilih **Copy**.
3. Kembali ke jendela Dolphin yang membuka folder `look-and-feel` lokal tadi.
4. Klik kanan di area kosong, lalu pilih **Paste**. Proses salin-tempel ini dipastikan berjalan lancar tanpa ada eror "Permission Denied" warna kuning.

---

## Langkah 4: Menerapkan Tema via System Settings
Setelah file berhasil dipindahkan, langkah terakhir adalah mengaktifkannya:
1. Buka **System Settings** (Pengaturan Sistem).
2. Di panel sebelah kiri, pilih menu **Colors & Themes** (Warna & Tema).
3. Di dalam menu tersebut, klik sub-menu **Splash Screen** di bagian paling bawah.
4. Tema Fallout pilihan Anda (Vault atau PipBoy) sekarang sudah otomatis muncul di dalam daftar.
5. Klik gambar tema Fallout tersebut, lalu klik tombol **Apply** di pojok kanan bawah.

---

## Langkah 5: Selesai dan Uji Coba
Tutup semua jendela aplikasi dan lakukan **Log Out** atau **Restart** pada laptop Anda. Ketika Anda login kembali, animasi loading masuk ke desktop kini sudah berubah menjadi ala dunia Fallout yang keren!
