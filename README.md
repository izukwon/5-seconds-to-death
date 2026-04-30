<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>15 Seconds to Death (Chaos Edition)</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Orbitron:wght@400;700&family=Roboto:wght@400;700&display=swap" rel="stylesheet">
    <style>
        :root {
            --death-red: #ff0000;
            --m-gold: #d4af37;
            --m-blue: #000833;
        }

        body {
            background: radial-gradient(circle, #1a0000 0%, #000000 100%);
            color: white;
            font-family: 'Roboto', sans-serif;
            height: 100vh;
            margin: 0;
            overflow: hidden;
            display: flex;
            align-items: center;
            justify-content: center;
            touch-action: manipulation;
        }

        h1, .timer-font { font-family: 'Orbitron', sans-serif; }

        .hex-button {
            position: absolute; 
            background: linear-gradient(180deg, #2a0000 0%, #000000 100%);
            border: 2px solid var(--death-red);
            padding: 8px 16px;
            cursor: pointer;
            transition: transform 0.1s linear, background 0.2s;
            min-width: 140px;
            max-width: 180px;
            clip-path: polygon(10% 0%, 90% 0%, 100% 50%, 90% 100%, 10% 100%, 0% 50%);
            text-align: center;
            user-select: none;
            z-index: 10;
            font-size: 0.875rem;
            word-wrap: break-word;
        }

        @media (min-width: 768px) {
            .hex-button {
                padding: 10px 25px;
                min-width: 200px;
                max-width: none;
                font-size: 1rem;
            }
        }

        .hex-button:hover {
            border-color: white;
            box-shadow: 0 0 15px var(--death-red);
        }

        .hex-button.correct { background: #00ff00 !important; color: black !important; border-color: white !important; }
        .hex-button.wrong { background: #ff0000 !important; color: white !important; }

        .question-box {
            width: 90%;
            max-width: 800px;
            background: rgba(0, 0, 0, 0.85);
            border: 2px solid var(--death-red);
            padding: 20px;
            text-align: center;
            font-size: 1.1rem;
            font-weight: bold;
            z-index: 5;
            box-shadow: 0 0 30px rgba(255, 0, 0, 0.2);
            pointer-events: auto;
        }

        @media (min-width: 768px) {
            .question-box {
                padding: 30px;
                font-size: 1.4rem;
            }
        }

        .timer-container {
            width: 100%;
            height: 6px;
            background: #333;
            position: fixed;
            top: 0;
            left: 0;
            z-index: 50;
        }

        #timer-bar {
            height: 100%;
            background: var(--death-red);
            width: 100%;
            transition: width 0.1s linear;
        }

        .lives-display {
            position: fixed;
            top: 15px;
            left: 15px;
            font-size: 1rem;
            color: var(--death-red);
            z-index: 40;
        }

        .score-display {
            position: fixed;
            top: 15px;
            right: 15px;
            font-size: 1rem;
            color: var(--m-gold);
            z-index: 40;
        }

        @media (min-width: 768px) {
            .lives-display, .score-display {
                font-size: 1.5rem;
                top: 20px;
            }
        }

        .game-over-screen {
            position: fixed;
            inset: 0;
            background: black;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            z-index: 100;
            padding: 20px;
        }

        @keyframes shake {
            0% { transform: translate(1px, 1px) rotate(0deg); }
            10% { transform: translate(-1px, -2px) rotate(-1deg); }
            20% { transform: translate(-3px, 0px) rotate(1deg); }
            30% { transform: translate(3px, 2px) rotate(0deg); }
            40% { transform: translate(1px, -1px) rotate(1deg); }
            50% { transform: translate(-1px, 2px) rotate(-1deg); }
        }

        .critical-time {
            animation: shake 0.5s infinite;
            color: red !important;
        }

        .hidden-target {
            cursor: pointer;
            transition: opacity 0.2s;
        }
        .hidden-target:active { opacity: 0.5; }
    </style>
</head>
<body>

    <div id="timer-ui" class="timer-container hidden">
        <div id="timer-bar"></div>
    </div>

    <div id="game-app" class="w-full h-full relative overflow-hidden flex flex-col items-center justify-center">
        
        <!-- Main Menu -->
        <div id="menu" class="text-center space-y-6 md:space-y-10 animate-pulse px-4">
            <div>
                <h1 class="text-4xl md:text-8xl font-black tracking-tighter text-red-600">15 SECONDS</h1>
                <h2 class="text-xl md:text-4xl font-bold text-white italic">TO DEATH</h2>
                <p class="text-xs md:text-sm text-gray-500 uppercase tracking-widest mt-2">Chaos Edition</p>
            </div>
            
            <div class="grid grid-cols-1 sm:grid-cols-2 gap-4 max-w-2xl mx-auto pt-6">
                <button onclick="startGame('tv')" class="hex-button relative !static w-full sm:w-64 mx-auto">TV SERIES</button>
                <button onclick="startGame('brands')" class="hex-button relative !static w-full sm:w-64 mx-auto">BRANDS</button>
                <button onclick="startGame('history')" class="hex-button relative !static w-full sm:w-64 mx-auto">HISTORY</button>
                <button onclick="startGame('logic')" class="hex-button relative !static w-full sm:w-64 mx-auto text-yellow-500">LOGIKA</button>
            </div>
        </div>

        <!-- Gameplay -->
        <div id="game-ui" class="hidden w-full h-full flex flex-col items-center justify-center">
            <div class="lives-display font-bold" id="lives-text">LIVES: 3</div>
            <div class="score-display font-bold" id="score-text">CASH: $0</div>
            <div id="countdown-text" class="timer-font text-4xl md:text-6xl fixed top-12 md:top-10 font-bold opacity-30">15.00</div>

            <div id="question" class="question-box"></div>

            <div id="chaos-arena" class="w-full h-full absolute inset-0 pointer-events-none">
                <!-- Tombol bergerak -->
            </div>
        </div>

        <!-- Game Over -->
        <div id="game-over" class="hidden game-over-screen">
            <h1 class="text-5xl md:text-7xl font-black text-red-700 mb-4 text-center">YOU DIED</h1>
            <p id="final-score" class="text-xl md:text-3xl text-white mb-10">Total Skor: $0</p>
            <button onclick="resetToMenu()" class="hex-button relative !static w-64">RESPAWN</button>
        </div>
    </div>

    <script>
        const triviaData = {
            tv: [
                { q: "Siapa pemeran Wednesday Addams di Netflix?", a: ["Jenna Ortega", "Emma Myers", "Sadie Sink", "Millie Bobby"], c: 0 },
                { q: "Apa nama pulau di 'Squid Game'?", a: ["Jeju", "Nami", "Dokdo", "Sungsan"], c: 0 },
                { q: "Tahun berapa 'Friends' pertama kali tayang?", a: ["1990", "1994", "1996", "2000"], c: 1 },
                { q: "Di 'The Last of Us', apa penyebab kiamat?", a: ["Virus", "Jamur", "Nuklir", "Zombie"], c: 1 },
                { q: "Siapa identitas asli Professor di Money Heist?", a: ["Sergio Marquina", "Berlin", "Denver", "Rio"], c: 0 },
                { q: "Siapa karakter utama 'Peaky Blinders'?", a: ["Thomas Shelby", "Arthur Shelby", "John Shelby", "Michael Gray"], c: 0 },
                { q: "Apa nama kota di serial 'Stranger Things'?", a: ["Hawkins", "Derry", "Riverdale", "Sunnydale"], c: 0 },
                { q: "Berapa jumlah total 'Game of Thrones' season?", a: ["6", "7", "8", "9"], c: 2 },
                { q: "Drama Korea 'Moving' tayang di platform apa?", a: ["Netflix", "Disney+", "Viu", "Prime"], c: 1 },
                { q: "Serial 'The Bear' fokus pada profesi apa?", a: ["Chef", "Polisi", "Dokter", "Pengacara"], c: 0 },
                { q: "Apa nama naga milik Daenerys yang paling besar?", a: ["Drogon", "Viserion", "Rhaegal", "Balerion"], c: 0 },
                { q: "Siapa pembunuh asli di serial 'Stranger Things' Season 4?", a: ["Vecna", "Demogorgon", "Mind Flayer", "Brenner"], c: 0 },
                { q: "Better Call Saul adalah spin-off dari serial apa?", a: ["Breaking Bad", "Narcos", "Ozark", "The Wire"], c: 0 },
                { q: "Apa nama perusahaan fiktif di 'Succession'?", a: ["Waystar Royco", "E Corp", "Pied Piper", "Hooli"], c: 0 },
                { q: "Siapa karakter utama di 'One Piece' live action?", a: ["Monkey D. Luffy", "Zoro", "Nami", "Usopp"], c: 0 },
                { q: "Serial Netflix 'Dark' berasal dari negara mana?", a: ["Jerman", "Spanyol", "Prancis", "Denmark"], c: 0 },
                { q: "Apa nama kapal di serial 'The Mandalorian'?", a: ["Razor Crest", "Millennium Falcon", "Slave I", "X-Wing"], c: 0 },
                { q: "Siapa pemeran Geralt of Rivia di 'The Witcher' S1-S3?", a: ["Henry Cavill", "Liam Hemsworth", "Tom Hardy", "Jason Momoa"], c: 0 },
                { q: "Serial 'The Crown' menceritakan tentang keluarga apa?", a: ["Kerajaan Inggris", "Kennedy", "Kardashian", "Romanov"], c: 0 },
                { q: "Apa nama distrik utama di 'Hunger Games'?", a: ["Capitol", "Panem", "Distrik 12", "Distrik 13"], c: 0 },
                { q: "Di serial 'Friends', apa pekerjaan Chandler Bing?", a: ["Data Analysis", "Chef", "Aktor", "Paleontologis"], c: 0 },
                { q: "Siapa nama asli 'The Professor' di Money Heist?", a: ["Sergio", "Andres", "Anibal", "Paco"], c: 0 },
                { q: "Serial 'Ted Lasso' bertema olahraga apa?", a: ["Sepak Bola", "Basket", "Baseball", "Tenis"], c: 0 },
                { q: "Apa nama kota tempat tinggal 'The Simpsons'?", a: ["Springfield", "Quahog", "South Park", "Shelbyville"], c: 0 },
                { q: "Siapa pencipta serial 'Squid Game'?", a: ["Hwang Dong-hyuk", "Bong Joon-ho", "Park Chan-wook", "Lee Chang-dong"], c: 0 },
                { q: "Apa warna kostum 'Power Rangers' pertama Tommy Oliver?", a: ["Hijau", "Putih", "Merah", "Hitam"], c: 0 },
                { q: "Serial 'The Boys' menceritakan tentang apa?", a: ["Superhero Korup", "Detektif Anak", "Grup Band", "Mafia Italia"], c: 0 },
                { q: "Apa nama sekolah di 'Sex Education'?", a: ["Moordale", "Riverdale", "Liberty High", "Las Encinas"], c: 0 },
                { q: "Siapa pemeran utama serial 'Euphoria'?", a: ["Zendaya", "Sydney Sweeney", "Hunter Schafer", "Jacob Elordi"], c: 0 },
                { q: "Serial 'Black Mirror' bertema tentang apa?", a: ["Teknologi Dystopia", "Horor Klasik", "Komedi Romantis", "Sejarah Dunia"], c: 0 }
            ],
            brands: [
                { q: "Apa nama pendiri Microsoft?", a: ["Steve Jobs", "Bill Gates", "Elon Musk", "Mark Z."], c: 1 },
                { q: "Slogan 'Real Magic' milik brand apa?", a: ["Pepsi", "Coca Cola", "Red Bull", "Sprite"], c: 1 },
                { q: "Logo brand apa yang menggunakan siluet burung biru (lama)?", a: ["Twitter", "Facebook", "Instagram", "Reddit"], c: 0 },
                { q: "Louis Vuitton berasal dari negara mana?", a: ["Italia", "Prancis", "Inggris", "USA"], c: 1 },
                { q: "Konsol game PlayStation dibuat oleh?", a: ["Nintendo", "Microsoft", "Sony", "Sega"], c: 2 },
                { q: "Brand jam tangan mewah dengan logo mahkota?", a: ["Omega", "Rolex", "Patek", "Cartier"], c: 1 },
                { q: "Mobil listrik Tesla didirikan oleh?", a: ["Elon Musk", "Jeff Bezos", "Larry Page", "Tim Cook"], c: 0 },
                { q: "Aplikasi video pendek dari ByteDance?", a: ["SnackVideo", "TikTok", "Reels", "YouTube"], c: 1 },
                { q: "Brand olahraga dengan logo 3 garis?", a: ["Nike", "Adidas", "Puma", "Reebok"], c: 1 },
                { q: "Produsen ban yang memberikan rating restoran?", a: ["Bridgestone", "Michelin", "Dunlop", "Pirelli"], c: 1 },
                { q: "Perusahaan induk dari Google adalah?", a: ["Alphabet", "Meta", "Amazon", "Tesla"], c: 0 },
                { q: "Apa nama sistem operasi smartphone milik Apple?", a: ["iOS", "Android", "Windows", "HarmonyOS"], c: 0 },
                { q: "Brand kopi yang berasal dari Seattle, AS?", a: ["Starbucks", "Dunkin", "Costa", "Luwak"], c: 0 },
                { q: "Apa warna dominan logo Ferrari?", a: ["Merah", "Kuning", "Hitam", "Biru"], c: 0 },
                { q: "Brand mie instan paling populer di Indonesia?", a: ["Indomie", "Sedaap", "Sarimi", "Supermi"], c: 0 },
                { q: "Perusahaan raksasa e-commerce milik Jeff Bezos?", a: ["Amazon", "Alibaba", "eBay", "Shopee"], c: 0 },
                { q: "Apa produk utama dari brand 'Canon'?", a: ["Kamera", "Sepatu", "Jam", "Mobil"], c: 0 },
                { q: "Brand kosmetik 'Fenty Beauty' milik siapa?", a: ["Rihanna", "Beyonce", "Selena Gomez", "Kylie Jenner"], c: 0 },
                { q: "Logo brand apa yang berbentuk tanda centang?", a: ["Nike", "Adidas", "Reebok", "Puma"], c: 0 },
                { q: "Apa nama layanan streaming musik milik Spotify?", a: ["Spotify", "Apple Music", "Joox", "Deezer"], c: 0 },
                { q: "Negara asal brand furnitur IKEA?", a: ["Swedia", "Denmark", "Norwegia", "Finlandia"], c: 0 },
                { q: "Apa singkatan dari brand otomotif BMW?", a: ["Bayerische Motoren Werke", "British Motor Works", "Best Motor World", "Berlin Motor Works"], c: 0 },
                { q: "Perusahaan mana yang memproduksi konsol Switch?", a: ["Nintendo", "Sony", "Microsoft", "Sega"], c: 0 },
                { q: "Apa nama asisten virtual buatan Amazon?", a: ["Alexa", "Siri", "Cortana", "Bixby"], c: 0 },
                { q: "Brand fashion 'Zara' berasal dari negara mana?", a: ["Spanyol", "Italia", "Prancis", "Jerman"], c: 0 },
                { q: "Apa nama perusahaan game pembuat 'Fortnite'?", a: ["Epic Games", "Ubisoft", "EA", "Activision"], c: 0 },
                { q: "Brand mewah yang logonya dua huruf G berhadapan?", a: ["Gucci", "Givency", "Goyard", "Gap"], c: 0 },
                { q: "Apa nama platform pembayaran digital milik PayPal?", a: ["PayPal", "Stripe", "Venmo", "Payoneer"], c: 0 },
                { q: "Brand mainan balok susun asal Denmark?", a: ["Lego", "Mattel", "Hasbro", "Barbie"], c: 0 },
                { q: "Logo brand apa yang berbentuk apel digigit?", a: ["Apple", "Blackberry", "Orange", "Android"], c: 0 }
            ],
            history: [
                { q: "Siapa penemu benua Amerika?", a: ["Vasco da Gama", "C. Columbus", "Magelhaens", "Marco Polo"], c: 1 },
                { q: "Tahun berapa Indonesia merdeka?", a: ["1944", "1945", "1946", "1950"], c: 1 },
                { q: "Raja terkenal dari Kerajaan Majapahit?", a: ["Ken Arok", "Hayam Wuruk", "Mulawarman", "Purnawarman"], c: 1 },
                { q: "Peristiwa 'Boston Tea Party' terjadi di negara?", a: ["Inggris", "Amerika Serikat", "Prancis", "Belanda"], c: 1 },
                { q: "Siapa pemimpin Nazi di PD II?", a: ["Mussolini", "Hitler", "Stalin", "Churchill"], c: 1 },
                { q: "Pyramida Giza dibangun oleh bangsa?", a: ["Maya", "Mesir Kuno", "Inca", "Yunani"], c: 1 },
                { q: "Perang Diponegoro berakhir pada tahun?", a: ["1825", "1830", "1845", "1850"], c: 1 },
                { q: "Siapa presiden wanita pertama di dunia?", a: ["Megawati", "S. Bandaranaike", "Indira Gandhi", "Corazon Aquino"], c: 1 },
                { q: "Kapan Tembok Cina mulai dibangun?", a: ["Dinasti Qin", "Dinasti Han", "Dinasti Ming", "Dinasti Tang"], c: 0 },
                { q: "Nama asli Pangeran Diponegoro?", a: ["Antasari", "Mustahar", "Ontowiryo", "Surapati"], c: 2 },
                { q: "Kapan Perang Dunia I dimulai?", a: ["1914", "1918", "1939", "1945"], c: 0 },
                { q: "Siapa penemu lampu pijar?", a: ["Thomas Edison", "Nikola Tesla", "Einstein", "Newton"], c: 0 },
                { q: "Apa nama kapal yang tenggelam tahun 1912?", a: ["Titanic", "Britannic", "Olympic", "Lusitania"], c: 0 },
                { q: "Siapa tokoh yang menjahit bendera Merah Putih?", a: ["Fatmawati", "Kartini", "Cut Nyak Dien", "Megawati"], c: 0 },
                { q: "Negara mana yang menjatuhkan bom di Hiroshima?", a: ["Amerika Serikat", "Jerman", "Inggris", "Uni Soviet"], c: 0 },
                { q: "Siapa penemu mesin cetak pertama?", a: ["Johannes Gutenberg", "James Watt", "Alexander Bell", "Galileo"], c: 0 },
                { q: "Tahun berapa Uni Soviet runtuh?", a: ["1991", "1989", "1995", "2000"], c: 0 },
                { q: "Apa nama perjanjian kemerdekaan RI dengan Belanda?", a: ["Linggarjati", "Renville", "Roem-Royen", "KMB"], c: 3 },
                { q: "Siapa kaisar pertama Romawi?", a: ["Augustus", "Julius Caesar", "Nero", "Caligula"], c: 0 },
                { q: "Di mana letak Candi Borobudur?", a: ["Magelang", "Yogyakarta", "Solo", "Semarang"], c: 0 },
                { q: "Siapa manusia pertama di ruang angkasa?", a: ["Yuri Gagarin", "Neil Armstrong", "Buzz Aldrin", "John Glenn"], c: 0 },
                { q: "Apa nama perang antara AS dan Uni Soviet (1947-1991)?", a: ["Perang Dingin", "Perang Teluk", "Perang Saudara", "Perang Vietnam"], c: 0 },
                { q: "Revolusi Perancis dimulai pada tahun?", a: ["1789", "1776", "1804", "1815"], c: 0 },
                { q: "Siapa penulis naskah proklamasi Indonesia?", a: ["Sayuti Melik", "Soekarno", "Moh Hatta", "Ahmad Soebardjo"], c: 0 },
                { q: "Kerajaan Islam pertama di Indonesia?", a: ["Samudera Pasai", "Demak", "Mataram", "Aceh"], c: 0 },
                { q: "Siapa penemu benua Australia?", a: ["James Cook", "Marco Polo", "Vasco da Gama", "Amerigo Vespucci"], c: 0 },
                { q: "Apa nama bom atom yang dijatuhkan di Nagasaki?", a: ["Fat Man", "Little Boy", "Big Boy", "Tsar Bomba"], c: 0 },
                { q: "Tahun berapa Sumpah Pemuda dibacakan?", a: ["1928", "1920", "1930", "1945"], c: 0 },
                { q: "Siapa perdana menteri pertama Inggris wanita?", a: ["Margaret Thatcher", "Theresa May", "Elizabeth II", "Diana"], c: 0 },
                { q: "Apa nama mata uang kuno Kerajaan Majapahit?", a: ["Gobog", "Rupiah", "Gulden", "Yen"], c: 0 }
            ],
            logic: [
                { q: "Ada berapa huruf 'f' dalam 'Fifteen Seconds to Death'?", a: ["0", "1", "2", "3"], c: 1 },
                { q: "Apa yang selalu datang tapi tidak pernah tiba?", a: ["Hujan", "Besok", "Masa Lalu", "Kado"], c: 1 },
                { q: "Semakin banyak diambil, semakin besar ia. Apakah itu?", a: ["Lubang", "Uang", "Ilmu", "Umur"], c: 0 },
                { q: "Bulan apa yang memiliki 28 hari?", a: ["Februari", "Januari", "Semua Bulan", "Maret"], c: 2 },
                { q: "Jika ada 3 apel dan kamu ambil 2, berapa apel kamu punya?", a: ["1", "2", "3", "0"], c: 1 },
                { q: "Warna apa yang dihasilkan dari merah + biru?", a: ["Hijau", "Ungu", "Orange", "Cokelat"], c: 1 },
                { q: "Ibu Budi punya 3 anak: Ali, Abi, dan...?", a: ["Budi", "Abu", "Ari", "Cici"], c: 0 },
                { q: "Berapa banyak angka 9 dari 1 sampai 100?", a: ["10", "11", "20", "21"], c: 2 },
                { q: "Pencet kata 'MATI' untuk selamat.", a: ["Selamat", "Mati", "Hidup", "Kabur"], c: 1 },
                { q: "Coba klik kata 'Skor' di layar.", a: ["Klik", "Ini", "Salah", "Coba"], hidden: "Skor" },
                { q: "Ada berapa banyak air dalam gelas kosong?", a: ["Penuh", "Setengah", "Tidak Ada", "Sedikit"], c: 2 },
                { q: "Apa yang pecah saat kamu menyebut namanya?", a: ["Kaca", "Hati", "Kesunyian", "Janji"], c: 2 },
                { q: "Jika sebuah kereta listrik melaju ke utara, asapnya ke mana?", a: ["Selatan", "Timur", "Barat", "Tidak Ada"], c: 3 },
                { q: "Siapa ayah dari saudara perempuanmu?", a: ["Kakek", "Paman", "Ayah", "Abang"], c: 2 },
                { q: "Mana yang lebih berat: 1kg Besi atau 1kg Kapas?", a: ["Besi", "Kapas", "Sama Saja", "Tidak Tahu"], c: 2 },
                { q: "Ada berapa angka nol dalam satu juta?", a: ["5", "6", "7", "8"], c: 1 },
                { q: "Seekor merak bertelur di atas atap, telurnya jatuh ke mana?", a: ["Kanan", "Kiri", "Pecah", "Merak Tidak Bertelur"], c: 3 },
                { q: "Apa yang naik tapi tidak pernah turun?", a: ["Balon", "Umur", "Asap", "Harga"], c: 1 },
                { q: "Memiliki ekor tapi tidak memiliki tubuh. Apakah itu?", a: ["Ular", "Koin", "Layangan", "Bayangan"], c: 1 },
                { q: "Memiliki mata tapi tidak bisa melihat. Apakah itu?", a: ["Kentang", "Jarum", "Badai", "Semua Benar"], c: 3 },
                { q: "Dalam balap lari, jika kamu menyalip orang terakhir, kamu posisi berapa?", a: ["Terakhir", "Kedua Terakhir", "Pertama", "Mustahil"], c: 3 },
                { q: "Pencet huruf 'L' pada tulisan 'LIVES' di kiri atas.", a: ["Klik", "Sini", "Salah", "Nggak Ada"], hidden: "Lives" },
                { q: "Pencet tanda titik '.' pada timer di atas.", a: ["Pencet", "Titik", "Mana", "Susah"], hidden: "TimerDot" },
                { q: "Berapa hasil dari 1 + 1 x 0?", a: ["0", "1", "2", "3"], c: 1 },
                { q: "Apa nama game ini?", a: ["Millionaire", "Trivia Chaos", "15 Seconds to Death", "5 Seconds"], c: 2 },
                { q: "Klik tulisan 'CASH' di kanan atas.", a: ["Duit", "Kaya", "Salah", "Uang"], hidden: "Cash" },
                { q: "Warna label Skor adalah...", a: ["Merah", "Emas", "Biru", "Putih"], c: 1 },
                { q: "Apa kebalikan dari 'Mati'?", a: ["Lahir", "Hidup", "Bangkit", "Makan"], c: 1 },
                { q: "Klik angka '1' di timer atas.", a: ["Satu", "Dua", "Klik", "Mana"], hidden: "TimerOne" },
                { q: "Siapa kakek dari cucu Andra?", a: ["Ayah Andra", "Andra", "Anak Andra", "Cucu Andra"], c: 1 }
            ]
        };

        let currentQuestions = [];
        let currentIndex = 0;
        let score = 0;
        let lives = 3;
        let timeLeft = 1500; 
        let timerInterval;
        let movementInterval;
        let answerButtons = [];
        let isGameActive = false;

        // Inisialisasi awal saat halaman dimuat
        window.onload = function() {
            resetToMenu();
        };

        function startGame(category) {
            currentQuestions = [...triviaData[category]].sort(() => Math.random() - 0.5);
            document.getElementById('menu').classList.add('hidden');
            document.getElementById('game-ui').classList.remove('hidden');
            document.getElementById('timer-ui').classList.remove('hidden');
            document.getElementById('game-over').classList.add('hidden');
            isGameActive = true;
            nextQuestion();
        }

        function resetToMenu() {
            // Reset Data
            score = 0;
            lives = 3;
            currentIndex = 0;
            isGameActive = false;
            
            // Hentikan semua timer
            clearInterval(timerInterval);
            clearInterval(movementInterval);
            
            // Atur Tampilan Awal Secara Tegas
            const menu = document.getElementById('menu');
            const gameUi = document.getElementById('game-ui');
            const timerUi = document.getElementById('timer-ui');
            const gameOver = document.getElementById('game-over');
            const arena = document.getElementById('chaos-arena');

            if (menu) menu.classList.remove('hidden');
            if (gameUi) gameUi.classList.add('hidden');
            if (timerUi) timerUi.classList.add('hidden');
            if (gameOver) gameOver.classList.add('hidden');
            if (arena) arena.innerHTML = '';
            
            // Update UI teks
            const scoreText = document.getElementById('score-text');
            const livesText = document.getElementById('lives-text');
            if (scoreText) scoreText.innerHTML = `CASH: $0`;
            if (livesText) {
                livesText.innerText = `LIVES: 3`;
                livesText.classList.remove('critical-time');
            }
            
            const countdownText = document.getElementById('countdown-text');
            if (countdownText) {
                countdownText.innerText = "15.00";
                countdownText.classList.remove('critical-time');
            }
            
            const timerBar = document.getElementById('timer-bar');
            if (timerBar) timerBar.style.width = '100%';
        }

        function nextQuestion() {
            if (currentIndex >= currentQuestions.length || lives <= 0) {
                gameOver();
                return;
            }

            clearInterval(timerInterval);
            clearInterval(movementInterval);
            document.getElementById('chaos-arena').innerHTML = '';
            answerButtons = [];
            
            const qData = currentQuestions[currentIndex];
            const qBox = document.getElementById('question');
            
            if (qData.hidden) {
                qBox.innerHTML = qData.q;
                
                document.getElementById('lives-text').innerHTML = `LIVES: ${lives}`;
                document.getElementById('score-text').innerHTML = `CASH: $${score.toLocaleString()}`;
                document.getElementById('countdown-text').innerHTML = "15.00";

                if (qData.hidden === "Skor" || qData.hidden === "Cash") {
                    document.getElementById('score-text').innerHTML = `CASH: <span onclick="handleCorrectClick()" class="hidden-target text-white underline font-bold">$${score.toLocaleString()}</span>`;
                } else if (qData.hidden === "Lives") {
                    document.getElementById('lives-text').innerHTML = `<span onclick="handleCorrectClick()" class="hidden-target text-white underline font-bold">L</span>IVES: ${lives}`;
                } else if (qData.hidden === "TimerDot") {
                    document.getElementById('countdown-text').innerHTML = `15<span onclick="handleCorrectClick()" class="hidden-target text-red-500 underline font-bold">.</span>00`;
                } else if (qData.hidden === "TimerOne") {
                    document.getElementById('countdown-text').innerHTML = `<span onclick="handleCorrectClick()" class="hidden-target text-red-500 underline font-bold">1</span>5.00`;
                }
            } else {
                qBox.innerText = qData.q;
                document.getElementById('score-text').innerText = `CASH: $${score.toLocaleString()}`;
                document.getElementById('lives-text').innerText = `LIVES: ${lives}`;
            }

            const speedFactor = window.innerWidth < 768 ? 5 : 8;

            qData.a.forEach((text, idx) => {
                const btn = document.createElement('div');
                btn.className = 'hex-button pointer-events-auto';
                btn.innerHTML = text;
                
                document.getElementById('chaos-arena').appendChild(btn);
                const rect = btn.getBoundingClientRect();
                
                const pos = {
                    x: Math.random() * (window.innerWidth - rect.width),
                    y: Math.random() * (window.innerHeight - rect.height),
                    vx: (Math.random() - 0.5) * speedFactor,
                    vy: (Math.random() - 0.5) * speedFactor,
                    width: rect.width,
                    height: rect.height
                };
                
                btn.style.left = pos.x + 'px';
                btn.style.top = pos.y + 'px';
                
                btn.onclick = (e) => {
                    e.stopPropagation();
                    if (qData.hidden) { handleWrongClick(btn); }
                    else if (idx === qData.c) { handleCorrectClick(btn); }
                    else { handleWrongClick(btn); }
                };

                answerButtons.push({ element: btn, pos: pos });
            });

            timeLeft = 1500;
            startTimer();
            startMovement();
        }

        function startTimer() {
            timerInterval = setInterval(() => {
                timeLeft--;
                const bar = document.getElementById('timer-bar');
                const txt = document.getElementById('countdown-text');
                
                if (bar) bar.style.width = (timeLeft / 15) + '%';
                
                const qData = currentQuestions[currentIndex];
                if (txt && (!qData || !qData.hidden || (qData.hidden !== "TimerDot" && qData.hidden !== "TimerOne"))) {
                    txt.innerText = (timeLeft / 100).toFixed(2);
                }

                if (txt && timeLeft < 300) { 
                    txt.classList.add('critical-time');
                } else if (txt) {
                    txt.classList.remove('critical-time');
                }

                if (timeLeft <= 0) {
                    handleTimeOut();
                }
            }, 10);
        }

        function startMovement() {
            movementInterval = setInterval(() => {
                answerButtons.forEach(b => {
                    b.pos.x += b.pos.vx;
                    b.pos.y += b.pos.vy;

                    if (b.pos.x <= 0 || b.pos.x >= window.innerWidth - b.pos.width) b.pos.vx *= -1;
                    if (b.pos.y <= 0 || b.pos.y >= window.innerHeight - b.pos.height) b.pos.vy *= -1;

                    b.pos.x = Math.max(0, Math.min(b.pos.x, window.innerWidth - b.pos.width));
                    b.pos.y = Math.max(0, Math.min(b.pos.y, window.innerHeight - b.pos.height));

                    b.element.style.left = b.pos.x + 'px';
                    b.element.style.top = b.pos.y + 'px';
                });
            }, 20);
        }

        function handleCorrectClick(btn) {
            clearInterval(timerInterval);
            clearInterval(movementInterval);
            if (btn) btn.classList.add('correct');
            score += 1000 * (currentIndex + 1);
            
            setTimeout(() => {
                currentIndex++;
                nextQuestion();
            }, 800);
        }

        function handleWrongClick(btn) {
            clearInterval(timerInterval);
            clearInterval(movementInterval);
            if (btn) btn.classList.add('wrong');
            lives--;
            updateLivesUI();
            
            setTimeout(() => {
                if (lives <= 0) gameOver();
                else {
                    currentIndex++;
                    nextQuestion();
                }
            }, 800);
        }

        function handleTimeOut() {
            clearInterval(timerInterval);
            clearInterval(movementInterval);
            lives--;
            updateLivesUI();
            
            const qBox = document.getElementById('question');
            if (qBox) qBox.innerHTML = "<span class='text-red-500'>WAKTU HABIS!</span>";

            setTimeout(() => {
                if (lives <= 0) gameOver();
                else {
                    currentIndex++;
                    nextQuestion();
                }
            }, 1200);
        }

        function updateLivesUI() {
            const lText = document.getElementById('lives-text');
            if (lText) {
                lText.innerText = `LIVES: ${lives}`;
                if (lives <= 1) lText.classList.add('critical-time');
                else lText.classList.remove('critical-time');
            }
        }

        function gameOver() {
            isGameActive = false;
            clearInterval(timerInterval);
            clearInterval(movementInterval);
            
            const gameUi = document.getElementById('game-ui');
            const timerUi = document.getElementById('timer-ui');
            const gameOverScreen = document.getElementById('game-over');
            const finalScore = document.getElementById('final-score');

            if (gameUi) gameUi.classList.add('hidden');
            if (timerUi) timerUi.classList.add('hidden');
            if (gameOverScreen) gameOverScreen.classList.remove('hidden');
            if (finalScore) finalScore.innerText = `Total Skor: $${score.toLocaleString()}`;
        }

        window.onresize = () => {
            if (isGameActive) {
                answerButtons.forEach(b => {
                    b.pos.x = Math.min(b.pos.x, window.innerWidth - b.pos.width);
                    b.pos.y = Math.min(b.pos.y, window.innerHeight - b.pos.height);
                });
            }
        };
    </script>
</body>
</html>
