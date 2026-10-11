# Tutorial Lengkap: Membuat Website Portal "The Beatles" 🎸

Tutorial ini akan memandu Anda dari awal hingga akhir dalam membuat halaman web statis modern menggunakan HTML5, CSS3 (Flexbox & Box Model), dan menjalankannya di lingkungan Arch Linux menggunakan Visual Studio Code.

---

## Tahap 1: Persiapan Aplikasi (Khusus Arch Linux)

Sebelum mulai coding, kita pastikan *tools* yang digunakan sudah siap.

1. **Install Visual Studio Code (VS Code)**
   Buka terminal (Konsole/Alacritty) dan ketik perintah berikut. **Ingat, jangan gunakan `sudo` saat memakai `yay`**:
   ```bash
   yay -S visual-studio-code-bin
   ```
   *Tunggu hingga proses selesai dan masukkan password jika diminta oleh sistem pada tahap akhir.*

2. **Install Ekstensi Live Server**
   * Buka aplikasi **Visual Studio Code**.
   * Klik ikon kotak-kotak di menu sebelah kiri (Extensions) atau tekan `Ctrl+Shift+X`.
   * Ketik **Live Server** di kolom pencarian.
   * Klik **Install** pada ekstensi buatan *Ritwick Dey*.

---

## Tahap 2: Persiapan Folder dan File Project

Sesuai standar pembuatan web, kita harus memisahkan file struktur (HTML) dan file tampilan (CSS).

1. Buat folder baru di komputer Anda, beri nama **`Tugas-Web-Beatles`**.
2. Buka VS Code, klik menu **File > Open Folder...** lalu pilih folder `Tugas-Web-Beatles` tersebut.
3. Di dalam VS Code, buat dua file baru:
   * Buat file bernama **`index.html`** (Ini adalah standar nama file utama website).
   * Buat file bernama **`style.css`**.

---

## Tahap 3: Menulis Kerangka HTML

Buka file **`index.html`**. Salin dan tempel (copy-paste) kode di bawah ini. Kode ini berisi struktur website yang sudah dihubungkan dengan file `style.css`.

```html
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>The Beatles - Official</title>
    <!-- Menghubungkan HTML dengan CSS Eksternal -->
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <!-- Bagian Header -->
    <header id="header">
        <h1>The Beatles</h1>
        <p>Kelompok musik rock asal Liverpool, Inggris yang dibentuk pada tahun 1960. Secara luas dianggap sebagai band paling berpengaruh dalam sejarah musik populer.</p>
    </header>

    <main class="container">
        <!-- Bagian Pengantar & Formulir -->
        <section id="section1">
            <p class="intro">Temukan sejarah dan melodi yang mengubah dunia.</p>
            <a class="btn" href="https://youtu.be/NCtzkaL2t_Y?si=XesRDWyiScV4HohU" target="_blank">Lihat Dokumenter</a>

            <form action="submit" class="form-box">
                <input type="text" name="name" placeholder="Masukkan nama Anda">
                <button type="submit" class="btn-submit">Kirim</button>
            </form>
        </section>

        <!-- Bagian Daftar Album (Grid/Flexbox) -->
        <section id="section2">
            <h2 class="judul-section">Diskografi Populer</h2>
            
            <div class="cards">
                <!-- Kartu Album 1 -->
                <article class="card">
                    <img alt="Abbey Road" src="https://github.com/user-attachments/assets/2af7c17c-e707-452e-ba48-f95524c4a5a6" />
                    <div class="card-content">
                        <h3>Abbey Road (1969)</h3>
                        <p>Album ke-11 ikonik dengan sampul penyeberangan jalan. Memuat hits "Come Together".</p>
                    </div>
                </article>
                
                <!-- Kartu Album 2 -->
                <article class="card">
                    <img alt="Sgt. Pepper's" src="https://github.com/user-attachments/assets/2d599b01-bb84-4c49-a32a-d69b3f88fb4b" />
                    <div class="card-content">
                        <h3>Sgt. Pepper's (1967)</h3>
                        <p>Titik balik eksperimentasi musik psikadelik dan konsep album secara keseluruhan.</p>
                    </div>
                </article>
                
                <!-- Kartu Album 3 -->
                <article class="card">
                    <img alt="Let It Be" src="https://github.com/user-attachments/assets/15eeb598-298f-43a0-9a00-a6ad7c3318e4" />
                    <div class="card-content">
                        <h3>Let It Be (1970)</h3>
                        <p>Album studio terakhir sebelum pembubaran. Berisi lagu hits abadi "Let It Be".</p>
                    </div>
                </article>
            </div>
        </section>
    </main>

    <!-- Bagian Footer -->
    <footer>
        <p>© 2026 Roihan Nabil</p>
    </footer>
</body>
</html>
```

*Jangan lupa tekan `Ctrl+S` untuk menyimpan file `index.html`.*

---

## Tahap 4: Menulis Desain CSS (Dengan Foto Background)

Buka file **`style.css`**. Salin dan tempel kode di bawah ini. Desain ini menggunakan tema cerah yang bersih, efek bayangan (shadow), layout Flexbox, dan **gambar latar belakang (background image)** pada Header.

