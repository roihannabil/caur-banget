/* Tipografi & Background Dasar */
body {
    font-family: Arial, sans-serif;
    background-color: #121212; /* Tema gelap ala web musik */
    color: #ffffff;
    margin: 0;
    line-height: 1.6;
}

/* Membatasi lebar konten agar tidak memenuhi layar penuh */
.container {
    width: 90%;
    max-width: 1100px;
    margin: 0 auto;
    padding: 20px 0;
}

/* ID Selector untuk Header */
#header {
    background-color: #000000;
    color: white;
    text-align: center;
    padding: 40px 20px;
    border-bottom: 2px solid #333;
}

#header h1 {
    font-size: 48px;
    margin-bottom: 10px;
}

/* Form & Tombol */
.form-box {
    margin-top: 20px;
    margin-bottom: 40px;
}

input[type="text"] {
    padding: 10px;
    width: 250px;
    border: none;
    border-radius: 4px;
}

.btn {
    display: inline-block;
    background-color: #d80000;
    color: white;
    padding: 10px 20px;
    text-decoration: none;
    border: none;
    border-radius: 4px;
    font-weight: bold;
    cursor: pointer;
    transition: background-color 0.2s;
}

.btn:hover {
    background-color: #ff1a1a;
}

/* Flexbox untuk menyusun kartu album */
.cards {
    display: flex;
    gap: 20px;
    flex-wrap: wrap; /* Item akan turun ke baris bawah jika layar sempit */
    margin-top: 20px;
}

/* Box Model untuk Kartu Album */
.card {
    flex: 1 1 300px; /* Fleksibilitas ukuran kartu */
    background-color: #1e1e1e;
    padding: 20px;
    border-radius: 8px;
    text-align: center;
    transition: transform 0.2s; /* Animasi mulus saat di-hover */
}

/* Efek Hover untuk menghidupkan tampilan */
.card:hover {
    transform: translateY(-8px); /* Kartu terangkat sedikit saat disentuh mouse */
    box-shadow: 0 10px 20px rgba(0,0,0,0.5);
}

.card img {
    width: 100%;
    height: auto;
    border-radius: 4px;
    box-shadow: 0 4px 8px rgba(0,0,0,0.3);
}

.card h3 {
    margin-top: 15px;
    color: #d80000;
}

footer {
    text-align: center;
    padding: 20px;
    background-color: #000;
    margin-top: 50px;
    border-top: 1px solid #333;
}
