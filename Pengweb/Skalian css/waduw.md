<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <title>The Beatles</title>
    <!-- Kita menggunakan Internal CSS agar bisa dibaca langsung di HP -->
    <style>
        /* Mengatur font, warna latar, dan jarak dasar */
        body {
            font-family: Arial, sans-serif;
            background-color: #f5f7fa;
            color: #333;
            margin: 0;
            line-height: 1.6;
        }

        /* Membatasi lebar agar konten tidak terlalu melebar */
        .container {
            width: 90%;
            max-width: 1100px;
            margin: 0 auto;
        }

        /* Tampilan Header utama */
        #header {
            background-color: #193056;
            color: white;
            padding: 40px 20px;
            text-align: center;
        }

        #header h1 {
            font-size: 36px;
            margin-bottom: 10px;
        }

        #header p {
            max-width: 800px;
            margin: 0 auto;
        }

        /* Bagian tombol dan formulir */
        #section1 {
            margin: 40px 0;
            text-align: center;
        }

        .btn {
            display: inline-block;
            background-color: #193056;
            color: white;
            padding: 10px 20px;
            text-decoration: none;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            font-weight: bold;
            margin: 10px;
        }

        .form-box input {
            padding: 10px;
            width: 250px;
            border: 1px solid #ccc;
            border-radius: 4px;
        }

        /* Judul Album */
        .judul-section {
            text-align: center;
            color: #193056;
            margin-bottom: 20px;
        }

        /* Layout Flexbox untuk Kartu Album */
        .cards {
            display: flex;
            gap: 20px;
            flex-wrap: wrap;
        }

        /* Pengaturan masing-masing Kartu */
        .card {
            flex: 1 1 280px;
            background-color: white;
            padding: 25px;
            border: 2px solid #193056;
            border-radius: 8px;
            text-align: center;
        }

        /* Memastikan gambar album proporsional */
        .card img {
            width: 100%;
            height: auto;
            border-radius: 4px;
        }

        .card h3 {
            color: #193056;
            font-size: 18px;
            margin-top: 15px;
        }

        .card p {
            font-size: 14px;
        }

        /* Tampilan Footer */
        footer {
            text-align: center;
            padding: 20px;
            background-color: #e0e0e0;
            margin-top: 40px;
        }
    </style>
</head>
<body>
    <header id="header">
        <h1>The Beatles</h1>
        <p>Kelompok musik rock asal Liverpool, Inggris yang dibentuk pada tahun 1960. Beranggotakan John Lennon, Paul McCartney, George Harrison, dan Ringo Starr.</p>
    </header>

    <main class="container">
        <section id="section1">
            <p class="intro">Silahkan pencet link dibawah untuk informasi lebih lanjut.</p>
            <a class="btn" href="https://youtu.be/NCtzkaL2t_Y?si=XesRDWyiScV4HohU" target="_blank">Info lebih lanjut</a>

            <form action="submit" class="form-box">
                <input type="text" name="name" placeholder="Masukkan nama Anda">
                <button type="submit" class="btn">Kirim</button>
            </form>
        </section>

        <section id="section2">
            <h2 class="judul-section">Album Populer The Beatles</h2>
            <div class="cards">
                <article class="card">
                    <img alt="Abbey Road" src="https://github.com/user-attachments/assets/2af7c17c-e707-452e-ba48-f95524c4a5a6" />
                    <h3>Abbey Road (1969)</h3>
                    <p>Album ke-11 The Beatles yang sangat ikonik dengan sampul penyeberangan jalan Abbey Road. Memuat lagu legendaris seperti "Come Together".</p>
                </article>
                
                <article class="card">
                    <img alt="Sgt. Pepper's" src="https://github.com/user-attachments/assets/2d599b01-bb84-4c49-a32a-d69b3f88fb4b" />
                    <h3>Sgt. Pepper's Lonely Hearts Club Band (1967)</h3>
                    <p>Salah satu album paling berpengaruh dalam sejarah. Menjadi titik balik eksperimentasi musik psikadelik dan konsep album secara keseluruhan.</p>
                </article>
                
                <article class="card">
                    <img alt="Let It Be" src="https://github.com/user-attachments/assets/15eeb598-298f-43a0-9a00-a6ad7c3318e4" />
                    <h3>Let It Be (1970)</h3>
                    <p>Album studio terakhir yang dirilis sebelum pembubaran mereka. Berisi lagu hits abadi seperti "Let It Be" dan "Across the Universe".</p>
                </article>
            </div>
        </section>
    </main>

    <footer>
        <p>© 2026 Roihan Nabil</p>
    </footer>
</body>
</html>
