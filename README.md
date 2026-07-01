<!doctype html>
<html>
<head>
  <meta charset="utf-8" />
  <title>Rotating Globe with Webs → 8s GIF</title>
  <style>
    body { margin: 0; background: #000; color: #fff; font-family: sans-serif; display:flex; flex-direction:column; align-items:center; }
    #container { width: 900px; height: 600px; margin-top:10px; }
    #controls { margin: 8px 0; }
    button { margin-right: 8px; }
    #status { font-size: 13px; }
  </style>
</head>
<body>
  <h3>Rotating globe with animated webs — will record an 8s GIF</h3>
  <div id="controls">
    <button id="startBtn">Start (auto-record 8s)</button>
    <button id="regenerateBtn">Regenerate webs</button>
    <span id="status">Idle</span>
  </div>
  <div id="container"></div>

  <!-- Three.js -->
  <script src="https://unpkg.com/three@0.156.0/build/three.min.js"></script>
  <script src="https://unpkg.com/three@0.156.0/examples/js/controls/OrbitControls.js"></script>

  <!-- gif.js (and worker) from CDN -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/gif.js/0.2.0/gif.js"></script>

  <script>
  // CONFIG
  const DURATION = 8.0; // seconds of GIF
  const FPS = 24;       // frames per second
  const WIDTH = 900;
  const HEIGHT = 600;
  const NUM_WEBS = 22;  // number of connection arcs
  const USE_EARTH_TEXTURE = false; // set true if you have a public texture url to use
  const EARTH_TEXTURE_URL = 'https://threejs.org/examples/textures/earth_atmos_2048.jpg';
  // If you want to use the Map.md image you supplied, replace EARTH_TEXTURE_URL with its public URL
  // (note: GitHub user-attachments URLs can have CORS restrictions; best to host a public image)

  // Scene setup
  const container = document.getElementById('container');
  const renderer = new THREE.WebGLRenderer({preserveDrawingBuffer: true, antialias: true});
  renderer.setSize(WIDTH, HEIGHT);
  container.appendChild(renderer.domElement);

  const scene = new THREE.Scene();
  scene.background = new THREE.Color(0x000000);

  const camera = new THREE.PerspectiveCamera(45, WIDTH/HEIGHT, 0.1, 1000);
  camera.position.set(0, 0, 3.6);

  const controls = new THREE.OrbitControls(camera, renderer.domElement);
  controls.enablePan = false;
  controls.enableZoom = false;

  // Lights
  const amb = new THREE.AmbientLight(0xffffff, 0.6);
  scene.add(amb);
  const dir = new THREE.DirectionalLight(0xffffff, 0.7);
  dir.position.set(5,3,5);
  scene.add(dir);

  // Globe
  const sphereGeom = new THREE.SphereGeometry(1, 64, 64);
  const loader = new THREE.TextureLoader();
  const textureURL = USE_EARTH_TEXTURE ? EARTH_TEXTURE_URL : 'https://threejs.org/examples/textures/earth_atmos_2048.jpg';
  const sphereMat = new THREE.MeshPhongMaterial({
    map: loader.load(textureURL),
    shininess: 5
  });
  const globe = new THREE.Mesh(sphereGeom, sphereMat);
  scene.add(globe);

  // Web group
  const webGroup = new THREE.Group();
  scene.add(webGroup);

  // Utility: lat/lon -> vector3
  function latLonToVector3(lat, lon, radius=1.001) {
    const phi = (90 - lat) * Math.PI/180;
    const theta = (lon + 180) * Math.PI/180;
    const x = -radius * Math.sin(phi) * Math.cos(theta);
    const z = radius * Math.sin(phi) * Math.sin(theta);
    const y = radius * Math.cos(phi);
    return new THREE.Vector3(x,y,z);
  }

  // Create curved arc between two points on sphere
  function createArcMesh(lat1, lon1, lat2, lon2, options={}) {
    const start = latLonToVector3(lat1, lon1);
    const end = latLonToVector3(lat2, lon2);
    // Midpoint: lift outward to make arc visible
    const mid = new THREE.Vector3().addVectors(start, end).multiplyScalar(0.5);
    mid.normalize().multiplyScalar(1.35); // raise mid outwards
    // Curve
    const curve = new THREE.CatmullRomCurve3([start, mid, end]);
    const points = curve.getPoints(120);
    const geometry = new THREE.BufferGeometry().setFromPoints(points);
    const material = new THREE.LineDashedMaterial({
      color: options.color || 0x00ffea,
      dashSize: 0.02,
      gapSize: 0.04,
      linewidth: 1.5, // note: many browsers ignore linewidth
      transparent: true,
      opacity: 0.85
    });
    const line = new THREE.Line(geometry, material);
    line.computeLineDistances();
    // Attach animation properties
    line.userData = { dashSpeed: (Math.random()*0.02 + 0.01) * (Math.random() > 0.5 ? 1 : -1), phase: Math.random() };
    return line;
  }

  // Generate random webs across globe
  function generateWebs(n) {
    // clear existing
    while (webGroup.children.length) webGroup.remove(webGroup.children[0]);
    for (let i=0;i<n;i++){
      const lat1 = (Math.random()*180)-90;
      const lon1 = (Math.random()*360)-180;
      // second point near the other hemisphere sometimes
      const lat2 = (Math.random()*180)-90;
      const lon2 = (Math.random()*360)-180;
      const col = new THREE.Color().setHSL(0.53 + Math.random()*0.2, 0.9, 0.5);
      const line = createArcMesh(lat1, lon1, lat2, lon2, {color: col.getHex()});
      webGroup.add(line);
    }
  }

  generateWebs(NUM_WEBS);

  // Animation parameters
  let startTime = null;
  const fullRotateRadians = Math.PI * 2; // one full rotation
  const rotationPerSecond = fullRotateRadians / DURATION; // rotate so one full turn during GIF duration

  // Animation loop
  function animate(timestamp) {
    if (!startTime) startTime = timestamp;
    const t = (timestamp - startTime) / 1000.0; // seconds elapsed
    // rotate entire globe (and webs) together
    globe.rotation.y = rotationPerSecond * t;
    webGroup.rotation.y = rotationPerSecond * t;

    // animate dash offset for each line
    webGroup.children.forEach(line => {
      line.material.dashOffset += line.userData.dashSpeed;
      // optionally pulsate opacity
      const op = 0.6 + 0.4 * Math.abs(Math.sin( t*0.9 + line.userData.phase*6.28 ));
      line.material.opacity = op;
    });

    renderer.render(scene, camera);
  }

  // For continuous rendering (so capture draws each frame)
  let rafId;
  function renderLoop(ts) {
    animate(ts);
    rafId = requestAnimationFrame(renderLoop);
  }
  rafId = requestAnimationFrame(renderLoop);

  // GIF capture logic
  const startBtn = document.getElementById('startBtn');
  const regenerateBtn = document.getElementById('regenerateBtn');
  const status = document.getElementById('status');

  startBtn.addEventListener('click', startCapture);
  regenerateBtn.addEventListener('click', () => { generateWebs(NUM_WEBS); status.textContent = 'Regenerated webs'; });

  function startCapture() {
    // Prepare GIF
    const totalFrames = Math.round(DURATION * FPS);
    const frameDelay = Math.round(1000 / FPS); // ms
    status.textContent = `Recording ${DURATION}s → ${totalFrames} frames at ${FPS} FPS...`;

    // gif.js requires correct path to worker; use CDN path
    const gif = new GIF({
      workers: 2,
      quality: 10,
      width: WIDTH,
      height: HEIGHT,
      workerScript: 'https://cdnjs.cloudflare.com/ajax/libs/gif.js/0.2.0/gif.worker.js'
    });

    let frameCount = 0;
    let captureStart = performance.now();
    // Temporarily speed up rotation to ensure exactly one rotation in DURATION — reset startTime
    startTime = performance.now();

    function captureFrame() {
      // renderLoop already renders; we just grab the canvas contents
      gif.addFrame(renderer.domElement, {copy: true, delay: frameDelay});
      frameCount++;
      status.textContent = `Recording... ${Math.round((frameCount/totalFrames)*100)}%`;
      if (frameCount < totalFrames) {
        // schedule next frame at correct time
        setTimeout(captureFrame, frameDelay);
      } else {
        status.textContent = 'Finalizing GIF (this may take a few seconds)...';
        gif.on('finished', function(blob) {
          const url = URL.createObjectURL(blob);
          const a = document.createElement('a');
          a.href = url;
          a.download = 'globe-webs-8s.gif';
          a.click();
          status.textContent = 'Done: downloaded globe-webs-8s.gif';
        });
        gif.render();
      }
    }
    // Start capture loop slightly delayed to let any UI settle
    setTimeout(captureFrame, 120);
  }

  // Auto-start when loaded (optional): comment out if you prefer manual start
  window.addEventListener('load', () => {
    // do not auto-start; wait for button — but you can uncomment to auto-start
    // startBtn.click();
  });
  </script>
</body>
</html>
