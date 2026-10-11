<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <title>The Beatles</title>
    <!-- Memanggil file CSS eksternal -->
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <header id="header">
        <h1>The Beatles</h1>
        <p>Kelompok musik rock asal Liverpool yang mengubah industri musik dunia.</p>
    </header>

    <main class="container">
        <section id="section1">
            <p class="intro">Silakan pencet link di bawah untuk informasi lebih lanjut.</p>
            <a href="https://youtu.be/NCtzkaL2t_Y?si=XesRDWyiScV4HohU" target="_blank" class="btn">Info lebih lanjut</a>

            <form action="submit" class="form-box">
                <input type="text" name="name" placeholder="Masukkan nama Anda">
                <button type="submit" class="btn">Kirim</button>
            </form>
        </section>

        <section id="section2">
            <h2>Album Populer The Beatles</h2>
            <!-- Menggunakan class cards untuk Flexbox layout -->
            <div class="cards">
                <article class="card">
                    <img width="721" height="721" alt="Abbey Road" src="https://github.com/user-attachments/assets/2af7c17c-e707-452e-ba48-f95524c4a5a6" />
                    <h3>Abbey Road (1969)</h3>
                    <p>Album ke-11 yang sangat ikonik dengan memuat lagu-lagu legendaris seperti "Come Together" dan "Here Comes the Sun".</p>
                </article>
                
                <article class="card">
                    <img width="721" height="721" alt="Sgt. Pepper's" src="https://github.com/user-attachments/assets/2d599b01-bb84-4c49-a32a-d69b3f88fb4b" />
                    <h3>Sgt. Pepper's Lonely Hearts Club Band (1967)</h3>
                    <p>Titik balik eksperimentasi musik psikadelik dan konsep album secara keseluruhan.</p>
                </article>
                
                <article class="card">
                    <img width="721" height="721" alt="Let It Be" src="https://github.com/user-attachments/assets/15eeb598-298f-43a0-9a00-a6ad7c3318e4" />
                    <h3>Let It Be (1970)</h3>
                    <p>Album studio terakhir yang dirilis sebelum pembubaran mereka. Berisi lagu hits abadi seperti "Let It Be".</p>
                </article>
            </div>
        </section>
    </main>

    <footer>
        <p>&copy; 2026 Roihan Nabil</p>
    </footer>
</body>
</html>
