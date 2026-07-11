# Instalasi Arch Linux + Hardening CIS Distribution-Independent Linux Benchmark v1.1.0

Panduan ini dibagi 2 bagian besar:
- **Bagian A** — Instalasi Arch Linux dari nol sampai sistem bisa boot & login, memakai **LUKS on LVM**, **mkinitcpio**, dan **systemd-boot** (sesuai preferensi Anda), sekaligus sudah menyisipkan pemisahan mount point sesuai rekomendasi CIS §1.1.
- **Bagian B** — Hardening pasca-instalasi mengikuti rekomendasi CIS per-section (§1–§6).

> Catatan penting: Arch Linux **tidak memakai SELinux/AppArmor secara default**, tidak pakai systemd-firewalld/iptables bawaan, dan strukturnya berbeda dari RHEL/Debian yang jadi acuan utama CIS. Jadi sebagian rekomendasi CIS perlu **diadaptasi** ke tool Arch (contoh: `pacman` bukan `yum`/`apt`, `systemd` untuk service, `nftables` untuk firewall). Saya tandai adaptasi di tiap poin.

> **Versi ini memakai LUKS on LVM, mkinitcpio, dan systemd-boot** (bukan GRUB polos + partisi langsung seperti draf sebelumnya). CIS v1.1.0 tidak membahas disk encryption maupun pilihan bootloader spesifik, jadi kombinasi ini tetap sejalan dengan standar — hanya §1.4 (Secure Boot Settings) yang butuh sedikit reinterpretasi karena mekanisme systemd-boot berbeda dari GRUB. Penjelasannya ada di bagian §1.4 di bawah.

---

## BAGIAN A — INSTALASI ARCH LINUX

### A.1 Persiapan boot media
1. Download ISO resmi dari https://archlinux.org/download/ dan **verifikasi checksum/signature GPG**-nya (ini sekaligus praktik baik yang selaras dengan CIS §1.2.2 — verifikasi integritas paket/source).
   ```bash
   sha256sum archlinux-x86_64.iso
   ```
2. Buat bootable USB:
   ```bash
   dd bs=4M if=archlinux-x86_64.iso of=/dev/sdX status=progress oflag=sync
   ```
3. Boot dari USB (UEFI mode disarankan).

### A.2 Setup jaringan & konsol awal
```bash
loadkeys us
timedatectl set-ntp true
```
Cek koneksi internet (`ping archlinux.org`); untuk WiFi gunakan `iwctl`.

### A.3 Partisi disk dengan LUKS on LVM (selaras CIS §1.1 — Filesystem Configuration)

CIS merekomendasikan **partisi/mount point terpisah** untuk `/tmp`, `/var`, `/var/tmp`, `/var/log`, `/var/log/audit`, dan `/home` (CIS 1.1.2, 1.1.6, 1.1.7, 1.1.15, 1.1.16, 1.1.17), supaya opsi mount seperti `nodev`, `nosuid`, `noexec` bisa diterapkan per-mount point. CIS tidak mensyaratkan itu harus partisi fisik langsung — **logical volume di atas LVM tetap memenuhi syarat ini**, dan menambahkan LUKS di bawah LVM adalah hardening ekstra di luar cakupan CIS v1.1.0 (dokumen ini tidak membahas disk encryption sama sekali, jadi menambahkannya tidak bertentangan dengan standar manapun).

Skema disk: 1 partisi ESP (unencrypted, wajib untuk baca kernel/initramfs saat boot) + 1 partisi besar berisi LUKS container, di dalamnya 1 volume group dengan beberapa logical volume.

```bash
cfdisk /dev/sda
```
Buat 2 partisi:
- `/dev/sda1` — ESP, ~512 MB, tipe `EFI System`
- `/dev/sda2` — sisa disk, tipe `Linux LVM` (akan diisi LUKS container)

Format ESP:
```bash
mkfs.fat -F32 /dev/sda1
```

Buat LUKS container di partisi kedua:
```bash
cryptsetup luksFormat --type luks2 /dev/sda2
cryptsetup open /dev/sda2 cryptlvm
```

