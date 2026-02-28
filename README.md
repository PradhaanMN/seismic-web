# ⬡ Seismic Web — Bio-Inspired Perimeter Intrusion Detection

A real-time seismic signal simulation and analysis dashboard running entirely in the browser — no server, no backend. Built with PyScript (Pyodide + NumPy) and vanilla JavaScript canvas rendering.

**Live Demo:** [seismic-web.netlify.app](https://seismic-web.netlify.app)

---

## Overview

Seismic Web simulates a 64-sensor geophone perimeter array deployed around a 100 m × 70 m field. When an intruder enters the field, seismic waves radiate outward and arrive at each sensor at slightly different times. The system uses **Time Difference of Arrival (TDOA)** to estimate the intruder's position and classifies the threat type from the signal's frequency content.

Every component runs client-side:
- Signal generation and DSP in **Python** (via PyScript/Pyodide WASM)
- 60 fps canvas rendering in **JavaScript**
- No dependencies, no build step — just open `index.html`

---

## Features

### Simulation
- **64 perimeter sensors** on a 20×14 grid (5 m/cell → 100 m × 70 m field)
- **Moving target** with random-walk velocity perturbation
- Three intruder types with physically-motivated frequency signatures:
  - 🚶 **Human** — footstep cadence, 1–2 Hz seismic dominant
  - 🚗 **Vehicle** — engine vibration, ~50 Hz with harmonics
  - 🐾 **Animal** — paw-fall bursts, 4–6 Hz dominant

### Signal Processing (Python / NumPy)
- Exponential amplitude decay: `s(t) = A·e^(−αd)·f_type(t) + η(t)`, α = 0.12 m⁻¹
- Hanning-windowed FFT with per-trigger adaptive frequency axis
- SNR calculation: `SNR = 10·log₁₀(P_signal / σ²)`
- **Adaptive noise floor** — 15th-percentile RMS across sliding windows
- **TDOA grid-search localisation** — minimises `Σ(Δt_ij − Δt̂_ij)²` over 18×12 search space

### Tracking & Estimation
- **Kalman filter** — constant-velocity model smooths TDOA position estimates, reducing measurement noise
- Position history trail on canvas (80-point rolling buffer)
- TDOA estimated position (cyan diamond) and Kalman-filtered position (purple diamond) both drawn live

### Dashboard
- **4-panel signal controls**: noise σ, sample rate fₛ, wave speed v, target speed
- **8 engineering metrics**: dominant frequency, confidence, estimated X/Y, SNR, position error, wavelength λ, period T
- **4 mini-charts** in 2×2 grid:
  - Waveform s(t) with amplitude axis
  - FFT |X(f)| with adaptive frequency scale and passband shading
  - Position X(t) and Y(t) — true vs estimated
  - TDOA Δt arrival time bar chart (8 nearest sensors)
- **Classification banner** with confidence bar
- **Event log** with timestamps
- **⚙ System Internals modal** — live formula panel (signal model, TDOA equation, grid-search, signal quality + noise floor)

---

## Architecture

```
index.html
├── <py-config>          PyScript package declaration (numpy, matplotlib)
├── <script>             JavaScript — RAF loop, canvas rendering, Kalman filter
│   ├── buildSensors()   64-point perimeter geometry
│   ├── mainLoop()       60 fps: update target → render grid → render mini charts
│   ├── runPythonAnalysis()  calls Python every 2 s via JS↔Py bridge
│   ├── kalmanUpdate()   simplified scalar Kalman filter for position smoothing
│   └── drawKalmanEstimate()  purple diamond overlay on grid canvas
└── <script type="py">   Python DSP engine
    ├── generate_signal()     signal model with decay + noise
    ├── fft_analysis()        Hanning FFT, passband detection, dominant freq
    ├── adaptive_noise_floor()  15th-percentile RMS noise estimation
    ├── tdoa_estimate()       grid-search TDOA localisation
    └── py_analyse_js()       entry point, returns JSON to JavaScript

styles.css               All layout and neon UI styling
```

**JS ↔ Python bridge:**
```js
js.window.pyAnalyse = create_proxy(py_analyse_js)  // exposed from Python
const result = JSON.parse(String(window.pyAnalyse(...args)))  // called from JS
```

---

## Running Locally

No install needed. Just serve the files over HTTP (required for PyScript WASM loading):

```bash
# Python
python -m http.server 8080

# Node
npx serve .
```

Then open `http://localhost:8080`.

> **Note:** First load downloads the Pyodide WASM runtime + NumPy (~8 MB). Subsequent loads are cached.

---

## Tech Stack

| Layer | Technology |
|---|---|
| DSP / Python | [PyScript 2024.11.1](https://pyscript.net/) + Pyodide WASM |
| Numerics | NumPy |
| Rendering | HTML5 Canvas (vanilla JS) |
| Fonts | Orbitron, Share Tech Mono (Google Fonts) |
| Hosting | Netlify (auto-deploy from GitHub) |

---

## Signal Model

```
s(t) = A · e^(−αd) · f_type(t) + η(t)

A       — source amplitude
α       — soil attenuation coefficient (0.12 m⁻¹)
d       — distance from source to nearest sensor
f_type  — type-specific waveform (human/vehicle/animal)
η(t)    — Gaussian noise, σ configurable via slider
```

**TDOA localisation:**
```
Δt_ij = (d_i − d_j) / v_s

Minimise:  Σ ( Δt_ij − Δt̂_ij(x,y) )²   over all candidate grid points (x,y)
```

---

## Project Structure

```
seismic-web/
├── index.html      Main app (HTML + inline JS + inline PyScript)
├── styles.css      UI stylesheet
└── README.md
```

---

## Version History

| Commit | Changes |
|---|---|
| `b8d7012` | v3.0 — 64 sensors, moving target, 3 mini charts, 4 sliders |
| `de4463b` | v3.1 — axis labels, TDOA chart, system internals, 8 metrics |
| `27716b4` | Corrected frequencies, adaptive FFT scale, header fix |
| `a16859e` | Compact layout fits 100vh, brighter neon palette |
| `c92e30b` | Horizontal 4-chart strip below grid |
| `16d1a13` | 2×2 chart grid, spinner loader, System Internals modal, Kalman filter, adaptive noise floor |
| `12eacdd` | Fix canvas-area flex column — charts full-width below grid |