```css
/* 1. Pengaturan Dasar (Tipografi & Warna Latar Utama) */
body {
    font-family: Arial, sans-serif;
    background-color: #f4f6f9;
    color: #333333;
    margin: 0;
    line-height: 1.6;
}

/* Membatasi lebar konten agar rapi di layar monitor besar */
.container {
    width: 90%;
    max-width: 1000px;
    margin: 0 auto;
    padding: 20px 0;
}

/* 2. Desain Header dengan Background Gambar */
#header {
    /* linear-gradient memberi efek kaca gelap di atas foto agar teks putih tetap terbaca */
    background-image: linear-gradient(rgba(15, 23, 42, 0.75), rgba(15, 23, 42, 0.75)), url('https://upload.wikimedia.org/wikipedia/commons/d/df/The_Fabs.JPG');
    background-size: cover;
    background-position: center;
    color: white;
    padding: 100px 20px; /* Padding besar agar foto terlihat luas */
    text-align: center;
    border-bottom: 5px solid #3b82f6; /* Garis biru di bawah foto */
}

#header h1 {
    font-size: 46px;
    margin: 0 0 10px 0;
    text-transform: uppercase;
    letter-spacing: 3px;
}

#header p {
    max-width: 700px;
    margin: 0 auto;
    color: #e2e8f0;
    font-size: 18px;
}

/* 3. Desain Kotak Info & Formulir */
#section1 {
    margin: 40px 0;
    text-align: center;
    background-color: #ffffff;
    padding: 30px;
    border-radius: 10px;
    box-shadow: 0 4px 6px rgba(0,0,0,0.05);
}

.intro {
    font-size: 18px;
    color: #334155;
    margin-bottom: 20px;
}

/* Desain Tombol Interaktif */
.btn, .btn-submit {
    display: inline-block;
    background-color: #3b82f6; /* Biru cerah */
    color: white;
    padding: 12px 25px;
    text-decoration: none;
    border: none;
    border-radius: 8px;
    font-weight: bold;
    cursor: pointer;
    transition: background-color 0.2s, transform 0.2s;
    box-shadow: 0 4px 6px rgba(59, 130, 246, 0.3);
}

/* Efek saat tombol disentuh mouse (Hover) */
.btn:hover, .btn-submit:hover {
    background-color: #2563eb;
    transform: translateY(-2px); /* Tombol sedikit terangkat */
}

.form-box {
    display: flex;
    justify-content: center;
    gap: 10px;
    margin-top: 20px;
}

.form-box input {
    padding: 12px;
    width: 250px;
    border: 2px solid #e2e8f0;
    border-radius: 8px;
    outline: none;
}

/* 4. Desain Bagian Album (Flexbox & Cards) */
.judul-section {
    text-align: center;
    color: #0f172a;
    margin: 60px 0 30px 0;
    font-size: 28px;
}

/* Container Flexbox agar kartu berjajar menyamping */
.cards {
    display: flex;
    gap: 25px;
    flex-wrap: wrap; /* Otomatis turun ke bawah jika layar sempit */
    justify-content: center;
}

/* Desain per-kartu album */
.card {
    flex: 1 1 280px; /* Fleksibilitas ukuran kartu */
    max-width: 300px;
    background-color: #ffffff;
    border-radius: 12px;
    overflow: hidden; /* Mencegah gambar keluar dari lengkungan */
    box-shadow: 0 10px 20px rgba(0,0,0,0.08);
    transition: transform 0.3s ease;
}

/* Efek saat kartu album disentuh mouse */
.card:hover {
    transform: translateY(-8px);
}

.card img {
    width: 100%;
    height: auto;
    display: block;
    border-bottom: 3px solid #3b82f6;
}

.card-content {
    padding: 20px;
    text-align: center;
}

.card-content h3 {
    color: #0f172a;
    font-size: 19px;
    margin: 0 0 10px 0;
}

.card-content p {
    font-size: 14px;
    color: #64748b;
    margin: 0;
}

/* 5. Desain Footer */
footer {
    text-align: center;
    padding: 25px 20px;
    background-color: #0f172a;
    color: #94a3b8;
    margin-top: 60px;
    font-size: 14px;
}
```

*Jangan lupa tekan `Ctrl+S` untuk menyimpan file `style.css`.*

---

## Tahap 5: Menjalankan Website (Go Live)

Sekarang saatnya melihat hasilnya!
1. Buka kembali file `index.html` di dalam VS Code.
2. Lihat ke pojok kanan bawah layar VS Code Anda.
3. Klik tulisan **"Go Live"** (Atau klik kanan di area teks HTML lalu pilih "Open with Live Server").
4. Browser bawaan sistem Anda (misalnya Firefox/Chrome) akan otomatis terbuka dan menampilkan website yang sangat rapi.

**Keuntungan Live Server:**
Jika Anda mengubah teks HTML atau mengganti warna di CSS lalu menyimpannya (`Ctrl+S`), browser akan otomatis me-refresh halaman tanpa perlu Anda klik tombol reload. Sangat cocok untuk eksperimen CSS!