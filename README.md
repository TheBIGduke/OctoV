# OctoV - Real-time Audio Visualizer

[![Three.js](https://img.shields.io/badge/Three.js-3D%20WebGL-blue.svg)](https://threejs.org/)
[![Web Audio API](https://img.shields.io/badge/Web%20Audio%20API-Supported-green.svg)](https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API)
[![HTML5](https://img.shields.io/badge/HTML5-Ready-orange.svg)](https://developer.mozilla.org/en-US/docs/Web/Guide/HTML/HTML5)

OctoV is an immersive, real-time audio visualizer built with **Three.js**. It captures system audio and transforms it into a dynamic, pulsating orb of light and energy. Featuring two distinct visual modes for different audio sources (e.g., user voice vs. TTS), smooth state transitions, and a glowing post-processing pipeline, OctoV provides a captivating visual experience directly in your web browser.

---

## Features

- **Real-time Audio Visualization** – Dynamic sphere reacts to live audio frequencies.
- **System Audio Capture** – Automatically finds and uses a specific system audio output (e.g., "Monitor of Built-in Audio") for seamless experience.
- **Dual Visual Modes** – Switch between 'USER' and 'TTS' modes, each with unique colors, sensitivity, and motion.
- **GLSL Shader Effects** – Custom vertex shaders with Perlin noise for organic, fluid-like distortions.
- **Post-Processing Pipeline** – UnrealBloomPass for a vibrant, high-quality glow effect.
- **Interactive 3D Scene** – OrbitControls for rotating, panning, and zooming the visualizer.
- **Smooth Transitions** – Linear interpolation (lerp) for graceful changes between visual states.
- **Event-Driven Control** – Responds to custom `sourceChange` events or WebSocket messages to switch modes externally.

---

## Table of Contents

- [Requirements](#requirements)
- [Installation](#installation)
- [Configuration](#configuration)
- [Usage](#usage)
- [Project Structure](#project-structure)
- [API Reference](#api-reference)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)

---

## Requirements

### Hardware

- A computer with a functioning audio output and input system.
- A microphone (for 'USER' mode testing) or system audio loopback device.

### Software

- A modern web browser with support for WebGL and the Web Audio API (e.g., Chrome, Firefox, Edge).
- A local web server for development (recommended to avoid CORS issues).
- (Optional) A WebSocket server for remote mode switching.

---

## Installation

This project is a single HTML file and does not require a complex installation process.

1. **Get the Code:**  
   Download or clone the repository to your local machine.
    ```bash
    git clone https://github.com/your-username/OctoV.git
    ```

2. **Set up a Local Server (Recommended):**  
   For the best experience and to avoid browser security restrictions, serve the `OctoV.html` file from a local web server. The Live Server extension for VS Code is an excellent option.

   Or use Python's built-in server:
    ```bash
    python -m http.server
    ```

---

## Configuration

All configuration is done directly within the `<script type="module">` block in `OctoV.html`.

### Audio Device Configuration

The most critical setting is the name of the audio device you wish to capture.

#### Finding Your Audio Device Names

**Method 1: Browser Console (Easiest)**

1. Open `OctoV.html` in your browser
2. Open Developer Tools (press `F12`)
3. Go to the **Console** tab
4. If OctoV cannot find the configured device, it will automatically print a list of all available audio devices with their labels
5. Copy the exact label of the device you want to use

**Method 2: System Commands (Ubuntu/Linux)**

To find the names of your audio input and output devices on Ubuntu, use:

List audio output devices (sinks):
```bash
pactl list short sinks
```

List audio input devices (sources):
```bash
pactl list short sources
```

**Important Note for Ubuntu Users:**  
System audio outputs that can be captured for visualization are typically labeled as **"Monitor of ..."** (e.g., `"Monitor of Built-in Audio Analog Stereo"`). These monitor devices capture what's being played through your speakers/headphones, making them perfect for visualizing system audio like TTS or music playback.

**Method 3: Browser Permissions Dialog**

When you first load OctoV, your browser will request audio permissions. Some browsers show a dropdown list of available devices in this dialog.

#### Update the Target Device

Modify the device label in the `setupAudio()` call for each mode:

```javascript
// For TTS mode (system audio - use "Monitor of..." device)
setupAudio('Monitor of Built-in Audio Analog Stereo');

// For USER mode (microphone - use regular input device)
setupAudio('Built-in Audio Analog Stereo');
```

**Tip:** Device names must match exactly, including capitalization and spacing.

---

### Visual Mode Configuration

You can customize the appearance and behavior of the 'USER' and 'TTS' modes within the `sourceChange` event listener in `OctoV.html`.

#### Where and What You Can Modify

Open `OctoV.html` and look for this section inside the `<script type="module">` block:

```javascript
window.addEventListener('sourceChange', (event) => {
    const source = event.detail.source;
    currentMode = source;

    if (source === 'USER') {
        // --- Customize USER mode ---
        targetColor.set(0x32a873).multiplyScalar(2); // Color (hex)
        targetScaleValue = 0.6;                      // Scale (base size)
        targetSensitivity = 0.15;                    // Sensitivity (audio reactivity)
        targetRotationSpeed = 7.0;                   // Rotation speed (auto-rotate)
        setupAudio('Built-in Audio Analog Stereo');  // Audio device label
    } else if (source === 'TTS') {
        // --- Customize TTS mode ---
        targetColor.set(0x00ddff).multiplyScalar(1.2); // Color (hex)
        targetScaleValue = 1.0;                        // Scale (base size)
        targetSensitivity = 0.20;                      // Sensitivity (audio reactivity)
        targetRotationSpeed = 0.5;                     // Rotation speed (auto-rotate)
        setupAudio('Monitor of Built-in Audio Analog Stereo'); // Audio device label
    }
});
```

**Important:**  
You must update the values in this transition section (`targetColor`, `targetScaleValue`, `targetSensitivity`, `targetRotationSpeed`, and `setupAudio(...)`) to customize the look, feel, and audio source for each mode. These values control the color, size, sensitivity, and rotation speed of the visualizer, as well as which audio device is used for each mode.

#### How to Modify

**Color:**  
Change the hex value in `targetColor.set(0xHEXCODE)` to any valid hex color.
```javascript
targetColor.set(0xff0000); // Red
```

**Scale:**  
Adjust `targetScaleValue` to make the sphere larger or smaller.
```javascript
targetScaleValue = 0.8; // Slightly larger
```

**Sensitivity:**  
Change `targetSensitivity` to control how strongly the sphere reacts to audio. Higher values = more movement.
```javascript
targetSensitivity = 0.3; // More reactive
```

**Rotation Speed:**  
Set `targetRotationSpeed` to control how fast the scene auto-rotates.
```javascript
targetRotationSpeed = 2.0; // Faster rotation
```

**Audio Device:**  
Change the string in `setupAudio('...')` to match your desired input/output device label.

---

### WebSocket Integration

OctoV supports remote mode switching via WebSocket messages.  
Send a JSON message like `{"source": "USER"}` or `{"source": "TTS"}` to the WebSocket server (default: `ws://localhost:9030`).

---

## Usage

1. **Launch the Application**  
   Open the `OctoV.html` file in a web browser, preferably via a local server. The application will immediately request audio permissions and attempt to find the configured audio device.

2. **Interact with the Scene**
    - **Click and Drag:** Rotate the camera around the sphere.
    - **Scroll Wheel:** Zoom in and out.
    - **Right-Click and Drag:** Pan the camera.

   The scene will automatically rotate based on the `targetRotationSpeed` of the current mode.

3. **Change Visual Modes**
    - **Via Console:**  
      Dispatch a custom event in the browser console:
      ```javascript
      window.dispatchEvent(new CustomEvent('sourceChange', { detail: { source: 'USER' } }));
      window.dispatchEvent(new CustomEvent('sourceChange', { detail: { source: 'TTS' } }));
      ```
    - **Via WebSocket:**  
      Send a JSON message to the WebSocket server as described above.

---

## Project Structure

```
OctoV/
├── OctoV.html      # Main application file (contains all HTML, CSS, and JavaScript)
└── README.md       # This file
```

---

## API Reference

### Custom Events

| Event         | Description                                             | Payload (event.detail)         |
|---------------|--------------------------------------------------------|-------------------------------|
| sourceChange  | Triggers a smooth transition to a new visual mode.     | { source: 'USER' } or { source: 'TTS' } |

### Shader Uniforms

| Uniform        | Type   | Description                                                        |
|----------------|--------|--------------------------------------------------------------------|
| u_time         | float  | A continuously increasing value (from clock.getElapsedTime()) for animation. |
| u_frequency    | float  | The average audio frequency data from the AnalyserNode. Drives distortion. |
| u_color        | vec3   | The base color of the sphere's wireframe.                          |
| u_sensitivity  | float  | A multiplier that controls how strongly u_frequency affects displacement. |

---

## Troubleshooting

### Common Issues

#### No Visualization / Sphere is Static

- **Check Audio Permissions:** Ensure you have granted the site permission to use your microphone/audio. If you denied it, you may need to reset permissions in your browser's site settings.
- **Check Audio Output:** Make sure audio is playing through the device specified in the `setupAudio()` call.
- **Check Console for Errors:** Open the developer console (F12). The application will log errors if the device cannot be found or if there is a problem with the Web Audio API.
- **Verify Device Name:** Ensure the device name in your code exactly matches the name shown in the browser console (including capitalization and spacing).

#### "Error: Could not find audio device"

This means the string in the `setupAudio()` call does not match any of your system's available audio devices.

**Solution:**  
1. Open the developer console (F12)
2. Reload the page
3. A list of available devices and their labels will be printed
4. Copy the exact label of the device you want
5. Update the device name in the code

**For Ubuntu users:** Remember that system audio capture requires a "Monitor of..." device name.

#### "Error: You must grant microphone permission"

This occurs if you click "Block" when the browser asks for audio permissions.

**Solution:**  
1. Go to your browser's settings for this site (often by clicking the lock icon in the URL bar)
2. Change the "Microphone" or "Audio" permission to "Allow"
3. Reload the page

#### Poor Performance or Lag

- **Reduce Bloom:** The UnrealBloomPass can be computationally expensive. Try reducing `bloomPass.strength` or increasing `bloomPass.threshold` in the `init()` function.
- **Reduce Geometry Detail:** In the `init()` function, lower the detail of the `IcosahedronGeometry`.
    ```javascript
    // From:
    const sphereGeometry = new THREE.IcosahedronGeometry(1, 10);
    // To a lower value:
    const sphereGeometry = new THREE.IcosahedronGeometry(1, 5);
    ```

#### No System Audio Visualization (Ubuntu)

If you're not seeing visualization when system audio plays:

1. Make sure you're using a "Monitor of..." device name
2. Verify the monitor device exists: `pactl list short sources | grep monitor`
3. Ensure audio is actually playing through the monitored output
4. Check the browser console for any error messages

---

## Contributing

Contributions are welcome!

1. Fork the repository.
2. Create a new branch (`git checkout -b feature/your-feature-name`).
3. Make your changes.
4. Commit your changes (`git commit -m 'Add some amazing feature'`).
5. Push to the branch (`git push origin feature/your-feature-name`).
6. Open a Pull Request.

---

Made with [Three.js](https://threejs.org/) for the creative coding community.