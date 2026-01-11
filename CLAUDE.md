# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with this repository.

## Project Overview

This is a **Cyberpunk Gesture-Interactive Particle System** - a high-performance, single-file HTML application featuring 12,000 neon particles controlled by hand gestures. It runs entirely in the browser using webcam-based computer vision.

## Tech Stack

- **Three.js** (v0.160.0) - WebGL rendering engine for particle system
- **MediaPipe Hands** - Real-time hand tracking and gesture recognition
- **GLSL** - Custom vertex/fragment shaders for particle glow effects
- **Web Audio API** - Procedural audio generation (glass-like sounds)
- **CSS** - Cyberpunk visual effects (scanlines, vignette, grid overlay)

## Project Structure

```
gesture-interactive-particle-system/
├── cyberpink.html      # Single-file application (~920 lines)
├── README.md           # User documentation and controls
├── IMPLEMENTATION_PLAN.md  # Technical specification and architecture
└── LICENSE             # MIT License
```

The entire application is contained in `cyberpink.html` with embedded CSS (~150 lines), JavaScript (~700 lines), and HTML structure.

## Running the Application

```bash
# Option 1: Python server (recommended)
python -m http.server 8000
# Then open http://localhost:8000/cyberpink.html

# Option 2: Any local server (Live Server, etc.)
# Direct file:// opening may have webcam permission issues
```

**Requirements:** Modern browser (Chrome/Edge recommended), webcam access

## Architecture Overview

### Core Systems

1. **Particle System** (`geometry`, `material`, `particleSystem`)
   - 12,000 particles using `THREE.Points` with BufferGeometry
   - Custom shader material with additive blending
   - Positions, colors, sizes stored in Float32Array buffers

2. **Hand Tracking** (`onResults`, `countFingers`)
   - MediaPipe processes webcam at 640x480
   - Detects up to 2 hands with finger counting
   - Coordinates transformed to scene space: `(0.5 - landmark.x) * 400`

3. **State Machine** (`state` object)
   - Modes: `TEXT`, `NEBULA`, `CATCH`, `COMBO`, `EXPLOSION`
   - Tracks left/right hand positions, finger counts, current color/shape

4. **Animation Loop** (`animate()`)
   - Updates particle positions via lerping toward targets
   - Applies physics forces (repulsion, ripple effects)
   - Handles mode-specific behaviors (catch, explosion, etc.)

### Gesture Mapping

**Left Hand (Command Interface):**
- 1 finger → "Hello" (Cyan)
- 2 fingers → "Gemini3" (Yellow)
- 3 fingers → "Opus 4.5" (Pink)
- 4 fingers → "Goodbye" (Green)
- 5 fingers → Catch Mode

**Right Hand (Physics Interface):**
- Fist → Repulsion field (XY plane only)
- Open hand → Nebula mode with water ripple
- 1-3 fingers → Name text triggers

**Combo:** Both hands open creates bouncing sphere (football)

## Key Functions

| Function | Purpose |
|----------|---------|
| `generateTextCoords(text)` | Rasterizes text to particle coordinates via canvas |
| `updateTargetPositions(type)` | Sets particle target positions for current mode |
| `countFingers(landmarks)` | Counts extended fingers from MediaPipe landmarks |
| `updateLeftHandLogic(fingers)` | Maps left hand gestures to text/color states |
| `animate()` | Main render loop - updates positions, colors, forces |

## Important Parameters (CONF object)

```javascript
particleCount: 12000   // Total particles
particleSize: 2.4      // Base size
lerpSpeed: 0.16        // Movement interpolation speed
```

## Shader Details

- **Vertex Shader:** Size attenuation by camera distance (`200.0 / -mvPosition.z`)
- **Fragment Shader:** Circular particles with gaussian glow (`exp(-r2 * 4.0)`)
- **Blending:** Additive for neon glow effect

## Common Development Tasks

### Adding a New Text Gesture

1. Add color to `CONF.colors` object
2. Add case in `updateLeftHandLogic()` or `updateRightHandText()`
3. Set `state.targetShape` and `state.currentColor`

### Modifying Particle Physics

- Repulsion force: Line ~776-784 in `animate()` loop
- Water ripple: Line ~762-771 (sine wave formula)
- Lerp speed: `CONF.lerpSpeed` or per-mode in animate()

### Changing Visual Effects

- Particle glow: Fragment shader (`fragmentShader` variable)
- Scanlines: CSS `.scanlines` class
- HUD styling: CSS `.hud` classes

## Testing Notes

- Test in well-lit environment for hand detection
- Chrome/Edge provide best WebGL performance
- Click "INITIALIZE SYSTEM" prompt to enable audio
- FPS counter in top-left HUD shows performance
