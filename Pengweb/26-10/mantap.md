<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>UTS Perpustakaan Digital - Koleksi Fantasi</title>
    
    <style>
        /* =========================================
           1. RESET & PENGATURAN DASAR
           ========================================= */
        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: #f8f9fa;
            color: #1e293b;
            line-height: 1.6;
        }

        /* =========================================
           2. HEADER & CONTAINER
           ========================================= */
        header {
            background-color: #0f172a;
            color: #ffffff;
            padding: 40px 20px;
            text-align: center;
            border-bottom: 4px solid #64748b;
        }

        header h1 {
            font-size: 30px;
            font-weight: bold;
            margin-bottom: 8px;
        }

        header p {
            color: #cbd5e1;
            font-size: 15px;
        }

        .container {
            width: 90%;
            max-width: 1100px;
            margin: 40px auto;
        }

        .section-title {
            text-align: center;
            margin-bottom: 30px;
            color: #0f172a;
            font-size: 22px;
        }

        /* =========================================
           3. LAYOUT KARTU BUKU (FLEXBOX)
           ========================================= */
        .cards {
            display: flex;
            gap: 24px;
            flex-wrap: wrap;
            justify-content: center;
        }

        .card {
            background-color: #ffffff;
            flex: 1 1 300px;
            max-width: 340px;
            border-radius: 10px;
            box-shadow: 0 4px 12px rgba(0, 0, 0, 0.05);
            overflow: hidden;
            display: flex;
            flex-direction: column;
            border: 1px solid #e2e8f0;
        }

        .card-header {
            padding: 24px 20px;
            color: #ffffff;
            height: 140px;
        }

        /* Tema Warna per Buku */
        .bg-hp { background-color: #740001; } 
        .bg-got { background-color: #5c0f0f; } 
        .bg-lotr { background-color: #2e5a2c; } 

        .badge-tag {
            display: inline-block;
            font-size: 11px;
            font-weight: bold;
            text-transform: uppercase;
            letter-spacing: 1px;
            background-color: rgba(255, 255, 255, 0.2);
            padding: 4px 8px;
            border-radius: 4px;
            margin-bottom: 10px;
        }

        .card-header h2 {
            font-size: 18px;
            line-height: 1.3;
        }

        .card-body {
            padding: 20px;
            display: flex;
            flex-direction: column;
            flex-grow: 1;
        }

        .info-list {
            list-style: none;
            margin-bottom: 20px;
            font-size: 13px;
        }

        .info-list li {
            margin-bottom: 8px;
            display: flex;
            justify-content: space-between;
            border-bottom: 1px dashed #e2e8f0;
            padding-bottom: 4px;
        }

        .info-label { color: #64748b; font-weight: bold; }
        .info-value { font-weight: bold; color: #0f172a; text-align: right; }

        .card-footer {
            margin-top: auto;
            display: flex;
            justify-content: space-between;
            align-items: center;
            padding-top: 10px;
        }

        .status-available {
            font-size: 13px;
            font-weight: bold;
            color: #16a34a;
        }

        .btn-lanjut {
            background-color: #0f172a;
            color: #ffffff;
            padding: 8px 16px;
            border-radius: 6px;
            text-decoration: none;
            font-size: 13px;
            font-weight: bold;
        }

        .btn-lanjut:hover { background-color: #334155; }

        /* =========================================
           4. TAMPILAN DETAIL (Trik CSS :target)
           ========================================= */
        .detail-section {
            /* Sembunyikan semua detail secara default */
            display: none; 
            margin-top: 40px;
            padding-top: 40px;
            border-top: 2px dashed #cbd5e1;
        }

        /* TRIK CSS: Tampilkan hanya jika ID-nya diklik dari tautan (menjadi target) */
        .detail-section:target {
            display: block;
        }

        .detail-container {
            display: flex;
            gap: 30px;
            background-color: #ffffff;
            padding: 30px;
            border-radius: 10px;
            box-shadow: 0 4px 12px rgba(0, 0, 0, 0.05);
            border: 1px solid #e2e8f0;
            position: relative; /* Agar tombol tutup bisa diposisikan di dalam */
        }

        /* Tombol Tutup (Silang) */
        .btn-tutup {
            position: absolute;
            top: 20px;
            right: 20px;
            text-decoration: none;
            color: #ef4444;
            font-weight: bold;
            font-size: 14px;
            border: 1px solid #ef4444;
            padding: 4px 10px;
            border-radius: 4px;
        }
        .btn-tutup:hover {
            background-color: #ef4444;
            color: white;
        }

        .detail-left img {
            width: 150px;
            border-radius: 4px;
            box-shadow: 0 2px 6px rgba(0,0,0,0.2);
        }

        .detail-right {
            flex: 1;
            padding-right: 20px;
        }

        .detail-right h3 {
            font-size: 22px;
            color: #0f172a;
            margin-bottom: 5px;
        }

        .detail-right .author {
            color: #64748b;
            font-size: 15px;
            margin-bottom: 10px;
        }

        .detail-right .rating {
            color: #d97706; 
            font-size: 16px;
            font-weight: bold;
            margin-bottom: 15px;
        }

        .detail-right .synopsis {
            color: #334155;
            font-size: 14px;
            line-height: 1.7;
        }

        footer {
            text-align: center;
            padding: 24px;
            background-color: #e2e8f0;
            font-size: 14px;
            margin-top: 40px;
        }
    </style>
</head>
<body>

    <header>
        <h1>Portal Perpustakaan Digital</h1>
        <p>Katalog Koleksi Buku Fantasi Epik Terbaik</p>
    </header>

    <main class="container">
        
        <section>
            <h2 class="section-title">Buku Terpopuler</h2>

            <div class="cards">
                <!-- BUKU 1: HARRY POTTER -->
                <article class="card">
                    <div class="card-header bg-hp">
                        <span class="badge-tag">Fantasi Anak</span>
                        <h2>Harry Potter and the Sorcerer's Stone</h2>
                    </div>
                    <div class="card-body">
                        <ul class="info-list">
                            <li><span class="info-label">Penulis:</span><span class="info-value">J.K. Rowling</span></li>
                            <li><span class="info-label">Tahun Terbit:</span><span class="info-value">1997</span></li>
                            <li><span class="info-label">Penerbit:</span><span class="info-value">Bloomsbury</span></li>
                        </ul>
                        <div class="card-footer">
                            <span class="status-available">✓ Tersedia</span>
                            <!-- Tautan ini menargetkan ID #buku1 di bawah -->
                            <a href="#buku1" class="btn-lanjut">Lebih Lanjut</a>
                        </div>
                    </div>
                </article>

                <!-- BUKU 2: GAME OF THRONES -->
                <article class="card">
                    <div class="card-header bg-got">
                        <span class="badge-tag">Epik Fantasi</span>
                        <h2>The World of Ice & Fire: History of Westeros</h2>
                    </div>
                    <div class="card-body">
                        <ul class="info-list">
                            <li><span class="info-label">Penulis:</span><span class="info-value">George R.R. Martin</span></li>
                            <li><span class="info-label">Tahun Terbit:</span><span class="info-value">2014</span></li>
                            <li><span class="info-label">Penerbit:</span><span class="info-value">Bantam</span></li>
                        </ul>
                        <div class="card-footer">
                            <span class="status-available">✓ Tersedia</span>
                            <!-- Tautan ini menargetkan ID #buku2 di bawah -->
                            <a href="#buku2" class="btn-lanjut">Lebih Lanjut</a>
                        </div>
                    </div>
                </article>

                <!-- BUKU 3: LORD OF THE RINGS -->
                <article class="card">
                    <div class="card-header bg-lotr">
                        <span class="badge-tag">High Fantasy</span>
                        <h2>The Fellowship of the Ring (LOTR #1)</h2>
                    </div>
                    <div class="card-body">
                        <ul class="info-list">
                            <li><span class="info-label">Penulis:</span><span class="info-value">J.R.R. Tolkien</span></li>
                            <li><span class="info-label">Tahun Terbit:</span><span class="info-value">1954</span></li>
                            <li><span class="info-label">Penerbit:</span><span class="info-value">Allen & Unwin</span></li>
                        </ul>
                        <div class="card-footer">
                            <span class="status-available">✓ Tersedia</span>
                            <!-- Tautan ini menargetkan ID #buku3 di bawah -->
                            <a href="#buku3" class="btn-lanjut">Lebih Lanjut</a>
                        </div>
                    </div>
                </article>
            </div>
        </section>

        <!-- =========================================
             AREA DETAIL BUKU (Hanya Muncul Saat Diklik)
             ========================================= -->

        <!-- DETAIL BUKU 1 -->
        <section id="buku1" class="detail-section">
            <h2 class="section-title">Informasi Detail Buku</h2>
            <div class="detail-container">
                <a href="#" class="btn-tutup">Tutup ✕</a>
                <div class="detail-left">
                    <img src="https://images-na.ssl-images-amazon.com/images/S/compressed.photo.goodreads.com/books/1474154022i/3.jpg" alt="Cover Harry Potter">
                </div>
                <div class="detail-right">
                    <h3>Harry Potter and the Sorcerer's Stone</h3>
                    <p class="author">J.K. Rowling</p>
                    <p class="rating">⭐⭐⭐⭐ 4.47</p>
                    <div class="synopsis">
                        <p>Harry Potter sama sekali tidak tahu betapa terkenalnya dia. Itu karena ia dibesarkan oleh bibi dan pamannya yang menyedihkan, yang sangat takut Harry akan mengetahui bahwa dia sebenarnya adalah seorang penyihir.</p>
                        <br>
                        <p>Namun semua itu berubah ketika sebuah surat misterius tiba, mengundangnya ke Sekolah Sihir Hogwarts. Di sanalah petualangan ajaibnya dimulai.</p>
                    </div>
                </div>
            </div>
        </section>

        <!-- DETAIL BUKU 2 -->
        <section id="buku2" class="detail-section">
            <h2 class="section-title">Informasi Detail Buku</h2>
            <div class="detail-container">
                <a href="#" class="btn-tutup">Tutup ✕</a>
                <div class="detail-left">
                    <img src="<img width="388" height="515" alt="image" src="https://github.com/user-attachments/assets/1008cd60-7917-4b81-aed8-61f25fe955c9" />
" alt="Cover Game of Thrones">
                </div>
                <div class="detail-right">
                    <h3>The World of Ice & Fire: The Untold History of Westeros</h3>
                    <p class="author">George R.R. Martin, Elio M. García Jr., Linda Antonsson</p>
                    <p class="rating">⭐⭐⭐⭐ 4.25</p>
                    <div class="synopsis">
                        <p>Sejarah Westeros dan wilayah di luarnya yang belum pernah diceritakan sebelumnya. Dilengkapi dengan ratusan halaman materi baru dari George R.R. Martin.</p>
                        <br>
                        <p>Jika masa lalu adalah prolog, maka mahakarya George R.R. Martin ini menjanjikan pengantar yang sangat memukau untuk dunia epik <i>A Song of Ice and Fire</i>.</p>
                    </div>
                </div>
            </div>
        </section>

        <!-- DETAIL BUKU 3 -->
        <section id="buku3" class="detail-section">
            <h2 class="section-title">Informasi Detail Buku</h2>
            <div class="detail-container">
                <a href="#" class="btn-tutup">Tutup ✕</a>
                <div class="detail-left">
                    <img src="https://images-na.ssl-images-amazon.com/images/S/compressed.photo.goodreads.com/books/1654215925i/61215351.jpg" alt="Cover Lord of The Rings">
                </div>
                <div class="detail-right">
                    <h3>The Fellowship of the Ring (The Lord of the Rings #1)</h3>
                    <p class="author">J.R.R. Tolkien</p>
                    <p class="rating">⭐⭐⭐⭐ 4.38</p>
                    <div class="synopsis">
                        <p>Di sebuah desa yang tenang di Shire, seorang hobbit muda bernama Frodo Baggins dipercayakan dengan tugas yang sangat besar. Ia menerima sebuah cincin ajaib dari pamannya, Bilbo.</p>
                        <br>
                        <p>Frodo harus melakukan perjalanan berbahaya melintasi Middle-earth menuju Mount Doom untuk menghancurkan Cincin Kekuasaan sebelum jatuh ke tangan Pangeran Kegelapan, Sauron.</p>
                    </div>
                </div>
            </div>
        </section>

    </main>

    <footer>
        <p>&copy; 2026 Roihan Nabil - UTS Pengembangan Web (HTML & CSS)</p>
    </footer>

</body>
</html>
