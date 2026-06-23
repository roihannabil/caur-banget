# utk kedua server

punya anu : https://asciinema.org/a/WTeVJ6VOCPTtGiID

punya gw : https://asciinema.org/a/wjSvjQmghfz4bwj3

# Disable Module Kernel

> **Catatan:**
> - Kalau module kernel **tidak ada/tidak aktif**, tidak perlu di-disable.
> - Kalau tidak ada output apa-apa setelah command, berarti module tersebut memang tidak aktif.

## Cek Module Kernel

### Module Kernel Cramfs
```bash
sudo lsmod | grep cramfs
```

### Module Kernel Freevxfs
```bash
sudo lsmod | grep freevxfs
```

### Module Kernel hfs
```bash
sudo lsmod | grep hfs
```

### Module Kernel hfsplus
```bash
sudo lsmod | grep hfsplus
```

### Module Kernel jffs2
```bash
sudo lsmod | grep jffs2
```

### Module Kernel overlayfs
```bash
sudo lsmod | grep overlayfs
```

### Module Kernel squashfs
```bash
sudo lsmod | grep squashfs
```

### Module Kernel udf
```bash
sudo lsmod | grep udf
```

### Module Kernel usb-storage
```bash
sudo lsmod | grep usb-storage
```

> Walaupun output `usb-storage` kosong (artinya tidak aktif), module ini **tetap harus diblokir**.

## Blokir Module usb-storage

1. Buka/edit file konfigurasi modprobe:
   ```bash
   sudo nvim /etc/modprobe.d/01-custom.conf
   ```
   Nama file `01-custom.conf` bebas, bisa diganti sesuai keinginan.

2. Tambahkan baris berikut di dalam file:
   ```
   install usb-storage /bin/false
   blacklist usb-storage
   ```
   - `/bin/false` → **disable**
   - `/bin/true` → **enable** (aktif)

<img width="183" height="95" alt="Screenshot From 2026-06-24 01-40-40" src="https://github.com/user-attachments/assets/066ebcc6-3403-465f-b985-30ec43ad4ef7" />

3. Simpan dan keluar dari nvim:
   ```
   Esc lalu :wq
   ```

4. Regenerate initramfs agar perubahan diterapkan:
   ```bash
   sudo mkinitcpio -P
   ```

5. Selesai — proses disable module kernel sudah berhasil dilakukan.
