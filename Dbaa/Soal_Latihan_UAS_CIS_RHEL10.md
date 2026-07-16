# Soal Latihan UAS — CIS Red Hat Enterprise Linux 10 Benchmark v1.0.1

**Format:** Pilihan ganda, jawaban bisa lebih dari satu (multi-select) kecuali disebutkan lain.
**Jumlah soal:** 50
**Sumber:** CIS RHEL 10 Benchmark v1.0.1 (dokumen yang diunggah)

---

### 1. Mengapa filesystem seperti `cramfs`, `freevxfs`, `hfs`, `jffs2`, dan `udf` direkomendasikan untuk dinonaktifkan sebagai kernel module?
A. Karena filesystem tersebut tidak mendukung enkripsi
B. Untuk mengurangi local attack surface pada sistem
C. Karena filesystem tersebut sudah deprecated oleh kernel Linux
D. Karena tidak digunakan secara umum dan jarang mendapat maintenance/dukungan berkelanjutan

### 2. Perintah mana yang benar untuk memverifikasi bahwa kernel module `cramfs` tidak dapat dimuat (not loadable)?
A. `lsmod | grep cramfs`
B. `modprobe --showconfig | grep -P -- '\b(install|blacklist)\h+cramfs\b'`
C. `rmmod cramfs`
D. `systemctl status cramfs`

### 3. Opsi mount apa saja yang direkomendasikan CIS untuk partisi `/dev/shm`? (pilih semua yang benar)
A. `nodev`
B. `nosuid`
C. `noexec`
D. `ro` (read-only)

### 4. Apa fungsi opsi mount `nodev` pada `/dev/shm`?
A. Mencegah eksekusi biner dari partisi tersebut
B. Mencegah keberadaan special device file pada partisi tersebut
C. Mencegah proses SUID berjalan
D. Membuat partisi menjadi read-only

### 5. Tiga mode operasi SELinux menurut CIS Benchmark adalah:
A. Enforcing, Permissive, Disabled
B. Active, Passive, Inactive
C. Strict, Targeted, MLS
D. On, Off, Audit-only

### 6. Mode SELinux yang direkomendasikan sebagai default untuk produksi adalah:
A. Permissive
B. Disabled
C. Enforcing
D. Targeted

### 7. Bagaimana cara memverifikasi mode SELinux yang sedang berjalan saat ini?
A. `sestatus -v`
B. `getenforce`
C. `cat /etc/selinux/config`
D. `semanage permissive -l`

### 8. Jika SELinux dalam keadaan disabled dan ingin diaktifkan kembali, langkah tambahan apa yang wajib dilakukan sebelum reboot?
A. `touch /.autorelabel`
B. `setenforce 1`
C. `semanage login -a`
D. `restorecon -Rv /`

### 9. Nilai `SELINUXTYPE` yang dianggap sesuai (compliant) menurut CIS adalah:
A. `targeted` atau `mls`
B. `strict` atau `minimum`
C. `default` atau `custom`
D. `unconfined` atau `permissive`

### 10. Mengapa CIS merekomendasikan bootloader (GRUB) diberi password?
A. Agar user biasa tidak bisa mengubah kernel parameter saat boot untuk bypass keamanan (single-user mode)
B. Agar sistem lebih cepat booting
C. Untuk mencegah dual-boot dengan OS lain
D. Untuk mengenkripsi partisi boot

### 11. Parameter kernel `fs.protected_hardlinks` dan `fs.protected_symlinks` berfungsi untuk:
A. Mempercepat filesystem
B. Mencegah user mengeksploitasi race condition melalui hard link/symbolic link pada direktori yang bisa ditulis banyak user
C. Mengenkripsi symlink
D. Menonaktifkan symlink sepenuhnya

### 12. Fungsi `kernel.randomize_va_space = 2` adalah:
A. Menonaktifkan ASLR (Address Space Layout Randomization)
B. Mengaktifkan ASLR secara penuh untuk stack, heap, dan library
C. Mengacak nomor PID proses
D. Mengacak alamat MAC pada network interface

### 13. `kernel.dmesg_restrict` yang diaktifkan bertujuan untuk:
A. Membatasi user non-root membaca output buffer kernel ring (`dmesg`)
B. Menonaktifkan logging kernel
C. Mempercepat proses boot
D. Menyembunyikan pesan error dari systemd

