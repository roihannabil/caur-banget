# i use arch btw

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


cryptsetup luksFormat /dev/system/crab
# Ketik YES (huruf kapital), set passphrase rahasia kamu

cryptsetup open /dev/system/crab cryptcrab

mkfs.ext4 /dev/system/root
mkfs.ext4 /dev/system/home
mkfs.ext4 /dev/mapper/cryptcrab
mkswap /dev/system/swap

# Mount ke folder /mnt installer
mount /dev/system/root /mnt
mount --mkdir /dev/nvme0n1p1        /mnt/boot   # Mengarah ke EFI bawaan laptop (260M)
mount --mkdir /dev/system/home       /mnt/home
mount --mkdir /dev/mapper/cryptcrab  /mnt/home/user
swapon /dev/system/swap

pacstrap /mnt base linux linux-headers linux-firmware lvm2 cryptsetup networkmanager nano sudo grub efibootmgr os-prober pipewire pipewire-pulse pipewire-alsa pipewire-jack wireplumber amd-ucode

genfstab -U /mnt >> /mnt/etc/fstab
# Note: Buka /mnt/etc/fstab, pastikan baris / mengarah ke /dev/system/root

arch-chroot /mnt

# Jam & Lokasi
ln -sf /usr/share/zoneinfo/Asia/Jakarta /etc/localtime
hwclock --systohc

# Locale
nano /etc/locale.gen  # Hapus tanda pagar (#) di baris en_US.UTF-8 UTF-8
locale-gen
echo "LANG=en_US.UTF-8" > /etc/locale.conf
echo "arch-utama" > /etc/hostname

# User & Root Password
useradd -m -G wheel -s /bin/bash namauser
passwd namauser
passwd root  # Set password untuk root
EDITOR=nano visudo  # Hapus tanda pagar (#) di baris %wheel ALL=(ALL:ALL) ALL

# Install Lingkungan Desktop KDE Plasma & Driver Grafis AMD
pacman -S xorg plasma-desktop sddm konsole dolphin ark kate gwenview egl-wayland network-manager-applet bluez bluez-utils bluedevil plasma-pa firefox git wget curl fastfetch htop mesa xf86-video-amdgpu vulkan-radeon vlc packagekit-packagekitd plasma-discover

# Aktifkan Layanan Desktop
systemctl enable NetworkManager
systemctl enable sddm
systemctl enable bluetooth

# Pasang crypttab & mkinitcpio
echo "cryptcrab  /dev/system/crab  none  luks" >> /etc/crypttab

nano /etc/mkinitcpio.conf
# 1. Baris MODULES=() isi menjadi: MODULES=(amdgpu)
# 2. Baris HOOKS=(...) ubah urutannya secara presisi menjadi:
# HOOKS=(base systemd autodetect microcode modconf kms keyboard sd-vconsole block lvm2 sd-encrypt filesystems fsck)

mkinitcpio -P

mkdir -p /var/lib/os-prober

nano /etc/default/grub
# 1. Cari baris GRUB_DISABLE_OS_PROBER=false, pastikan TIDAK ada tanda pagar (#)
# 2. Sesuaikan baris CMDLINE agar mengarah ke root LVM:
# GRUB_CMDLINE_LINUX="root=/dev/system/root"

grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=ARCH
grub-mkconfig -o /boot/grub/grub.cfg

exit
umount -R /mnt
reboot
