<img width="1875" height="2063" alt="House_Targaryen" src="https://github.com/user-attachments/assets/f91312d9-b068-4d54-81ae-9528658e7727" />



<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Perpustakaan Diagonalley</title>
    
    <style>
        /* =========================================================
           PALET WARNA TEMA "GAME OF THRONES"
           ========================================================= */
        :root {
            --primary-color: #5c0f0f;  /* Merah darah gelap (Targaryen Red) - Sesuai Logo */
            --nav-bg: #1a1a1a;         /* Hitam Obsidian/Batu untuk navigasi */
            --secondary-color: #f4f1ea;/* Putih tulang/Perkamen tua untuk background */
            --text-color: #2b2b2b;     /* Abu-abu sangat gelap untuk teks */
            --white: #ffffff;          /* Putih bersih untuk isi kartu */
            --accent-color: #c5a059;   /* Emas redup (Gold) untuk aksen/hover */
            --tag-bg: #ebd6d6;         /* Merah sangat muda untuk tag */
        }

        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
        }

        body {
            /* Teks biasa menggunakan font modern agar tetap mudah dibaca */
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: var(--secondary-color);
            color: var(--text-color);
            line-height: 1.6;
        }

        /* Judul-judul menggunakan font Serif klasik agar terasa seperti buku fantasi tua */
        h1, h2, h3 {
            font-family: 'Georgia', 'Times New Roman', Times, serif;
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
            padding: 20px;
            border-bottom: 3px solid var(--accent-color); /* Garis emas di bawah header */
        }

        .header-container {
            display: flex;
            align-items: center;
            justify-content: center;
            gap: 20px;
            max-width: 1100px;
            margin: 0 auto;
        }

        .header-logo {
            width: 80px; 
            height: auto;
            border-radius: 50%; /* Membuat logo menjadi bulat sempurna */
            box-shadow: 0 0 10px rgba(0,0,0,0.5); 
        }

        .header-text { text-align: left; }
        .header-text h1 { font-size: 32px; margin-bottom: 5px; }
        .header-text p { font-style: italic; color: #e2d1d1; }

        nav {
            background-color: var(--nav-bg);
            position: sticky;
            top: 0;
            z-index: 100;
            border-bottom: 1px solid #000000;
            box-shadow: 0 2px 5px rgba(0,0,0,0.2);
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
            font-size: 15px;
            padding: 5px 12px;
            border-radius: 4px;
            transition: 0.3s;
            text-transform: uppercase; /* Huruf kapital semua untuk menu */
            letter-spacing: 1px;
        }

        .nav-links a:hover { 
            background-color: var(--primary-color); 
            color: var(--white);
        }

        section {
            padding: 40px 0;
            border-bottom: 2px dashed #d1c8b8;
        }

        .section-title {
            text-align: center;
            font-size: 26px;
            color: var(--primary-color);
            margin-bottom: 30px;
            text-transform: uppercase;
            letter-spacing: 2px;
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
            border: 2px solid #d1c8b8;
            background-color: var(--white);
            border-radius: 30px;
            outline: none;
            font-family: inherit;
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
        .hidden-radio { display: none; }

        .filter-btn {
            padding: 8px 20px;
            background-color: #e0d8c8; /* Warna perkamen sedikit gelap */
            color: var(--text-color);
            border-radius: 20px;
            cursor: pointer;
            font-weight: bold;
            transition: 0.3s;
            display: inline-block;
            border: 1px solid #d1c8b8;
        }

        .filter-btn:hover { background-color: #d1c8b8; }

        /* Mewarnai tombol saat aktif */
        #radio-semua:checked ~ #kategori .btn-semua,
        #radio-fiksi:checked ~ #kategori .btn-fiksi,
        #radio-teknologi:checked ~ #kategori .btn-teknologi,
        #radio-ilmu:checked ~ #kategori .btn-ilmu,
        #radio-jurnal:checked ~ #kategori .btn-jurnal {
            background-color: var(--primary-color);
            color: var(--white);
            border-color: var(--primary-color);
        }

        /* Logika Menyembunyikan Kartu */
        #radio-fiksi:checked ~ #katalog .card:not(.kategori-fiksi) { display: none; }
        #radio-fiksi:checked ~ #jurnal { display: none; } 

        #radio-teknologi:checked ~ #katalog .card:not(.kategori-teknologi) { display: none; }
        #radio-teknologi:checked ~ #jurnal { display: none; } 

        #radio-ilmu:checked ~ #katalog .card:not(.kategori-ilmu) { display: none; }
        #radio-ilmu:checked ~ #jurnal { display: none; } 

        #radio-jurnal:checked ~ #katalog { display: none; } 

        /* =========================================================
           GAYA KARTU BUKU & JURNAL
           ========================================================= */
        .cards {
            display: flex;
            flex-wrap: wrap;
            gap: 24px;
            justify-content: center;
        }

        .card {
            background-color: var(--white);
            flex: 1 1 250px;
            max-width: 300px;
            border-radius: 8px; /* Ujung kartu sedikit lebih tegas */
            border: 1px solid #d1c8b8;
            box-shadow: 0 4px 8px rgba(0,0,0,0.05);
            padding: 24px;
            display: flex;
            flex-direction: column;
            transition: transform 0.2s, box-shadow 0.2s;
        }

        .card:hover {
            transform: translateY(-6px);
            box-shadow: 0 12px 20px rgba(0,0,0,0.12);
        }

        .kategori-tag {
            font-size: 11px;
            font-weight: bold;
            background-color: var(--tag-bg);
            color: var(--primary-color);
            padding: 4px 10px;
            border-radius: 4px;
            align-self: flex-start;
            margin-bottom: 15px;
            text-transform: uppercase;
            letter-spacing: 1px;
        }

        .card h3 { font-size: 20px; margin-bottom: 10px; color: var(--primary-color); line-height: 1.3; }
        .card p { font-size: 14px; margin-bottom: 5px; }
        .penulis { color: #5a5a5a; font-weight: bold; }

        /* Tombol Detail */
        .card-footer {
            margin-top: auto; 
            padding-top: 18px;
            border-top: 1px solid #f4f1ea;
            display: flex;
            justify-content: flex-end;
        }

        .btn-detail {
            background-color: var(--nav-bg); /* Tombol hitam elegan */
            color: var(--white);
            padding: 8px 16px;
            border-radius: 4px;
            text-decoration: none;
            font-size: 12px;
            font-weight: bold;
            transition: 0.3s;
            text-transform: uppercase;
            letter-spacing: 1px;
        }

        .btn-detail:hover { background-color: var(--accent-color); }

        footer {
            background-color: var(--nav-bg);
            color: var(--white);
            text-align: center;
            padding: 25px;
            margin-top: 50px;
            border-top: 3px solid var(--primary-color);
        }

        /* =========================================================
           RESPONSIVITAS MOBILE (MEDIA QUERY)
           ========================================================= */
        @media (max-width: 768px) {
            .cards { flex-direction: column; align-items: center; }
            .card { width: 100%; max-width: 100%; }
            .nav-links { flex-direction: column; text-align: center; gap: 10px; }
            
            .header-container { flex-direction: column; text-align: center; }
            .header-text { text-align: center; }
        }
    </style>
</head>
<body>

    <header>
        <div class="header-container">
            <img src="https://github.com/user-attachments/assets/f91312d9-b068-4d54-81ae-9528658e7727" alt="Logo Perpustakaan" class="header-logo">
            
            <div class="header-text">
                <h1>Perpustakaan Universitas Jackpot</h1>
                <p>Bukan kamu yang memilih buku, melainkan buku yang memilihmu</p>
            </div>
        </div>
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
            <p style="text-align: center; max-width: 750px; margin: 0 auto; font-size: 16px;">
                Langkahkan kaki Anda ke dalam lorong ilmu pengetahuan tanpa batas. Di Perpustakaan Universitas Jackpot, kami menyimpan berbagai 'perkamen' fiksi epik, literatur teknologi masa depan, hingga jurnal akademik yang siap membuka portal wawasan Anda.
            </p>
        </section>

        <!-- FITUR PENCARIAN & FILTER -->
        <section id="kategori">
            <h2 class="section-title">Cari Koleksi</h2>
            
            <div class="search-filter-container">
                <input type="text" class="search-bar" placeholder="Ketik judul, penulis, atau kata kunci (Visual Only)...">
                
                <div class="filter-buttons">
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
                    <span class="kategori-tag">Jurnal</span>
                    <h3 class="judul-buku">Jurnal Visi Pustaka</h3>
                    <p class="penulis">Penerbit: Perpusnas RI</p>
                    <p>Fokus: Kajian literasi, kearsipan, dan layanan informasi.</p>
                    <div class="card-footer">
                        <a href="https://ejournal.perpusnas.go.id/" target="_blank" class="btn-detail">Lihat Detail ➔</a>
                    </div>
                </article>

                <!-- JURNAL 2 -->
                <article class="card">
                    <span class="kategori-tag">Jurnal</span>
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