### 14. System-wide crypto policy pada RHEL yang **tidak** direkomendasikan digunakan adalah:
A. DEFAULT
B. FUTURE
C. LEGACY
D. FIPS

### 15. Mengapa CIS merekomendasikan menonaktifkan algoritma hash SHA1 dalam crypto policy?
A. SHA1 sudah dianggap lemah secara kriptografis dan rentan terhadap collision attack
B. SHA1 tidak didukung oleh RHEL 10
C. SHA1 memperlambat proses enkripsi
D. SHA1 hanya digunakan untuk checksum file, bukan keamanan

### 16. File apa saja yang perlu dikonfigurasi dengan warning banner sesuai kebijakan (bukan menampilkan info OS/versi)?
A. `/etc/motd`
B. `/etc/issue`
C. `/etc/issue.net`
D. `/etc/hostname`

### 17. Mengapa isi banner login sebaiknya tidak menampilkan nama sistem operasi dan versinya?
A. Agar tidak memberi informasi yang memudahkan penyerang (information disclosure)
B. Karena melanggar lisensi Red Hat
C. Karena akan memperlambat login
D. Karena systemd tidak mendukung teks panjang

### 18. Pada konfigurasi GDM (GNOME Display Manager), pengaturan berikut ini termasuk rekomendasi CIS **kecuali**:
A. GDM login banner dikonfigurasi
B. GDM disable-user-list diaktifkan
C. GDM screen lock dikonfigurasi
D. GDM auto-login untuk mempercepat akses administrator

### 19. Layanan (service) berikut yang direkomendasikan untuk **tidak digunakan** pada server produksi hardened menurut CIS meliputi: (pilih semua yang benar)
A. `telnet-server`
B. `rsync`
C. `chronyd`
D. `tftp-server`

### 20. Mengapa layanan `telnet` dianggap tidak aman dan direkomendasikan untuk dihapus?
A. Karena mentransmisikan data termasuk kredensial dalam bentuk plaintext (tidak terenkripsi)
B. Karena sudah tidak kompatibel dengan RHEL 10
C. Karena menggunakan port yang sama dengan SSH
D. Karena membutuhkan lisensi tambahan

### 21. Layanan client yang direkomendasikan untuk tidak diinstal karena berpotensi tidak aman meliputi:
A. `ftp` client
B. `telnet` client
C. `openssh-client`
D. `ldap` client (tanpa TLS)

### 22. Mengapa time synchronization (chrony/NTP) penting dari sisi keamanan sistem?
A. Agar timestamp log akurat, yang penting untuk investigasi forensik dan korelasi kejadian keamanan
B. Agar CPU clock lebih presisi
C. Untuk mempercepat booting
D. Untuk load balancing jaringan

### 23. Mengapa `chronyd` direkomendasikan untuk **tidak** dijalankan sebagai user root?
A. Menerapkan prinsip least privilege agar dampak kompromi layanan tersebut minimal
B. Karena root tidak memiliki izin membaca file konfigurasi chrony
C. Karena systemd melarangnya secara default
D. Karena akan menyebabkan konflik dengan NTP

### 24. Kernel module jaringan berikut yang direkomendasikan dinonaktifkan karena jarang digunakan dan menambah attack surface meliputi:
A. `dccp`
B. `sctp`
C. `tipc`
D. `ipv4`

### 25. Parameter kernel `net.ipv4.conf.all.accept_redirects = 0` berfungsi untuk:
A. Mencegah sistem menerima ICMP redirect yang dapat digunakan untuk serangan man-in-the-middle
B. Menonaktifkan IPv4 sepenuhnya
C. Mempercepat routing
D. Menonaktifkan ping

### 26. Fungsi utama `firewalld` dalam konteks CIS Host Based Firewall adalah:
A. Menyediakan manajemen firewall berbasis zone yang dapat dikonfigurasi secara dinamis
B. Mengganti fungsi SELinux
C. Mengenkripsi seluruh traffic jaringan
D. Menggantikan iptables sepenuhnya tanpa kompatibilitas

