# Cyberpunk Gesture-Interactive Particle System

A high-performance, single-file HTML application featuring a cyberpunk interactive particle system controlled by hand gestures. Built with **Three.js** for WebGL rendering and **MediaPipe Hands** for real-time computer vision.

![License](https://img.shields.io/badge/license-MIT-blue.svg)

## 🌌 Overview

This project simulates a "Neural Link" interface where users can interact with a cloud of 12,000 neon particles using hand gestures. It runs entirely in the browser and features a retro-futuristic cyberpunk aesthetic complete with scanlines, glitches, and procedural audio.

## ✨ Features

-   **Real-time Hand Tracking**: Detects left and right hands independently using MediaPipe.
-   **Interactive Physics**:
    -   **Repulsion Field**: Push particles away with a fist.
    -   **Nebula/Ripple**: Create wave effects with an open hand.
    -   **Explosions**: Catch and release particles to trigger dynamic scatters.
-   **Gesture Commands**: Form shapes and text by holding up different numbers of fingers.
-   **Procedural Audio**: Generates glass-like sound effects dynamically using the Web Audio API.
-   **Visual Effects**:
    -   Custom GLSL shaders for glowing "glass-like" particles.
    -   CRT scanlines, vignettes, and chromatic aberration glitches.
    -   Dynamic HUD (Heads-Up Display).

## 🎮 Controls & Gestures

Ensure your webcam is enabled and you are in a well-lit environment.

### 👋 Left Hand (Command Interface)
Controls the **Target Shape** of the particles.

| Gesture | Resulting Text/Shape | Color Theme |
| :--- | :--- | :--- |
| **1 Finger** | "Hello" | Neon Blue |
| **2 Fingers** | "Gemini3" | Neon Yellow |
| **3 Fingers** | "Opus 4.5" | Neon Pink |
| **4 Fingers** | "Goodbye" | Neon Green |
| **5 Fingers (Open)** | **Catch Mode** (Attracts particles) | - |

### ✋ Right Hand (Physics Interface)
Controls the **Physics Forces** applied to the particles.

| Gesture | Effect |
| :--- | :--- |
| **Fist (0 Fingers)** | **Repulsion Field**: Pushes particles away from your hand. |
| **Open Hand (5 Fingers)** | **Nebula Mode**: Creates a water-ripple effect and spreads particles. |
| **1 Finger** | Text: "Diza" (Neon Purple) |
| **2 Fingers** | Text: "Gaby" (Coral) |
| **3 Fingers** | Text: "Solomon" (Gold) |

### 💥 Combo Moves
*   **The Drop**: Open your **Left Hand** (Attract) to gather particles, then quickly **Close** both hands to trigger a physics explosion.

## 🚀 Quick Start

1.  **Clone the repository**:
    ```bash
    git clone https://github.com/udohsolomon/gesture-interactive-particle-system.git
    ```
2.  **Run the file**:
    *   Simply open `cyberpink.html` in a modern web browser (Chrome/Edge recommended).
    *   *Note*: For best camera performance, it is recommended to run this via a local server (e.g., VS Code "Live Server" or Python `http.server`) rather than opening the file directly, to avoid browser security restrictions on the webcam.

    **Using Python:**
    ```bash
    python -m http.server 8000
    # Open http://localhost:8000/cyberpink.html
    ```

3.  **Initialize**:
    *   Click the **[ CLICK TO INITIALIZE SYSTEM ]** prompt on the screen to enable audio and the physics engine.

## 🛠️ Technologies

*   **[Three.js](https://threejs.org/)**: WebGL rendering engine.
*   **[MediaPipe Hands](https://developers.google.com/mediapipe)**: Machine learning-based hand tracking.
*   **GLSL**: Custom shader programming for particle effects.
*   **Web Audio API**: Real-time sound synthesis.

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---
*Concept & Implementation by Solomon Udoh*
