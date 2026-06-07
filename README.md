# Automated KPI Validation & Reporting System  — Zero Traffic Detection (LTE / NR)

> **Built and used in production level**  
> Python · MySQL · pandas · SQLAlchemy · openpyxl

---

## The Problem This Solves

Every morning, the network operations team had to manually go through large KPI Excel files to find which LTE/NR cells had zero traffic — a process that took around **4 hours** and was done entirely by hand.

The problem with doing it manually:
- It took up the first half of every working day
- Human error meant some zero-traffic cells were missed or wrongly flagged
- There was no history — if someone asked "how many times has this cell been zero-traffic this month?", there was no easy answer
- The report had to be prepared and distributed by a specific person each morning — a single point of failure

This script eliminates all of that.

---

## What It Does

Runs once each day. Takes the raw KPI files as input. Produces a validated, structured Excel report and stores everything in MySQL — in under 5 minutes.

```
Raw KPI Files (Excel / CSV)
        ↓
Load & parse data
        ↓
Apply business rules → identify zero-traffic cells
        ↓
Enrich with engineer mapping & availability data
        ↓
Merge with online cell records
        ↓
Generate structured Excel report  →  Operations team
        ↓
Store results in MySQL             →  Historical tracking
```

---

## Business Rules Applied

A cell is flagged as zero-traffic only when **all** of the following are true — this prevents false positives from cells that are legitimately offline:

| Condition | Value | Why It Matters |
|---|---|---|
| Date | Latest date only | Avoid flagging old resolved issues |
| Traffic hours monitored | = 4 hours | Sufficient observation window |
| NR traffic volume | = 0 | Actual zero traffic, not missing data |
| Administrative state | UNLOCKED | Cell is supposed to be active |
| Operational state | ENABLED | Cell is not in maintenance |

Only cells that fail all five checks together are flagged — filtering out noise and reducing false escalations.

---

## Key Results

- **4 hours → under 5 minutes** — daily reporting effort reduced by 98%
- Zero-traffic cells that previously went undetected for days are now caught and escalated **same day**
- Operations team can now query history: *"How many times has this cell had zero traffic this month?"* — previously impossible
- Report is generated and ready automatically — no dependency on any individual to prepare or send it

---

## Output

**Excel Report** — 3 sheets:
- `Consolidated` — all zero-traffic and online cells with full attributes
- `Summary` — aggregated view by UPC (network unit)
- `Zero Traffic Days` — per-cell count of zero-traffic occurrences over time

**MySQL Database** — 3 tables:
- `consol_non_consol` — consolidated cell data (zero-traffic + online)
- `summary` — UPC-level summary for management reporting
- `no_of_days_zerotraffic` — historical zero-traffic day count per cell

The MySQL tables enable trend analysis and historical queries that the Excel-only process never could.

---

## Tech Stack

| Tool | What It Does in This Project |
|---|---|
| Python (pandas) | Loads and processes raw KPI files, applies business rule filters |
| openpyxl | Reads input Excel files, writes structured multi-sheet output reports |
| SQLAlchemy | Connects to MySQL and auto-creates tables — no manual DDL required |
| MySQL | Stores validated results for historical tracking and downstream queries |
| config.json | Manages all file paths and parameters — no hardcoding in the script |

---

## How to Run

**1. Configure paths**

Update `config.json` with your file locations:
```json
{
  "kpi_file": "path/to/daily_kpi.xlsx",
  "reference_file": "path/to/cell_reference.xlsx",
  "output_path": "path/to/output/",
  "db_host": "localhost",
  "db_name": "telecom_kpi"
}
```

**2. Install dependencies**
```bash
pip install pandas openpyxl sqlalchemy mysql-connector-python
```

**3. Run**
```bash
python zero_traffic_automation.py
```

The script creates the MySQL database and tables automatically on first run.

**Sample input files** are available here:  
[Google Drive — Sample KPI Files](https://drive.google.com/drive/folders/1Y4I8nlI1w8makIy5cD4ootJW68-cD3cZ?usp=sharing)

---

## Repository Structure

```
Telecommunication-Network-KPI-Automation-Python-SQL/
├── zero_traffic_automation.py   # Main script
├── config.json                  # File paths and DB config
├── requirements.txt             # Python dependencies
└── README.md
```

---

## Context

This project was built to solve a real operational problem. The detection logic, business rules, and output format were developed by working directly with the network optimization and operations teams to match their actual escalation workflows. Because of data confidentiality, the raw data is replaced with imagined data.

It is the same automation referenced in my [resume](https://github.com/VishwanathMalli) under professional experience.

---

*Built by Vishwanath Malli 
