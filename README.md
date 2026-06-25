<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>MC GPS Camera Pro</title>
    <style>
        :root {
            --bg-color: #000;
            --stamp-bg: rgba(30, 30, 30, 0.85);
            --text-main: #ffffff;
            --accent: #ffb300; /* Screenshot wali Yellow line */
        }

        body, html {
            margin: 0; padding: 0; width: 100%; height: 100%;
            background-color: var(--bg-color); font-family: 'Segoe UI', Roboto, Helvetica, Arial, sans-serif;
            overflow: hidden; color: var(--text-main);
        }

        #camera-container { position: relative; width: 100%; height: 100%; }
        #camera-view { width: 100%; height: 100%; object-fit: cover; }

        /* Live Preview Overlay */
        #live-overlay {
            position: absolute; bottom: 120px; left: 10px; right: 10px;
            pointer-events: none; z-index: 5;
        }

        /* Top Tab - Logo Hatane ke baad sirf Text */
        .stamp-tab {
            background-color: var(--stamp-bg);
            display: inline-flex; align-items: center;
            padding: 6px 12px; border-radius: 8px 8px 0 0;
            font-size: 0.8rem; font-weight: 700; margin-left: auto;
            position: absolute; right: 0; top: -28px;
            letter-spacing: 0.5px;
        }

        /* Main Stamp Box */
        .stamp-box {
            background-color: var(--stamp-bg);
            border-radius: 8px 0 8px 8px; padding: 15px;
            box-shadow: 0 4px 10px rgba(0,0,0,0.5);
        }

        .stamp-top-row { display: flex; align-items: center; margin-bottom: 8px; }
        #live-time { font-size: 2.2rem; font-weight: bold; letter-spacing: 1px; }
        
        .vertical-divider {
            width: 3px; height: 40px; background-color: var(--accent);
            margin: 0 15px; border-radius: 2px;
        }

        .date-col { display: flex; flex-direction: column; }
        #live-date { font-size: 1.1rem; font-weight: bold; }
        #live-day { font-size: 1.1rem; font-weight: bold; }

        #live-short-address { font-size: 0.95rem; font-weight: bold; margin-bottom: 3px; }
        #live-long-address { font-size: 0.75rem; line-height: 1.3; margin-bottom: 3px; }
        #live-latlon { font-size: 0.75rem; font-weight: bold; }

        /* Controls */
        #controls {
            position: absolute; bottom: 0; width: 100%; height: 100px;
            background-color: rgba(15, 30, 60, 0.92); display: flex; justify-content: center; 
            align-items: center; z-index: 10;
        }

        #capture-btn {
            width: 70px; height: 70px; background-color: white;
            border: 5px solid rgba(255,255,255,0.3); border-radius: 50%;
            cursor: pointer; outline: none; transition: transform 0.1s;
        }
        #capture-btn:active { transform: scale(0.9); }

        #flip-btn {
            background: rgba(255,255,255,0.15); border: none; border-radius: 50%;
            position: absolute; right: 30px; width: 45px; height: 45px;
            display: flex; justify-content: center; align-items: center; cursor: pointer;
        }
        #flip-btn svg { fill: white; width: 24px; height: 24px; }

        #gallery-btn {
            position: absolute; left: 30px; width: 45px; height: 45px;
            border-radius: 5px; border: 2px solid white; background-color: #222;
            background-size: cover; background-position: center; cursor: pointer;
        }

        /* Gallery Screen */
        #gallery-screen, #photo-viewer {
            position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            background-color: #000; z-index: 100; display: none; flex-direction: column;
        }
        .ui-header { height: 60px; background: #111; display: flex; align-items: center; padding: 0 20px; font-size: 1.1rem; }
        .back-icon { margin-right: 20px; cursor: pointer; font-size: 1.6rem; }
        .gallery-grid { flex: 1; display: grid; grid-template-columns: repeat(3, 1fr); gap: 2px; overflow-y: auto; padding: 2px; }
        .grid-item { aspect-ratio: 1; background-size: cover; background-position: center; cursor: pointer;}
        #viewer-img { flex: 1; object-fit: contain; width: 100%; }
        .viewer-actions { height: 80px; background: #111; display: flex; justify-content: space-around; align-items: center; }
        .btn-action { background: none; border: none; color: white; display: flex; flex-direction: column; align-items: center; font-size: 0.8rem; cursor: pointer; }
        .btn-action svg { width: 24px; height: 24px; fill: white; margin-bottom: 5px; }

        #canvas { display: none; }
    </style>
</head>
<body>

    <div id="camera-container">
        <video id="camera-view" autoplay playsinline muted></video>
        
        <div id="live-overlay">
            <div class="stamp-tab">MC GPS Camera</div>
            <div class="stamp-box">
                <div class="stamp-top-row">
                    <div id="live-time">00:00 PM</div>
                    <div class="vertical-divider"></div>
                    <div class="date-col">
                        <div id="live-date">-- ----- ----</div>
                        <div id="live-day">-------</div>
                    </div>
                </div>
                <div id="live-short-address">Fetching Location...</div>
                <div id="live-long-address">Acquiring detailed GPS address.</div>
                <div id="live-latlon">Lat --.------° Long --.------°</div>
            </div>
        </div>
    </div>

    <div id="controls">
        <div id="gallery-btn" onclick="openGallery()"></div>
        <button id="capture-btn" onclick="capturePhoto()"></button>
        <button id="flip-btn" onclick="flipCamera()">
            <svg viewBox="0 0 24 24"><path d="M19.95 11c-.47-4.11-3.66-7.3-7.77-7.77V1a1 1 0 0 0-1.74-.7l-3.32 3.32a1 1 0 0 0 0 1.41l3.32 3.32A1 1 0 0 0 12.23 7.6V5.05c2.72.41 4.9 2.59 5.31 5.31H16a1 1 0 0 0-1 1 1 1 0 0 0 1 1h5a1 1 0 0 0 1-1v-5a1 1 0 0 0-1-1 1 1 0 0 0-1 1Zm-5.71 8.35c-2.72-.41-4.9-2.59-5.31-5.31H10a1 1 0 0 0 1-1 1 1 0 0 0-1-1H5a1 1 0 0 0-1 1v5a1 1 0 0 0 1 1 1 1 0 0 0 1-1v-2.55c.47 4.11 3.66 7.3 7.77 7.77V23a1 1 0 0 0 1.74.7l3.32-3.32a1 1 0 0 0 0-1.41l-3.32-3.32a1 1 0 0 0-1.04-.26 1 1 0 0 0-.7.96Z"/></svg>
        </button>
    </div>

    <div id="gallery-screen">
        <div class="ui-header"><span class="back-icon" onclick="closeGallery()">←</span> Gallery</div>
        <div class="gallery-grid" id="grid"></div>
    </div>

    <div id="photo-viewer">
        <div class="ui-header"><span class="back-icon" onclick="closeViewer()">←</span> View</div>
        <img id="viewer-img">
        <div class="viewer-actions">
            <button class="btn-action" onclick="sharePhoto()">
                <svg viewBox="0 0 24 24"><path d="M18 16.08c-.76 0-1.44.3-1.96.77L8.91 12.7c.05-.23.09-.46.09-.7s-.04-.47-.09-.7l7.05-4.11c.54.5 1.25.81 2.04.81 1.66 0 3-1.34 3-3s-1.34-3-3-3-3 1.34-3 3c0 .24.04.47.09.7L8.04 9.81C7.5 9.31 6.79 9 6 9c-1.66 0-3 1.34-3 3s1.34 3 3 3c.79 0 1.5-.31 2.04-.81l7.12 4.16c-.05.21-.08.43-.08.65 0 1.61 1.31 2.92 2.92 2.92s2.92-1.31 2.92-2.92c0-1.61-1.31-2.92-2.92-2.92z"/></svg>
                Share
            </button>
            <button class="btn-action" style="color: #ff5252;" onclick="deletePhoto()">
                <svg style="fill: #ff5252;" viewBox="0 0 24 24"><path d="M6 19c0 1.1.9 2 2 2h8c1.1 0 2-.9 2-2V7H6v12zM19 4h-3.5l-1-1h-5l-1 1H5v2h14V4z"/></svg>
                Delete
            </button>
        </div>
    </div>

    <canvas id="canvas"></canvas>

    <script>
        const video = document.getElementById('camera-view');
        const canvas = document.getElementById('canvas');
        const ctx = canvas.getContext('2d');
        const galleryBtn = document.getElementById('gallery-btn');
        const liveTime = document.getElementById('live-time'), liveDate = document.getElementById('live-date'), liveDay = document.getElementById('live-day');
        const liveShortAddress = document.getElementById('live-short-address'), liveLongAddress = document.getElementById('live-long-address'), liveLatLon = document.getElementById('live-latlon');

        let currentLat = "0.000000", currentLon = "0.000000", shortAddText = "City, State, भारत 🇮🇳", longAddText = "Fetching...", facingMode = 'environment', stream = null;
        let photos = JSON.parse(localStorage.getItem('mc_final_gallery_v3')) || [], viewIdx = -1;

        const hindiDateFmt = new Intl.DateTimeFormat('hi-IN-u-nu-latn', { day: 'numeric', month: 'long', year: 'numeric' });
        const hindiDayFmt = new Intl.DateTimeFormat('hi-IN', { weekday: 'long' });

        async function initCamera() {
            if (stream) stream.getTracks().forEach(t => t.stop());
            try {
                stream = await navigator.mediaDevices.getUserMedia({ video: { facingMode: facingMode, width: { ideal: 1920 }, height: { ideal: 1080 } }, audio: false });
                video.srcObject = stream;
            } catch (e) { alert("Camera Permission Error."); }
        }

        function flipCamera() { facingMode = facingMode === 'environment' ? 'user' : 'environment'; initCamera(); }

        function updateClock() {
            const now = new Date();
            liveTime.innerText = now.toLocaleTimeString('en-US', { hour: '2-digit', minute: '2-digit' });
            liveDate.innerText = hindiDateFmt.format(now);
            liveDay.innerText = hindiDayFmt.format(now);
        }
        setInterval(updateClock, 1000); updateClock();

        async function fetchAddress(lat, lon) {
            try {
                const res = await fetch(`https://nominatim.openstreetmap.org/reverse?format=json&lat=${lat}&lon=${lon}&zoom=18&addressdetails=1&accept-language=en`);
                const data = await res.json();
                if(data && data.address) {
                    shortAddText = `${data.address.city || data.address.town || ""}, ${data.address.state || ""}, भारत 🇮🇳`.replace(/^, /, '');
                    liveShortAddress.innerText = shortAddText;
                    longAddText = `${data.display_name}, भारत`;
                    liveLongAddress.innerText = longAddText;
                }
            } catch (e) { console.log("GPS fetch error."); }
        }

        function startGPS() {
            if ("geolocation" in navigator) {
                navigator.geolocation.watchPosition(pos => {
                    currentLat = pos.coords.latitude.toFixed(6); currentLon = pos.coords.longitude.toFixed(6);
                    liveLatLon.innerText = `Lat ${currentLat}° Long ${currentLon}°`;
                    if(longAddText.includes("Fetching")) fetchAddress(currentLat, currentLon);
                }, null, { enableHighAccuracy: true });
            }
        }

        function capturePhoto() {
            canvas.width = video.videoWidth; canvas.height = video.videoHeight;
            if(facingMode === 'user') { ctx.translate(canvas.width, 0); ctx.scale(-1, 1); }
            ctx.drawImage(video, 0, 0, canvas.width, canvas.height);
            if(facingMode === 'user') ctx.setTransform(1, 0, 0, 1, 0, 0);

            const w = canvas.width, h = canvas.height;
            const padX = w * 0.03, padY = h * 0.04, boxW = w - (padX * 2), boxH = h * 0.22, boxY = h - boxH - padY;

            // 1. Top Small Tab (Logo Hataya gaya)
            const tabW = w * 0.28, tabH = h * 0.04, tabX = w - padX - tabW, tabY = boxY - tabH;
            ctx.fillStyle = "rgba(30, 30, 30, 0.85)";
            if(ctx.roundRect) { ctx.beginPath(); ctx.roundRect(tabX, tabY, tabW, tabH, [10, 10, 0, 0]); ctx.fill(); }
            else ctx.fillRect(tabX, tabY, tabW, tabH);
            
            ctx.fillStyle = "#ffffff"; ctx.textAlign = "center"; ctx.textBaseline = "middle";
            ctx.font = `bold ${h * 0.021}px sans-serif`;
            ctx.fillText("MC GPS Camera", tabX + (tabW/2), tabY + (tabH/2));

            // 2. Main Box
            ctx.fillStyle = "rgba(30, 30, 30, 0.85)";
            if(ctx.roundRect) { ctx.beginPath(); ctx.roundRect(padX, boxY, boxW, boxH, [10, 0, 10, 10]); ctx.fill(); }
            else ctx.fillRect(padX, boxY, boxW, boxH);

            const iX = padX + (w * 0.02); let iY = boxY + (h * 0.04);

            // Time & Date Block
            ctx.textAlign = "left"; ctx.fillStyle = "#ffffff"; ctx.font = `bold ${h * 0.055}px sans-serif`;
            const tS = liveTime.innerText; ctx.fillText(tS, iX, iY);
            const tW = ctx.measureText(tS).width;
            ctx.fillStyle = "#ffb300"; ctx.fillRect(iX + tW + (w * 0.02), iY - (h*0.025), w * 0.003, h * 0.05);
            ctx.fillStyle = "#ffffff"; ctx.font = `bold ${h * 0.028}px sans-serif`;
            ctx.fillText(liveDate.innerText, iX + tW + (w * 0.045), iY - (h*0.012));
            ctx.fillText(liveDay.innerText, iX + tW + (w * 0.045), iY + (h*0.018));

            // Address Lines
            iY += h * 0.06; ctx.font = `bold ${h * 0.025}px sans-serif`; ctx.fillText(shortAddText, iX, iY);
            iY += h * 0.035; ctx.font = `normal ${h * 0.02}px sans-serif`;
            if(longAddText.length > 80) { ctx.fillText(longAddText.substring(0, 80) + "-", iX, iY); iY += h * 0.025; ctx.fillText(longAddText.substring(80), iX, iY); }
            else ctx.fillText(longAddText, iX, iY);
            iY += h * 0.035; ctx.font = `bold ${h * 0.018}px sans-serif`;
            ctx.fillText(`Lat ${currentLat}° Long ${currentLon}°`, iX, iY);

            const img = canvas.toDataURL('image/jpeg', 0.9);
            photos.unshift(img); if(photos.length > 15) photos.pop();
            localStorage.setItem('mc_final_gallery_v3', JSON.stringify(photos));
            updateGalleryThumb();
            const link = document.createElement('a'); link.download = `MCGPS_${Date.now()}.jpg`; link.href = img; link.click();
        }

        function updateGalleryThumb() { if(photos.length > 0) galleryBtn.style.backgroundImage = `url(${photos[0]})`; }
        function openGallery() {
            const grid = document.getElementById('grid'); grid.innerHTML = '';
            photos.forEach((p, i) => { const d = document.createElement('div'); d.className = 'grid-item'; d.style.backgroundImage = `url(${p})`; d.onclick = () => openViewer(i); grid.appendChild(d); });
            document.getElementById('gallery-screen').style.display = 'flex';
        }
        function closeGallery() { document.getElementById('gallery-screen').style.display = 'none'; }
        function openViewer(i) { viewIdx = i; document.getElementById('viewer-img').src = photos[i]; document.getElementById('photo-viewer').style.display = 'flex'; }
        function closeViewer() { document.getElementById('photo-viewer').style.display = 'none'; }
        function deletePhoto() { if(confirm("Delete photo?")) { photos.splice(viewIdx, 1); localStorage.setItem('mc_final_gallery_v3', JSON.stringify(photos)); updateGalleryThumb(); closeViewer(); openGallery(); } }
        async function sharePhoto() { try { const r = await fetch(photos[viewIdx]); const b = await r.blob(); const f = new File([b], 'MCGPS.jpg', { type: 'image/jpeg' }); if (navigator.canShare && navigator.canShare({ files: [f] })) await navigator.share({ files: [f] }); } catch (e) {} }

        updateGalleryThumb(); initCamera(); startGPS();
    </script>
</body>
</html>
