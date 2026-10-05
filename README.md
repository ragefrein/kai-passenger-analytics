# Indonesian Railway Passenger Analytics

An exploratory data analysis project that examines railway passenger trends in Indonesia using official data from Statistics Indonesia (Badan Pusat Statistik, or BPS). The project transforms monthly BPS data from yearly CSV files into a structured dataset and presents the results through summaries and visualizations.

## Project Links

- GitHub repository: [ragefrein/kai-passenger-analytics](https://github.com/ragefrein/kai-passenger-analytics)
- BPS data source: [Railway Passenger Data](https://www.bps.go.id/id/statistics-table/2/NzIjMg==/jumlah-penumpang-kereta-api.html)

## Project Overview

The BPS source data is provided as one CSV file per year. The first three rows of each file contain metadata; the following rows contain passenger categories, monthly values from January to December, and an annual column.

`main.ipynb` loads all yearly files, extracts the year from each file's metadata, removes the unused annual column, and converts the data from wide format to long format. During cleaning, the `-` marker is treated as `0`, and passenger counts are converted to numeric values. The month names are ordered chronologically and combined with the year to create a time-series date column.

## Data and Analysis Summary

- The combined dataset contains **1,188 rows** covering **2016-2026**.
- Passenger counts are reported in **thousands of people**.
- Available categories include Jabodetabek, Non-Jabodetabek Java, Java, Non-Java, Airport Railway, MRT, LRT, Whoosh high-speed railway, and Total.
- **July** has the highest combined monthly passenger value among non-`Total` categories in the available dataset.
- The aggregate **Java (Jabodetabek + Non-Jabodetabek)** category has the largest cumulative value.
- Annual values decline during 2020-2021 and increase again in the following years through 2025.
- **2026 should be treated as a partial year** if all months are not yet available, so it should not be compared directly with complete years.

These observations reflect the values recorded in the BPS table. `Java (Jabodetabek + Non-Jabodetabek)` and `Total` are aggregate categories; they should not be added to their component categories when calculating unique passenger totals.

## Project Goals

- Understand how railway passenger volumes change over time.
- Compare monthly patterns across passenger categories.
- Convert public BPS data into a more analysis-friendly format.
- Produce clear visualizations for exploration and data analysis learning.

## Analysis Features

- Read all yearly CSV files from the `data/` directory.
- Extract the year from each source file's metadata.
- Reshape monthly data from wide format to long format.
- Replace `-` values with `0` and convert passenger counts to numeric values.
- Order months from January through December.
- Compare monthly trends by category and year.
- Create static visualizations with Matplotlib and Seaborn.
- Create interactive time-series visualizations with Plotly.

## Project Structure

```text
.
├── data/                         # Local BPS CSV files; ignored by Git
│   ├── Jumlah Penumpang Kereta Api, 2016.csv
│   ├── Jumlah Penumpang Kereta Api, 2017.csv
│   ├── ...
│   ├── Jumlah Penumpang Kereta Api, 2026.csv
│   └── combined_data.csv
├── main.ipynb                    # Main analysis notebook
├── penumpang_kereta.png          # Passenger visualization
├── tren_bulanan_kereta.png       # Monthly trend visualization
├── tren_tahunan_kereta.png       # Annual trend visualization
├── .gitignore
└── README.md
```

The local CSV files are intentionally excluded from version control by `.gitignore`. Download or place the source files in `data/` before running the notebook.

## Technology Stack

- Python 3.10 or newer
- Jupyter Notebook or JupyterLab
- pandas
- Matplotlib
- Seaborn
- Plotly

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

If PowerShell blocks script activation, use the following command for the current session:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
```

### 3. Install dependencies

```powershell
python -m pip install --upgrade pip
python -m pip install pandas matplotlib seaborn plotly jupyter ipykernel
```

Register the environment as a Jupyter kernel:

```powershell
python -m ipykernel install --user --name kai-passenger-analytics --display-name "Python (kai-passenger-analytics)"
```

## Run the Notebook

1. Open `main.ipynb` in VS Code or JupyterLab.
2. Select the `Python (kai-passenger-analytics)` kernel.
3. Set the working directory to the project root, where `main.ipynb` and `data/` are located.
4. Run the cells from top to bottom.

The notebook searches for source files with:

```python
glob.glob("data/*.csv")
```

Keep the `data/` directory in the project root so the source files can be found.

## Processed Data Format

The processed dataset uses the following columns:

| Column             | Description                                    |
| ------------------ | ---------------------------------------------- |
| `Kategori`         | Railway region or service category             |
| `Bulan`            | Month name in Indonesian                       |
| `Jumlah Penumpang` | Passenger count in thousands of people         |
| `Tahun`            | Observation year                               |
| `Bulan_num`        | Month number from 1 to 12                      |
| `Tanggal`          | Date value used for time-series visualizations |

## Visualization Outputs

- `tren_bulanan_kereta.png`: monthly passenger comparison by category and year.
- `tren_tahunan_kereta.png`: annual passenger trend summary.
- `penumpang_kereta.png`: passenger dynamics across the observation period.

The interactive Plotly chart is displayed directly in the notebook output.

## Reproducibility Note

The processing cells create `list_data`, while the visualization cells use `data_utama` and `Tanggal`. When starting from a fresh kernel, make sure the combined dataframe is created before running the visualization cells:

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

This is necessary because variables from a previous notebook kernel session are not guaranteed to exist after reopening or restarting the notebook.

## Attribution

This project is intended for analysis and learning. The data is sourced from BPS. Please retain the source attribution and follow the terms applicable to the official BPS data.
