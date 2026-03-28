# 🫀 VitalSense — Contactless Vital Signs Monitor

> **Real-time, contactless physiological monitoring using just a webcam.**  
> No wearables. No sensors. Just your face and a camera.

VitalSense uses **remote Photoplethysmography (rPPG)** to extract blood volume pulse signals from subtle skin color changes captured via a standard webcam. It detects faces, identifies registered users, and continuously computes a comprehensive suite of physiological metrics — all in real time.

---

## ✨ Key Features

| Feature | Description |
|---------|-------------|
| **💓 Heart Rate** | Real-time HR via FFT (Welch's periodogram) + adaptive peak detection with EMA smoothing |
| **📊 HRV Analysis** | Time-domain (RMSSD, SDNN, pNN50) and frequency-domain (LF/HF ratio via Lomb-Scargle) |
| **🌬️ Respiration Rate** | Extracted from respiratory-induced intensity modulation of the rPPG signal |
| **😰 Stress Score** | Multi-factor model using LF/HF ratio, RMSSD, HR, respiration, pNN50, and eye metrics |
| **😴 Fatigue Risk** | Combines HRV depression, HR drift, PERCLOS, EAR, blink rate, and gaze stability |
| **🧠 Cognitive Load** | Estimated from HR trends, HRV, blink suppression, gaze stability, and pupil dilation |
| **😊 Mood Detection** | 2D arousal-valence model mapping to: Happy, Calm, Stressed, Anxious, Fatigued, Neutral |
| **👥 Multi-Person Support** | Simultaneous tracking & identification of multiple faces with per-face vitals |
| **👁️ Eye Analysis** | Blink rate, PERCLOS, gaze stability, saccade rate, Eye Aspect Ratio (EAR) |
| **🤖 AI Health Suggestions** | Groq-powered (Llama 3.3 70B) personalized health recommendations based on live vitals |
| **📈 Predictive Analytics** | Stress trend prediction, fatigue forecasting, and anomaly detection (Z-score) |
| **🗄️ Vitals History** | Per-user vitals stored in MongoDB with daily/weekly/monthly analytics |

---

## 🧬 How It Works

### Signal Processing Pipeline

```
Webcam Frame → Face Detection (MediaPipe) → ROI Extraction (forehead, cheeks)
       ↓
  Spatial RGB Averaging → rPPG Signal Extraction
       ↓
  ┌─── POS Algorithm (Wang et al., 2017) ───┐
  │    CHROM Algorithm (De Haan et al., 2013) │  → SNR-Weighted Fusion
  │    GREEN Fallback (Verkruysse et al., 2008)│
  └────────────────────────────────────────────┘
       ↓
  Butterworth Bandpass Filter (0.75–2.5 Hz)
       ↓
  HR Estimation (Welch FFT + Peak Detection) → HRV Analysis → Derived Metrics
       ↓
  WebSocket → Real-time Dashboard
```

### rPPG Algorithms

| Algorithm | Reference | Role |
|-----------|-----------|------|
| **POS** | Wang et al., IEEE Trans. Biomed. Eng., 2017 | Primary — projection orthogonal to skin-tone vector |
| **CHROM** | De Haan & Jeanne, IEEE Trans. Biomed. Eng., 2013 | Primary — chrominance-based pulse extraction |
| **GREEN** | Verkruysse et al., Opt. Express, 2008 | Fallback — green channel extraction for low-quality conditions |

The final BVP signal is an **SNR-weighted fusion** of POS + CHROM, with automatic fallback to GREEN when both primaries have low signal quality. Signal quality is continuously monitored via spectral SNR.

---

## 🏗️ Tech Stack

### Backend
- **FastAPI** — async REST API + WebSocket server
- **MediaPipe Face Mesh** — 468-landmark face detection & tracking
- **NumPy / SciPy** — signal processing (Butterworth filters, Welch PSD, Lomb-Scargle)
- **OpenCV** — frame decoding & image processing
- **MongoDB (Motor)** — async database for user profiles & vitals history
- **Groq API (Llama 3.3 70B)** — AI-powered health suggestions
- **Uvicorn** — ASGI server with hot reload

### Frontend
- **React 19** + **Vite 8** — modern frontend toolchain
- **Framer Motion** — smooth animations and transitions
- **Recharts** — real-time vitals charting
- **Lucide React** — icon system
- **Tailwind CSS** — utility-first styling
- **React Router** — client-side routing

---

## 📂 Project Structure

```
├── backend/
│   ├── main.py                 # FastAPI app entry point, WebSocket handler
│   ├── camera_processor.py     # Frame processing pipeline (per-face state)
│   ├── ippg_signal.py          # rPPG extraction (POS, CHROM, GREEN fusion)
│   ├── signal_processing.py    # Bandpass filter, FFT HR, peak detection
│   ├── hrv_analysis.py         # Time & frequency domain HRV metrics
│   ├── respiration_detection.py# Respiratory rate from BVP envelope
│   ├── metrics_engine.py       # Stress, fatigue, cognitive load, mood
│   ├── roi_extraction.py       # Facial ROI extraction (forehead, cheeks)
│   ├── face_detection.py       # MediaPipe face mesh wrapper
│   ├── face_registration.py    # User registration + face embedding
│   ├── eye_analysis.py         # Blink rate, PERCLOS, gaze, EAR, saccades
│   ├── prediction_engine.py    # Trend prediction & anomaly detection
│   ├── gemini_service.py       # AI vitals validation service
│   ├── database.py             # MongoDB connection & collections
│   ├── models.py               # Pydantic models
│   └── auth.py                 # Authentication routes
│
├── frontend/
│   ├── src/
│   │   ├── pages/
│   │   │   └── Monitor.jsx     # Main monitoring dashboard
│   │   ├── components/
│   │   │   ├── CameraFeed.jsx  # Webcam capture & frame streaming
│   │   │   ├── VitalsPanel.jsx # Real-time vitals display
│   │   │   ├── HealthAlerts.jsx# AI-powered health alerts
│   │   │   ├── Charts.jsx      # Live vitals charts
│   │   │   ├── FaceBoxes.jsx   # Face detection overlay
│   │   │   ├── HeatmapView.jsx # Signal heatmap visualization
│   │   │   └── ...             # Landing page components
│   │   └── index.css           # Global styles
│   └── package.json
│
├── docker-compose.yml
└── README.md
```

---

## 🚀 Getting Started

### Prerequisites
- **Python 3.10+**
- **Node.js 18+**
- **MongoDB** (local or Atlas)
- A **webcam**

### 1. Clone the repository

```bash
git clone https://github.com/Hackverse-healthcare2026/wiet-hackverse-2-0-hackathon-project-submission-healthcare-wh03_visionx4.git
cd wiet-hackverse-2-0-hackathon-project-submission-healthcare-wh03_visionx4
```

### 2. Start the Backend

```bash
cd backend
pip install -r requirements.txt
python main.py
```
The API server starts at `http://localhost:8000`.

### 3. Start the Frontend

```bash
cd frontend
npm install
npm run dev
```
The frontend runs at `http://localhost:5173`.

### 4. Open the App

Navigate to `http://localhost:5173`, click **Start Monitoring**, and allow camera access.

---

## 📊 Metrics Reference

### Core Vitals
| Metric | Method | Normal Range |
|--------|--------|-------------|
| Heart Rate | Welch FFT + peak detection fusion | 60–100 BPM |
| HRV (RMSSD) | Root mean square of successive IBI differences | 20–80 ms |
| HRV (SDNN) | Standard deviation of IBI | 10–200 ms |
| pNN50 | % of successive IBI diffs > 50ms | 0–50% |
| LF/HF Ratio | Lomb-Scargle periodogram (0.04–0.4 Hz) | 0.5–2.0 |
| Respiration Rate | BVP envelope bandpass (0.15–0.5 Hz) | 12–20 BPM |

### Derived Metrics
| Metric | Model | Output |
|--------|-------|--------|
| Stress Score | LF/HF + RMSSD + HR + respiration + pNN50 + eye metrics | 0–100 |
| Fatigue Risk | RMSSD + SDNN + pNN50 + PERCLOS + EAR + blink rate | 0–100 |
| Cognitive Load | LF/HF + HRV + blink suppression + gaze stability + pupil size | Low / Medium / High |
| Mood | 2D arousal-valence model (HR × HRV × stress × gaze) | Happy / Calm / Neutral / Anxious / Stressed / Fatigued |

---

## 🔬 Accuracy & Validation

### Algorithm Validation
- rPPG algorithms ported from the **rPPG-Toolbox** (peer-reviewed benchmark)
- POS and CHROM report **MAE of 2–5 BPM** on benchmark datasets (UBFC-rPPG, PURE)
- Dual-estimator cross-validation: FFT and peak detection must agree within 15 BPM

### Signal Quality Awareness
- Real-time **spectral SNR** computed for both POS and CHROM independently
- Automatic fallback to GREEN channel when signal quality drops
- **Malik's criterion** for IBI outlier rejection (20% deviation filter)
- **Tarvainen detrending** (regularised least-squares) for illumination drift removal

### HRV Standard Compliance
- Time-domain metrics (RMSSD, SDNN, pNN50) follow **Task Force of ESC/NASPE** standards
- Frequency-domain uses **Lomb-Scargle** periodogram (handles unevenly spaced IBIs)
- Physiological clamping prevents biologically implausible readings

---

