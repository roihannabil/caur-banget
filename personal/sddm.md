<img width="4064" height="3048" alt="1000231983" src="https://github.com/user-attachments/assets/b391d0ca-dba2-4fb7-9987-61fac31ce559" />


# Warnet

Aduh, maaf banget! Sepertinya sistem saya sedang mengalami gangguan teknis dalam membuatkan file unduhan kustom secara langsung di sini.
Agar Anda tidak terhambat dan tetap bisa menyimpan file .md ini di laptop, kita pakai cara alternatif yang sangat mudah. Anda bisa membuatnya sendiri dalam hitungan detik:
### Cara Membuat File .md Sendiri:
 1. Buka aplikasi **Text Editor** bawaan (misalnya **KWrite**, **Kate**, atau **Mousepad**) di KDE Plasma Anda.
 2. *Copy* (salin) seluruh teks di dalam kotak abu-abu di bawah ini:
```markdown
# Panduan Cara Memasang Tema SDDM (Billing Warnet) di Arch Linux / KDE Plasma 6

Panduan ini dibuat khusus untuk memperbaiki struktur konfigurasi SDDM yang mengalami error `is not a directory` dan menerapkan tema kustom **Billing-Warnet-KDE-SDDM-main** secara manual melalui Terminal.

---

## Langkah 1: Masuk Sebagai Root (Administrator)
Buka aplikasi **Konsole** atau **Terminal**, lalu jalankan perintah berikut untuk masuk ke mode root agar semua perintah berikutnya tidak memerlukan teks `sudo` lagi:

```bash
sudo su

```
*Masukkan password laptop Anda jika diminta (teks password memang tidak akan muncul saat diketik, langsung tekan Enter saja).*
## Langkah 2: Memperbaiki Struktur Folder Konfigurasi yang Rusak
Error Path '/etc/sddm.conf.d' is not a directory terjadi karena jalur tersebut berupa file usang, bukan sebuah folder. Kita perlu menghapusnya dan membuat folder yang benar dengan perintah berikut:
```bash
rm -rf /etc/sddm.conf.d
mkdir -p /etc/sddm.conf.d

```
## Langkah 3: Membuat dan Mengisi File Konfigurasi Tema
Setelah foldernya berhasil dibuat secara benar, buat file konfigurasi baru di dalamnya menggunakan teks editor nano:
```bash
nano /etc/sddm.conf.d/theme.conf

```
Di dalam layar hitam editor nano yang kosong, ketikkan teks berikut secara persis:
```text
[Theme]
Current=Billing-Warnet-KDE-SDDM-main

```
### Cara Menyimpan File Nano:
 1. Tekan tombol **Ctrl + O** pada keyboard, lalu tekan **Enter** untuk menyimpan.
 2. Tekan tombol **Ctrl + X** untuk keluar dari editor nano dan kembali ke terminal biasa.
## Langkah 4: Memasang Dependensi Grafis (Paling Krusial!)
KDE Plasma 6 menggunakan Qt6, sedangkan tema kustom seperti tema Billing Warnet ini sering kali masih membutuhkan beberapa komponen Qt5 agar aspek visualnya tidak *crash* atau kembali ke tampilan dasar (layar biru polos).
Jalankan perintah ini untuk memasang semua pustaka (*library*) pendukung yang diperlukan:
```bash
pacman -S --needed qt6-5compat qt5-graphicaleffects qt5-quickcontrols2

```
*Jika muncul pilihan instalasi atau konfirmasi [Y/n], langsung tekan **Enter** saja terus sampai prosesnya selesai.*
## Langkah 5: Selesai dan Uji Coba
Semua rintangan struktur file dan dependensi sistem kini sudah diperbaiki. Tutup terminal Anda dan lakukan restart pada laptop dengan perintah:
```bash
reboot

```
Setelah laptop menyala kembali, Anda akan langsung disambut oleh layar login legendaris bertemakan billing warnet!
```

3. *Paste* (tempel) teks tersebut ke dalam Text Editor.
4. Klik **Save As**, lalu beri nama file tersebut **`Panduan_SDDM.md`** (pastikan ekstensinya berakhiran `.md`, bukan `.txt`).

Dengan cara ini, Anda sudah punya file dokumen Markdown resmi yang tersimpan aman di laptop Anda!

```
<img width="4064" height="3048" alt="1000231982" src="https://github.com/user-attachments/assets/fb8e74c9-28c0-4e53-b751-6652c78b6365" />






