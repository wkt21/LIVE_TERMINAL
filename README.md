# Rotating Globe with Animated Webs — 8s GIF Generator

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
