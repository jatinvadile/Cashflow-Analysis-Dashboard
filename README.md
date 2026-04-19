# 💰 Cashflow Analytics & Financial Monitoring Platform

> An end-to-end financial intelligence system built on **Power BI**, **Python**, and **REST API integration** — consolidating multi-bank, multi-currency cash data into a single real-time dashboard with automated exchange rate management.

---

## 📌 Table of Contents

- [Overview](#-overview)
- [Business Problem](#-business-problem)
- [Architecture](#-architecture)
- [Tech Stack](#-tech-stack)
- [Solution Layers](#-solution-layers)
- [Dashboard Breakdown](#-dashboard-breakdown)
- [Data Modeling](#-data-modeling)
- [Key Logic Implemented](#-key-logic-implemented)
- [Key Insights](#-key-insights-generated)
- [Business Impact](#-business-impact)
- [Challenges & Solutions](#-challenges--solutions)
- [Getting Started](#-getting-started)
- [Future Enhancements](#-future-enhancements)

---

## 📖 Overview

This project is a **complete Cashflow Analytics and Financial Monitoring solution** that tracks inflows, outflows, balances, and currency distribution across five independent banking sources. A Python-powered exchange rate automation layer fetches live USD ↔ TZS rates daily, enabling consistent and accurate multi-currency reporting across the entire platform.

The dashboard provides stakeholders with a **centralized view of financial health** — monitoring liquidity positions, identifying cash flow trends, and supporting faster treasury decisions.

---

## 🔴 Business Problem

| Pain Point | Impact |
|---|---|
| Financial data scattered across 5 separate bank sources | No unified view of total position |
| No centralized balance or liquidity visibility | Decision-making delayed |
| Exchange rates updated manually | Multi-currency reporting inconsistent |
| USD vs TZS exposure not quantified | Currency risk unmonitored |
| Manual consolidation every reporting cycle | High effort, high error rate |

---

## 🏗️ Architecture

```
  ┌──────┐  ┌──────┐  ┌──────┐  ┌──────┐  ┌──────┐
  │ ABSA │  │ CRDB │  │ SCB  │  │ NMB  │  │ ECO  │   ← Raw bank files
  └──┬───┘  └──┬───┘  └──┬───┘  └──┬───┘  └──┬───┘
     │         │          │         │          │
     ▼         ▼          ▼         ▼          ▼
 fnTransform fnTransform fnTransform fnTransform fnTransform
  ABSA        CRDB        SCB        NMB        ECO
     │         │          │         │          │       ← Custom M functions
     └─────────┴──────────┴─────────┴──────────┘         per bank source
                          │
                          ▼
               ┌─────────────────────┐
               │     Bank_Master      │   ← Consolidated clean table
               │  (15 queries total)  │
               └──────────┬──────────┘
                          │
          ┌───────────────┴──────────────┐
          │                              │
          ▼                              ▼
 ┌─────────────────┐          ┌──────────────────────┐
 │  Reference      │          │  Python Script        │
 │  Tables         │          │  Exchange Rate API    │
 │  · Bank Master  │          │  (USD ↔ TZS Live)     │
 │  · Call Deposit │          │  → Rate information   │
 │  · USD to TZS   │          │  → USD to TZS table   │
 └────────┬────────┘          └──────────┬────────────┘
          └──────────────┬───────────────┘
                         ▼
             ┌───────────────────────┐
             │   Star Schema Model   │
             │   Fact + Dimensions   │
             └───────────┬───────────┘
                         ▼
             ┌───────────────────────┐
             │       Power BI        │
             │  · Overview Dashboard │
             │  · Inflow / Outflow   │
             │  · DAX KPIs & Slicers │
             └───────────────────────┘
```

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| **Data Sources** | Excel / CSV files (ABSA, CRDB, ECO, NMB, SCB) |
| **Data Transformation** | Power Query (M Language) |
| **Exchange Rate Automation** | Python · Requests · Pandas |
| **External API** | Live forex API (USD ↔ TZS) |
| **Data Modeling** | Power BI (Star Schema) |
| **Visualization** | Power BI |
| **Analytics** | DAX (Data Analysis Expressions) |

---

## 💡 Solution Layers

### 🔹 Layer 1 — Data Consolidation (Power Query — 15 Queries)

The Power Query layer uses a **reusable function pattern** — one custom `fnTransform` function per bank source — keeping transformation logic DRY and easy to maintain.

**Complete Query List (as built in Power BI):**

| # | Query Name | Type | Purpose |
|---|---|---|---|
| 1 | `Bank Master` | Table | Master reference for all bank accounts |
| 2 | `Call Deposit & Restricted Cash` | Table | Tracks locked/restricted fund balances |
| 3 | `Rate information of TZS/USD...` | Table | Exchange rate dataset (Python output) |
| 4 | `USD to TZS` | Table | Converted currency lookup table |
| 5 | `CRDB` | Table | Transformed CRDB bank data |
| 6 | `fnTransformCRDB` | Function `fx` | Reusable cleansing function for CRDB |
| 7 | `ABSA` | Table | Transformed ABSA bank data |
| 8 | `fnTransformABSA` | Function `fx` | Reusable cleansing function for ABSA |
| 9 | `SCB` | Table | Transformed SCB bank data |
| 10 | `fnTransformSCB` | Function `fx` | Reusable cleansing function for SCB |
| 11 | `NMB` | Table | Transformed NMB bank data |
| 12 | `fnTransformNMB` | Function `fx` | Reusable cleansing function for NMB |
| 13 | `ECO` | Table | Transformed ECO bank data |
| 14 | `fnTransformECO` | Function `fx` | Reusable cleansing function for ECO |
| 15 | `Bank_Master` | Table | Final consolidated output table |

> All 15 queries load successfully with no errors — each bank table is fully transformed via its paired `fnTransform` function before being appended into `Bank_Master`.

**Pattern applied per bank source:**

```
Raw Bank File (ABSA / CRDB / SCB / NMB / ECO)
        │
        ▼
  fnTransform{Bank}     ← Custom M function: column rename,
        │                  type casting, null handling,
        │                  currency tagging, date formatting
        ▼
  Cleaned Bank Table
        │
        ▼
    Bank_Master          ← Appended consolidated output
```

**Why this pattern?**
- Each bank file has a different column structure — one function per source handles its specific quirks
- Changing cleansing logic for one bank only requires editing its `fnTransform` — no risk of breaking others
- `Bank_Master` is always the single clean output consumed by the data model

### 🔹 Layer 2 — Exchange Rate Automation (Python + API)

This is the key differentiator of the project. A Python script runs daily to:

```python
# Core workflow of the exchange rate automation script
# 1. Fetch live USD → TZS rate from forex API
# 2. Calculate reverse rate (TZS → USD)
# 3. Store with: Date | Currency Pair | Rate | API Timestamp
# 4. Check for duplicate entries before inserting
# 5. Maintain full historical exchange rate dataset
```

**Output dataset columns:**

| Column | Description |
|---|---|
| `Date` | Rate fetch date |
| `Currency Pair` | USD/TZS or TZS/USD |
| `Rate` | Exchange rate value |
| `API Timestamp` | Exact fetch time from API |

### 🔹 Layer 3 — Power BI Dashboard (Multi-Page)

Interactive, filterable reports across two focused dashboard pages covering overall position and detailed cash flow movement.

---

## 📊 Dashboard Breakdown

### 🟣 Page 1 — Overview Dashboard

**KPI Cards**

| KPI | Value |
|---|---|
| Total TZS | $3.06M |
| Closing Balance | $4.42M |
| Call Deposit | $14M |
| Restricted Cash | $1M |
| Total (USD) | $22.48M |
| Total Accounts | 11 |

**Visuals & Insights**

| Visual | Insight |
|---|---|
| Currency Proportion by Source | Identifies USD vs TZS dependency per bank |
| Call Deposit Distribution (Pie) | Shows where major deposits are concentrated |
| Bank Balance by Currency (Bar) | Compares currency exposure per source |
| Currency Distribution (Donut) | Overall USD vs TZS risk exposure |

---

### 📈 Page 2 — Inflow / Outflow Analysis

**Daily Trends**
- Line chart comparing daily inflows and outflows in both TZS and USD
- Highlights peak cash movement days and volatility patterns

**Monthly Trends**
- Source-wise monthly inflow and outflow breakdown
- Identifies highest-contributing sources and seasonal cash flow patterns

---

## 🗃️ Data Modeling

Designed using a **star schema** for Power BI performance:

```
                  ┌─────────────┐
                  │  dim_Date   │
                  └──────┬──────┘
                         │
┌──────────────┐   ┌─────▼──────────┐   ┌───────────────┐
│  dim_Source  ├───►  fact_Txn       ◄───┤ dim_Currency  │
│ (ABSA, CRDB) │   │  (Amount, Date │   │ (USD / TZS)   │
└──────────────┘   │   Type, Src)   │   └───────────────┘
                   └────────────────┘
```

**Fact Table — `fact_Transactions`**
- Amount, Date, Transaction Type (Inflow/Outflow), Source, Currency

**Dimension Tables**
- `dim_Date` — full calendar table with Year, Month, Quarter
- `dim_Source` — ABSA, CRDB, ECO, NMB, SCB with metadata
- `dim_Currency` — USD, TZS with live rate linkage

---

## ⚙️ Key Logic Implemented

### 1. Exchange Rate Conversion (Python + DAX)
- Python fetches live rate daily via API and stores history
- DAX uses the closest available rate for dynamic TZS ↔ USD conversion in all visuals

### 2. Closing Balance Logic
- Uses the **last available value** within the selected period — not a simple sum
- Ensures correct month-end and period-end balance reporting

### 3. Multi-Source Aggregation
- Consolidates all five bank sources while preserving drill-down to source level
- Avoids double-counting across sources with validated key logic

### 4. User Filtering
Dashboard supports dynamic filtering by:
- **Bank** (ABSA, CRDB, ECO, NMB, SCB)
- **Year-Month**
- **Currency** (USD / TZS)

---

## 🔍 Key Insights Generated

- Identified which bank sources hold the **maximum liquidity** at any given time
- Detected **imbalance between USD and TZS exposure** across sources
- Highlighted **peak inflow and outflow periods** for treasury planning
- Enabled accurate tracking of **restricted vs usable cash**
- Improved visibility into **call deposit concentration risk**

---

## 💼 Business Impact

| Outcome | Magnitude |
|---|---|
| Reduced manual reporting effort | 40–50% reduction |
| Multi-currency reporting accuracy | Consistent, API-driven, daily updated |
| Financial decision speed | Real-time dashboard vs. periodic reports |
| Data consolidation | 5 sources → 1 unified model |
| Exchange rate management | Fully automated, historical record maintained |

---

## 🧩 Challenges & Solutions

| Challenge | Solution |
|---|---|
| Multiple sources with different formats | Power Query standardization pipeline |
| Missing and inconsistent data | Cleansing logic with null handling and type enforcement |
| Correct closing balance calculation | DAX `LASTNONBLANK` / last-value logic per period |
| Accurate currency conversion | Python API script with daily historical storage |
| Duplicate exchange rate entries | Deduplication check before every API write |

---

## 🚀 Getting Started

### Prerequisites
- Power BI Desktop (latest)
- Python 3.8+
- Access to forex API (e.g., ExchangeRate-API, Open Exchange Rates)

### Python Setup

```bash
pip install requests pandas openpyxl
```

### Exchange Rate Script Configuration

```python
# config.py
API_KEY        = "your_api_key_here"
BASE_CURRENCY  = "USD"
TARGET_CURRENCY = "TZS"
OUTPUT_PATH    = "data/exchange_rates.csv"
```

```bash
python exchange_rate_fetcher.py
```

### Power BI Setup

1. Open `cashflow_dashboard.pbix` in Power BI Desktop
2. Update data source paths under **Transform Data → Data Source Settings**
3. Point to your local bank source files (ABSA, CRDB, ECO, NMB, SCB)
4. Set exchange rate file path to match `OUTPUT_PATH` above
5. Click **Refresh** — all pages update automatically

### Input File Requirements

Each bank source file must contain:

| Column | Description |
|---|---|
| `Date` | Transaction date |
| `Amount` | Transaction amount |
| `Type` | Inflow / Outflow |
| `Currency` | USD or TZS |
| `Source` | Bank identifier |

---

## 🔮 Future Enhancements

- [ ] **Predictive cash flow forecasting** using ML (Prophet / ARIMA) on historical inflow/outflow trends
- [ ] **Automated daily email reports** — scheduled Python job sends PDF snapshot to stakeholders
- [ ] **Multi-currency expansion** — support EUR, GBP, KES alongside USD and TZS
- [ ] **Alert system** — trigger notifications when closing balance falls below threshold
- [ ] **Direct bank API integration** — replace manual file uploads with live bank feed connections
- [ ] **Power BI Service deployment** — publish to cloud with scheduled refresh and row-level security (RLS)

---

## 👤 Author

**Aerunkar Abhishek**
Data & Analytics — Caterpillar Signs Private Limited

---

> *One dashboard. Five banks. Two currencies. Zero manual effort.*
