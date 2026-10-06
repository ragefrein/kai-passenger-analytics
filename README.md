# Indonesian Railway Passenger Analytics

An end-to-end analytics project on Indonesian railway passenger volumes, built on
official **BPS** (Badan Pusat Statistik / Statistics Indonesia) data. It covers
data cleaning, exploratory analysis, and **passenger forecasting through
December 2027** with uncertainty intervals.

The project ships two notebooks:

| Notebook | Purpose |
| --- | --- |
| `main.ipynb` | Data preparation and exploratory data analysis (EDA) |
| `forecast_2027.ipynb` | Forecasting (Sep 2026 - Dec 2027), growth analysis, and summary |

- GitHub repository: [ragefrein/kai-passenger-analytics](https://github.com/ragefrein/kai-passenger-analytics)
- BPS data source: [Railway Passenger Data](https://www.bps.go.id/id/statistics-table/2/NzIjMg==/jumlah-penumpang-kereta-api.html)

---

## Table of Contents

1. [Project Overview](#project-overview)
2. [Dataset](#dataset)
3. [EDA Summary](#eda-summary)
4. [Forecasting (Sep 2026 - Dec 2027)](#forecasting-sep-2026---dec-2027)
5. [Forecast Results](#forecast-results)
6. [Growth Analysis](#growth-analysis)
7. [Output Schema](#output-schema)
8. [Project Structure](#project-structure)
9. [Tech Stack](#tech-stack)
10. [Setup](#setup)
11. [How to Run](#how-to-run)
12. [Visualization Outputs](#visualization-outputs)
13. [Assumptions and Limitations](#assumptions-and-limitations)
14. [Attribution](#attribution)

---

## Project Overview

The BPS source data is published as one CSV file per year. The first three rows of
each file hold metadata (including the reporting year); the remaining rows list
passenger categories with monthly values (January-December) plus an annual column.

`main.ipynb` loads every yearly file, extracts the year from the metadata, drops
the redundant annual column, and reshapes the data from **wide to long** format.
During cleaning the `-` marker becomes `0`, and passenger counts are coerced to
numeric. Month names are ordered chronologically and combined with the year to
build a proper time-series date column (`Tanggal`).

`forecast_2027.ipynb` then consumes the processed dataset to produce monthly
forecasts, prediction intervals, a backtested model choice per category, and a
plain-language growth summary.

### Goals

- Understand how railway passenger volumes evolve over time.
- Compare monthly and annual patterns across passenger categories.
- Convert public BPS data into a tidy, analysis-friendly format.
- Produce clear, readable visualizations for exploration.
- Forecast passenger volumes for the months not yet published (Sep 2026 - Dec
  2027) with 80% and 95% prediction intervals.

---

## Dataset

- Combined dataset: **1,188 rows** spanning **2016-2026** (`data/combined_data.csv`).
- Raw values are reported in **thousands of people**; forecasts and charts use
  **millions of people** (converted by dividing by 1,000).
- Observations end in **August 2026**; September-December 2026 are not yet
  published and are forecast by the notebook.
- Categories:

| Category | Notes |
| --- | --- |
| `Jabodetabek` | Component |
| `Non Jabodetabek (Jawa)` | Component |
| `Jawa (Jabodetabek+Non Jabodetabek)` | **Aggregate** |
| `Non Jawa (Sumatera + Sulawesi)` | Component |
| `Kereta Bandara` (Airport Railway) | Starts 2024 |
| `MRT` | Starts 2024 |
| `LRT` | Starts 2024 |
| `Kereta cepat (Whoosh)` | Starts 2024 |
| `Total` | **Aggregate** |

> **Important:** `Jawa (...)` and `Total` are aggregate categories. Never add them
> to their component categories, which would double-count passengers.

---

## EDA Summary

- **July** records the highest combined monthly value among non-`Total`
  categories.
- The aggregate **`Jawa (Jabodetabek+Non Jabodetabek)`** category holds the
  largest cumulative volume.
- Annual volumes collapsed during **2020-2021** (COVID-19) and recovered in the
  following years through 2025.
- **2026 is a partial year** (data stops in August) and must not be compared
  directly with complete years.

These observations reflect the recorded BPS values, not independent measurements.

---

## Forecasting (Sep 2026 - Dec 2027)

`forecast_2027.ipynb` estimates monthly passenger volumes from **September 2026
through December 2027** (16 months) for **all nine categories**, plus annual
totals and uncertainty intervals.

### Approach

- **Training window:** 2022-2026 (post-COVID regime). The 2020-2021 shock is
  excluded because it is a structural break.
- **Candidate models**, compared with expanding-origin backtesting (12-month
  horizon): Seasonal Naive, ETS (Holt-Winters), SARIMA, and HistGradientBoosting
  with seasonal lag features.
- **Model selection:** the lowest backtest **MAPE per category** wins, so
  different categories may use different models.
- **Intervals:** 80% and 95% prediction intervals; all forecasts are clipped to
  be non-negative.
- **Robustness:** SARIMA rejects non-convergent fits and implausible forecasts;
  ETS falls back to a non-seasonal fit when fewer than two full cycles exist.

### Selected model per category

| Category | Model | Backtest MAPE |
| --- | --- | --- |
| `Jabodetabek` | SARIMA | 3.03% |
| `Jawa (Jabodetabek+Non Jabodetabek)` | SARIMA | 3.02% |
| `Kereta Bandara` | ML | 4.07% |
| `Kereta cepat (Whoosh)` | ML | 7.03% |
| `LRT` | ETS | 5.24% |
| `MRT` | ETS | 6.99% |
| `Non Jabodetabek (Jawa)` | SARIMA -> Seasonal Naive | 6.01% |
| `Non Jawa (Sumatera + Sulawesi)` | SARIMA -> Seasonal Naive | 6.86% |
| `Total` | ETS | 10.10% |

---

## Forecast Results

Headline numbers for the `Total` category (million passengers):

| Item | Value |
| --- | --- |
| Observed Jan-Aug 2026 | **384.74** |
| Forecast Sep-Dec 2026 | **199.50** (80%: 192.55 - 206.45) |
| **Estimated full-year 2026** | **584.23** |
| **Forecast full-year 2027** | **613.88** |
| 2027 - 80% interval | 593.03 - 634.73 |
| 2027 - 95% interval | 581.99 - 645.77 |
| Expected growth 2026 -> 2027 | **+5.07%** |

Annual 2027 forecast per category (million passengers):

| Category | 2027 forecast |
| --- | --- |
| `Total` | 613.88 |
| `Jawa (Jabodetabek+Non Jabodetabek)` | 484.38 |
| `Jabodetabek` | 386.89 |
| `Non Jabodetabek (Jawa)` | 103.86 |
| `MRT` | 51.98 |
| `LRT` | 41.77 |
| `Kereta Bandara` | 9.35 |
| `Non Jawa (Sumatera + Sulawesi)` | 7.70 |
| `Kereta cepat (Whoosh)` | 6.17 |

---

## Growth Analysis

To make the trajectory easy to read, `forecast_2027.ipynb` includes a dedicated
growth section:

- **Annual totals 2022-2027** (observed vs estimated/forecast) with the
  year-over-year growth rate overlaid.
- **Growth 2026 to 2027 by category** - estimated full-year 2026 versus the 2027
  forecast.
- **2027 monthly forecast** with month-over-month change.
- A **written summary** explaining the observed Jan-Aug 2026 data, the forecast
  Sep-Dec 2026, the estimated full-year 2026, and the 2027 forecast.

Estimated full-year 2026 vs 2027 forecast by category (million passengers):

| Category | Observed 2026 (Jan-Aug) | Forecast 2026 (Sep-Dec) | Est. 2026 (full) | Forecast 2027 | Growth |
| --- | --- | --- | --- | --- | --- |
| `Total` | 384.74 | 199.50 | 584.23 | 613.88 | +5.07% |
| `Jawa (Jabodetabek+Non Jabodetabek)` | 311.36 | 160.53 | 471.89 | 484.38 | +2.65% |
| `Jabodetabek` | 240.60 | 127.96 | 368.56 | 386.89 | +4.97% |
| `Non Jabodetabek (Jawa)` | 70.76 | 33.10 | 103.86 | 103.86 | 0.00% |
| `MRT` | 31.98 | 17.42 | 49.40 | 51.98 | +5.21% |
| `LRT` | 25.74 | 13.77 | 39.51 | 41.77 | +5.73% |
| `Kereta Bandara` | 6.25 | 3.12 | 9.36 | 9.35 | -0.18% |
| `Non Jawa (Sumatera + Sulawesi)` | 5.32 | 2.38 | 7.70 | 7.70 | 0.00% |
| `Kereta cepat (Whoosh)` | 4.08 | 2.06 | 6.13 | 6.17 | +0.51% |

The fastest-growing category is **LRT (+5.73%)**. Note that the estimated
full-year 2026 is a *blend* of observed (Jan-Aug) and forecast (Sep-Dec) months,
so the growth figure inherits the uncertainty of four forecast months; treat it
as indicative.

---

## Output Schema

The per-category forecast is written to `data/forecast_2027.csv`
(**144 rows** = 9 categories x 16 months):

| Column | Description |
| --- | --- |
| `Kategori` | Service category |
| `Tahun` | 2026 (Sep-Dec) or 2027 |
| `Bulan` | Month name (Indonesian) |
| `Bulan_num` | Month number 1-12 |
| `Tanggal` | Date, 2026-09-01 ... 2027-12-01 |
| `Ramalan` | Point forecast (million passengers) |
| `Batas_Bawah_80` / `Batas_Atas_80` | 80% interval bounds |
| `Batas_Bawah_95` / `Batas_Atas_95` | 95% interval bounds |
| `Model` | Selected model for the category |
| `MAPE_uji` | Backtest MAPE (%) |

The processed EDA dataset (`data/combined_data.csv`) uses these columns:

| Column | Description |
| --- | --- |
| `Kategori` | Railway region or service category |
| `Bulan` | Month name in Indonesian |
| `Jumlah Penumpang` | Passenger count in thousands of people |
| `Tahun` | Observation year |
| `Bulan_num` | Month number from 1 to 12 |
| `Tanggal` | Date value used for time-series visualizations |

---

## Project Structure

```text
.
├── data/                         # Local BPS CSV files; ignored by Git
│   ├── Jumlah Penumpang Kereta Api, 2016.csv
│   ├── ...
│   ├── Jumlah Penumpang Kereta Api, 2026.csv
│   ├── combined_data.csv         # Processed tidy dataset
│   └── forecast_2027.csv         # Monthly forecast output (Sep 2026 - Dec 2027)
├── main.ipynb                    # EDA notebook
├── forecast_2027.ipynb           # Forecasting + growth notebook
├── penumpang_kereta.png          # Passenger visualization
├── tren_bulanan_kereta.png       # Monthly trend visualization
├── tren_tahunan_kereta.png       # Annual trend visualization
├── forecast_2027.png             # Forecast visualization (3x3 categories)
├── .gitignore
└── README.md
```

The local CSV files (including `data/`) are intentionally excluded from version
control by `.gitignore`. Place the source files in `data/` before running.

---

## Tech Stack

- Python 3.10 or newer (developed on 3.12)
- Jupyter Notebook or JupyterLab
- pandas, NumPy
- Matplotlib, Seaborn
- Plotly (interactive charts)
- statsmodels (ETS, SARIMA, STL, ADF)
- scikit-learn (HistGradientBoosting forecast)

---

## Setup

### 1. Clone the repository

```powershell
git clone https://github.com/ragefrein/kai-passenger-analytics.git
cd kai-passenger-analytics
```

### 2. Create a virtual environment

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
```

If PowerShell blocks script activation, allow it for the current session only:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
```

### 3. Install dependencies

```powershell
python -m pip install --upgrade pip
python -m pip install pandas numpy matplotlib seaborn plotly statsmodels scikit-learn jupyter ipykernel
```

Register the environment as a Jupyter kernel:

```powershell
python -m ipykernel install --user --name kai-passenger-analytics --display-name "Python (kai-passenger-analytics)"
```

---

## How to Run

1. Place all BPS yearly CSV files in `data/`.
2. Run `main.ipynb` top to bottom to build `data/combined_data.csv`.
3. Run `forecast_2027.ipynb` top to bottom to produce the forecast, charts, and
   `data/forecast_2027.csv`.

Both notebooks expect the working directory to be the project root (where
`data/` lives), because they reference relative paths such as
`data/combined_data.csv` and write `forecast_2027.png`.

`main.ipynb` discovers source files with:

```python
glob.glob("data/*.csv")
```

### Reproducibility note

The EDA cells build `list_data`, while the visualization cells use `data_utama`.
When starting from a fresh kernel, make sure the combined dataframe is created
before the visualization cells run:

```python
data_utama = pd.concat(list_data, ignore_index=True)
data_utama["Bulan_num"] = data_utama["Bulan"].map(month_map)
data_utama["Tanggal"] = pd.to_datetime(
    data_utama["Tahun"].astype(str)
    + "-"
    + data_utama["Bulan_num"].astype(str)
    + "-01"
)
```

Variables from a previous kernel session are not guaranteed to exist after a
restart.

---

## Visualization Outputs

Static images:

- `tren_bulanan_kereta.png` - monthly passenger comparison by category and year.
- `tren_tahunan_kereta.png` - annual passenger trend summary.
- `penumpang_kereta.png` - passenger dynamics across the observation period.
- `forecast_2027.png` - 3x3 panel of observed history and forecast per category.

Interactive Plotly charts (rendered in the notebooks):

- `main.ipynb` - exploratory time-series charts.
- `forecast_2027.ipynb` - headline `Total` forecast with intervals, per-category
  panels, annual totals with growth rate, per-category growth, and monthly
  month-over-month change.

---

## Assumptions and Limitations

- The **post-COVID training window (2022-2026)** is short, so prediction intervals
  can be wide.
- **September-December 2026 are forecasts, not observations.** The estimated
  full-year 2026 is a blend and should not be treated as final data.
- **Short-lived categories** (Airport Railway, MRT, LRT, Whoosh, starting 2024)
  have few backtest folds and the least reliable estimates.
- **Annual intervals** are the sum of monthly bounds; this ignores cross-month
  correlation, so the true range may be wider.
- The models capture historical trend and seasonality only - **not** fare policy,
  new routes, or external shocks. Treat the output as an indicative estimate.
- `Total` and `Jawa (...)` are aggregates; never sum them with their components.

---

## Attribution

This project is intended for analysis and learning. The data is sourced from
**BPS (Badan Pusat Statistik)**. Please retain the source attribution and follow
the terms applicable to the official BPS data.