

### Langkah 1: Persiapan Sistem (Arch Linux)

Buka terminal dan instal paket sistem yang dibutuhkan:

```bash
sudo pacman -Syu
sudo pacman -S python python-pip python-virtualenv v4l-utils git

```

### Langkah 2: Ambil Projek dari GitHub

Unduh repository target dan masuk ke foldernya:

```bash
git clone https://github.com/claramiadevira/foto-kita-blur.git
cd foto-kita-blur

```

### Langkah 3: Buat Virtual Environment Khusus

Agar MediaPipe tidak error `AttributeError` karena ketidakcocokan dengan Python 3.14 bawaan Arch saat ini, kita buat venv yang bersih:

```bash
# Buat environment baru
python -m venv venv

# Aktifkan environment (Wajib!)
source venv/bin/activate

```

*(Pastikan muncul tanda `(venv)` di ujung kiri terminalmu).*

### Langkah 4: Install Pustaka yang Benar (Sesuai Spek MediaPipe)

Di dalam keadaan venv aktif, instal komponen yang dibutuhkan:

```bash
pip install --upgrade pip setuptools wheel
pip install opencv-python mediapipe

```

### Langkah 5: Buat File Skrip (`gesture.py`)

Buka text editor lewat terminal:

```bash
nano gesture.py

```

**Kopas (Paste) seluruh kode di bawah ini:**

```python
import cv2
import mediapipe as mp

# Inisialisasi MediaPipe Hands secara aman
mp_hands = mp.solutions.hands
hands = mp_hands.Hands(min_detection_confidence=0.7, min_tracking_confidence=0.5)
mp_draw = mp.solutions.drawing_utils

# Inisialisasi Kamera Webcam (0)
cap = cv2.VideoCapture(0)

print("Program berjalan... Arahkan tangan ke kamera. Tekan 'q' untuk keluar.")

while cap.isOpened():
    success, image = cap.read()
    if not success:
        print("Kamera tidak ditemukan.")
        break

    # Balik gambar secara horizontal (efek cermin) dan ubah ke RGB untuk MediaPipe
    image = cv2.flip(image, 1)
    rgb_image = cv2.cvtColor(image, cv2.COLOR_BGR2RGB)
    
    # Proses deteksi tangan
    results = hands.process(rgb_image)

    # Jika tangan terdeteksi oleh sistem
    if results.multi_hand_landmarks:
        for hand_landmarks in results.multi_hand_landmarks:
            # Menggambar titik persendian tangan di layar
            mp_draw.draw_landmarks(image, hand_landmarks, mp_hands.HAND_CONNECTIONS)
            
            # Efek blur (51, 51) pada layar saat tangan terdeteksi
            image = cv2.GaussianBlur(image, (51, 51), 0)

    # Tampilkan hasil gambar ke jendela komputer
    cv2.imshow('Foto Kita Blur - Hand Gesture', image)

    # Tekan tombol 'q' pada keyboard untuk keluar
    if cv2.waitKey(1) & 0xFF == ord('q'):
        break

cap.release()
cv2.destroyAllWindows()

```

*Simpan perubahan di nano dengan menekan **Ctrl + O**, lalu **Enter**, dan keluar dengan **Ctrl + X**.*

### Langkah 6: Jalankan Program

Jalankan skripnya menggunakan perintah:

```bash
python gesture.py

```

---

### Cara Menyalakan Lagi di Lain Waktu (Jika Terminal Ditutup)

Jika besok-besok kamu ingin menyalakan fiturnya lagi, kamu tidak perlu mengulang instalasi. Cukup buka terminal baru dan ketik:

```bash
cd foto-kita-blur
source venv/bin/activate
python gesture.py

```
