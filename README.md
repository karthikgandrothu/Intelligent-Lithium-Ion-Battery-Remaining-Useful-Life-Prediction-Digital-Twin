# Dedicated Lithium-Ion Battery Digital Twin Application

A research-grade **Lithium-Ion Battery Digital Twin** web application powered by a modular **Python Flask backend** and a minimalist **React + TypeScript + Vite + Tailwind CSS** frontend.

This application is strictly focused on operational Digital Twin capabilities using the NASA/PML-UCF Accelerated Life Testing (ALT) dataset (26 battery packs). 

All preliminary data analysis, EDA, preprocessing pipelines, feature engineering, and 5-model benchmarking (Random Forest, XGBoost, SVR, LSTM, GRU) are preserved separately in the project research notebooks:
1. `battery_rul_dataset_analysis_preprocessing.ipynb`
2. `battery_rul_five_model_training_evaluation.ipynb`

---

## 1. Digital Twin Subsystems (Web Interface)

The web interface contains exclusively four dedicated Digital Twin views:

1. **Digital Twin Overview** (`OverviewPage.tsx`)
   - Pack ID, cohort, latest telemetry timestamp, and operating mode indicator (Charge, Discharge, Rest/OCV).
   - Measured physical telemetry (voltage, current, temperature) with clear sensor distinction.
   - Reference Coulomb-counting discharge capacity and calculated State of Health (SOH %).
   - Predicted Remaining Useful Life (RUL) with $95\%$ confidence bounds and inference model selector.
   - Ground-truth comparison and degradation trajectory charts with censorship-safe indicators.

2. **Virtual Battery Explorer** (`VirtualBatteryExplorerPage.tsx`)
   - Persistent virtual state machine synchronized to historical telemetry replay.
   - Telemetry controls: Play, Pause, Resume, Reset, Step Advance (10 to 10,000 observations), and Replay Speed (0.5x to 100x).
   - Scrubbable chronological timeline slider with millisecond-resolution dataset index navigation.
   - Real-time delta inspector showing exact state diffs ($\Delta V$, $\Delta I$, $\Delta T$, $\Delta \text{Index}$) upon each replay tick.
   - Server-side chunked telemetry streaming preventing browser memory exhaustion.

3. **What-if Simulator** (`WhatIfAnalysisPage.tsx`)
   - Physics-guided operational counterfactual simulation under load derating ($5\text{A}$–$19\text{A}$) and thermal stress.
   - Power-law kinetic model ($k_{\text{deg}} \propto I^{0.65} \cdot \exp\left(-\frac{E_a}{R T}\right)$) calibrated against ALT empirical trajectories.
   - Comparative dual-trajectory visualization (Baseline vs. Counterfactual), absolute & relative RUL gain, and out-of-distribution domain boundary guards.

4. **Digital Twin Event History** (`DigitalTwinEventHistoryPage.tsx`)
   - Immutable SQLite transaction ledger recording replay transitions, pack switches, state synchronizations, what-if executions, and telemetry anomalies.
   - Multi-criteria filtering by severity (`ALL`, `INFO`, `WARNING`, `CRITICAL`), battery pack ID, and full-text keyword search.
   - Expandable JSON payload inspector for technical diagnostics.

---

## 2. Architecture & File Structure

```
battery-digital-twin/
├── backend/
│   ├── app/
│   │   ├── main.py                   # Flask 3.1 application factory with CORS
│   │   ├── api/
│   │   │   ├── routes.py             # Modular Flask Blueprint (/api/*)
│   │   │   └── schemas.py            # Pydantic data schemas
│   │   ├── core/
│   │   │   └── config.py             # App settings, paths, ALT dataset configuration
│   │   ├── database/
│   │   │   ├── db.py                 # SQLAlchemy SQLite engine & sessionmaker
│   │   │   └── models.py             # TwinState, ReplaySession, AuditLog, ScenarioRun
│   │   ├── services/
│   │   │   ├── data_service.py       # ALT telemetry loader & trajectory provider
│   │   │   └── inference_service.py  # Model loader (RF, XGBoost, SVR) & RUL inference
│   │   ├── simulation/
│   │   │   └── simulator.py          # Physics-guided counterfactual simulator
│   │   └── twin/
│   │       ├── replay_manager.py     # Chronological replay state controller
│   │       └── twin_engine.py        # Virtual battery state engine
│   ├── tests/                        # 14 Pytest test cases (100% pass)
│   └── requirements.txt
│
└── frontend/
    ├── src/
    │   ├── components/               # Header, Sidebar, Badge
    │   ├── pages/                    # 4 dedicated Digital Twin pages
    │   │   ├── OverviewPage.tsx
    │   │   ├── VirtualBatteryExplorerPage.tsx
    │   │   ├── WhatIfAnalysisPage.tsx
    │   │   └── DigitalTwinEventHistoryPage.tsx
    │   ├── services/                 # Typed Axios REST API client
    │   ├── types/                    # TypeScript interfaces
    │   └── App.tsx                   # Main 4-tab application router
    ├── package.json
    └── vite.config.ts                # Vite dev server with /api proxy to Flask (:5000)
```

---

## 3. Quick Start & Execution

### Prerequisites
- Python 3.10+
- Node.js 18+

### Step 1: Start the Flask Backend
```bash
cd battery-digital-twin/backend
pip install -r requirements.txt
python app/main.py
```
*The Flask server initializes SQLite persistence and listens on `http://127.0.0.1:5000`.*

### Step 2: Start the React Frontend
```bash
cd battery-digital-twin/frontend
npm install
npm run dev
```
*The Vite development server launches at `http://127.0.0.1:5173` with automatic reverse proxying of `/api/*` to the Flask backend.*

### Step 3: Run Backend Test Suite
```bash
cd battery-digital-twin/backend
python -m pytest tests/ -v
```
*(All 14 tests pass across API routes, twin state transitions, model inference, and scenario simulations).*
