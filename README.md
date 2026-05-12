# Bsc_Thesis: Spatiotemporal Variation of Glacial Snowline Altitudes in the Caucasus

This repository contains all Jupyter notebooks for the data analysis conducted as part of my Bachelor's thesis. The project reconstructs and analyses long-term Snowline Altitude (SLA) trends across 7 glaciers in the Caucasus using a fusion of Landsat/Sentinel-2 and MODIS satellite data spanning from 2000-2025.

---

## Study Glaciers

The analysis covers the following seven glaciers:

Ulluchiran, Lekziri, Caneri, Bezengi, Dyh-Cu, Agashtan, Karaugom

---

## Repository Structure

```
Bsc_Thesis/
├── notebooks/
│   ├── 01_LandsatSentinel_Preprocessing.ipynb
│   ├── 02_MODIS_Preprocessing.ipynb
│   ├── 03_Weighted_Regression.ipynb
│   ├── 04_LS_MODIS_combine.ipynb
│   ├── 05_Trend_Analysis.ipynb
│   └── Figures.ipynb
├── results/
│   ├── Weighted_regression/
│   ├── Annual_Trend_Analysis/
│   └── ...
├── data/                  # Not included (see Data section below)
│   ├── LS_Data/
│   │   ├── raw_sla/
│   │   └── weighted_smoothed/
│   ├── MODIS_data/
│   │   ├── raw_fulltime/
│   │   ├── weighted_data/
│   │   └── predicted_sla/
│   └── LS_MODIS_combined/
│       └── combination_post2017/
├── .gitignore
└── README.md
```

---

## Analysis Workflow

The notebooks are designed to be run sequentially. Each notebook feeds into the next, forming a complete pipeline from raw satellite data to long-term trend analysis.

### Notebook 01 — Landsat & Sentinel-2 Preprocessing
**`01_LandsatSentinel_Preprocessing.ipynb`**

Cleans and smooths raw SLA data derived from Landsat and Sentinel-2 imagery, originally extracted via Google Earth Engine.

- Loads per-glacier CSV files and filters for the summer season (May–October, 2000–2025)
- Assigns a **classification weight** based on `percentage_classified` (penalising images with less than 70% clear pixels)
- Assigns a **velocity weight** based on the rate of SLA change between consecutive images (penalising jumps above 40 m/day)
- Combines both weights into a single `LS_weight_final` (70% classification, 30% velocity)
- Compares **row-based (5-image)** vs. **time-aware (20-day)** smoothing methods, both weighted and unweighted
- Exports the final processed datasets to `data/LS_Data/weighted_smoothed/`

---

### Notebook 02 — MODIS Preprocessing
**`02_MODIS_Preprocessing.ipynb`**

Processes raw MODIS Near-Infrared (NIR) data for each glacier.

- Loads MODIS Terra NIR observations and filters for summer months (May–October)
- Assigns a **zenith angle weight** (penalising images with sensor zenith > 50°)
- Assigns a **clear pixel weight** (penalising images with less than 70% clear pixels)
- Combines both weights multiplicatively into `MODIS_weight_final`
- Tests three time-aware smoothing windows (7, 14 and 21 days) and exports the results to `data/MODIS_data/weighted_data/`

---

### Notebook 03 — Weighted Regression
**`03_Weighted_Regression.ipynb`**

Establishes a statistical relationship between MODIS NIR values and Landsat/Sentinel-2 SLA observations, used to reconstruct SLA from MODIS data.

- Merges MODIS and Landsat/Sentinel-2 datasets
- Excludes low-quality observations (weight < 0.2 for either sensor)
- Fits both an **OLS (Ordinary Least Squares)** and a **WLS (Weighted Least Squares)** regression per glacier, using `LS_weight_final` as regression weights
- Applies the fitted WLS model to the full MODIS time series to generate a continuous **SLA proxy** (`predicted_SLA_smoothed_WLS`)
- Exports regression summaries to `results/Weighted_regression/` and predicted SLA datasets to `data/MODIS_data/predicted_sla/`

