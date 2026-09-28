<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>GitHub Media Player</title>
    <link href="https://fonts.googleapis.com/css2?family=Poppins:wght@300;400;600&display=swap" rel="stylesheet">
    <style>
        :root {
            --bg: #121212;
            --card-bg: #1e1e1e;
            --accent: #ff4d6d;
            --accent-hover: #ff758f;
            --text: #ffffff;
            --subtext: #b3b3b3;
        }

        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
            font-family: 'Poppins', sans-serif;
        }

        body {
            background-color: var(--bg);
            color: var(--text);
            min-height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
            padding: 20px;
        }

        .player-container {
            width: 100%;
            max-width: 800px;
            background: var(--card-bg);
            border-radius: 20px;
            padding: 30px;
            box-shadow: 0 15px 35px rgba(0,0,0,0.6);
            display: flex;
            flex-direction: column;
            gap: 20px;
        }

        /* Screen Area for Video/Audio Visualizer */
        .display-screen {
            width: 100%;
            height: 320px;
            background: #000;
            border-radius: 12px;
            overflow: hidden;
            display: flex;
            justify-content: center;
            align-items: center;
            position: relative;
        }

        video, audio {
            width: 100%;
            height: 100%;
            object-fit: contain;
        }

        .audio-placeholder {
            display: flex;
            flex-direction: column;
            align-items: center;
            gap: 15px;
            color: var(--accent);
        }

        .audio-placeholder .icon {
            font-size: 4rem;
            animation: pulse 2s infinite ease-in-out;
        }

        @keyframes pulse {
            0%, 100% { transform: scale(1); opacity: 0.8; }
            50% { transform: scale(1.1); opacity: 1; }
        }

        /* Track Info */
        .track-info {
            text-align: center;
        }

        .track-title {
            font-size: 1.3rem;
            font-weight: 600;
            margin-bottom: 4px;
            white-space: nowrap;
            overflow: hidden;
            text-overflow: ellipsis;
        }

        .track-artist {
            font-size: 0.9rem;
            color: var(--subtext);
        }

        /* Seek Bar & Timers */
        .timeline-container {
            display: flex;
            align-items: center;
            gap: 12px;
        }

        .time-text {
            font-size: 0.85rem;
            color: var(--subtext);
            width: 45px;
            text-align: center;
        }

        .seek-bar {
            flex-grow: 1;
            height: 6px;
            accent-color: var(--accent);
            cursor: pointer;
        }

        /* Controls */
        .controls {
            display: flex;
            justify-content: center;
            align-items: center;
            gap: 15px;
            flex-wrap: wrap;
        }

        .btn {
            background: rgba(255, 255, 255, 0.08);
            border: none;
            color: var(--text);
            padding: 12px 18px;
            border-radius: 50%;
            cursor: pointer;
            font-size: 1.1rem;
            transition: all 0.2s ease;
            display: flex;
            align-items: center;
            justify-content: center;
        }

        .btn:hover {
            background: rgba(255, 255, 255, 0.2);
            transform: scale(1.08);
        }

        .btn-primary {
            background: var(--accent);
            padding: 18px 22px;
            font-size: 1.4rem;
        }

        .btn-primary:hover {
            background: var(--accent-hover);
        }

        .btn.active {
            color: var(--accent);
            background: rgba(255, 77, 109, 0.2);
        }

        /* Extra Tools & Playlist */
        .bottom-section {
            display: flex;
            flex-direction: column;
            gap: 15px;
            border-top: 1px solid rgba(255,255,255,0.1);
            padding-top: 20px;
        }

        .action-bar {
            display: flex;
            justify-content: space-between;
            align-items: center;
            flex-wrap: wrap;
            gap: 10px;
        }

        .add-btn-label {
            background: var(--accent);
            color: white;
            padding: 10px 20px;
            border-radius: 20px;
            cursor: pointer;
            font-size: 0.9rem;
            font-weight: 600;
            display: inline-block;
            transition: background 0.2s;
        }

        .add-btn-label:hover {
            background: var(--accent-hover);
        }

        #file-input {
            display: none;
        }

        .playlist {
            max-height: 200px;
            overflow-y: auto;
            border-radius: 10px;
            background: rgba(0,0,0,0.3);
            border: 1px solid rgba(255,255,255,0.05);
        }

        .playlist-item {
            padding: 12px 16px;
            display: flex;
            justify-content: space-between;
            align-items: center;
            cursor: pointer;
            border-bottom: 1px solid rgba(255,255,255,0.05);
            font-size: 0.9rem;
            transition: background 0.2s;
        }

        .playlist-item:hover {
            background: rgba(255,255,255,0.05);
        }

        .playlist-item.active {
            background: rgba(255, 77, 109, 0.15);
            color: var(--accent);
            font-weight: 600;
        }

        .badge {
            font-size: 0.75rem;
            padding: 2px 8px;
            border-radius: 10px;
            background: rgba(255,255,255,0.1);
            color: var(--subtext);
        }

        .badge.user-added {
            background: rgba(77, 255, 136, 0.2);
            color: #4dff88;
        }
    </style>
