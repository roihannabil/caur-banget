# Instalasi Arch Linux (Modified)

> ✏️ **Modifikasi:** Bagian `pacstrap` dan `pacman -S` sudah diperbarui sesuai permintaan.

---

## Konek Internet

Jika menggunakan kabel LAN tinggal colok.

Jika menggunakan WiFi:

```bash
iwctl
```

```bash
device list
station wlan0 scan
station wlan0 get-networks
station wlan0 connect NamaWifi
```

Alternatif jika phy0 belum dinyalakan, keluar dulu dari iwctl:

```bash
exit
```

```bash
rfkill list
```

Kalau ada tulisan `Soft blocked: yes` atau `Hard blocked: yes`:

```bash
rfkill unblock all
rfkill unblock wifi
```

```bash
ip link set wlan0 up
```

Update sistem waktu:

```bash
timedatectl
```

---

## Partisi Disk

Di live environment kamu dapat melihat disk yang dibagi menjadi *block device* seperti `/dev/sda`, `/dev/nvme0n1`. Untuk mengidentifikasi device ini gunakan `lsblk` atau `fdisk`.

```bash
fdisk /dev/nvme0n1
```

Buat satu disk untuk LVM dan boot jika dibutuhkan.

### Format Disk

Jika buat boot, **jangan diformat ulang** kalau sebelumnya sudah ada karena dapat merusak sistem.

#### Untuk Boot

```bash
mkfs.fat -F 32 /dev/nvme0n1p6
```

Lalu buat folder dan mount bootnya, kemudian buat LVM.

Untuk PV (contoh di disk saya):

```bash
pvcreate /dev/nvme0n1p5
```

Untuk VG:

```bash
vgcreate sawit /dev/nvme0n1p5
```

Buat LV sesuai layout CIS:

```bash
lvcreate -L 30G sawit -n lroot
lvcreate -L 8G  sawit -n lvar
lvcreate -L 4G  sawit -n lvtmp
lvcreate -L 4G  sawit -n lvlog
lvcreate -L 2G  sawit -n lvaud
lvcreate -l 100%FREE sawit -n home
```

Lihat LVM:

```bash
sudo lvdisplay
```

Buat enkripsi:

```bash
sudo cryptsetup luksFormat /dev/sawit/lvroot
```

Lihat status:

```bash
sudo cryptsetup status /dev/sawit/lvroot
```

Open untuk membuat virtual device yang mapping enkripsi (`cryptroot` = nama virtual):

```bash
cryptsetup open /dev/sawit/lvroot cryptroot
```

Untuk lihat LUKS header information:

```bash
cryptsetup luksDump /dev/sawit/lvroot
```

Format partisi:

```bash
mkfs.ext4 /dev/mapper/cryptroot
mkfs.ext4 /dev/sawit/lvar
mkfs.ext4 /dev/sawit/lvtmp
mkfs.ext4 /dev/sawit/lvlog
mkfs.ext4 /dev/sawit/lvaud
mkfs.ext4 /dev/sawit/home
```

Tambahkan format boot jika perlu.

---

## Mount Partisi

```bash
mount /dev/mapper/cryptroot /mnt
mount --mkdir /dev/nvme0n1p4 /mnt/boot/efi
mount --mkdir -o rw,nodev,nosuid,relatime /dev/proc/lvar /mnt/var
mount --mkdir -o rw,nodev,nosuid,noexec,relatime /dev/sawit/ltmp /mnt/var/tmp
mount --mkdir -o rw,nodev,nosuid,noexec,relatime /dev/sawit/lvlog /mnt/var/log
mount --mkdir -o rw,nodev,nosuid,noexec,relatime /dev/sawit/lvaud /mnt/var/log/audit
mount --mkdir -o rw,nodev,nosuid,noexec,relatime /dev/sawit/home /mnt/home
```

---

## Mirror

Jika diperlukan (misalnya download masih lambat):

```bash
nano /etc/pacman.d/mirrorlist
```

---

## Install Packages Penting

> ⚠️ **DIMODIFIKASI** — Ditambahkan: `linux-hardened-headers` dan `lvm2` eksplisit sesuai permintaan.

```bash
pacstrap /mnt base linux-hardened linux-hardened-headers linux-firmware lvm2
```

> Catatan: Paket tambahan seperti `networkmanager`, `nano`, `sudo`, `grub`, `efibootmgr`, `os-prober`, `cryptsetup`, dan `amd-ucode` (jika pakai AMD) dapat ditambahkan di perintah di atas sesuai kebutuhan hardware kamu.

---

## fstab

Agar setelah Arch Linux di-boot, sistem tahu format partisi disk yang ada di mana, dibuatlah file `/etc/fstab`. File ini adalah daftar partisi yang harus otomatis dimount saat Linux menyala.

```bash
genfstab -U /mnt >> /mnt/etc/fstab
```

---

## Chroot

Masuk ke system environment:

```bash
arch-chroot /mnt
```

Set time:

```bash
ln -sf /usr/share/zoneinfo/Asia/Jakarta /etc/localtime
hwclock --systohc
```

### Localization

Masuk ke folder:

```bash
nano /etc/locale.gen
```

Buka komentarnya:

```
en_US.UTF-8 UTF-8
```

Jalankan:

```bash
locale-gen
```

Ketik:

```bash
echo "LANG=en_US.UTF-8" > /etc/locale.conf
```

Hostname:

```bash
nano /etc/hostname
```

Ketik nama hostname.

### Tambah User

```bash
useradd -m -G wheel -s /bin/bash kautsar
```

Masuk ke visudo:

```bash
EDITOR=nano visudo
```

Uncomment baris berikut:

```
# %wheel ALL=(ALL:ALL) ALL
```

Set password user:

```bash
passwd kautsar
```

### Enable NetworkManager

```bash
systemctl enable NetworkManager
```

Set password root:

```bash
passwd
```

---

## Install Paket Server (pacman)

> ⚠️ **DIMODIFIKASI** — Ditambahkan sesuai permintaan.

```bash
pacman -S nginx php-fpm php php-gd php-intl mariadb firewalld openssh git wget curl audit unzip
```

---

## Boot Loader

Install GRUB:

```bash
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=slims
```

Update GRUB (tambahkan UUID enkripsi):

```bash
echo "cryptsetup luksUUID /dev/mapper/lvroot" >> /etc/default/grub
```

```bash
nano /etc/default/grub
```

Tambahkan di bagian `GRUB_CMDLINE_LINUX`:

```
rd.luks.name=device-UUID=cryptroot root=/dev/mapper/cryptroot
```

Generate config GRUB:

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

---

## initramfs

Masuk ke:

```bash
nano /etc/mkinitcpio.conf
```

Cari baris HOOKS dan ubah menjadi:

```
HOOKS=(base systemd autodetect microcode modconf kms keyboard block lvm2 sd-encrypt filesystems fsck)
```

Jalankan:

```bash
mkinitcpio -P
```

Nyalakan os-prober di `/etc/default/grub`, lalu update bootloader:

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

---

## Reboot

```bash
exit
umount -R /mnt
reboot
```
