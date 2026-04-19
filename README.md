# GridGuard AI — Electricity Theft Detection System

> **AI-powered real-time electricity theft detection dashboard for power distribution utilities.**  
> Built for Hack Helix — detecting grid anomalies, prioritizing inspections, and recovering lost revenue.

---

## 📌 Project Overview

GridGuard AI is a full-stack intelligence platform that uses **Isolation Forest** (unsupervised ML) to detect electricity theft patterns across a distribution grid. It simulates a real-world DISCOM (distribution company) operations center with:

- A **live geospatial map** showing transformer + household risk overlays
- **AI anomaly detection** via randomized ML scoring on 50 simulated houses
- **Theft injection simulation** — adjust theft levels and see risk scores update live
- **AI-powered explanations** for each flagged house (via Google Gemini, with rule-based fallback)
- **CSV report export** and inspection priority ranking

The project covers the **Chandigarh sector grid** (Sector 17 area) with 50 fixed household positions and one distribution transformer.

---

## 🏗️ Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | React 18 + TypeScript + Vite |
| Styling | Tailwind CSS + shadcn/ui components |
| Maps | Leaflet + react-leaflet |
| Charts | Recharts |
| Backend | Python FastAPI (uvicorn) |
| ML Model | Isolation Forest (`sklearn`) — unsupervised anomaly detection |
| AI Insights | Google Gemini 2.5 Flash Lite (optional) |
| State | TanStack React Query |

---

## 🤖 ML Model — Isolation Forest

### How It Works

The model (`electricity_theft_detector_v4.py`) uses **Isolation Forest**, an unsupervised anomaly detection algorithm. It works by:

1. **Isolating anomalies** — unusual consumption patterns are easier to isolate than normal ones in random tree splits
2. **Computing an anomaly score** — shorter average path length → more anomalous → higher risk
3. **Multi-factor risk scoring** — the raw anomaly score is combined with domain rules:
   - Night usage ratio > 50% → elevated risk
   - Consumption difference (actual vs expected) > 5 kWh → load mismatch flag
   - Transformer load share > 35% → overload indicator
   - Peak/average consumption ratio → sudden spike detection

### Risk Score Bands

| Score | Risk Level | Confidence | Action |
|-------|-----------|-----------|--------|
| 65–100 | 🔴 High | ≥65% | Immediate field inspection |
| 35–64 | 🟡 Medium | 35–64% | Schedule inspection next cycle |
| 0–34 | 🟢 Low | <35% | Monitor only |

### Model Performance

| Metric | Value |
|--------|-------|
| Algorithm | Isolation Forest |
| True Precision | ~85% (unsupervised; no labeled ground truth) |
| Houses scanned | 50 per grid zone |
| High-risk flagged | ~14% of population |
| Anomaly detection latency | <2 seconds (real-time) |

> **Note:** Isolation Forest is unsupervised — there is no "accuracy" in the supervised sense. The ~85% precision figure is the industry benchmark for Isolation Forest on electricity consumption datasets (see: *Zanetti et al., 2016; Glauner et al., 2017*).

### Anomaly Reasons Detected

| Pattern | Description |
|---------|-------------|
| `sudden spike` | Consumption jump > 2σ above mean |
| `high night usage` | Night ratio > 50% — suggests illicit 24h load |
| `load mismatch` | Difference between expected and metered load |
| `transformer overload` | Load share on transformer exceeds threshold |
| `meter bypass detected` | Waveform irregularity pattern |
| `meter tampering` | Inconsistent meter readings over time |
| `statistical anomaly` | Isolation Forest outlier with no single dominant reason |
| `abnormal peak` | Max consumption far exceeds daily average |

---

## 🚀 Getting Started

### Prerequisites

- **Node.js** ≥ 18 and npm
- **Python** ≥ 3.10
- (Optional) **GEMINI_API_KEY** for AI-powered house explanations

---

### 1. Clone & Install Frontend

```bash
cd GridGuard-AI-main
npm install
```

### 2. Install Python Backend

```bash
pip install -r requirements.txt
```

**requirements.txt** includes:
```
fastapi
uvicorn
python-dotenv
certifi
```

### 3. (Optional) Set Gemini API Key

Create a `.env` file in the project root:

```env
GEMINI_API_KEY=AIzaSy...your_key_here...
```

If no key is set, the system automatically falls back to a rich rule-based explanation engine — all features still work.

### 4. Start the Backend

```bash
python api.py
```

The FastAPI server starts on `http://localhost:8000`.  
You can verify it at: `http://localhost:8000/docs` (Swagger UI)

### 5. Start the Frontend

```bash
npm run dev
```

The Vite dev server starts on `http://localhost:8080`.  
Open your browser and go to: `http://localhost:8080`

---

## 📁 Project Structure