</head>
<body>

    <div class="player-container">
        
        <!-- Media Display Area -->
        <div class="display-screen">
            <video id="video-element" style="display: none;"></video>
            <div id="audio-placeholder" class="audio-placeholder">
                <div class="icon">🎵</div>
                <p style="color: var(--subtext); font-size: 0.9rem;">Audio Playing</p>
            </div>
        </div>

        <!-- Track Information -->
        <div class="track-info">
            <div class="track-title" id="track-title">Select or Add a Song</div>
            <div class="track-artist" id="track-artist">Media Player</div>
        </div>

        <!-- Seek Timeline -->
        <div class="timeline-container">
            <span class="time-text" id="current-time">0:00</span>
            <input type="range" class="seek-bar" id="seek-bar" value="0" min="0" max="100">
            <span class="time-text" id="total-duration">0:00</span>
        </div>

        <!-- Playback Controls -->
        <div class="controls">
            <button class="btn" id="shuffle-btn" onclick="toggleShuffle()" title="Shuffle">🔀</button>
            <button class="btn" onclick="prevTrack()" title="Previous">⏮</button>
            <button class="btn" onclick="skipTime(-10)" title="Rewind 10s">⏪</button>
            <button class="btn btn-primary" id="play-btn" onclick="togglePlay()" title="Play/Pause">▶</button>
            <button class="btn" onclick="stopTrack()" title="Stop">⏹</button>
            <button class="btn" onclick="skipTime(10)" title="Forward 10s">⏩</button>
            <button class="btn" onclick="nextTrack()" title="Next">⏭</button>
            <button class="btn" id="loop-btn" onclick="toggleLoop()" title="Loop">🔁</button>
        </div>

        <!-- Bottom Action Bar & Playlist -->
        <div class="bottom-section">
            <div class="action-bar">
                <h3>Playlist</h3>
                <label class="add-btn-label">➕ Add Your Song
                    <input type="file" id="file-input" accept="audio/*,video/*" multiple onchange="handleUserFileUpload(event)">
                </label>
            </div>

            <div class="playlist" id="playlist-container">
                <!-- Playlist items injected here -->
            </div>
        </div>

    </div>

    <!-- Hidden Audio Player for mp3/wav handling -->
    <audio id="audio-element" style="display: none;"></audio>

    <script>
        // Default Repository Songs (Put your GitHub media file paths here)
        let playlist = [
            { title: "Sample Track 1", src: "song1.mp3", type: "audio", source: "repo" },
            { title: "Sample Video 1", src: "video1.mp4", type: "video", source: "repo" }
        ];

        let currentIndex = 0;
        let isPlaying = false;
        let isShuffle = false;
        let isLoop = false;

        const audioElement = document.getElementById('audio-element');
        const videoElement = document.getElementById('video-element');
        const playBtn = document.getElementById('play-btn');
        const seekBar = document.getElementById('seek-bar');
        const currentTimeEl = document.getElementById('current-time');
        const totalDurationEl = document.getElementById('total-duration');
        const trackTitleEl = document.getElementById('track-title');
        const trackArtistEl = document.getElementById('track-artist');
        const audioPlaceholder = document.getElementById('audio-placeholder');
        const playlistContainer = document.getElementById('playlist-container');

        let activeMedia = audioElement;

        function getMediaType(filename) {
            const ext = filename.split('.').pop().toLowerCase();
            return ['mp4', 'webm', 'ogg', 'mov'].includes(ext) ? 'video' : 'audio';
        }

        function loadTrack(index) {
            if (playlist.length === 0) return;
            currentIndex = index;
            const track = playlist[currentIndex];

            // Pause both
            audioElement.pause();
            videoElement.pause();

            trackTitleEl.innerText = track.title;
            trackArtistEl.innerText = track.source === 'user' ? 'Added by Visitor' : 'GitHub Library';

            if (track.type === 'video') {
                audioPlaceholder.style.display = 'none';
                videoElement.style.display = 'block';
                activeMedia = videoElement;
            } else {
                videoElement.style.display = 'none';
                audioPlaceholder.style.display = 'flex';
                activeMedia = audioElement;
            }

            activeMedia.src = track.src;
            activeMedia.load();
            renderPlaylist();

            if (isPlaying) {
                activeMedia.play();
            }
        }

        function togglePlay() {
            if (playlist.length === 0) return;
            if (isPlaying) {
                activeMedia.pause();
                playBtn.innerText = '▶';
            } else {
                activeMedia.play();
                playBtn.innerText = '⏸';
            }
            isPlaying = !isPlaying;
        }

        function stopTrack() {
            activeMedia.pause();
            activeMedia.currentTime = 0;
            isPlaying = false;
            playBtn.innerText = '▶';
        }

        function prevTrack() {
            currentIndex = (currentIndex - 1 + playlist.length) % playlist.length;
            loadTrack(currentIndex);
            if (isPlaying) activeMedia.play();
        }

        function nextTrack() {
            if (isShuffle) {
                currentIndex = Math.floor(Math.random() * playlist.length);
            } else {
                currentIndex = (currentIndex + 1) % playlist.length;
            }
            loadTrack(currentIndex);
            if (isPlaying) activeMedia.play();
        }

        function skipTime(seconds) {
            activeMedia.currentTime += seconds;
        }

        function toggleShuffle() {
            isShuffle = !isShuffle;
            document.getElementById('shuffle-btn').classList.toggle('active', isShuffle);
        }

        function toggleLoop() {
            isLoop = !isLoop;
            document.getElementById('loop-btn').classList.toggle('active', isLoop);
        }

        // Handle Seekbar & Timers
        function setupMediaListeners(media) {
            media.addEventListener('timeupdate', () => {
                if (media === activeMedia && media.duration) {
                    const progress = (media.currentTime / media.duration) * 100;
                    seekBar.value = progress;
                    currentTimeEl.innerText = formatTime(media.currentTime);
                    totalDurationEl.innerText = formatTime(media.duration);
                }
            });

            media.addEventListener('ended', () => {
                if (isLoop) {
                    activeMedia.currentTime = 0;
                    activeMedia.play();
                } else {
                    nextTrack();
                }
            });
        }

        setupMediaListeners(audioElement);
        setupMediaListeners(videoElement);

        seekBar.addEventListener('input', () => {
            if (activeMedia.duration) {
                activeMedia.currentTime = (seekBar.value / 100) * activeMedia.duration;
            }
        });

        function formatTime(seconds) {
            const min = Math.floor(seconds / 60);
            const sec = Math.floor(seconds % 60);
            return `${min}:${sec < 10 ? '0' : ''}${sec}`;
        }

        // Allow Users to Add Local Media
        function handleUserFileUpload(event) {
            const files = event.target.files;
            for (let file of files) {
                const objectURL = URL.createObjectURL(file);
                const type = getMediaType(file.name);
                playlist.push({
                    title: file.name,
                    src: objectURL,
                    type: type,
                    source: 'user'
                });
            }
            renderPlaylist();
            if (playlist.length === files.length) {
                loadTrack(0);
            }
        }

        function renderPlaylist() {
            playlistContainer.innerHTML = '';
            playlist.forEach((track, idx) => {
                const item = document.createElement('div');
                item.className = `playlist-item ${idx === currentIndex ? 'active' : ''}`;
                item.onclick = () => {
                    isPlaying = true;
                    playBtn.innerText = '⏸';
                    loadTrack(idx);
                };

                item.innerHTML = `
                    <span>${idx + 1}. ${track.title}</span>
                    <span class="badge ${track.source === 'user' ? 'user-added' : ''}">
                        ${track.type.toUpperCase()} • ${track.source === 'user' ? 'Visitor' : 'GitHub'}
                    </span>
                `;
                playlistContainer.appendChild(item);
            });
        }

        // Initial Load
        renderPlaylist();
        if (playlist.length > 0) {
            loadTrack(0);
        }
    </script>
</body>
</html>
