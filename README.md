# 🌀 Nonparametric Actigraphy Clustering

A modular Python pipeline for **extracting, contextualising, and clustering** circadian and sleep features from wrist actigraphy data. Developed for a study of blind adults living near the equator in Brazil, and readily adaptable to other actigraphy datasets.

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.18922404.svg)](https://doi.org/10.5281/zenodo.18922404)
[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/Pugliane/nonparametric-actigraphy-clustering/blob/main/nonparametric_actigraphy_clustering.ipynb)
[![bioRxiv](https://img.shields.io/badge/bioRxiv-2026.03.19.712663-b31b1b)](https://doi.org/10.64898/2026.03.19.712663)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](./LICENSE)
[![Python 3.11](https://img.shields.io/badge/python-3.11-blue.svg)](https://www.python.org/)

---

## 📖 About

This repository contains three integrated Jupyter notebooks developed for the study:

> **Pugliane, K. C., França, L. G. S., Leocadio-Miguel, M., & Araújo, J. F. (2026).**  
> *Low-latitude environmental regularity sustains non-photic entrainment in blind adults.*  
> bioRxiv. https://doi.org/10.64898/2026.03.19.712663

The pipeline moves from **geospatial and environmental context** → **actigraphy feature extraction** → **unsupervised clustering with SHAP-based explainability**, producing publication-ready figures and statistical outputs throughout.

![Key results figure](Highlights_RA_IS_QPActivity_LRI_L5_layout221.png)

---

## 🔬 Study Overview

Most research on circadian rhythms in blind individuals has been conducted at high latitudes (North America, Europe), where pronounced seasonal variation in photoperiod may compound the loss of photic input. This study asked whether the exceptionally stable environmental conditions near the equator (~5°S, Rio Grande do Norte, Brazil) could sustain circadian entrainment even in the absence of light perception.

**58 blind adults** (21–77 years; 43.1% female) wore wrist actigraphy continuously for **four weeks**. Pupillary light reflex (PLR) was used as a proxy for intact ipRGC photoreception (22 reactive, 36 non-reactive). A semi-supervised machine learning approach — combining PCA, KMeans clustering, and SHAP feature importance — was applied to 23 nonparametric actigraphy variables.

Two distinct circadian phenotypes emerged:

| Phenotype | n | % | Key features |
|-----------|---|---|--------------|
| 🟢 **Higher Circadian Stability (HCS)** | 42 | 72% | High RA, IS, QPActivity, LRI; consolidated rhythms |
| 🔴 **Lower Circadian Stability (LCS)** | 16 | 28% | Fragmented rhythms, lower regularity and amplitude |

Notably, **64% of PLR-non-reactive individuals** (23 of 36) fell in the HCS group — approximately **1.6× higher** than previously reported for blind cohorts at higher latitudes. Cluster membership explained substantially more variance in circadian metrics than PLR status alone (e.g., RA: R² = 0.61 for cluster vs. R² = 0.15 for PLR).

---

## 🗂️ Pipeline Overview

| # | Notebook | Description |
|---|----------|-------------|
| 1 | `geographical_and_seasonal_context_of_study_participants.ipynb` | Maps participant location (Rio Grande do Norte, Brazil), estimates photoperiod using `astral`, and retrieves NASA POWER temperature/radiation data |
| 2 | `nonparametric_actigraphy_feature_extraction.ipynb` | Loads actigraphy files, applies manually defined off-wrist masks, and extracts nonparametric circadian/sleep variables using `pyActigraphy` |
| 3 | `nonparametric_actigraphy_clustering.ipynb` | Z-score normalisation → multicollinearity screening → PCA → KMeans clustering → Random Forest + SHAP feature importance → between-group comparisons and regression |

---

## ✨ Features

- 🗺️ **Geospatial figures** — maps of Brazil and Rio Grande do Norte with participant location, using `geopandas` and `geobr`
- ☀️ **Photoperiod & temperature panels** — seasonal variation estimated from `astral` + NASA POWER API
- 🏃 **Actigraphy feature extraction** — 24 nonparametric metrics via `pyActigraphy`, with manual off-wrist masking applied per participant
- 🧬 **Multiple circadian domains** — activity, light, temperature, and sleep-derived variables
- 🤖 **Unsupervised clustering** — PCA dimensionality reduction + KMeans with silhouette-based optimisation (k = 2–10 evaluated)
- 🔍 **Explainability** — SHAP values (via Random Forest classifier) identify which features most distinguish each cluster
- 📊 **Publication-ready outputs** — figures, Excel/CSV exports, regression tables

---

## 📚 Data Dictionary

### 🗃️ Raw Data Sources

| Source | Description | Format |
|--------|-------------|--------|
| **Wrist actigraphy** | Participants wore an **ActTrust 1** device (Condor Instruments, São Paulo, Brazil) on the non-dominant wrist continuously for four weeks. The device simultaneously records **triaxial wrist activity** (counts/epoch), **ambient light exposure** (lux), and **peripheral skin temperature** (°C) at a fixed epoch resolution. Raw files follow the naming pattern `*_LogTAT_conferido.txt`. Valid days required ≥ 16 h of wear. | `.txt` (tab-delimited) |
| **Off-wrist mask database** | A manually curated CSV listing the participant ID and time intervals during which the device was removed (bathing, high-impact activities). Applied before any metric computation in notebook 2. | `.csv` |
| **Pupillary Light Reflex (PLR)** | Binary classification (reactive / non-reactive) based on pupillary constriction to a 600-lumen white LED flashlight assessed in a darkened room. Used as a proxy for intact ipRGC-mediated non-image-forming photoreception. Stored as a column in the participant metadata table: `1 = reactive`, `0 = non-reactive`. | Column in participant spreadsheet |
| **Geospatial data** | Official municipal boundary polygons for Brazil and Rio Grande do Norte, accessed via the `geobr` package (source: IBGE). Municipality polygon centroids used as geographic reference points for all environmental calculations. | GeoDataFrame (in-memory, notebook 1) |
| **Environmental data** | Daily meteorological records (air temperature, surface solar radiation) retrieved from the **NASA POWER Project API** (`https://power.larc.nasa.gov`) per municipality centroid. Sunrise and sunset times computed from centroid coordinates using the `astral` library. During data collection, mean photoperiod was 12.11 ± 0.23 h (annual amplitude ~44 min) and mean air temperature was 27.0 ± 1.1 °C. | JSON → DataFrame (notebook 1) |

---

### 📐 Derived Actigraphy Metrics

All metrics below are computed from the raw actigraphy signals using `pyActigraphy`. Variables are extracted per participant across the full valid recording period.

### Stability

| Variable | Abbreviation | Description | Reference |
|----------|-------------|-------------|-----------|
| Interdaily Stability | **IS** | Regularity and synchronisation of the activity pattern with the 24-h light–dark cycle across days; higher = more stable | Witting et al., 1990 |
| Sleep Regularity Index | **SRI** | Day-to-day consistency of sleep–wake timing; higher = more regular | Sletten et al., 2023 |

### Fragmentation

| Variable | Abbreviation | Description | Reference |
|----------|-------------|-------------|-----------|
| Intradaily Variability | **IV** | Frequency of transitions between rest and activity; higher = more fragmented | Witting et al., 1990 |
| Rest-to-Activity probability | **kRA** | Probability of switching from rest to activity | Lim et al., 2011 |
| Activity-to-Rest probability | **kAR** | Probability of switching from activity to rest | Lim et al., 2011 |
| Fraction of Sleep over Daytime | **FSoD** | Proportion of daytime epochs classified as sleep; higher = more daytime sleep | David et al., 2022 |

### Amplitude

| Variable | Abbreviation | Description | Reference |
|----------|-------------|-------------|-----------|
| Relative Amplitude | **RA** | Contrast between the most active 10-h period (M10) and least active 5-h period (L5); range 0–1, higher = greater amplitude | Van Someren et al., 1999 |
| Most Active 10-h Mean | **M10** | Mean activity level during the 10 most active consecutive hours | Gonçalves et al., 2015 |
| Least Active 5-h Mean | **L5** | Mean activity level during the 5 least active consecutive hours | Gonçalves et al., 2015 |
| Average Daily Activity Total | **ADAT** | Overall mean daily activity level | Buchman et al., 2012 |

### Phase Markers

| Variable | Abbreviation | Description | Reference |
|----------|-------------|-------------|-----------|
| Sleep Midpoint | **SleepMidpoint** | Average clock time of the midpoint of the main sleep episode (minutes from midnight) | Forbes et al., 2012 |
| M10 Onset | **M10o** | Clock time marking the start of the 10-h most active window (minutes from midnight) | Boudebesse et al., 2014 |
| L5 Onset | **L5o** | Clock time marking the start of the 5-h least active window (minutes from midnight) | Boudebesse et al., 2014 |
| Center of Gravity | **CoG** | Average timing of activity accumulation; a phase orientation marker (hours) | Kenagy, 1980 |

### Light Exposure

| Variable | Abbreviation | Description | Reference |
|----------|-------------|-------------|-----------|
| Light Regularity Index | **LRI** | Day-to-day regularity of light exposure patterns; higher = more regular | Hand et al., 2023 |
| M10 Onset (light) | **M10olight** | Clock time marking the onset of maximum 10-h light exposure (minutes from midnight) | Hammad et al., 2024 |
| L5 Onset (light) | **L5olight** | Clock time marking the onset of minimum 5-h light exposure (minutes from midnight) | Hammad et al., 2024 |

### Frequency Domain (Activity, Light & Temperature)

| Variable | Abbreviation | Description | Reference |
|----------|-------------|-------------|-----------|
| QP Statistic | **QPActivity / QPLight / QPTemperature** | Rhythm prominence (strength) based on the chi-square periodogram; higher = stronger rhythm | Sokolove & Bushell, 1978 |
| Period Estimate | **PeriodActivity / PeriodLight / PeriodTemperature** | Period length (minutes) corresponding to the dominant QP peak | Sokolove & Bushell, 1978 |

> **Note on variable exclusion:** AUC (area under the activity curve) was computed but excluded prior to clustering due to high redundancy with ADAT (Pearson r > 0.85, VIF > 5.0). The final clustering used 23 variables.

---

## 📥 Input Data Format

The clustering notebook (notebook 3) expects a **CSV or Excel file** where each row represents one participant.

### Required metadata columns

```text
Participant, Sex, Age, PLR, Diabetes status, Light Perception
```

### Feature columns

```text
SRI, SleepMidpoint, RA, IV, IVm, IS, ISm,
M10, M10o, L5, L5o, ADAT, CoG, FSoD,
kRA, kAR, LRI, M10olight, L5olight,
PeriodActivity, QPActivity, PeriodLight, QPLight,
PeriodTemperature, QPTemperature
```

### Encoding rules

- Binary variables must be encoded as `0 = No / Female`, `1 = Yes / Male`
- Column names must match exactly
- Actigraphy files for notebook 2 should follow the naming pattern: `*_LogTAT_conferido.txt`

### Sample rows

```csv
Participant,Sex,Age,PLR,Diabetes status,SRI,SleepMidpoint,RA,...,Light Perception
P01F45R0,1,45,1,0,42.59,1567,0.852,...,1
P02F34R0,1,34,1,0,15.24,1551,0.803,...,1
```

---

## 🚀 Getting Started

### Option A — Run in Google Colab (no setup required)

Click the badge above or open notebook 3 directly:

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/Pugliane/nonparametric-actigraphy-clustering/blob/main/nonparametric_actigraphy_clustering.ipynb)

### Option B — Run locally

#### 1. Clone the repository

```bash
git clone https://github.com/Pugliane/nonparametric-actigraphy-clustering.git
cd nonparametric-actigraphy-clustering
```

#### 2. Create and activate a virtual environment

```bash
python -m venv venv

# macOS / Linux
source venv/bin/activate

# Windows
venv\Scripts\activate
```

#### 3. Install dependencies

```bash
pip install -r requirements.txt
```

#### 4. Launch Jupyter and run in order

```bash
jupyter notebook
```

Open the notebooks in sequence: **Notebook 1 → Notebook 2 → Notebook 3**.

---

## 📦 Dependencies

### Core scientific computing

| Package | Version | Purpose |
|---------|---------|---------|
| `numpy` | 1.26.4 | Numerical computing |
| `pandas` | 2.2.2 | Data wrangling and Excel/CSV I/O |
| `scipy` | 1.11.4 | Statistical tests (Shapiro–Wilk, Levene, Mann–Whitney U) |
| `statsmodels` | 0.14.1 | GLMs, OLS regression with HC3 robust standard errors |
| `openpyxl` | latest | Excel file export |

### Machine learning & explainability

| Package | Version | Purpose |
|---------|---------|---------|
| `scikit-learn` | 1.6.1 | PCA, KMeans, silhouette scoring, Random Forest classifier |
| `shap` | 0.44.1 | SHAP feature importance for cluster characterisation |
| `joblib` | latest | Parallel computation |
| `numba` | 0.59.1 | JIT compilation (SHAP dependency) |

### Actigraphy analysis

| Package | Version | Purpose |
|---------|---------|---------|
| `pyActigraphy` | 1.2.1 | Actigraphy file parsing; nonparametric metric computation (IS, IV, RA, M10, L5, SRI, kRA, kAR, LRI, CoG, FSoD, QP, Period) |

### Visualisation

| Package | Version | Purpose |
|---------|---------|---------|
| `matplotlib` | 3.8.2 | Base plotting; publication-ready figure export |
| `seaborn` | 0.13.2 | Statistical visualisation (violin plots, correlation matrices) |
| `plotly` | 5.24.1 | Interactive figures |

### Geospatial & environmental

| Package | Version | Purpose |
|---------|---------|---------|
| `geopandas` | latest | Geospatial data processing |
| `geobr` | latest | Official Brazilian spatial datasets (IBGE) |
| `astral` | latest | Sunrise/sunset and photoperiod calculation by coordinates |
| `requests` | latest | NASA POWER API access for temperature/radiation data |
| `adjustText` | latest | Non-overlapping label placement in maps |

### Utilities

| Package | Version | Purpose |
|---------|---------|---------|
| `tqdm` | latest | Progress bars |
| `Pillow` | latest | Image processing |
| `python-docx` | 1.1.2 | Word document export |
| `pytz` | latest | Timezone handling |
| `ipykernel` | 6.29.5 | Jupyter kernel support |

See [`requirements.txt`](./requirements.txt) for pinned versions.

---

## 🔧 Customising the Pipeline

You can adapt the pipeline to your own actigraphy dataset:

- **Add or remove features** — edit the variables list in the settings block at the top of notebook 3
- **Change the clustering algorithm** — swap KMeans for DBSCAN, hierarchical, or GMM
- **Adjust multicollinearity thresholds** — the default Pearson r > 0.85 and VIF > 5.0 cutoffs can be modified in the preprocessing section
- **Change the PCA variance threshold** — default is ≥80% cumulative variance
- **Update file paths** — set your data directory and filenames at the top of each notebook
- **Adjust off-wrist masking** — provide your own mask database CSV for notebook 2

---

## 📄 Citation

If you use this pipeline in your research, please cite both the software and the paper:

**Software:**
```bibtex
@software{pugliane_2026_nonparametric,
  author    = {Pugliane},
  title     = {Pugliane/nonparametric-actigraphy-clustering: Initial release},
  year      = {2026},
  publisher = {Zenodo},
  version   = {v1.0},
  doi       = {10.5281/zenodo.18922404},
  url       = {https://doi.org/10.5281/zenodo.18922404}
}
```

**Paper:**
```bibtex
@article{pugliane_2026_lowlatitude,
  author  = {Pugliane, Karen C. and França, Lucas G. S. and Leocadio-Miguel, Mario and Araújo, John F.},
  title   = {Low-latitude environmental regularity sustains non-photic entrainment in blind adults},
  journal = {bioRxiv},
  year    = {2026},
  doi     = {10.64898/2026.03.19.712663},
  url     = {https://doi.org/10.64898/2026.03.19.712663}
}
```

---

## 👥 Authors

| Role | Names | Affiliation |
|------|-------|-------------|
| Analysis & Development | Karen C. Pugliane | Graduate Program in Psychobiology, UFRN, Brazil |
| Principal Investigators | Lucas G. S. França | Northumbria University, UK; King's College London, UK |
| | Mario Leocadio-Miguel | Northumbria University, UK |
| | John F. Araújo | UFRN, Brazil; Federal University of Delta do Parnaíba, Brazil |

Correspondence: lucas.franca@northumbria.ac.uk

---

## 🤝 Related Tools

This pipeline was developed alongside other open tools from the Circadia Lab:

- ⚡ [**ACTT_validation_study**](https://github.com/circadia-bio/ACTT_validation_study) — R/Quarto pipeline validating the ActTrust® device for energy expenditure estimation and PA classification; provides the cut-points and regression equations used with the same device
- 🌙 [**Sleep Diaries**](https://github.com/circadia-bio/SleepDiaries) — open-source React Native sleep diary app for consensus-based sleep data collection (iOS, Android, web)
- 🔬 [**circadia-bio**](https://github.com/circadia-bio) — the Circadia Lab GitHub organisation

---

## 📄 Licence

Released under the [MIT License](./LICENSE).

Copyright © Pugliane, França, Leocadio-Miguel & Araújo