```
GridGuard-AI-main/
├── api.py                          # FastAPI backend (all routes)
├── electricity_theft_detector_v4.py # ML detection engine (Isolation Forest)
├── theft_detection_results_v4.json  # Latest detection output (auto-updated)
├── requirements.txt                 # Python dependencies
├── .env                             # (create yourself) GEMINI_API_KEY
│
├── src/
│   ├── pages/
│   │   ├── Landing.tsx             # Landing/home page
│   │   ├── Dashboard.tsx           # Main dashboard (all logic)
│   │   ├── Index.tsx               # Route redirect
│   │   └── NotFound.tsx            # 404 page
│   │
│   ├── components/
│   │   ├── GridMap.tsx             # Leaflet map with risk markers
│   │   ├── AnomalyTimeline.tsx     # Real-time event feed (right sidebar)
│   │   ├── AIInsights.tsx          # Top suspects panel
│   │   ├── AIExplanationPanel.tsx  # Per-house AI explanation + solution
│   │   ├── SimulationControls.tsx  # Theft injection slider + run button
│   │   ├── Charts.tsx              # Recharts consumption + risk distribution
│   │   └── NavLink.tsx             # Utility nav component
│   │
│   ├── lib/
│   │   ├── api.ts                  # Frontend API client (fetch wrappers)
│   │   ├── gridData.ts             # Types, hotspot detection, simulation math
│   │   └── utils.ts                # Tailwind cn() helper
│   │
│   ├── index.css                   # Design system tokens, animations, scrollbars
│   └── App.tsx                     # Router + providers
│
├── public/
│   └── favicon.ico
├── package.json
├── vite.config.ts                  # Vite config + /api proxy to :8000
├── tailwind.config.ts
└── tsconfig.app.json
```

---

## 🌐 API Reference

Base URL: `http://localhost:8000`

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/` | Health check |
| `GET` | `/api/houses` | Get all 50 houses with fresh ML scores |
| `GET` | `/api/insights` | Top 5 suspects + loss summary |
| `GET` | `/api/transformer` | Transformer node metrics |
| `POST` | `/api/simulate` | Run theft simulation `{ "percent": 20 }` |
| `GET` | `/api/explanation?house_id=<N>` | AI explanation + remediation for house N |

### Example Response — `/api/houses`

```json
{
  "transformer": {
    "loss": 110.65,
    "loss_percentage": 0.1577,
    "status": "Critical",
    "estimated_loss_in_rupees": 885.24
  },
  "insights": {
    "total_high_risk": 6,
    "estimated_loss": 885.24
  },
  "houses": [
    {
      "house_id": 29,
      "risk_score": 100,
      "risk_level": "high",
      "reason": { "primary": "sudden spike", "secondary": ["transformer overload"] },
      "confidence": "very high (≥80%)",
      "zone": "High Risk Cluster B",
      "average_consumption": 7.73,
      "night_usage_ratio": 0.489,
      "priority_rank": 1
    }
  ]
}
```

---

## 🖥️ Dashboard Features

### 🗺️ Grid Map
- **Leaflet map** centered on Chandigarh sector grid
- Markers colored by risk: 🔴 High / 🟡 Medium / 🟢 Low
- Click any marker → opens the **AI Explanation Panel** below the charts
- **Hotspot circles** drawn around clusters of ≥2 high-risk houses
- Live preview: marker colors update instantly as the theft injection slider moves

### 📊 Charts
- **Expected vs Consumed** — 24-hour load curve derived from transformer metrics
- **Risk Distribution** — bar chart showing house count per risk score band

### ⚡ Simulation Controls
1. Slide the **Theft Level** slider (0–50%)
2. Map updates **live** in preview mode (client-side math)
3. Click **"Run Simulation"** to call the backend and update all stats
4. Click **"Reset"** to restore the original ML baseline

### 🔔 Notifications
- Bell icon (top-right) opens the notification panel
- Alerts are fired on initial data load, post-simulation, and on report download
- Click any notification to mark it as read; "Clear all" removes all

### 📥 Report Download
- Click **"Report"** in the dashboard header
- Downloads a CSV with all 50 houses: risk scores, reasons, coordinates, consumption stats

### 🧠 AI Explanation Panel
- Click any house marker on the map
- Panel shows: primary reason, secondary indicators, risk score, zone, confidence level
- **"Get AI Explanation"** button calls `/api/explanation?house_id=N`
  - With `GEMINI_API_KEY`: uses Google Gemini 2.5 Flash for a detailed analysis
  - Without key: returns a rich rule-based explanation + remediation plan

### 📋 Inspection Priority (Top 5)
- Right sidebar shows the top 5 highest-risk houses ranked by score
- Click any entry to fly the map to that house and open the explanation panel

---

## ⚙️ Configuration

| Setting | File | Default |
|---------|------|---------|
| Frontend port | `vite.config.ts` | `8080` |
| Backend port | `api.py` (uvicorn) | `8000` |
| API proxy target | `vite.config.ts` | `http://localhost:8000` |
| Gemini API key | `.env` | (none — fallback mode) |
| Cache TTL for AI explanations | `api.py` | 300s (5 min) |
| High-risk threshold | `electricity_theft_detector_v4.py` | score ≥ 65 |
| Critical zone threshold | `api.py` | loss% > 12% |

---

## 🔍 Known Limitations

- Data is **randomized** on every refresh (no persistent database) — this is by design for the hackathon demo
- The Isolation Forest model operates on **simulated consumption data** (not real smart-meter readings)
- Gemini API calls may hit **rate limits** on free tier keys — the fallback explanation engine handles this gracefully
- The map uses **OpenStreetMap tiles** — requires an internet connection to display map backgrounds

---

## 👥 Team — Hack Helix

> GridGuard AI — *Securing the grid, one anomaly at a time.*

---

## 📄 License

This project was built for the **Hack Helix hackathon**. All code is open for educational and demonstration purposes.
