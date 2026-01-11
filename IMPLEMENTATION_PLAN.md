# Enhanced Cyberpunk MediaPipe Particle System - Implementation Plan

## Enhanced Replication Prompt

Act as an expert **frontend** creative developer specializing in **Three.js**, **WebGL**, and **Computer Vision (MediaPipe Hands)**. Your task is to create a high-performance, single-file HTML application featuring a cyberpunk interactive particle system with advanced visual effects and optimized performance.

---

## Visual Style

### Theme: Hardcore Cyberpunk
- **Background:** Deep black (0x000000) with:
  - Animated moving grid (50px cells, subtle cyan tint)
  - Horizontal scanlines (4px spacing, slow vertical scroll animation)
  - Radial vignette effect (40% transparent center, 80% dark edges)
  - Optional CRT screen curvature effect

### HUD System
- **Font:** Orbitron (Google Fonts) or monospace fallback
- **Color:** Neon Cyan (#00FFFF) with text-shadow glow (10px, 20px, 30px spread)
- **Layout:**
  - Top-left: System Status, Particle Count (12,000 UNITS), FPS, Sector
  - Top-right: Detection Mode
  - Bottom-left: Left Hand Command Display
  - Bottom-right: Right Hand Physics Mode
- **Effects:** Glitch animation on state changes (0.3s transform jitter)

### Particle Material
- **Blending:** `THREE.AdditiveBlending` for neon glow
- **Depth Write:** Disabled for proper transparency
- **Custom Shader:**
  - Vertex: Size attenuation by distance (300.0 / -mvPosition.z)
  - Fragment: Circular shape with soft edge falloff (smoothstep + pow 1.5)

---

## Core Parameters

| Parameter | Value | Description |
|-----------|-------|-------------|
| Particle Count | 12,000 | Total particles in system |
| Particle Size | 2.4 | Base size (scaled by device pixel ratio) |
| Lerp Factor | 0.16 | Position interpolation speed (fast return) |
| Repulsion Strength | 0.8 | Force magnitude for scatter effect |
| Repulsion Radius | 0.15 | Interaction distance threshold |
| Nebula Spread | 2.5 | 3D distribution range for nebula mode |
| Basketball Radius | 0.4 | Size of basketball formation |
| Bounce Frequency | 8 | Hz for bouncing trajectory |
| Bounce Amplitude | 0.02 | Y-axis displacement for bounce |

---

## Color Palette

| State | Hex Code | RGB | Description |
|-------|----------|-----|-------------|
| Hello | 0x00FFFF | (0, 255, 255) | Neon Blue/Cyan |
| Gemini 3 | 0xFFFF00 | (255, 255, 0) | Neon Yellow |
| Opus 4.5 | 0xFF00FF | (255, 0, 255) | Neon Pink/Magenta |
| Goodbye | 0x00FF88 | (0, 255, 136) | Neon Green |
| Basketball | 0xFF8800 | (255, 136, 0) | Orange |
| Default | 0x00FFFF | (0, 255, 255) | Cyan |

---

## Interaction Logic (Detailed Requirements)

### 1. Left Hand (Command Controller)

Detect the number of extended fingers to switch the target shape and color of the particles:

| Fingers | Text | Color | HUD Display |
|---------|------|-------|-------------|
| 1 | "Hello" | Neon Blue (0x00FFFF) | CMD: HELLO |
| 2 | "Gemini 3" | Neon Yellow (0xFFFF00) | CMD: GEMINI_3 |
| 3 | "Opus 4.5" | Neon Pink (0xFF00FF) | CMD: OPUS_4.5 |
| 4 | "Goodbye" | Neon Green (0x00FF88) | CMD: GOODBYE |
| 5 (Open Palm) | N/A | N/A | MODE: CATCH |

**Implementation Details:**
- Use canvas pixel scanning to generate text coordinates (75px bold font)
- Pre-generate coordinates for all text strings on initialization
- Apply slight randomness (0.02 units) to particle positions for natural look
- Smooth color transitions using RGB lerping

### 2. Right Hand (Physics Interactor)

**Tracking Point:** Always track the **Index Finger Tip** (MediaPipe Landmark 8)

#### State A: Pointing/Fist (Default - fingers < 5)
- **Interaction:** Strong Scatter effect on touch
- **Physics:**
  - Calculate distance from finger tip to each particle
  - Apply repulsion force: `force = (1 - dist/radius) * strength`
  - **CRITICAL:** Pure planar repulsion (XY axis ONLY)
  - **DO NOT** apply any Z-axis bulge or distortion
- **Velocity Damping:** 0.92 per frame

#### State B: Open Hand (5 Fingers)
- **Trigger:** Nebula Mode - particles scatter to fill entire 3D screen space
- **Distribution:** Random positions within ±2.5 units (X), ±1.875 units (Y), ±1.25 units (Z)
- **Water Ripple Effect:**
  - Trigger: Hand movement through particles
  - Formula: `displacement = amplitude * sin(dist * frequency - time * speed)`
  - Parameters: frequency=15, amplitude=0.1, speed=5, decay=0.3
  - Apply ripple to Z-axis position only
  - Fade with exponential distance decay

### 3. Dual Hand Combo (Ultimate Effect)

**Condition:** Right Hand Open (Nebula active) **AND** Left Hand Open (Catch active) simultaneously

**Effect Logic:**

1. **Attraction Phase:**
   - All particles instantly attracted to left hand palm center
   - Palm center = average of landmarks 0, 5, 17

2. **Basketball Shape:**
   - Use **Fibonacci Sphere Distribution** (golden angle = PI * (3 - sqrt(5)))
   - Formula for each particle i:
     ```
     y = 1 - (i / (count-1)) * 2
     radiusAtY = sqrt(1 - y*y)
     theta = goldenAngle * i
     x = cos(theta) * radiusAtY
     z = sin(theta) * radiusAtY
     ```
   - Scale to basketball radius (0.4 units)

3. **Basketball Texture:**
   - Color: Orange (0xFF8800)
   - Black lines for texture:
     - Horizontal equator: |y| < 0.05
     - Vertical meridians: |sin(angle * 2)| < 0.1
     - Curved seams: |y - sin(angle * 2) * 0.5| < 0.08

4. **Motion Trajectory:**
   - **Bouncing/Jumping trajectory** using high-frequency Y-axis sine wave
   - Formula: `y_offset = amplitude * |sin(time * frequency + i * 0.01)|`
   - Creates effect of thousands of energetic bouncing beads

5. **Rotation:**
   - Continuous Y-axis rotation at 0.5 radians/second

---

## Technical Implementation

### Three.js Setup
```javascript
// Renderer
renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));

// Camera
camera = new THREE.PerspectiveCamera(75, aspect, 0.1, 1000);
camera.position.z = 2;
```

### Particle System
```javascript
// BufferGeometry attributes
geometry.setAttribute('position', new THREE.BufferAttribute(positions, 3));
geometry.setAttribute('color', new THREE.BufferAttribute(colors, 3));

// Shader Material
material = new THREE.ShaderMaterial({
    transparent: true,
    blending: THREE.AdditiveBlending,
    depthWrite: false
});
```

### MediaPipe Configuration
```javascript
hands.setOptions({
    maxNumHands: 2,
    modelComplexity: 1,
    minDetectionConfidence: 0.7,
    minTrackingConfidence: 0.5
});
```

### Finger Detection Algorithm
```javascript
// Thumb: compare x-position of tip (4) vs IP joint (3)
thumbExtended = |landmarks[4].x - landmarks[3].x| > 0.04

// Other fingers: compare y-position of tip vs PIP joint
fingerExtended = landmarks[tip].y < landmarks[pip].y - 0.02
```

---

## Performance Optimizations

1. **Rendering:**
   - Limit device pixel ratio to 2
   - Use typed arrays (Float32Array) for all particle data
   - Only update `needsUpdate` when necessary

2. **MediaPipe:**
   - Process at camera frame rate (not render frame rate)
   - Use modelComplexity: 1 (balanced)

3. **Memory:**
   - Pre-allocate all arrays at initialization
   - Reuse objects in animation loop

4. **Coordinate Conversion:**
   - Flip X for mirror view: `(1 - landmark.x) * 4 - 2`
   - Scale Y: `(0.5 - landmark.y) * 3`
   - Scale Z: `-landmark.z * 2`

---

## File Structure

Single `index.html` file containing:
- HTML structure (~50 lines)
- CSS styling (~200 lines)
- JavaScript application (~600 lines)
- **Total:** ~850 lines (optimized)

---

## Test Plan

### Test Cases

| ID | Test | Steps | Expected Result | Status |
|----|------|-------|-----------------|--------|
| TC01 | 1-finger gesture | Show 1 finger on left hand | "Hello" in Neon Blue | [ ] |
| TC02 | 2-finger gesture | Show 2 fingers on left hand | "Gemini 3" in Neon Yellow | [ ] |
| TC03 | 3-finger gesture | Show 3 fingers on left hand | "Opus 4.5" in Neon Pink | [ ] |
| TC04 | 4-finger gesture | Show 4 fingers on left hand | "Goodbye" in Neon Green | [ ] |
| TC05 | Left open palm | Open left hand fully | "Catch Mode" activates | [ ] |
| TC06 | Right pointer scatter | Point at particles | XY scatter (no Z bulge) | [ ] |
| TC07 | Right open nebula | Open right palm | 3D particle scatter | [ ] |
| TC08 | Water ripple | Move open right hand | Visible sine wave | [ ] |
| TC09 | Basketball combo | Both hands open | Orange basketball forms | [ ] |
| TC10 | Basketball bounce | Observe basketball | Bouncing trajectory | [ ] |
| TC11 | Basketball texture | Inspect basketball | Black lines visible | [ ] |
| TC12 | FPS stability | Run for 5 minutes | Consistent 60fps | [ ] |
| TC13 | Hand transitions | Rapidly change gestures | Smooth transitions | [ ] |
| TC14 | No hands | Remove hands from view | Return to default state | [ ] |

### Browser Compatibility
- [ ] Chrome (recommended)
- [ ] Firefox
- [ ] Edge
- [ ] Safari (WebGL support varies)

---

## Verification Checklist

### Functionality
- [ ] All 5 left-hand gestures correctly detected
- [ ] Right hand scatter is XY-only (no Z-axis bulge)
- [ ] Nebula mode fills 3D space evenly
- [ ] Water ripple visible during hand movement
- [ ] Basketball forms when both hands open
- [ ] Basketball has black line texture pattern
- [ ] Bouncing trajectory animation visible
- [ ] Smooth transitions between all states

### Visual Quality
- [ ] Colors match specification exactly
- [ ] Neon glow effect visible on particles
- [ ] Scanlines visible and animated
- [ ] Vignette darkening at edges
- [ ] Grid background subtly visible
- [ ] HUD readable and styled correctly
- [ ] Glitch effect on state changes

### Performance
- [ ] 60fps maintained throughout
- [ ] No memory leaks after 10 minutes
- [ ] MediaPipe detection < 33ms/frame
- [ ] Smooth particle animations

---

## Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                         index.html                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────────┐                      ┌──────────────┐         │
│  │   Webcam     │─────────────────────>│  MediaPipe   │         │
│  │   Video      │                      │   Hands      │         │
│  └──────────────┘                      └──────┬───────┘         │
│         │                                     │                  │
│         │ (background)              ┌─────────┴─────────┐       │
│         ▼                           │                   │       │
│  ┌──────────────┐           ┌───────▼──────┐   ┌───────▼──────┐│
│  │   Video      │           │  Left Hand   │   │  Right Hand  ││
│  │   Display    │           │  Processor   │   │  Processor   ││
│  │  (opacity)   │           └───────┬──────┘   └───────┬──────┘│
│  └──────────────┘                   │                   │       │
│                                     │                   │       │
│  ┌──────────────────────────────────┴───────────────────┤       │
│  │                  State Machine                        │       │
│  │  ┌────────┐ ┌────────┐ ┌────────┐ ┌────────────────┐ │       │
│  │  │ Text   │ │ Nebula │ │ Catch  │ │   Basketball   │ │       │
│  │  │ Mode   │ │ Mode   │ │ Mode   │ │   Combo Mode   │ │       │
│  │  └────┬───┘ └────┬───┘ └────┬───┘ └───────┬────────┘ │       │
│  └───────┼──────────┼──────────┼─────────────┼──────────┘       │
│          │          │          │             │                   │
│          └──────────┴──────────┴─────────────┘                   │
│                              │                                   │
│                              ▼                                   │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                   Particle System                         │   │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐    │   │
│  │  │ Positions    │  │   Colors     │  │  Velocities  │    │   │
│  │  │ (Float32x3)  │  │ (Float32x3)  │  │ (Float32x3)  │    │   │
│  │  └──────────────┘  └──────────────┘  └──────────────┘    │   │
│  │                         │                                 │   │
│  │                         ▼                                 │   │
│  │  ┌──────────────────────────────────────────────────┐    │   │
│  │  │              Custom Shader Material               │    │   │
│  │  │   - Vertex: Size attenuation                     │    │   │
│  │  │   - Fragment: Circular + soft glow               │    │   │
│  │  │   - Blending: Additive                           │    │   │
│  │  └──────────────────────────────────────────────────┘    │   │
│  └──────────────────────────────────────────────────────────┘   │
│                              │                                   │
│                              ▼                                   │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                   Three.js Renderer                       │   │
│  │              requestAnimationFrame loop                   │   │
│  └──────────────────────────────────────────────────────────┘   │
│                              │                                   │
│                              ▼                                   │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                    Visual Overlays                        │   │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────────┐  │   │
│  │  │Scanlines│  │Vignette │  │  Grid   │  │     HUD     │  │   │
│  │  └─────────┘  └─────────┘  └─────────┘  └─────────────┘  │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Key Enhancements Over Original Specification

1. **Sharper Text Formation** - Higher pixel sampling density (step=3)
2. **Stronger Neon Glow** - Custom shader with pow(1.5) falloff
3. **Visible Water Ripple** - Clear sine wave with configurable parameters
4. **Better Depth Perception** - Size attenuation by camera distance
5. **Smoother Transitions** - RGB color lerping between states
6. **Complete Basketball Effect** - Fibonacci sphere + black texture lines
7. **Performance Optimized** - Typed arrays, efficient shader
8. **Enhanced HUD** - Glitch effects, color-coded status
9. **Visible Grid/Scanlines** - CSS animations for cyberpunk aesthetic
10. **Proper XY Constraint** - Strict planar repulsion (no Z-axis)

---

## Implementation Files

- **Main Application:** `/mnt/c/Users/Solomon-PC/Documents/upwork/mediapipe/index.html`
- **Reference Video:** `/mnt/c/Users/Solomon-PC/Documents/upwork/mediapipe/5XG3zhipsnlG4fCu.mp4`
- **Reference Frames:** `frame_0_0.png` through `frame_4_1190.png`

---

*Created: 2026-01-10*
*Status: Implementation Complete*
*Estimated Lines: ~850 (single file)*
