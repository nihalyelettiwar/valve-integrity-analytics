# Well Integrity Surveillance Analytics Suite

A Python-based analytics project that automates valve pressure test surveillance across offshore oil & gas assets — turning raw test records into risk rankings, exception reports, and visual dashboards.

> **Note:** All data used in this project is **fully synthetic**, generated to mirror realistic offshore well integrity workflows. No proprietary or confidential field data has been used.

---

## Project Background

In offshore oil & gas operations, safety-critical valves (SCSSV, PWV, PMV, AMV) must be pressure-tested on a regular schedule — typically every 180–210 days. Engineers manually review hundreds of test records across multiple assets to identify:
- Valves that **failed** their pressure test (leaking)
- Wells that are **overdue** for their next test
- Valves showing **deteriorating trends** across test cycles

This project automates that entire workflow using Python and pandas, producing outputs that an asset integrity team could use directly in weekly reviews.

---

## Dataset

| File | Description |
|---|---|
| `data/valve_test_dataset_raw.csv` | Original raw dataset (156 well-valve combinations, 1 test cycle) |
| `data/valve_tests_clean.csv` | Cleaned & standardised version with snake_case columns |

**Dataset structure:**
- **39 wells** across 3 assets: European Yard (8), British Palm (14), France Initial State (17)
- **4 valve types per well:** SCSSV, PWV, PMV, AMV
- **5 test cycles** per valve → 780 rows total
- **Date range:** October 2023 – March 2026

**Core business rule (Pass/Fail logic):**
```
ΔP = Tubing Pressure After Build-up − Tubing Pressure After Bled

If ΔP > Leak Rate  →  FAILED  (valve is leaking)
If ΔP ≤ Leak Rate  →  PASSED  (valve is holding)
```

**Recommended test intervals:**
- SCSSV: every **210 days**
- PWV / PMV / AMV: every **180 days**

---

## Project Structure

```
valve_integrity_project/
│
├── data/
│   ├── valve_test_dataset_raw.csv     # Raw input data
│   └── valve_tests_clean.csv          # Cleaned output from Notebook 1
│
├── notebooks/
│   ├── 01_data_ingestion_exploration.ipynb   # Load, validate, explore
│   ├── 02_analysis_engine.ipynb              # Risk classification & scoring
│   └── 03_visualisations.ipynb              # 7 publication-quality charts
│
├── outputs/
│   ├── valve_tests_enriched.csv              # Full 780 rows + trend metrics
│   ├── integrity_summary_valve_level.csv     # 156 rows — one per well-valve
│   ├── integrity_summary_well_level.csv      # 39 rows — one per well
│   └── exception_report.csv                 # Flagged wells only
│
├── requirements.txt
├── .gitignore
└── README.md
```

---

## Notebooks

### Notebook 1 — Data Ingestion & Exploration
- Loads CSV from Google Drive
- Checks nulls, duplicates, date formats
- **Validates the Pass/Fail rule** independently (recalculates ΔP and confirms 0 mismatches)
- Produces 2 overview charts
- Saves `valve_tests_clean.csv`

### Notebook 2 — Analysis Engine
Three analytical modules run in sequence:

**Module A — Barrier Status Engine**
Classifies each valve's current health based on its most recent test:

| Status | Rule |
|---|---|
| `HEALTHY` | Last test Passed, ΔP comfortably within limit |
| `MONITOR` | Last test Passed but ΔP > 75% of Leak Rate |
| `AT RISK` | Last test Failed |
| `CRITICAL` | Last test Failed AND 2+ failures in history |

**Module B — Overdue Test Detector**
- Calculates `days_since_last_test` relative to a reference date
- Flags any valve where `days_since_last_test > recommended_interval_days`

**Module C — Historical Trend Analyser**
- Computes `delta_p_slope` across all test cycles (linear regression)
- Flags `rising_dp_trend` where slope > 0.5 PSI per test cycle
- Counts consecutive failures

**Module D — Master Summary + Risk Score**

Risk Score (0–100) calculated per valve:

| Factor | Points |
|---|---|
| Last test Failed | +40 |
| Repeat failure (2+ ever) | +20 |
| Overdue test | +20 |
| Rising ΔP trend | +10 |
| ΔP > 50% of Leak Rate | +10 |

Risk Levels: `LOW (0–19)` → `MEDIUM (20–39)` → `HIGH (40–59)` → `CRITICAL (60–100)`

### Notebook 3 — Visualisations
Seven charts generated and saved to Drive:

| # | Chart | What it shows |
|---|---|---|
| 1 | Risk Level Distribution | Stacked bar by asset + portfolio donut |
| 2 | Barrier Status Heatmap | All 39 wells × 4 valves — colour-coded grid |
| 3 | ΔP vs Leak Rate Scatter | Every valve plotted against its failure threshold |
| 4 | Historical Failure Timeline | Failure events across 2.5 years by asset |
| 5 | ΔP Trend Lines | Deterioration curves for 6 focus wells |
| 6 | Risk Score by Valve Type | Box plots + barrier status stacked bar |
| 7 | Well Risk Scorecard | All 39 wells ranked — executive summary view |

---

## Key Findings (from synthetic dataset)

- **2 CRITICAL wells:** BP-13 (PWV) and FIS-03 (AMV) — last test Failed
- **51 MEDIUM risk** valves — passed but showing monitoring flags
- **37 MONITOR** valves — ΔP trending close to leak rate threshold
- **0 overdue** tests as of March 2026 reference date
- SCSSV valves carry the highest average risk score across the portfolio

---

## How to Run

### Option 1 — Google Colab (recommended)
1. Upload notebooks to [colab.research.google.com](https://colab.research.google.com)
2. Create folder structure in Google Drive:
   ```
   My Drive/valve_integrity_project/data/
   My Drive/valve_integrity_project/outputs/
   ```
3. Upload `valve_test_dataset_raw.csv` to `data/`
4. Run notebooks in order: `01` → `02` → `03`

### Option 2 — Local
```bash
git clone https://github.com/YOUR_USERNAME/valve-integrity-analytics.git
cd valve-integrity-analytics
pip install -r requirements.txt
jupyter notebook notebooks/01_data_ingestion_exploration.ipynb
```
> Update file paths in each notebook from `/content/drive/MyDrive/...` to your local path.

---

## Tech Stack

| Library | Purpose |
|---|---|
| `pandas` | Data loading, cleaning, groupby aggregations |
| `numpy` | Numerical calculations, linear regression for trend slope |
| `matplotlib` | All chart generation |
| `seaborn` | Theme and colour palette |

---

## Skills Demonstrated

- **Data engineering:** ingestion pipeline, validation, schema standardisation
- **Domain logic:** implementing petroleum engineering Pass/Fail rules in Python
- **Feature engineering:** delta_p slope, barrier status classification, risk scoring
- **Analytical thinking:** multi-factor risk model with transparent, explainable scoring
- **Data visualisation:** 7 chart types — heatmap, scatter, timeline, trend lines, scorecard
- **Workflow design:** modular notebooks with clean handoffs between stages

---

## About

Built as a portfolio project demonstrating Python analytics applied to well integrity surveillance workflows.  
Data is fully synthetic — inspired by real offshore valve testing practices.

---
