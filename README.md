<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>xatframe Radio Player</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        html, body {
            width: 100%;
            height: 100%;
            overflow: hidden;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            /* ภาพพื้นหลังหลักของคุณ */
            background: url('https://xatimg.com/image/j31wIIM7HMVC.png') no-repeat center center fixed;
            background-size: cover;
            color: #fff;
        }

        .xatframe-wrapper {
            position: relative;
            width: 100vw;
            height: 100vh;
            display: flex;
            justify-content: space-between;
            align-items: center;
            padding: 20px 40px;
        }

        /* ===== ด้านซ้าย: เครื่องเล่นเพลง / แผ่นเสียง ===== */
        .player-card {
            width: 250px;
            background: rgba(10, 16, 26, 0.85);
            backdrop-filter: blur(10px);
            border: 1px solid rgba(255, 255, 255, 0.1);
            border-radius: 12px;
            padding: 20px 15px;
            box-shadow: 0 8px 32px rgba(0, 0, 0, 0.6);
            display: flex;
            flex-direction: column;
            align-items: center;
            z-index: 10;
        }

        .player-title {
            font-size: 11px;
            color: #8b98a5;
            letter-spacing: 1px;
            margin-bottom: 15px;
            text-transform: uppercase;
        }

        /* เครื่องเล่นแผ่นเสียง */
        .turntable {
            position: relative;
            width: 130px;
            height: 130px;
            margin-bottom: 15px;
        }

        .vinyl {
            width: 100%;
            height: 100%;
            border-radius: 50%;
            background: radial-gradient(circle, #333 15%, #111 18%, #111 38%, #222 40%, #111 42%, #111 68%, #000 70%);
            border: 3px solid #222;
            display: flex;
            align-items: center;
            justify-content: center;
            box-shadow: 0 0 15px rgba(0, 0, 0, 0.8);
            animation: spin 4s linear infinite;
            animation-play-state: paused; /* เล่น animation เมื่อกดเล่นเพลง */
        }

        .vinyl.playing {
            animation-play-state: running;
        }

        .vinyl-center {
            width: 35px;
            height: 35px;
            background: #00f0ff;
            border-radius: 50%;
            border: 3px solid #fff;
            box-shadow: 0 0 10px rgba(0, 240, 255, 0.8);
        }

        @keyframes spin {
            100% { transform: rotate(360deg); }
        }

        /* ข้อมูลเพลงที่เล่นอยู่ */
        .song-info {
            text-align: center;
            width: 100%;
            margin-bottom: 12px;
        }

        .now-label {
            font-size: 10px;
            color: #00f0ff;
            font-weight: bold;
        }

        .song-title {
            font-size: 13px;
            font-weight: bold;
            margin-top: 3px;
            white-space: nowrap;
            overflow: hidden;
            text-overflow: ellipsis;
            color: #ffffff;
        }

        /* ปุ่มควบคุมเพลง */
        .controls {
            display: flex;
            align-items: center;
            justify-content: center;
            gap: 15px;
            margin-bottom: 12px;
        }

        .play-btn {
            width: 40px;
            height: 40px;
            border-radius: 50%;
            background: #00f0ff;
            border: none;
            color: #000;
            font-size: 16px;
            cursor: pointer;
            display: flex;
            align-items: center;
            justify-content: center;
            box-shadow: 0 0 12px rgba(0, 240, 255, 0.5);
            transition: all 0.2s;
        }

        .play-btn:hover {
            transform: scale(1.08);
            box-shadow: 0 0 18px rgba(0, 240, 255, 0.8);
        }

        .volume-box {
            width: 100%;
            display: flex;
            align-items: center;
            gap: 8px;
        }

        .volume-box input[type="range"] {
            width: 100%;
            accent-color: #00f0ff;
            cursor: pointer;
        }


        /* ===== ด้านขวา: เมนู Playlist ===== */
        .playlist-card {
            width: 260px;
            background: rgba(10, 16, 26, 0.85);
            backdrop-filter: blur(10px);
            border: 1px solid rgba(255, 255, 255, 0.1);
            border-radius: 12px;
            padding: 15px;
            box-shadow: 0 8px 32px rgba(0, 0, 0, 0.6);
            z-index: 10;
        }

        .playlist-header {
            font-size: 14px;
            font-weight: bold;
            color: #00f0ff;
            margin-bottom: 12px;
            border-bottom: 1px solid rgba(255, 255, 255, 0.1);
            padding-bottom: 8px;
        }

        .playlist-items {
            display: flex;
            flex-direction: column;
            gap: 8px;
        }

        .track-item {
            display: flex;
            align-items: center;
            gap: 10px;
            padding: 8px 10px;
            background: rgba(255, 255, 255, 0.03);
            border: 1px solid transparent;
            border-radius: 8px;
            cursor: pointer;
            transition: all 0.2s;
        }

        .track-item:hover, .track-item.active {
            background: rgba(0, 240, 255, 0.1);
            border-color: rgba(0, 240, 255, 0.4);
        }

        .track-num {
            font-size: 11px;
            color: #8b98a5;
            font-weight: bold;
        }

        .track-name {
            font-size: 12px;
            color: #e2e8f0;
            white-space: nowrap;
            overflow: hidden;
            text-overflow: ellipsis;
        }

        /* เว้นพื้นที่ตรงกลางสำหรับแชต xat */
        .chat-space {
            flex-grow: 1;
        }
    </style>
</head>
<body>

    <div class="xatframe-wrapper">
        
        <!-- ด้านซ้าย: เครื่องเล่นเพลง -->
        <div class="player-card">
            <div class="player-title">MYRADIOHIT PLAYER</div>
            
            <div class="turntable">
                <div class="vinyl" id="vinylRecord">
                    <div class="vinyl-center"></div>
                </div>
            </div>

            <div class="song-info">
                <div class="now-label">NOW PLAYING</div>
                <div class="song-title" id="songTitle">สตริงใหม่ล่าสุด</div>
            </div>

            <div class="controls">
                <button class="play-btn" id="playBtn" onclick="togglePlay()">▶</button>
            </div>

            <div class="volume-box">
                <span style="font-size: 12px;">🔈</span>
                <input type="range" id="volumeControl" min="0" max="1" step="0.01" value="0.8" oninput="changeVolume(this.value)">
            </div>

            <audio id="radioPlayer" preload="none">
                <source id="audioSource" src="https://ia600607.us.archive.org/28/items/sting-new/sting%20new.mp3" type="audio/mpeg">
            </audio>
        </div>

        <!-- เว้นช่องตรงกลางไว้สำหรับตัวแชต xat -->
        <div class="chat-space"></div>

        <!-- ด้านขวา: รายการ Playlist -->
        <div class="playlist-card">
            <div class="playlist-header">📋 Playlist</div>
            <div class="playlist-items">
                
                <div class="track-item active" onclick="playTrack('https://ia600607.us.archive.org/28/items/sting-new/sting%20new.mp3', 'สตริงใหม่ล่าสุด', this)">
                    <span class="track-num">#1</span>
                    <span class="track-name">สตริงใหม่ล่าสุด</span>
                </div>

                <div class="track-item" onclick="playTrack('https://ia902905.us.archive.org/5/items/thaistation/thaistation.mp3', 'เพลตริง (Thaistation)', this)">
                    <span class="track-num">#2</span>
                    <span class="track-name">เพลตริง (Thaistation)</span>
                </div>

                <div class="track-item" onclick="playTrack('https://ia601906.us.archive.org/3/items/radio_202607/1.radio.mp3', 'สตริง 1 Selection', this)">
                    <span class="track-num">#3</span>
                    <span class="track-name">สตริง 1 Selection</span>
                </div>

                <div class="track-item" onclick="playTrack('https://ia601501.us.archive.org/2/items/thaistation-v.-1/thaistation%20v.1.mp3', 'สตริง 2 ชม (Nonstop)', this)">
                    <span class="track-num">#4</span>
                    <span class="track-name">สตริง 2 ชม (Nonstop)</span>
                </div>

            </div>
        </div>

    </div>

    <script>
        const player = document.getElementById('radioPlayer');
        const playBtn = document.getElementById('playBtn');
        const vinyl = document.getElementById('vinylRecord');
        const songTitle = document.getElementById('songTitle');
        const audioSource = document.getElementById('audioSource');

        function togglePlay() {
            if (player.paused) {
                player.play();
                playBtn.innerText = '❚❚';
                vinyl.classList.add('playing');
            } else {
                player.pause();
                playBtn.innerText = '▶';
                vinyl.classList.remove('playing');
            }
        }

        function playTrack(url, title, element) {
            // เปลี่ยน Class แถบเลือก
            document.querySelectorAll('.track-item').forEach(item => item.classList.remove('active'));
            element.classList.add('active');

            // เปลี่ยนเพลง
            audioSource.src = url;
            songTitle.innerText = title;
            
            player.load();
            player.play();
            playBtn.innerText = '❚❚';
            vinyl.classList.add('playing');
        }

        function changeVolume(val) {
            player.volume = val;
        }
    </script>

</body>
</html>
