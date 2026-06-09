# Fallout 

---

<img width="1280" height="720" alt="pipboy" src="https://github.com/user-attachments/assets/5b0d3720-d6ed-4a20-9a34-ecd257bfb9cb" />

---

```markdown
# Panduan Solusi Alternatif Pemasangan Tema Fallout GRUB

Ketik perintah-perintah ini secara berurutan di posisi direktori `/tmp` kamu sekarang:

---

### 🛠️ Eksekusi Tema Fallout (Fix 100%)

**1. Kloning repositori tema Fallout murni:**
```bash
git clone [https://github.com/shvchk/fallout-grub-theme.git](https://github.com/shvchk/fallout-grub-theme.git) /tmp/fallout-v2

```
**2. Masuk ke folder hasil download-nya:**
```bash
cd /tmp/fallout-v2

```
**3. Jalankan skrip instalasi bawaannya:**
```bash
sudo ./install.sh

```
*(Tunggu sebentar sampai proses penyalinan aset gambar background RobCo hijau selesai ke sistem bootloader-mu)*.
### 🏁 Langkah Pengunci Final & Reboot
Setelah instalasinya selesai tanpa error, jalankan perintah ini agar menu GRUB kamu resmi diperbarui dengan tema barumu:
```bash
sudo grub-mkconfig -o /boot/grub/grub.cfg

```

```
reboot
``` 


<img width="1280" height="720" alt="menu fallout" src="https://github.com/user-attachments/assets/7dffbee8-8418-4498-ba39-bd28f0441710" />