---

### Notebook 04 — Data Fusion: Landsat/Sentinel-2 & MODIS
**`04_LS_MODIS_combine.ipynb`**

Combines both data sources into a single, continuous 26-year SLA timeline per glacier.

- Uses only MODIS-derived SLA before 2017 (limited Landsat/Sentinel-2 availability)
- After 2017, merges both datasets with Landsat/Sentinel-2 taking priority on co-observation days (higher spatial resolution)
- Quantifies the alignment between MODIS and Landsat/Sentinel-2 by computing the **Mean Absolute Difference (MAD)** on overlapping days
- Anchors seasonal boundaries: assigns the annual minimum SLA to May 1st and October 31st (assumed full snow cover)
- Exports the final composite time series to `data/LS_MODIS_combined/combination_post2017/`

---

### Notebook 05 — Trend Analysis
**`05_Trend_Analysis.ipynb`**

Detects and quantifies long-term SLA trends across all seven glaciers.

- Interpolates each composite time series to a consistent daily resolution (May–October)
- Calculates **annual mean SLA** for each glacier and fits a linear regression to estimate the long-term trend (m/yr)
- Computes **monthly trends** (May through October separately) to identify intra-seasonal variation
- Plots annual SLA anomalies for geographically close glaciers (Bezengi, Caneri, Dyh-Cu) to reveal regional signals
- Exports annual means and monthly trend statistics to `data/Trend_Analysis/`

---

### Notebook 06 — Figures
**`Figures.ipynb`**

Generates all publication-quality figures used in the thesis and presentation.

- Weighted regression scatter plots 
- Multi-glacier regression appendix 
- Seasonal SLA time series plots for individual glaciers and years

---

## Data

> **The `data/` folder is not included in this repository** (added to `.gitignore` due to file size).

To replicate the full analysis, you will need to set up the following directory structure and populate it with the corresponding raw data:

```
data/
├── LS_Data/
│   └── raw_sla/
│       └── SLA_<glacier_name>.csv      # Landsat/Sentinel-2 SLA data from Google Earth Engine
├── MODIS_data/
│   └── raw_fulltime/
│       └── Glacier_NIR_Terra_SCAF_<glacier_name>.csv   # MODIS NIR data from Google Earth Engine
```

The raw satellite data was extracted from **Google Earth Engine**. Please contact the author if you need assistance accessing the original datasets.

---

## Requirements

Install the required Python packages with:

```bash
pip install pandas numpy scipy statsmodels plotly matplotlib
```

Or if you are using a `conda` environment:

```bash
conda install pandas numpy scipy statsmodels plotly matplotlib
```

**Python version:** 3.9+

---

## How to Run

1. Clone the repository:
   ```bash
   git clone https://github.com/sruetimann/Bsc_Thesis.git
   cd Bsc_Thesis
   ```

2. Set up the `data/` directory with the required raw datasets (see [Data](#data) section above).

3. Run the notebooks **in order** from inside the `notebooks/` directory:
   ```
   01_LandsatSentinel_Preprocessing.ipynb
   02_MODIS_Preprocessing.ipynb
   03_Weighted_Regression.ipynb
   04_LS_MODIS_combine.ipynb
   05_Trend_Analysis.ipynb
   Figures.ipynb          ← run last, reads from results/
   ```

> **Note:** Notebooks 03 onward depend on the output files generated by earlier notebooks. Running them out of order will result in missing file errors.

---

## Results

Key outputs are stored in the `results/` folder:

- `results/Weighted_regression/` — Regression comparison tables (OLS vs WLS R², slope, intercept) and scatter plots
- `results/Annual_Trend_Analysis/` — Annual SLA means, trend slopes (m/yr), p-values, and regional anomaly plots

---

## Author

**Sophie Rütimann**  
Bachelor's Thesis — 2026
