# OctoV - Real-time Audio Visualizer

[![Three.js](https://img.shields.io/badge/Three.js-3D%20WebGL-blue.svg)](https://threejs.org/)
[![Web Audio API](https://img.shields.io/badge/Web%20Audio%20API-Supported-green.svg)](https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API)
[![HTML5](https://img.shields.io/badge/HTML5-Ready-orange.svg)](https://developer.mozilla.org/en-US/docs/Web/Guide/HTML/HTML5)

An immersive, real-time audio visualizer built with **Three.js**. OctoV captures system audio and transforms it into a dynamic, pulsating orb of light and energy. Featuring two distinct visual modes for different audio sources (e.g., user voice vs. TTS), smooth state transitions, and a glowing post-processing pipeline, OctoV provides a captivating visual experience directly in your web browser.

---

## Features

- **Real-time Audio Visualization** – Renders a dynamic sphere that reacts to live audio frequencies.
- **System Audio Capture** – Automatically finds and uses a specific system audio output (e.g., "Monitor of Built-in Audio") for a seamless experience.
- **Dual Visual Modes** – Switches between 'USER' and 'TTS' modes, each with unique colors, sensitivity, and motion characteristics.
- **GLSL Shader Effects** – Utilizes custom vertex shaders with Perlin noise to create organic, fluid-like distortions on the sphere's surface.
- **Post-Processing Pipeline** – Implements an UnrealBloomPass for a vibrant, high-quality glow effect.
- **Interactive 3D Scene** – Uses OrbitControls to allow users to rotate, pan, and zoom around the visualizer.
- **Smooth Transitions** – Leverages linear interpolation (lerp) for graceful changes between visual states.
- **Event-Driven Control** – Responds to custom `sourceChange` events to switch modes externally.

---

## Table of Contents

- [Requirements](#requirements)
- [Quick Start](#quick-start)
- [Installation](#installation)
- [Configuration](#configuration)
- [Usage](#usage)
- [Project Structure](#project-structure)
- [API Reference](#api-reference)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)

## Requirements
### Hardware

- A computer with a functioning audio output and input system.
- A microphone (for 'USER' mode testing) or system audio loopback device.

### Software

- A modern web browser with support for WebGL and the Web Audio API (e.g., Chrome, Firefox, Edge).
- A local web server for development (recommended to avoid CORS issues).

## Quick Start

1. **Clone the repository**

   ```bash
   git clone [https://github.com/your-username/OctoV.git](https://github.com/your-username/OctoV.git)
   cd OctoV
   ```

2. **Run a local server (e.g., using Python)**

   ```bash
   python -m http.server
   ```

3. **Open in your browser**
   Navigate to `http://localhost:8000` in your web browser.

4. **Grant Permissions**
   Allow the page to access your microphone/audio devices when prompted.

5. **Play Audio**
   Play music or speak into your microphone to see the visualization.

## Installation

This project is a single HTML file and does not require a complex installation process.

1. **Get the Code:**
   Download or clone the repository to your local machine.

   ```bash
   git clone [https://github.com/your-username/OctoV.git](https://github.com/your-username/OctoV.git)
   ```

2. **Set up a Local Server (Recommended):**
   For the best experience and to avoid potential browser security restrictions, serve the `index.html` file from a local web server. The Live Server extension for VS Code is an excellent option.

   Alternatively, you can use Python's built-in server:

   ```bash
   # For Python 3.x
   python -m http.server

   # For Python 2.x
   python -m SimpleHTTPServer
   ```

## Configuration

All configuration is done directly within the `<script type="module">` block in `index.html`.
### Audio Device Configuration

The most critical setting is the name of the audio device you wish to capture.

1. **Find Your Device Name:**
   The code logs a list of available audio devices to the browser's developer console (F12) if the target device isn't found. Look for the label of the device you want to use.

2. **Update the Target Device:**
   Modify the `TARGET_DEVICE_LABEL` constant with the name of your device.

   ```javascript
   // Change this string to match your system's audio monitor device
   const TARGET_DEVICE_LABEL = 'Monitor of Built-in Audio Analog Stereo';
   ```

### Visual Mode Configuration

You can customize the appearance and behavior of the 'USER' and 'TTS' modes within the `sourceChange` event listener.

```javascript
window.addEventListener('sourceChange', (event) => {
    const source = event.detail.source;
    currentMode = source;

    if (source === 'USER') {
        // --- Customize USER mode ---
        targetColor.set(0xebbf34).multiplyScalar(1); // Color
        targetScaleValue = 0.6;                      // Base size
        targetSensitivity = 0.1;                     // Reaction to audio
        targetRotationSpeed = 7.0;                   // Auto-rotation speed
    } else if (source === 'TTS') {
        // --- Customize TTS mode ---
        targetColor.set(0x00ddff).multiplyScalar(1.2);
        targetScaleValue = 1.0;
        targetSensitivity = 0.20;
        targetRotationSpeed = 0.5;
    }
});
```

### Post-Processing and Rendering

Adjust the bloom effect for performance or aesthetic preference.

```javascript
// In the init() function
const bloomPass = new UnrealBloomPass(/* ... */);
bloomPass.threshold = 0.4;  // Lower value = more bloom
bloomPass.strength = 0.7;   // How intense the bloom is
bloomPass.radius = 0.5;     // How far the bloom spreads
composer.addPass(bloomPass);
```

## Usage
### 1. Launch the Application

Open the `index.html` file in a web browser, preferably via a local server. The application will immediately request audio permissions and attempt to find the configured audio device.
### 2. Interact with the Scene

- **Click and Drag:** Rotate the camera around the sphere.
- **Scroll Wheel:** Zoom in and out.
- **Right-Click and Drag:** Pan the camera.

The scene will automatically rotate based on the `targetRotationSpeed` of the current mode.
### 3. Change Visual Modes (Advanced)

To switch between the 'USER' and 'TTS' visual styles, you must dispatch a custom `sourceChange` event from the browser's developer console.

```javascript
// To switch to USER mode
window.dispatchEvent(new CustomEvent('sourceChange', { detail: { source: 'USER' } }));

// To switch back to TTS mode
window.dispatchEvent(new CustomEvent('sourceChange', { detail: { source: 'TTS' } }));
```

## Project Structure

The project is self-contained within a single file for simplicity.

```
OctoV/
└── index.html      # Contains all HTML, CSS, and JavaScript code
```

## API Reference
### Custom Events

The visualizer is controlled by a single custom event.

| Event        | Description                                                        | Payload (event.detail)                     |
|--------------|--------------------------------------------------------------------|--------------------------------------------|
| `sourceChange` | Triggers a smooth transition to a new visual mode. Dispatched on the window object. | `{ source: 'USER' }` or `{ source: 'TTS' }` |

### Shader Uniforms

These values are passed to the GLSL shader material to control the visual effect in real-time.

| Uniform      | Type   | Description                                                                 |
|--------------|--------|-----------------------------------------------------------------------------|
| `u_time`     | float  | A continuously increasing value (from `clock.getElapsedTime()`) for animation. |
| `u_frequency`| float  | The average audio frequency data from the `AnalyserNode`. Drives distortion. |
| `u_color`    | vec3   | The base color of the sphere's wireframe.                                   |
| `u_sensitivity` | float | A multiplier that controls how strongly `u_frequency` affects displacement. |

## Troubleshooting
### Common Issues
- **No Visualization / Sphere is Static**

  - Check Audio Permissions: Ensure you have granted the site permission to use your microphone/audio. If you denied it, you may need to reset permissions in your browser's site settings.
  - Check Audio Output: Make sure audio is playing through the device specified in `TARGET_DEVICE_LABEL`.
  - Check Console for Errors: Open the developer console (F12). The application will log errors if the device cannot be found or if there is a problem with the Web Audio API.

- **"Error: Could not find audio device"**

  This means the string in `TARGET_DEVICE_LABEL` does not match any of your system's available audio devices.

  **Solution:** Open the developer console. A list of available devices and their labels will be printed. Copy the exact label of the device you want and paste it into the `TARGET_DEVICE_LABEL` constant in the code.

- **"Error: You must grant microphone permission"**

  This occurs if you click "Block" when the browser asks for audio permissions.

  **Solution:** Go to your browser's settings for this site (often by clicking the lock icon in the URL bar) and change the "Microphone" or "Audio" permission to "Allow". Reload the page.

- **Poor Performance or Lag**

  - Reduce Bloom: The `UnrealBloomPass` can be computationally expensive. Try reducing `bloomPass.strength` or increasing `bloomPass.threshold` in the `init()` function.
  - Reduce Geometry Detail: In the `init()` function, lower the detail of the `IcosahedronGeometry`.

    ```javascript
    // From:
    const sphereGeometry = new THREE.IcosahedronGeometry(1, 10);
    // To a lower value:
    const sphereGeometry = new THREE.IcosahedronGeometry(1, 5);
    ```

## Contributing

Contributions are welcome!

1. Fork the repository.
2. Create a new branch (`git checkout -b feature/your-feature-name`).
3. Make your changes.
4. Commit your changes (`git commit -m 'Add some amazing feature'`).
5. Push to the branch (`git push origin feature/your-feature-name`).
6. Open a Pull Request.

Made with Three.js for the creative coding community.