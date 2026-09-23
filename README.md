# Railway Data Engineering Pipeline

> A four-level, end-to-end data-engineering project on Indian railway train-schedule data —
> built with **Python & pandas** as part of my Data Engineering internship.

![Python](https://img.shields.io/badge/Python-3.10%2B-blue)
![pandas](https://img.shields.io/badge/pandas-2.x-orange)
![Notebook](https://img.shields.io/badge/Notebook-Jupyter%20%7C%20Colab-yellow)
![Status](https://img.shields.io/badge/Level%201-✔%20complete-brightgreen)
![Status](https://img.shields.io/badge/Level%202-✔%20complete-brightgreen)
![Status](https://img.shields.io/badge/Level%203-✔%20complete-brightgreen)

---

## Overview

Raw operational data is messy: inconsistent station spellings, missing values, and no
standard format. This project turns a raw train-schedule CSV into a clean, analysis-ready
dataset through four progressive levels — from exploration and cleaning to advanced
engineering — following real data-engineering practice:

- **Explicit, documented cleaning policies** (never silent defaults)
- **Reproducible notebooks** that run top-to-bottom in Google Colab
- **Raw → processed data layering** (`data/raw` → each level's `output/` folder)
- **One self-contained directory per level** — notebook, Colab guide and outputs live together

---

## Repository Structure

```
railway-data-engineering/
├── README.md
├── requirements.txt
├── .gitignore
├── data/
│   └── raw/
│       ├── Railway_info.csv        
│       
├── level3/
   ├── level3_advanced_data_analysis.ipynb
   ├── colab_cells.md
   └── output/
       ├── fig_weekly_distribution.png  
       ├── fig_top_routes.png           
       ├── fig_hub_heatmap.png          
       ├── weekly_summary.csv           
       └── insights_report.md           
```

---

## Project Roadmap

| Level | Focus | Notebook | Status |
|-------|-------|----------|--------|
| **1** | Data Exploration & Basic Operations | `level1/level1_data_exploration_and_basic_operations.ipynb` | Complete |
| **2** | Data Transformation & Aggregation | `level2/level2_data_transformation_and_aggregation.ipynb` | Complete |
| **3** | Advanced Data Analysis | `level3/level3_advanced_data_analysis.ipynb` | Complete |
| **4** | *(to be added)* | — | Pending |

### Level 1 — Data Exploration & Basic Operations

| Task | What the code does |
|------|--------------------|
| **1.1 Load & Inspect** | Loads the CSV (auto-detects `trains.csv` or opens a Colab upload widget), displays the first 10 rows, and profiles structure, dtypes and a missing-value report. |
| **1.2 Basic Statistics** | Counts trains and unique source/destination stations; ranks the most common source & destination stations. Uses a **defensive column resolver** so it adapts to header naming variations (`Source Station`, `source`, `From`, …). |
| **1.3 Data Cleaning** | Applies a documented missing-value policy and standardizes station names (trim, collapse spaces, upper-case). Exports `level1/output/trains_cleaned.csv`. |

**Cleaning policy (Level 1):**

1. Drop rows that carry no information at all.
2. Drop train records without an identifier (unusable rows).
3. Missing station names → explicit `UNKNOWN` placeholder.
4. Numeric measure columns → median imputation (identifiers are *never* imputed).
5. Remaining string columns (e.g. times) → `UNKNOWN` placeholder.

### Level 2 — Data Transformation & Aggregation 

| Task | What the code does |
|------|--------------------|
| **2.1 Filtering** | Filters services by operating day (default `Saturday` → 1,593 trains) and builds a dedicated dataframe for trains departing a chosen station (default: busiest hub `CST-MUMBAI` → 513 trains). |
| **2.2 Grouping & Aggregation** | Group-by source station with train counts (921 stations), plus average trains per operating day for each station. |
| **2.3 Enrichment** | Adds a `Day_Category` column (Weekday / Weekend) with an explicit `Unknown` fallback — and repairs 1,153 corrupted day tokens (`Fridayd` → `Friday`) discovered during profiling. |

### Level 3 — Advanced Data Analysis

| Task | What the code does |
|------|--------------------|
| **3.1 Pattern Analysis** | Bar plot of journeys per day of week (Friday busiest: 1,649 · Monday quietest: 1,503), top-10 routes & two-way corridors (busiest: CHENNAI BEACH ↔ TAMBARAM, 274 services), hub-concentration metric, and a hub × day-of-week heatmap. |
| **3.2 Correlation & Insights** | Pearson correlation of service volume vs day-of-week position (r=+0.381, weak) and vs weekend flag (r=+0.127, negligible); generates `insights_report.md` with data-driven insights & recommendations. |

---

## Dataset

The submission runs on **`Railway_info.csv`** — 11,113 train records across 900+ Indian
railway stations (bundled in `data/raw/`). A tiny `sample_trains.csv` is kept for
smoke-tests so notebooks always run out of the box.

| Column | Description |
|--------|-------------|
| `Train_No` | Train number (unique per record) |
| `Train_Name` | Train name / code |
| `Source_Station_Name` | Origin station |
| `Destination_Station_Name` | Terminus station |
| `days` | Day of the week the service runs |

**Headline stats (Level 1):** 11,113 records · 921 unique source stations · 924 unique
destination stations · busiest hub **CST-MUMBAI** (513 departures / 514 arrivals) ·
0 missing values.

---

## Getting Started

### Option A — Google Colab (recommended for submissions)

1. Open [colab.research.google.com](https://colab.research.google.com) → **Upload** the notebook from its level folder (`level1/`, `level2/`, …).
2. Run all cells. If `trains.csv` is not in the session, an upload widget appears — drop your CSV in.
3. The cleaned CSV is saved and auto-downloaded at the end.

### Option B — Local / Jupyter

```bash
git clone https://github.com/<your-username>/railway-data-engineering.git
cd railway-data-engineering
python -m venv .venv && source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -r requirements.txt
jupyter notebook level1/level1_data_exploration_and_basic_operations.ipynb
```

---

## ️ Tech Stack

- **Python 3.10+** — core language
- **pandas / NumPy** — loading, profiling, cleaning
- **Matplotlib** — publication-ready visualizations (bar plots, heatmap)
- **Jupyter / Google Colab** — reproducible analysis notebooks

---

## Engineering Notes

- **Defensive column resolution** — notebooks survive header renames across dataset versions.
- **Raw vs processed separation** — raw data is never overwritten; every level writes to its own `output/` folder, and Level N+1 reads Level N's output.
- **Documented policies** — every imputation/drop decision is stated in code comments, and printed before/after reports make the effect measurable on every run.
- **Data-quality fixes in the open** — corrupted day tokens (`Fridayd`, `Saturdayd`, …) are detected, counted and repaired with a conservative prefix rule instead of being silently dropped.
- **Versioned analysis artifacts** — every plot, summary CSV and the insights report are written to the level's `output/` folder, so results are reviewable without re-running.

---

## License

Educational project — internship submission. Free to use for learning purposes.

---

## Author

**[Sushant Shelar]**
Data Engineering Intern

- GitHub: [github.com/sushantshelar2707-ui](https://github.com/sushantshelar2707-ui)
- LinkedIn: [linkedin.com/in/sushant-s-924bb5352](https://www.linkedin.com/sushant-s-924bb5352)

