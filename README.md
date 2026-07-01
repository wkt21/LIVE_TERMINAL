# Rotating Globe with Animated Webs — 8s GIF Generator

<!doctype html>
<html>
<head>
<meta charset="utf-8"/>
<title>Cyber Map → 8s GIF Export (embedded image)</title>
<style>
  body{margin:0;background:#000;color:#0ff;font-family:system-ui,Segoe UI,Roboto,Arial}
  #controls{position:fixed;left:12px;top:12px;z-index:10;background:rgba(0,0,0,0.6);padding:10px;border-radius:6px;border:1px solid rgba(0,255,200,0.08)}
  button{margin-right:8px;padding:8px 12px;background:#013;color:#0ff;border:1px solid #06f;border-radius:4px;cursor:pointer}
  #status{display:inline-block;margin-left:8px;color:#afa}
  canvas{display:block;margin:0 auto;box-shadow:0 0 30px rgba(0,255,200,0.03)}
</style>
</head>
<body>
  <div id="controls">
    <button id="startBtn">Start (auto-record 8s)</button>
    <button id="regenBtn">Regenerate webs</button>
    <span id="status">Ready</span>
  </div>

  <canvas id="c" width="1365" height="768"></canvas>

  <!-- gif.js from CDN -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/gif.js/0.2.0/gif.js"></script>

  <script>
  // REPLACE the value below with your image data URI (data:image/png;base64,....)
  const MAP_DATA_URI = "data:image/png;base64,PASTE_YOUR_BASE64_HERE";

  // Config
  const DURATION = 8.0;        // seconds
  const FPS = 24;              // frames per second
  const FRAMES = Math.round(DURATION * FPS);
  const WIDTH = 1365;
  const HEIGHT = 768;
  const NUM_WEBS = 12;         // number of arcs

  // Canvas & ctx
  const canvas = document.getElementById('c');
  const ctx = canvas.getContext('2d');
  canvas.width = WIDTH;
  canvas.height = HEIGHT;

  const statusEl = document.getElementById('status');
  const startBtn = document.getElementById('startBtn');
  const regenBtn = document.getElementById('regenBtn');

  // Load embedded map
  const mapImg = new Image();
  mapImg.src = MAP_DATA_URI;
  mapImg.onload = () => {
    drawFrame(0); // initial preview
    statusEl.textContent = 'Map loaded';
  };
  mapImg.onerror = () => statusEl.textContent = 'Failed to load embedded image — check MAP_DATA_URI';

  // Web data
  let webs = [];

  function rand(min, max){ return Math.random() * (max - min) + min; }
  function pickPointEdgeBias() { return { x: rand(80, WIDTH - 80), y: rand(60, HEIGHT - 60) }; }
  function makeWeb() {
    const a = pickPointEdgeBias();
    const b = pickPointEdgeBias();
    const hue = Math.floor(rand(160, 300)); // cyan->magenta palettes
    const color = `hsl(${hue} 100% 60%)`;
    return { from: a, to: b, color, width: rand(2, 5), phase: Math.random(), speed: rand(0.8, 1.6) };
  }

  function regenerate() {
    webs = [];
    for (let i=0;i<NUM_WEBS;i++) webs.push(makeWeb());
    statusEl.textContent = 'Webs regenerated';
    drawFrame(0);
  }
  regenBtn.onclick = () => regenerate();

  function cubicPoint(p0,p1,p2,p3,t){
    const u = 1 - t;
    const tt = t*t, uu = u*u;
    const uuu = uu * u, ttt = tt * t;
    const x = uuu*p0.x + 3*uu*t*p1.x + 3*u*tt*p2.x + ttt*p3.x;
    const y = uuu*p0.y + 3*uu*t*p1.y + 3*u*tt*p2.y + ttt*p3.y;
    return {x,y};
  }

  function drawGlowArc(pathPoints, lineWidth, color, alpha=1.0) {
    ctx.save();
    ctx.globalCompositeOperation = 'lighter';
    for (let i=3;i>=0;i--){
      ctx.beginPath();
      ctx.lineWidth = lineWidth * (1 + i*1.6);
      ctx.strokeStyle = color;
      ctx.globalAlpha = alpha * (0.12 + 0.22*(i/3));
      ctx.shadowColor = color;
      ctx.shadowBlur = 10 + i*10;
      ctx.moveTo(pathPoints[0].x, pathPoints[0].y);
      for (let k=1;k<pathPoints.length;k++) ctx.lineTo(pathPoints[k].x, pathPoints[k].y);
      ctx.stroke();
    }
    ctx.beginPath();
    ctx.lineWidth = lineWidth;
    ctx.strokeStyle = '#fff';
    ctx.globalAlpha = alpha*0.12;
    for (let k=0;k<pathPoints.length;k++){
      if (k===0) ctx.moveTo(pathPoints[k].x, pathPoints[k].y);
      else ctx.lineTo(pathPoints[k].x, pathPoints[k].y);
    }
    ctx.stroke();
    ctx.restore();
  }

  function drawFrame(progress) {
    ctx.clearRect(0,0,WIDTH,HEIGHT);
    ctx.drawImage(mapImg, 0, 0, WIDTH, HEIGHT);
    ctx.fillStyle = 'rgba(0,8,10,0.45)';
    ctx.fillRect(0,0,WIDTH,HEIGHT);

    webs.forEach((w, idx) => {
      const dx = (w.to.x - w.from.x);
      const dy = (w.to.y - w.from.y);
      const dist = Math.hypot(dx,dy) || 1;
      const cx = (w.from.x + w.to.x)/2 + (-dy/dist)*rand(-0.25, 0.25)*dist;
      const cy = (w.from.y + w.to.y)/2 + (dx/dist)*rand(-0.25, 0.25)*dist;
      const p0 = w.from;
      const p1 = { x: (w.from.x + cx)/2, y: (w.from.y + cy)/2 };
      const p2 = { x: (cx + w.to.x)/2, y: (cy + w.to.y)/2 };
      const p3 = w.to;

      const samples = 80;
      const pathPts = [];
      for(let s=0;s<=samples;s++){
        const t = s / samples;
        pathPts.push(cubicPoint(p0,p1,p2,p3,t));
      }

      drawGlowArc(pathPts, w.width, w.color, 0.9);

      const localPos = ((progress * w.speed) + w.phase) % 1.0;
      const particleCount = 2 + Math.floor(w.width/2);
      for (let p=0;p<particleCount;p++){
        const t = (localPos + (p/particleCount)*0.08) % 1;
        const pt = cubicPoint(p0,p1,p2,p3, t);
        ctx.save();
        ctx.globalCompositeOperation = 'lighter';
        ctx.fillStyle = w.color;
        ctx.beginPath();
        ctx.shadowBlur = 12;
        ctx.shadowColor = w.color;
        ctx.globalAlpha = 0.95;
        ctx.arc(pt.x, pt.y, 2.5 + Math.abs(Math.sin((progress + p*0.4)*Math.PI))*2.5, 0, Math.PI*2);
        ctx.fill();
        ctx.restore();
      }

      const pulse = 0.5 + 0.5 * Math.sin(2*Math.PI*(progress*2 + idx*0.17));
      ctx.save();
      ctx.globalCompositeOperation = 'lighter';
      ctx.strokeStyle = w.color;
      ctx.lineWidth = 2;
      ctx.globalAlpha = 0.9;
      ctx.beginPath();
      ctx.shadowBlur = 30;
      ctx.shadowColor = w.color;
      ctx.arc(w.to.x, w.to.y, 6 + pulse*10, 0, Math.PI*2);
      ctx.stroke();
      ctx.restore();

      ctx.save();
      ctx.globalCompositeOperation = 'lighter';
      ctx.fillStyle = '#fff';
      ctx.globalAlpha = 0.92;
      ctx.beginPath();
      ctx.arc(w.to.x, w.to.y, 2.4, 0, Math.PI*2);
      ctx.fill();
      ctx.restore();
    });

    ctx.save();
    ctx.globalAlpha = 0.08;
    ctx.fillStyle = '#0ff';
    ctx.fillRect(12, 12, WIDTH-24, 1.2);
    ctx.restore();
  }

  // Recording via gif.js
  function recordGIF() {
    startBtn.disabled = true;
    regenBtn.disabled = true;
    statusEl.textContent = 'Preparing GIF...';

    const gif = new GIF({
      workers: 2,
      workerScript: 'https://cdnjs.cloudflare.com/ajax/libs/gif.js/0.2.0/gif.worker.js',
      quality: 10,
      width: WIDTH,
      height: HEIGHT,
      transparent: null,
      repeat: 0,
      background: '#000'
    });

    let i = 0;
    function addNext() {
      if (i >= FRAMES) {
        statusEl.textContent = 'Rendering GIF (this may take a moment)...';
        gif.render();
        return;
      }
      const progress = i / (FRAMES - 1);
      drawFrame(progress);
      statusEl.textContent = `Recording frame ${i+1}/${FRAMES} (${Math.round(100*(i+1)/FRAMES)}%)`;
      gif.addFrame(canvas, {copy: true, delay: 1000 / FPS});
      i++;
      // yield occasionally to keep UI responsive
      if (i % 24 === 0) setTimeout(addNext, 6);
      else addNext();
    }

    gif.on('finished', function(blob) {
      const url = URL.createObjectURL(blob);
      const a = document.createElement('a');
      a.href = url;
      a.download = 'cyber-map-8s.gif';
      document.body.appendChild(a);
      a.click();
      a.remove();
      URL.revokeObjectURL(url);
      statusEl.textContent = 'Done — GIF downloaded';
      startBtn.disabled = false;
      regenBtn.disabled = false;
    });

    addNext();
  }

  startBtn.onclick = async () => {
    regenerate();
    await new Promise(r => setTimeout(r, 120));
    statusEl.textContent = 'Recording starting...';
    recordGIF();
  };
  regenBtn.onclick = regenerate;

  // initial
  regenerate();
  </script>
</body>
</html>


An interactive Three.js visualization that renders a rotating globe with animated connection arcs and exports the animation as an 8-second GIF.

## Features

- **3D Globe**: Textured Earth sphere with ambient and directional lighting
- **Animated Web Connections**: Random curved arcs connecting points across the globe with dashed line animation
- **Dynamic Colors**: Each arc gets a random cyan-to-green hue
- **Pulsating Opacity**: Lines fade in and out for visual interest
- **GIF Export**: Record and download the full animation as an 8-second GIF at 24 FPS (192 frames)
- **Web Regeneration**: Instantly create new random connection patterns

## How to Use

1. Open `index.html` in a modern web browser (Chrome, Firefox, Edge, Safari)
2. Click **"Start (auto-record 8s)"** to begin recording the animation
3. The status bar shows recording progress
4. Once complete, the GIF downloads automatically as `globe-webs-8s.gif`
5. Click **"Regenerate webs"** to create a new random web pattern without recording

## Technical Details

- **Three.js**: 3D rendering engine
- **gif.js**: Client-side GIF encoding (uses WebWorkers)
- **CatmullRomCurve3**: Smooth curved arcs between random lat/lon points
- **LineDashedMaterial**: Animated dash offset for flowing effect
- **WebGL**: Hardware-accelerated graphics

### Configuration (in `index.html`)

```javascript
const DURATION = 8.0;        // GIF duration in seconds
const FPS = 24;              // Frames per second
const NUM_WEBS = 22;         // Number of connection arcs
const WIDTH = 900;           // Canvas width
const HEIGHT = 600;          // Canvas height
const USE_EARTH_TEXTURE = false;  // Use custom Earth texture
```

## Dependencies

All libraries are loaded from CDN:
- Three.js (0.156.0)
- OrbitControls (from Three.js examples)
- gif.js (0.2.0)

## Browser Support

- Chrome/Chromium: ✓
- Firefox: ✓
- Safari: ✓
- Edge: ✓

Requires WebGL support and ES6 JavaScript.

## Files

- `index.html` - Complete interactive application (open directly in browser)
- `README.md` - This documentation

## License

MIT or use freely for personal/educational projects.