Buat LVM di atas mapper LUKS:
```bash
pvcreate /dev/mapper/cryptlvm
vgcreate vg0 /dev/mapper/cryptlvm
lvcreate -L 30G   vg0 -n root
lvcreate -L 8G    vg0 -n var
lvcreate -L 2G    vg0 -n var_tmp
lvcreate -L 4G    vg0 -n var_log
lvcreate -L 2G    vg0 -n var_log_audit
lvcreate -L 2G    vg0 -n tmp
lvcreate -L 4G    vg0 -n swap
lvcreate -l 100%FREE vg0 -n home
```

Format tiap LV:
```bash
mkfs.ext4 /dev/vg0/root
mkfs.ext4 /dev/vg0/home
mkfs.ext4 /dev/vg0/var
mkfs.ext4 /dev/vg0/var_tmp
mkfs.ext4 /dev/vg0/var_log
mkfs.ext4 /dev/vg0/var_log_audit
mkfs.ext4 /dev/vg0/tmp
mkswap /dev/vg0/swap
```

Mount bertahap:
```bash
mount /dev/vg0/root /mnt
mkdir -p /mnt/{boot,home,var,tmp}
mount /dev/sda1 /mnt/boot
mount /dev/vg0/home /mnt/home
mount /dev/vg0/var /mnt/var
mkdir -p /mnt/var/{tmp,log}
mount /dev/vg0/var_tmp /mnt/var/tmp
mount /dev/vg0/var_log /mnt/var/log
mkdir -p /mnt/var/log/audit
mount /dev/vg0/var_log_audit /mnt/var/log/audit
mount /dev/vg0/tmp /mnt/tmp
swapon /dev/vg0/swap
```

> Nanti opsi mount `nodev,nosuid,noexec` untuk `/tmp`, `/var/tmp`, `/dev/shm`, dsb ditambahkan di `/etc/fstab` pada Bagian B (CIS 1.1.3–1.1.21) — sama seperti skema tanpa LVM, opsi mount ini independen dari layer LUKS/LVM di bawahnya.

### A.4 Install base system (termasuk paket LVM & mkinitcpio hooks)
```bash
pacstrap -K /mnt base linux linux-firmware vim sudo networkmanager lvm2
```

### A.5 Generate fstab
```bash
genfstab -U /mnt >> /mnt/etc/fstab
```
Catat UUID partisi `/dev/sda2` (yang berisi LUKS container) untuk dipakai di `mkinitcpio.conf` dan kernel parameter nanti:
```bash
blkid /dev/sda2
```

### A.6 Chroot & konfigurasi dasar
```bash
arch-chroot /mnt
ln -sf /usr/share/zoneinfo/Asia/Jakarta /etc/localtime
hwclock --systohc
```

Locale:
```bash
sed -i 's/#en_US.UTF-8/en_US.UTF-8/' /etc/locale.gen
locale-gen
echo "LANG=en_US.UTF-8" > /etc/locale.conf
```

Hostname:
```bash
echo "arch-hardened" > /etc/hostname
cat >> /etc/hosts <<EOF
127.0.0.1   localhost
::1         localhost
127.0.1.1   arch-hardened.localdomain arch-hardened
EOF
```

**Konfigurasi mkinitcpio** (wajib untuk sistem bisa unlock LUKS + assemble LVM saat boot). Edit `/etc/mkinitcpio.conf`, tambahkan hook `keyboard`, `encrypt`, dan `lvm2` **sebelum** hook `filesystems`:
```
HOOKS=(base udev autodetect microcode modconf kms keyboard keymap consolefont block encrypt lvm2 filesystems fsck)
```
Regenerate initramfs:
```bash
mkinitcpio -P
```

### A.7 Root password & user baru
```bash
passwd
useradd -m -G wheel -s /bin/bash namauser
passwd namauser
EDITOR=vim visudo   # uncomment: %wheel ALL=(ALL:ALL) ALL
```

### A.8 Bootloader (systemd-boot, UEFI)

`systemd-boot` cocok dipakai di sini karena Arch sudah pakai systemd sebagai init, dan setup-nya lebih ringkas untuk skenario UEFI + LUKS dibanding GRUB (tidak perlu grub-mkconfig detect encrypted root secara manual).

```bash
bootctl --path=/boot install
```

