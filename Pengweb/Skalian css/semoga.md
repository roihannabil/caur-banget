<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>The Beatles - Official</title>
    <style>
        /* Mengatur font dan background yang bersih */
        body {
            font-family: Arial, sans-serif;
            background-color: #f4f6f9; /* Abu-abu sangat terang/putih */
            color: #333333;
            margin: 0;
            line-height: 1.6;
        }

        .container {
            width: 90%;
            max-width: 1000px;
            margin: 0 auto;
            padding: 20px 0;
        }

        /* Header Biru Gelap yang Elegan */
        #header {
            background-color: #0f172a; /* Biru dongker gelap */
            color: white;
            padding: 50px 20px;
            text-align: center;
            border-bottom: 5px solid #3b82f6; /* Aksen garis biru terang */
        }

        #header h1 {
            font-size: 38px;
            margin: 0 0 10px 0;
            text-transform: uppercase;
            letter-spacing: 2px;
        }

        #header p {
            max-width: 700px;
            margin: 0 auto;
            color: #cbd5e1;
            font-size: 16px;
        }

        /* Box Informasi dan Tombol */
        #section1 {
            margin: 40px 0;
            text-align: center;
            background-color: #ffffff;
            padding: 30px;
            border-radius: 10px;
            box-shadow: 0 4px 6px rgba(0,0,0,0.05);
        }

        .btn {
            display: inline-block;
            background-color: #3b82f6; /* Biru terang */
            color: white;
            padding: 12px 25px;
            text-decoration: none;
            border: none;
            border-radius: 8px;
            font-weight: bold;
            margin-bottom: 15px;
            box-shadow: 0 4px 6px rgba(59, 130, 246, 0.3);
        }

        .form-box {
            display: flex;
            justify-content: center;
            flex-wrap: wrap;
            gap: 10px;
            margin-top: 15px;
        }

        .form-box input {
            padding: 12px;
            width: 250px;
            border: 2px solid #e2e8f0;
            border-radius: 8px;
            outline: none;
        }

        /* Layout Grid/Flexbox untuk Kartu Album */
        .judul-section {
            text-align: center;
            color: #0f172a;
            margin: 50px 0 30px 0;
            font-size: 26px;
        }

        .cards {
            display: flex;
            gap: 25px;
            flex-wrap: wrap;
            justify-content: center;
        }

        /* Desain Kartu Album */
        .card {
            flex: 1 1 280px;
            max-width: 300px;
            background-color: #ffffff;
            border-radius: 12px;
            overflow: hidden;
            box-shadow: 0 10px 20px rgba(0,0,0,0.08);
            border: 1px solid #f1f5f9;
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
            font-size: 18px;
            margin: 0 0 10px 0;
        }

        .card-content p {
            font-size: 14px;
            color: #64748b;
            margin: 0;
        }

        /* Footer */
        footer {
            text-align: center;
            padding: 25px 20px;
            background-color: #0f172a;
            color: #94a3b8;
            margin-top: 50px;
            font-size: 14px;
        }
    </style>
</head>
<body>
    <header id="header">
        <h1>The Beatles</h1>
        <p>Kelompok musik rock asal Liverpool, Inggris yang dibentuk pada tahun 1960. Secara luas dianggap sebagai band paling berpengaruh dalam sejarah musik populer.</p>
    </header>

    <main class="container">
        <section id="section1">
            <p style="font-size: 18px; color: #334155; margin-bottom: 20px;">Temukan sejarah dan melodi yang mengubah dunia.</p>
            <a class="btn" href="https://youtu.be/NCtzkaL2t_Y?si=XesRDWyiScV4HohU" target="_blank">Lihat Dokumenter</a>

            <form action="submit" class="form-box">
                <input type="text" name="name" placeholder="Masukkan nama Anda">
                <button type="submit" class="btn" style="margin-bottom: 0;">Kirim</button>
            </form>
        </section>

        <section id="section2">
            <h2 class="judul-section">Diskografi Populer</h2>
            
            <div class="cards">
                <article class="card">
                    <img alt="Abbey Road" src="https://github.com/user-attachments/assets/2af7c17c-e707-452e-ba48-f95524c4a5a6" />
                    <div class="card-content">
                        <h3>Abbey Road (1969)</h3>
                        <p>Album ke-11 ikonik dengan sampul penyeberangan jalan. Memuat hits "Come Together".</p>
                    </div>
                </article>
                
                <article class="card">
                    <img alt="Sgt. Pepper's" src="https://github.com/user-attachments/assets/2d599b01-bb84-4c49-a32a-d69b3f88fb4b" />
                    <div class="card-content">
                        <h3>Sgt. Pepper's (1967)</h3>
                        <p>Titik balik eksperimentasi musik psikadelik dan konsep album secara keseluruhan.</p>
                    </div>
                </article>
                
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

    <footer>
        <p>© 2026 Roihan Nabil</p>
    </footer>
</body>
</html>