### 27. Konfigurasi firewalld yang direkomendasikan terkait loopback traffic adalah:
A. Loopback traffic (lo) diizinkan (accept)
B. Traffic dari source address loopback yang datang dari interface non-loopback diblokir
C. Loopback traffic diblokir seluruhnya
D. Loopback interface dinonaktifkan

### 28. File konfigurasi utama SSH server yang permission-nya harus dibatasi (misalnya 600 root:root) adalah:
A. `/etc/ssh/sshd_config`
B. `/etc/ssh/ssh_host_*_key` (private host key)
C. `/etc/passwd`
D. `/etc/ssh/ssh_host_*_key.pub` (public host key)

### 29. Parameter `PermitRootLogin` pada `sshd_config` yang direkomendasikan CIS adalah:
A. `yes`
B. `no`
C. `without-password`
D. `forced-commands-only`

### 30. Mengapa `PermitEmptyPasswords` pada SSH harus di-disable?
A. Agar tidak ada akun yang bisa login SSH tanpa memasukkan password sama sekali
B. Agar password bisa disimpan dalam bentuk kosong
C. Untuk mempercepat proses autentikasi
D. Agar sesuai dengan standar enkripsi AES

### 31. Parameter `sshd` yang mengatur berapa kali percobaan autentikasi gagal sebelum koneksi diputus adalah:
A. `MaxAuthTries`
B. `MaxSessions`
C. `LoginGraceTime`
D. `ClientAliveCountMax`

### 32. Mengapa CIS merekomendasikan menonaktifkan `HostbasedAuthentication` dan `IgnoreRhosts=yes` pada SSH?
A. Untuk mencegah otentikasi berbasis trust hostname (`.rhosts`/`.shosts`) yang rentan spoofing
B. Karena fitur ini tidak didukung RHEL 10
C. Untuk mempercepat koneksi SSH
D. Agar sesuai dengan crypto policy FIPS

### 33. Baris konfigurasi `Defaults use_pty` pada `/etc/sudoers` berfungsi untuk:
A. Memaksa perintah sudo dijalankan melalui pseudo-terminal, mencegah proses background berbahaya tetap berjalan setelah program utama selesai
B. Mempercepat eksekusi perintah sudo
C. Membatasi jumlah user yang bisa menggunakan sudo
D. Mengaktifkan logging warna pada terminal

### 34. Tool yang **wajib** digunakan untuk mengedit file `/etc/sudoers` guna menghindari kesalahan sintaks adalah:
A. `nano`
B. `visudo`
C. `vim` langsung
D. `sed`

### 35. Modul PAM yang berfungsi mengunci akun setelah sejumlah percobaan login gagal (failed lockout) adalah:
A. `pam_faillock`
B. `pam_pwquality`
C. `pam_pwhistory`
D. `pam_unix`

### 36. Modul PAM `pam_pwhistory` digunakan untuk:
A. Mencatat riwayat percobaan sudo
B. Mencegah user menggunakan kembali password lama (password reuse) berdasarkan history
C. Mengenkripsi password baru
D. Membatasi panjang password

### 37. Berdasarkan CIS, panjang minimum password (`minlen`) yang direkomendasikan adalah:
A. 6 karakter
B. 8 karakter
C. 14 karakter atau lebih
D. 20 karakter

### 38. Parameter `pwquality.conf` yang mengatur jumlah karakter berbeda minimal antara password baru dan password lama adalah:
A. `difok`
B. `minlen`
C. `dcredit`
D. `retry`

### 39. Mengapa opsi `nullok` pada modul `pam_unix` harus dihapus/tidak digunakan?
A. Karena `nullok` memungkinkan akun login dengan password kosong
B. Karena `nullok` memperlambat autentikasi
C. Karena `nullok` menyebabkan konflik dengan SELinux
D. Karena `nullok` tidak didukung PAM versi baru

### 40. CIS merekomendasikan bahwa akun root harus menjadi satu-satunya akun dengan:
A. UID 0
B. GID 0 sebagai *primary group ID*-nya
C. Shell `/bin/false`
D. Password kosong