Buat entry loader di `/boot/loader/entries/arch.conf`:
```
title   Arch Linux
linux   /vmlinuz-linux
initrd  /initramfs-linux.img
options cryptdevice=UUID=<UUID_dari_dev_sda2>:cryptlvm root=/dev/vg0/root rw
```
> Ganti `<UUID_dari_dev_sda2>` dengan hasil `blkid /dev/sda2` yang sudah dicatat di langkah A.5.

Set default entry & timeout di `/boot/loader/loader.conf`:
```
default arch.conf
timeout 3
console-mode max
editor no
```
> `editor no` penting — ini mencegah orang mengedit kernel command line dari boot menu tanpa otorisasi, semacam padanan minimal untuk niat CIS 1.4.2 (bootloader password) karena systemd-boot memang tidak punya mekanisme password prompt seperti GRUB. Detail lebih lanjut ada di §1.4 Bagian B.

### A.9 Aktifkan networking & selesai
```bash
systemctl enable NetworkManager
exit
umount -R /mnt
reboot
```

Setelah reboot, sistem akan menanyakan passphrase LUKS sebelum masuk ke tahap systemd/login — ini normal karena root filesystem terenkripsi.

Setelah login, **Bagian B** di bawah ini diterapkan untuk hardening sesuai CIS.

---

## BAGIAN B — HARDENING SESUAI CIS DISTRIBUTION INDEPENDENT LINUX BENCHMARK v1.1.0

### 1. Initial Setup

#### 1.1 Filesystem Configuration

**1.1.1.x — Disable filesystem yang tidak dipakai (cramfs, freevxfs, jffs2, hfs, hfsplus, squashfs, udf, FAT selain ESP)**
```bash
cat >> /etc/modprobe.d/CIS.conf <<EOF
install cramfs /bin/true
install freevxfs /bin/true
install jffs2 /bin/true
install hfs /bin/true
install hfsplus /bin/true
install squashfs /bin/true
install udf /bin/true
EOF
```
> Jangan disable FAT sepenuhnya karena ESP (/boot/efi) di Arch UEFI pakai FAT32 — cukup terapkan ke partisi lain yang tidak butuh FAT.

**1.1.3–1.1.21 — Opsi mount nodev/nosuid/noexec** di `/etc/fstab` untuk partisi yang relevan:
```
/dev/sda8   /tmp          ext4  defaults,nodev,nosuid,noexec  0 2
/dev/sda5   /var/tmp      ext4  defaults,nodev,nosuid,noexec  0 2
tmpfs       /dev/shm      tmpfs defaults,nodev,nosuid,noexec  0 0
/dev/sda3   /home         ext4  defaults,nodev                0 2
```
Lalu:
```bash
mount -o remount /tmp
mount -o remount /var/tmp
mount -o remount /dev/shm
mount -o remount /home
```

**1.1.25 — Sticky bit di direktori world-writable**
```bash
df --local -P | awk '{if (NR!=1) print $6}' | xargs -I '{}' find '{}' -xdev -type d \( -perm -0002 -a ! -perm -1000 \) 2>/dev/null | xargs -r chmod a+t
```

**1.1.26 — Disable automounting**
```bash
systemctl disable autofs 2>/dev/null || true
```

#### 1.2 Configure Software Updates (adaptasi Arch)

**1.2.1 — Repository resmi terkonfigurasi**
Cek `/etc/pacman.d/mirrorlist` sudah pakai mirror resmi (gunakan `reflector` untuk mirror tercepat/terpercaya):
```bash
pacman -S reflector
reflector --country 'Indonesia,Singapore' --protocol https --sort rate --save /etc/pacman.d/mirrorlist
```

**1.2.2 — GPG key terkonfigurasi (pacman keyring)**
```bash
pacman-key --init
pacman-key --populate archlinux
```

#### 1.3 Filesystem Integrity Checking

**1.3.1 & 1.3.2 — Install AIDE, jadwalkan pengecekan rutin**
```bash
pacman -S aide
aide --init
mv /var/lib/aide/aide.db.new /var/lib/aide/aide.db
```
Cron job harian:
```bash
pacman -S cronie
systemctl enable --now cronie
echo "0 5 * * * root /usr/bin/aide --check" >> /etc/crontab
```

