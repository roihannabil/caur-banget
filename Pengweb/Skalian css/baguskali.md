<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>The Beatles - Official</title>
    <style>
        /* Mengatur font, warna latar gelap, dan reset jarak */
        body {
            font-family: 'Helvetica Neue', Arial, sans-serif;
            background-color: #0d0d0d; /* Hitam pekat */
            color: #e0e0e0;
            margin: 0;
            line-height: 1.6;
        }

        /* Membatasi lebar konten */
        .container {
            width: 90%;
            max-width: 1000px;
            margin: 0 auto;
            padding: 20px 0;
        }

        /* Tampilan Header yang lebih elegan */
        #header {
            background: linear-gradient(135deg, #1a1a1a, #000000);
            color: white;
            padding: 60px 20px;
            text-align: center;
            border-bottom: 4px solid #d90429; /* Aksen merah */
        }

        #header h1 {
            font-size: 42px;
            margin: 0 0 10px 0;
            letter-spacing: 2px;
            text-transform: uppercase;
        }

        #header p {
            max-width: 700px;
            margin: 0 auto;
            color: #aaaaaa;
            font-size: 16px;
        }

        /* Bagian tombol dan formulir */
        #section1 {
            margin: 50px 0;
            text-align: center;
            background-color: #1a1a1a;
            padding: 30px;
            border-radius: 12px;
            box-shadow: 0 8px 24px rgba(0,0,0,0.5);
        }

        .intro {
            font-size: 18px;
            margin-bottom: 20px;
        }

        .btn {
            display: inline-block;
            background-color: #d90429; /* Merah The Beatles */
            color: white;
            padding: 12px 24px;
            text-decoration: none;
            border: none;
            border-radius: 50px; /* Tombol membulat modern */
            cursor: pointer;
            font-weight: bold;
            transition: all 0.3s ease;
            margin-bottom: 20px;
        }

        .btn:hover {
            background-color: #ff1a3c;
            transform: translateY(-3px);
            box-shadow: 0 5px 15px rgba(217, 4, 41, 0.4);
        }

        .form-box {
            display: flex;
            justify-content: center;
            gap: 10px;
            flex-wrap: wrap;
        }

        .form-box input {
            padding: 12px 20px;
            width: 250px;
            border: 1px solid #333;
            border-radius: 50px;
            background-color: #262626;
            color: white;
            outline: none;
        }

        .form-box input:focus {
            border-color: #d90429;
        }

        /* Judul Section Album */
        .judul-section {
            text-align: center;
            color: #ffffff;
            margin: 60px 0 30px 0;
            font-size: 28px;
            text-transform: uppercase;
            letter-spacing: 1px;
        }

        /* Layout Flexbox untuk Kartu Album */
        .cards {
            display: flex;
            gap: 30px;
            flex-wrap: wrap;
            justify-content: center;
        }

        /* Pengaturan Kartu Modern */
        .card {
            flex: 1 1 280px;
            max-width: 320px;
            background-color: #1a1a1a;
            border-radius: 12px;
            overflow: hidden; /* Agar gambar tidak keluar dari sudut membulat */
            box-shadow: 0 10px 30px rgba(0,0,0,0.8);
            transition: transform 0.3s ease, box-shadow 0.3s ease;
        }

        .card:hover {
            transform: translateY(-10
            
