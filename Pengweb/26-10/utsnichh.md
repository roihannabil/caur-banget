<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Perpustakaan Diagonalley</title>
    
    <style>
        /* =========================================================
           VARIABEL WARNA & PENGATURAN DASAR
           ========================================================= */
        :root {
            --primary-color: #1e40af; 
            --secondary-color: #f3f4f6;
            --text-color: #333333;
            --white: #ffffff;
            --accent-color: #d97706; 
        }

        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: var(--secondary-color);
            color: var(--text-color);
            line-height: 1.6;
        }

        .container {
            width: 90%;
            max-width: 1100px;
            margin: 0 auto;
            padding: 20px 0;
        }

        /* =========================================================
           HEADER & NAVIGASI
           ========================================================= */
        header {
            background-color: var(--primary-color);
            color: var(--white);
            padding: 30px 20px;
            text-align: center;
        }

        header h1 { font-size: 28px; margin-bottom: 5px; }

        nav {
            background-color: #1e3a8a;
            position: sticky;
            top: 0;
            z-index: 100;
        }

        .nav-links {
            display: flex;
            justify-content: center;
            list-style: none;
            padding: 15px;
            gap: 20px;
        }

        .nav-links a {
            color: var(--white);
            text-decoration: none;
            font-weight: bold;
            font-size: 15px;
            padding: 5px 10px;
            border-radius: 4px;
            transition: background 0.3s;
        }

        .nav-links a:hover { background-color: #3b82f6; }

        section {
            padding: 40px 0;
            border-bottom: 2px dashed #cbd5e1;
        }

        .section-title {
            text-align: center;
            font-size: 24px;
            color: var(--primary-color);
            margin-bottom: 30px;
        }

        /* =========================================================
           KOLOM PENCARIAN (MOCKUP) & TOMBOL FILTER
           ========================================================= */
        .search-filter-container {
            display: flex;
            flex-direction: column;
            align-items: center;
            gap: 20px;
            margin-bottom: 30px;
        }

        .search-bar {
            width: 100%;
            max-width: 500px;
            padding: 12px 20px;
            font-size: 16px;
            border: 2px solid #cbd5e1;
            border-radius: 30px;
            outline: none;
        }

        .search-bar:focus { border-color: var(--primary-color); }

        .filter-buttons {
            display: flex;
            gap: 10px;
            flex-wrap: wrap;
            justify-content: center;
        }

        /* =========================================================
           TRIK FILTER MURNI CSS (TANPA JAVASCRIPT)
           ========================================================= */
        /* 1. Sembunyikan radio button asli */
        .hidden-radio { display: none; }

        /* 2. Ubah tampilan Label agar terlihat seperti Tombol */
        .filter-btn {
            padding: 8px 20px;
            background-color: #e2e8f0;
            color: var(--text-color);
            border-radius: 20px;
            cursor: pointer;
            font-weight: bold;
            transition: 0.3s;
            display: inline-block;
        }

        .filter-btn:hover { background-color: #cbd5e1; }

        /* 3. Mewarnai tombol/label saat Radio Button-nya terpilih (:checked) */
        #radio-semua:checked ~ #kategori .btn-semua,
        #radio-fiksi:checked ~ #kategori .btn-fiksi,
        #radio-teknologi:checked ~ #kategori .btn-teknologi,
        #radio-ilmu:checked ~ #kategori .btn-ilmu,
        #radio-jurnal:checked ~ #kategori .btn-jurnal {
            background-color: var(--primary-color);
            color: var(--white);
        }

        /* 4. Logika Penyaringan: Menyembunyikan kartu & section yang TIDAK SESUAI */
        #radio-fiksi:checked ~ #katalog .card:not(.kategori-fiksi) { display: none; }
        #radio-fiksi:checked ~ #jurnal { display: none; } /* Sembunyikan semua jurnal jika klik Fiksi */

        #radio-teknologi:checked ~ #katalog .card:not(.kategori-teknologi) { display: none; }
        #radio-teknologi:checked ~ #jurnal { display: none; } 

        #radio-ilmu:checked ~ #katalog .card:not(.kategori-ilmu) { display: none; }
        #radio-ilmu:checked ~ #jurnal { display: none; } 

        #radio-jurnal:checked ~ #katalog { display: none; } /* Sembunyikan semua buku jika klik Jurnal */

        /* =========================================================
           GAYA KARTU BUKU & JURNAL
           ========================================================= */
        .cards {
            display: flex;
            flex-wrap: wrap;
            gap: 20px;
            justify-content: center;
        }

        .card {
            background-color: var(--white);
            flex: 1 1 250px;
            max-width: 300px;
            border-radius: 10px;
            border: 1px solid #e2e8f0;
            box-shadow: 0 4px 6px rgba(0,0,0,0.05);
            padding: 20px;
            display: flex;
            flex-direction: column;
            transition: transform 0.3s;
        }

        .card:hover {
            transform: translateY(-5px);
            box-shadow: 0 10px 15px rgba(0,0,0,0.1);
        }

        .kategori-tag {
            font-size: 12px;
            font-weight: bold;
            background-color: #dbeafe;
            color: var(--primary-color);
            padding: 4px 8px;
            border-radius: 4px;
            align-self: flex-start;
            margin-bottom: 15px;
            text-transform: uppercase;
        }

        .card h3 { font-size: 18px; margin-bottom: 10px; color: #0f172a; line-height: 1.3; }
        .card p { font-size: 14px; margin-bottom: 5px; }
        .penulis { color: #64748b; font-weight: bold; }

        /* Tombol Detail */
        .card-footer {
            margin-top: auto; 
            padding-top: 15px;
            border-top: 1px solid #f1f5f9;
            display: flex;
            justify-content: flex-end;
        }

        .btn-detail {
            background-color: var(--primary-color);
            color: var(--white);
            padding: 6px 14px;
            border-radius: 5px;
            text-decoration: none;
            font-size: 12px;
            font-weight: bold;
            transition: 0.3s;
        }

        .btn-detail:hover { background-color: var(--accent-color); }

        footer {
            background-color: var(--primary-color);
            color: var(--white);
            text-align: center;
            padding: 20px;
            margin-top: 40px;
        }

        /* RESPONSIVITAS MOBILE */
        @media (max-width: 768px) {
            .cards { flex-direction: column; align-items: center; }
            .card { width: 100%; max-width: 100%; }
            .nav-links { flex-direction: column; text-align: center; gap: 10px; }
        }
    </style>
</head>
<body>

    <header>
        <h1>Perpustakaan Universitas Jackpot</h1>
        <p>Bukan kamu yang memilih buku, melainkan buku yang memilihmu </p>
    </header>

    <nav>
        <ul class="nav-links">
            <li><a href="#beranda">Beranda</a></li>
            <li><a href="#kategori">Pencarian</a></li>
            <li><a href="#katalog">Katalog Buku</a></li>
            <li><a href="#jurnal">Jurnal</a></li>
        </ul>
    </nav>

    <main class="container">
        
        <!-- RADIO BUTTON (Sembunyi) UNTUK LOGIKA FILTER CSS -->
        <input type="radio" name="filter-kategori" id="radio-semua" class="hidden-radio" checked>
        <input type="radio" name="filter-kategori" id="radio-fiksi" class="hidden-radio">
        <input type="radio" name="filter-kategori" id="radio-teknologi" class="hidden-radio">
        <input type="radio" name="filter-kategori" id="radio-ilmu" class="hidden-radio">
        <input type="radio" name="filter-kategori" id="radio-jurnal" class="hidden-radio">

        <section id="beranda">
            <h2 class="section-title">Selamat Datang !!!</h2>
            <p style="text-align: center; max-width: 700px; margin: 0 auto;">
                Jelajahi rak-rak ajaib kami yang menyimpan ribuan kisah dan misteri alam semesta. Mulai dari keajaiban buku fiksi, panduan ilmu komputer, hingga jurnal penelitian terpercaya hadir untuk menemani perjalanan akademis Anda.
            </p>
        </section>

        <!-- FITUR PENCARIAN & FILTER -->
        <section id="kategori">
            <h2 class="section-title">Cari Koleksi</h2>
            
            <div class="search-filter-container">
                <!-- Kolom pencarian pajangan (Mockup UI) untuk memenuhi poin soal -->
                <input type="text" class="search-bar" placeholder="Ketik judul, penulis, atau kata kunci (Visual Only)...">
                
                <div class="filter-buttons">
                    <!-- Tombol ini adalah LABEL yg terhubung ke input Radio di atas -->
                    <label for="radio-semua" class="filter-btn btn-semua">Semua Koleksi</label>
                    <label for="radio-fiksi" class="filter-btn btn-fiksi">Fiksi</label>
                    <label for="radio-teknologi" class="filter-btn btn-teknologi">Teknologi</label>
                    <label for="radio-ilmu" class="filter-btn btn-ilmu">Ilmu Perpustakaan</label>
                    <label for="radio-jurnal" class="filter-btn btn-jurnal">Jurnal Akademik</label>
                </div>
            </div>
        </section>

        <!-- KATALOG BUKU -->
        <section id="katalog">
            <h2 class="section-title">Katalog Buku</h2>
            <div class="cards">
                
                <!-- BUKU 1 -->
                <article class="card kategori-fiksi">
                    <span class="kategori-tag">Fiksi</span>
                    <h3 class="judul-buku">Harry Potter and the Sorcerer's Stone</h3>
                    <p class="penulis">Penulis: J.K. Rowling</p>
                    <p>Tahun: 1997 | Bloomsbury</p>
                    <div class="card-footer">
                        <a href="https://www.goodreads.com/book/show/3.Harry_Potter_and_the_Sorcerer_s_Stone" target="_blank" class="btn-detail">Lihat Detail ➔</a>
                    </div>
                </article>

                <!-- BUKU 2 -->
                <article class="card kategori-fiksi">
                    <span class="kategori-tag">Fiksi</span>
                    <h3 class="judul-buku">The World of Ice & Fire</h3>
                    <p class="penulis">Penulis: George R.R. Martin</p>
                    <p>Tahun: 2014 | Bantam</p>
                    <div class="card-footer">
                        <a href="https://www.goodreads.com/book/show/17345242" target="_blank" class="btn-detail">Lihat Detail ➔</a>
                    </div>
                </article>

                <!-- BUKU 3 -->
                <article class="card kategori-teknologi">
                    <span class="kategori-tag">Teknologi</span>
                    <h3 class="judul-buku">Dasar Pemrograman HTML dan CSS</h3>
                    <p class="penulis">Penulis: Jon Duckett</p>
                    <p>Tahun: 2011 | Wiley</p>
                    <div class="card-footer">
                        <a href="https://books.google.co.id/books?id=aGjaBTbT0o0C&printsec=copyright&hl=id#v=onepage&q&f=false" target="_blank" class="btn-detail">Lihat Detail ➔</a>
                    </div>
                </article>

                <!-- BUKU 4 -->
                <article class="card kategori-ilmu">
                    <span class="kategori-tag">Ilmu Perpustakaan</span>
                    <h3 class="judul-buku">Pengantar Ilmu Perpustakaan</h3>
                    <p class="penulis">Penulis: Sulistyo-Basuki</p>
                    <p>Tahun: 1991 | Gramedia</p>
                    <div class="card-footer">
                        <a href="https://www.goodreads.com/book/show/45319850-pengantar-ilmu-perpustakaan" target="_blank" class="btn-detail">Lihat Detail ➔</a>
                    </div>
                </article>

            </div>
        </section>

        <!-- JURNAL -->
        <section id="jurnal">
            <h2 class="section-title">Katalog Jurnal Akademik</h2>
            <div class="cards">
                
                <!-- JURNAL 1 -->
                <article class="card">
                    <span class="kategori-tag" style="background-color: #fce7f3; color: #be185d;">Jurnal</span>
                    <h3 class="judul-buku">Jurnal Visi Pustaka</h3>
                    <p class="penulis">Penerbit: Perpusnas RI</p>
                    <p>Fokus: Kajian literasi, kearsipan, dan layanan informasi.</p>
                    <div class="card-footer">
                        <a href="https://ejournal.perpusnas.go.id/" target="_blank" class="btn-detail">Lihat Detail ➔</a>
                    </div>
                </article>

                <!-- JURNAL 2 -->
                <article class="card">
                    <span class="kategori-tag" style="background-color: #fce7f3; color: #be185d;">Jurnal</span>
                    <h3 class="judul-buku">Jurnal Sistem Informasi (JSI)</h3>
                    <p class="penulis">Penerbit: Fasilkom UI</p>
                    <p>Fokus: Penerapan teknologi informasi di berbagai bidang.</p>
                    <div class="card-footer">
                        <a href="https://jsi.cs.ui.ac.id/index.php/jsi" target="_blank" class="btn-detail">Lihat Detail ➔</a>
                    </div>
                </article>

            </div>
        </section>

    </main>

    <footer>
        <p>&copy; 2026 Portal Perpustakaan Digital Jackpot - Roihan Nabil</p>
    </footer>

</body>
</html>