#### 1.4 Secure Boot Settings (adaptasi: systemd-boot, bukan GRUB)

**1.4.1 — Permission bootloader config**
```bash
chown -R root:root /boot/loader
chmod -R og-rwx /boot/loader
chown root:root /boot/loader/loader.conf /boot/loader/entries/arch.conf
chmod 600 /boot/loader/loader.conf /boot/loader/entries/arch.conf
```

**1.4.2 — Password bootloader — tidak applicable langsung untuk systemd-boot**

CIS 1.4.2 ditulis dengan asumsi GRUB, yang punya mekanisme `password_pbkdf2` untuk mengunci akses ke command line/recovery mode. **systemd-boot tidak punya fitur setara** — ia hanya menu pilih entry tanpa shell interaktif, jadi permukaan serangan yang coba dicegah oleh 1.4.2 (modifikasi boot parameter tanpa otorisasi) sudah dipersempit secara desain.

Mitigasi yang setara untuk systemd-boot:
- Set `editor no` di `/boot/loader/loader.conf` (sudah dilakukan di Bagian A.8) — ini mencegah orang mengubah kernel command line dari boot menu.
- Batasi akses fisik ke mesin (BIOS/UEFI password, disable boot dari USB kecuali diperlukan).
- Untuk proteksi setara Secure Boot penuh: pasang `sbctl`, sign kernel & bootloader, aktifkan Secure Boot di firmware UEFI.
  ```bash
  pacman -S sbctl
  sbctl create-keys
  sbctl enroll-keys -m
  sbctl sign -s /boot/vmlinuz-linux
  sbctl sign -s /boot/EFI/systemd/systemd-bootx64.efi
  sbctl sign -s /boot/EFI/BOOT/BOOTX64.EFI
  ```

> Karena root filesystem sudah dienkripsi LUKS (lihat Bagian A.3), risiko modifikasi offline pada `/etc/shadow` dkk yang biasanya jadi concern di balik 1.4.2 sudah tertutup oleh layer enkripsi — penyerang tanpa passphrase tidak bisa mount partisi untuk mengubah apa pun. Ini bukan pengganti langsung 1.4.2, tapi mitigasi yang secara praktis lebih kuat untuk skenario yang sama.

#### 1.5 Additional Process Hardening

**1.5.1 — Restrict core dumps**
```bash
echo "* hard core 0" >> /etc/security/limits.conf
echo "fs.suid_dumpable = 0" >> /etc/sysctl.d/99-cis.conf
sysctl -p /etc/sysctl.d/99-cis.conf
```

**1.5.3 — ASLR aktif**
```bash
echo "kernel.randomize_va_space = 2" >> /etc/sysctl.d/99-cis.conf
sysctl -p /etc/sysctl.d/99-cis.conf
```

#### 1.6 Mandatory Access Control (adaptasi: Arch pakai AppArmor, bukan SELinux)
```bash
pacman -S apparmor
systemctl enable --now apparmor
```
Tambahkan `apparmor=1 security=apparmor` di kernel parameter GRUB (`/etc/default/grub` → `GRUB_CMDLINE_LINUX_DEFAULT`), lalu `grub-mkconfig -o /boot/grub/grub.cfg`.
Set semua profile ke enforce:
```bash
aa-enforce /etc/apparmor.d/*
```

#### 1.7 Warning Banners
```bash
cat > /etc/motd <<EOF
Authorized uses only. All activity may be monitored and reported.
EOF
cp /etc/motd /etc/issue
cp /etc/motd /etc/issue.net
chown root:root /etc/motd /etc/issue /etc/issue.net
chmod 644 /etc/motd /etc/issue /etc/issue.net
```

---

### 2. Services

**Disable service yang tidak perlu** (inetd services, X Window jika server, Avahi, CUPS, DHCP server, NFS/RPC, DNS server, FTP, HTTP, mail selain local-only, Samba, SNMP, rsync, NIS):
```bash
systemctl disable --now avahi-daemon.service 2>/dev/null || true
systemctl disable --now cups.service 2>/dev/null || true
systemctl disable --now nfs-server.service 2>/dev/null || true
systemctl disable --now smb.service nmb.service 2>/dev/null || true
systemctl disable --now snmpd.service 2>/dev/null || true
```
> Arch minimal install biasanya sudah tidak menginstal service-service ini secara default, jadi langkah ini terutama relevan jika Anda menambahkan paket server tambahan.