### 41. Mengapa system account (seperti `daemon`, `bin`, `sync`) direkomendasikan tidak memiliki *valid login shell* (misalnya diarahkan ke `/sbin/nologin`)?
A. Agar akun tersebut tidak dapat digunakan untuk login interaktif oleh siapapun, mengurangi permukaan serangan
B. Agar proses sistem berjalan lebih cepat
C. Karena systemd mewajibkannya
D. Agar akun tersebut otomatis terhapus

### 42. Permission yang direkomendasikan untuk file `/etc/shadow` adalah:
A. `644`
B. `000` atau `600`/`0000` (sangat restriktif, tidak world-readable)
C. `777`
D. `755`

### 43. Mengapa AIDE (Advanced Intrusion Detection Environment) direkomendasikan diinstal dalam konteks CIS Logging and Auditing?
A. Untuk melakukan filesystem integrity checking guna mendeteksi perubahan tak sah pada file sistem
B. Untuk menggantikan fungsi auditd
C. Untuk mengenkripsi seluruh filesystem
D. Untuk mempercepat proses booting

### 44. Layanan yang bertanggung jawab mencatat event keamanan tingkat kernel (system call auditing) pada RHEL adalah:
A. `rsyslog`
B. `auditd`
C. `journald`
D. `chronyd`

### 45. Mengapa CIS merekomendasikan audit terhadap proses-proses yang start sebelum `auditd` aktif (`audit=1` pada boot parameter)?
A. Agar semua aktivitas sejak awal proses boot tetap tercatat, tidak ada celah waktu yang tidak termonitor
B. Agar sistem boot lebih cepat
C. Agar auditd tidak perlu dijalankan sebagai service
D. Untuk menonaktifkan logging non-esensial

### 46. Parameter `audit_backlog_limit` berfungsi untuk:
A. Menentukan jumlah maksimum event audit yang dapat di-buffer sebelum userspace auditd siap memproses, mencegah kehilangan log saat boot
B. Membatasi ukuran file log audit
C. Menentukan retensi log dalam hari
D. Mengatur kompresi log audit

### 47. Konfigurasi `rsyslog` yang direkomendasikan terkait pengiriman log ke server terpusat adalah:
A. Mengonfigurasi rsyslog untuk mengirim log ke remote log host
B. Menonaktifkan seluruh forwarding log
C. Hanya menyimpan log secara lokal tanpa backup
D. Mengganti rsyslog dengan FTP server

### 48. Mengapa CIS merekomendasikan memastikan tidak ada file atau direktori yang *unowned* (tanpa user/group pemilik yang valid)?
A. File tanpa owner yang valid berpotensi menjadi celah karena bisa jadi milik akun yang sudah dihapus dan disalahgunakan
B. Agar filesystem lebih cepat diakses
C. Karena SELinux mewajibkan semua file punya owner root
D. Agar backup lebih mudah dilakukan

### 49. File SUID/SGID pada sistem direkomendasikan untuk:
A. Ditinjau secara manual (manual review) karena berpotensi disalahgunakan untuk privilege escalation
B. Dihapus semuanya tanpa terkecuali
C. Diabaikan karena tidak berisiko
D. Otomatis di-nonaktifkan oleh SELinux

### 50. `umask` default untuk root dan user pada CIS direkomendasikan diatur ke nilai yang cukup restriktif, yaitu:
A. `000`
B. `022` atau lebih ketat (misalnya `027`)
C. `777`
D. `666`

---

## Kunci Jawaban

1. B, D
2. B
3. A, B, C
4. B
5. A
6. C
7. B
8. A
9. A
10. A
11. B
12. B
13. A
14. C
15. A
16. A, B, C
17. A
18. D
19. A, B, D
20. A
21. A, B, D
22. A
23. A
24. A, B, C
25. A
26. A
27. A, B
28. A, B
29. B
30. A
31. A
32. A
33. A
34. B
35. A
36. B
37. C
38. A
39. A
40. A, B
41. A
42. B
43. A
44. B
45. A
46. A
47. A
48. A
49. A
50. B

---

**Catatan belajar:** Format soal UAS kamu menyebutkan bahwa jawaban bisa bersifat jamak (multi-select), jadi latihan ini sengaja mencampur soal single-answer dan multi-answer agar terbiasa membaca opsi dengan teliti — jangan asumsikan hanya ada 1 jawaban benar di tiap soal.