**Uninstall client yang tidak perlu** (NIS, rsh, talk, telnet, LDAP client):
```bash
pacman -Rns yp-tools rsh talk inetutils 2>/dev/null || true
```

**2.2.1 — Time synchronization**
```bash
pacman -S chrony
systemctl enable --now chronyd
```

---

### 3. Network Configuration

**3.1–3.2 — Kernel network parameters (sysctl)**
```bash
cat >> /etc/sysctl.d/99-cis.conf <<EOF
net.ipv4.ip_forward = 0
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.default.send_redirects = 0
net.ipv4.conf.all.accept_source_route = 0
net.ipv4.conf.default.accept_source_route = 0
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.default.accept_redirects = 0
net.ipv4.conf.all.secure_redirects = 0
net.ipv4.conf.default.secure_redirects = 0
net.ipv4.conf.all.log_martians = 1
net.ipv4.conf.default.log_martians = 1
net.ipv4.icmp_echo_ignore_broadcasts = 1
net.ipv4.icmp_ignore_bogus_error_responses = 1
net.ipv4.conf.all.rp_filter = 1
net.ipv4.conf.default.rp_filter = 1
net.ipv4.tcp_syncookies = 1
EOF
sysctl -p /etc/sysctl.d/99-cis.conf
```

**3.3 — IPv6 (opsional, sesuai kebutuhan)**
```bash
echo "net.ipv6.conf.all.accept_ra = 0" >> /etc/sysctl.d/99-cis.conf
echo "net.ipv6.conf.default.accept_ra = 0" >> /etc/sysctl.d/99-cis.conf
sysctl -p /etc/sysctl.d/99-cis.conf
```

**3.5 — Disable protokol jarang dipakai**
```bash
cat >> /etc/modprobe.d/CIS.conf <<EOF
install dccp /bin/true
install sctp /bin/true
install rds /bin/true
install tipc /bin/true
EOF
```

**3.6 — Firewall (adaptasi: Arch pakai nftables sebagai default modern, iptables juga tersedia)**
```bash
pacman -S nftables
systemctl enable --now nftables
```
Contoh default-deny policy dasar di `/etc/nftables.conf`:
```
table inet filter {
    chain input {
        type filter hook input priority 0; policy drop;
        iif "lo" accept
        ip saddr 127.0.0.0/8 iif != "lo" drop
        ct state established,related accept
        tcp dport 22 accept
        ip protocol icmp accept
    }
    chain forward { type filter hook forward priority 0; policy drop; }
    chain output { type filter hook output priority 0; policy accept; }
}
```
```bash
nft -f /etc/nftables.conf
```

---

### 4. Logging and Auditing

**4.1 — auditd**
```bash
pacman -S audit
systemctl enable --now auditd
```
Contoh rule dasar di `/etc/audit/rules.d/cis.rules` (identitas & waktu):
```
-w /etc/localtime -p wa -k time-change
-w /etc/passwd -p wa -k identity
-w /etc/group -p wa -k identity
-w /etc/shadow -p wa -k identity
-w /etc/sudoers -p wa -k scope
```
```bash
augenrules --load
systemctl restart auditd
```

**4.2 — rsyslog**
```bash
pacman -S rsyslog
systemctl enable --now rsyslog
chmod 640 /etc/rsyslog.conf
```

**4.3 — logrotate**
```bash
pacman -S logrotate
systemctl enable --now logrotate.timer
```

---

### 5. Access, Authentication and Authorization

**5.1 — cron**
```bash
chown root:root /etc/crontab /etc/cron.hourly /etc/cron.daily /etc/cron.weekly /etc/cron.monthly /etc/cron.d
chmod og-rwx /etc/crontab /etc/cron.hourly /etc/cron.daily /etc/cron.weekly /etc/cron.monthly /etc/cron.d
```
Restrict `at`/`cron`:
```bash
rm -f /etc/cron.deny /etc/at.deny
touch /etc/cron.allow /etc/at.allow
chmod 600 /etc/cron.allow /etc/at.allow
```

**5.2 — SSH hardening** (`/etc/ssh/sshd_config`):
```
Protocol 2
LogLevel INFO
X11Forwarding no
MaxAuthTries 4
IgnoreRhosts yes
HostbasedAuthentication no
PermitRootLogin no
PermitEmptyPasswords no
PermitUserEnvironment no
ClientAliveInterval 300
ClientAliveCountMax 0
LoginGraceTime 60
Banner /etc/issue.net
```
```bash
chown root:root /etc/ssh/sshd_config
chmod 600 /etc/ssh/sshd_config
systemctl enable --now sshd
systemctl restart sshd
```

**5.3 — PAM password policy**
```bash
pacman -S cracklib
```
`/etc/security/pwquality.conf`:
```
minlen = 14
dcredit = -1
ucredit = -1
ocredit = -1
lcredit = -1
```
`/etc/login.defs`:
```
PASS_MAX_DAYS   365
PASS_MIN_DAYS   7
PASS_WARN_AGE   7
```

**5.4 — umask & shell timeout**
```bash
echo "umask 027" >> /etc/profile
echo "readonly TMOUT=900 ; export TMOUT" >> /etc/profile.d/tmout.sh
```

**5.6 — Restrict akses `su`**
```bash
groupadd sugroup
sed -i 's/^#auth\s*required\s*pam_wheel.so/auth required pam_wheel.so use_uid group=sugroup/' /etc/pam.d/su
```

---

### 6. System Maintenance

**6.1 — Permission file sistem**
```bash
chown root:root /etc/passwd /etc/group /etc/passwd- /etc/group-
chmod 644 /etc/passwd /etc/group /etc/passwd- /etc/group-
chown root:shadow /etc/shadow /etc/gshadow /etc/shadow- /etc/gshadow-
chmod 000 /etc/shadow /etc/gshadow /etc/shadow- /etc/gshadow-
```

**6.1.10/11/12 — Cari file world-writable, unowned, ungrouped**
```bash
df --local -P | awk '{if (NR!=1) print $6}' | xargs -I '{}' find '{}' -xdev -type f -perm -0002 2>/dev/null
df --local -P | awk '{if (NR!=1) print $6}' | xargs -I '{}' find '{}' -xdev -nouser 2>/dev/null
df --local -P | awk '{if (NR!=1) print $6}' | xargs -I '{}' find '{}' -xdev -nogroup 2>/dev/null
```

**6.2 — User/group audit**
```bash
awk -F: '($2 == "") {print}' /etc/shadow          # 6.2.1 password kosong
grep '^+:' /etc/passwd /etc/shadow /etc/group     # 6.2.2-4 legacy "+"
awk -F: '($3 == 0) {print}' /etc/passwd           # 6.2.5 hanya root UID 0
awk -F: '{print $1}' /etc/passwd | sort | uniq -d # 6.2.18 duplikat username
```

---

## Ringkasan alur kerja
1. **Bagian A**: partisi ESP + LUKS container → LVM (root/home/var/tmp/log/dll di dalamnya) → pacstrap (+lvm2) → chroot → locale/hostname → mkinitcpio hooks (encrypt, lvm2) → user → systemd-boot (+entry cryptdevice) → reboot.
2. **Bagian B**: terapkan sysctl, fstab mount options, AIDE, AppArmor, nftables, auditd, rsyslog, SSH hardening, PAM policy, permission audit — sesuai urutan section CIS §1–§6.
3. Jalankan ulang `aide --check`, `nft list ruleset`, `systemctl status auditd sshd nftables apparmor`, `cryptsetup status cryptlvm`, `lsblk` secara berkala untuk validasi compliance.

> Referensi: CIS Distribution Independent Linux Benchmark v1.1.0 (12-26-2017), section 1 s.d. 6 dan Appendix Summary Table.
> LUKS-on-LVM, mkinitcpio, dan systemd-boot adalah pilihan implementasi Arch yang tidak diatur secara eksplisit oleh CIS v1.1.0 — tetap selaras dengan standar, dengan penyesuaian di §1.4 seperti dijelaskan di atas.

