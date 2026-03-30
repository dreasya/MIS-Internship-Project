# Time Series Forecasting — Energy Consumption & Retail Demand

> Two end-to-end time series forecasting projects using **Facebook Prophet** and classical decomposition techniques. Applied to real-world datasets: **16 years of hourly US energy grid data** (PJME) and a **multi-dimensional retail sales dataset** — covering EDA, feature engineering, model training, and evaluation.

---

## Table of Contents

- [Project Overview](#project-overview)
- [Notebooks](#notebooks)
- [Datasets](#datasets)
- [Methodology](#methodology)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Getting Started](#getting-started)
- [Results](#results)
- [Key Learnings](#key-learnings)

---

## Project Overview

This project contains two independent but thematically linked forecasting notebooks, each tackling a different domain and granularity of time series data:

| Notebook | Domain | Granularity | Model | Target |
|---|---|---|---|---|
| Energy Consumption | Power grid (US East) | Hourly | Prophet | Energy use in MW |
| Demand Forecasting | Retail sales | Monthly | Prophet | Sales revenue ($) |

Both notebooks follow the same disciplined pipeline: exploratory analysis at multiple time scales → feature engineering → model training → forecast evaluation.

---

## Notebooks

### 1. `Energy_Consumption_-_Time_series_forecasting.ipynb`

Forecasts hourly energy consumption for PJM's eastern US grid using the **PJME_hourly** dataset (January 2002 – August 2018, ~145,000 hourly records). The notebook performs a deep multi-scale EDA — zooming from the full 16-year series down to individual days — before engineering time-based and holiday features and training a Prophet model with US federal holidays baked in.

### 2. `Demand_Forcasting.ipynb`

Forecasts monthly retail sales from a Superstore-style transactional dataset. The notebook explores sales trends broken down by region, state, city, product category, sub-category, and shipping mode, before aggregating to monthly granularity and forecasting 24 months ahead with Prophet.

---

## Datasets

### PJME Hourly Energy Consumption
- **Source:** PJM Interconnection LLC (regional transmission organization, eastern US)
- **Coverage:** January 2002 – August 2018
- **Records:** ~145,000 hourly observations
- **Target column:** `PJME_MW` — energy consumption in megawatts
- **Missing values:** None

### Retail Sales (`train.csv`)
- **Type:** Transactional retail / superstore orders
- **Columns include:** Order Date, Ship Date, Ship Mode, Customer, Region, State, City, Category, Sub-Category, Product, Sales
- **Granularity after resampling:** Monthly average sales
- **Forecast horizon:** 24 months ahead

---

## Methodology

Both notebooks follow this pipeline:

```
Raw Data
   │
   ▼
Data Cleaning & Type Conversion
   │  - Parse datetimes, set as index
   │  - Check & handle missing values
   ▼
Exploratory Data Analysis (EDA)
   │  - Multi-scale visual analysis
   │    (full series → yearly → seasonal → monthly → weekly → daily)
   │  - Seasonal decomposition (trend, seasonality, residual)
   │  - Sales breakdown by category / region / ship mode
   ▼
Feature Engineering
   │  - Time features: hour, day of week, month, quarter,
   │    day of year, week of year
   │  - Binary US federal holiday flag
   ▼
Train / Test Split
   │  - Energy: split at 2015-01-01
   │    (train = 2002–2014, test = 2015–2018)
   │  - Demand: full fit, 24-month future projection
   ▼
Prophet Model
   │  - Holiday integration (USFederalHolidayCalendar)
   │  - Uncertainty interval: 95%
   │  - Component decomposition plots
   ▼
Evaluation
     - MAE, RMSE, MAPE (energy)
     - MSE, MPE / prediction accuracy % (demand)
```

---

### Facebook Prophet — Why It Was Chosen

Prophet is well-suited to both use cases because it:
- Natively handles **daily, weekly, and yearly seasonality** without manual specification
- Integrates **custom holiday effects** out of the box
- Is **robust to missing data** and outliers
- Produces **uncertainty intervals** alongside point forecasts
- Decomposes forecasts into interpretable **trend + seasonality + holiday** components

---

### Feature Engineering Details (Energy Notebook)

Beyond Prophet's native seasonality handling, the energy notebook engineers explicit time features for model context:

| Feature | Description |
|---|---|
| `hour` | Hour of day (0–23) |
| `dayofweek` | Day of week (0=Monday) |
| `quarter` | Calendar quarter (1–4) |
| `month` | Month of year (1–12) |
| `year` | Calendar year |
| `dayofyear` | Day number within year (1–365) |
| `dayofmonth` | Day of month (1–31) |
| `weekofyear` | ISO week number |
| `holiday` | Binary US federal holiday flag |

---

## Tech Stack

| Purpose | Library |
|---|---|
| Data manipulation | `pandas`, `numpy` |
| Visualisation | `matplotlib`, `seaborn` |
| Time series decomposition | `statsmodels` |
| Forecasting | `prophet` (Facebook/Meta) |
| Interactive plots | `plotly` (via `prophet.plot`) |
| Holiday calendar | `holidays`, `pandas.tseries.holiday` |
| Error metrics | `sklearn.metrics` |

---

## Project Structure

```
time-series-forecasting/
│
├── Energy_Consumption_-_Time_series_forecasting.ipynb
├── Demand_Forcasting.ipynb
│
├── PJME_hourly.csv          # PJM energy consumption dataset
├── train.csv                # Retail sales transactional dataset
│
└── README.md
```

---

## Getting Started

### Prerequisites

- Python 3.9+
- Jupyter Notebook or JupyterLab

### Installation

```bash
# 1. Clone the repository
git clone https://github.com/your-username/time-series-forecasting.git
cd time-series-forecasting

# 2. Create and activate a virtual environment
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate

# 3. Install dependencies
pip install -r requirements.txt

# 4. Launch Jupyter
jupyter notebook
```

### Requirements

```
pandas
numpy
matplotlib
seaborn
prophet
statsmodels
scikit-learn
holidays
plotly
jupyter
```

---

## Results

### Energy Consumption Forecasting (PJME)

Evaluated on the held-out test set (2015–2018):

| Metric | Score |
|---|---|
| **MAE** | 3,102.92 MW |
| **RMSE** | 4,116.50 MW |
| **MAPE** | 9.62% |

The model successfully captures multi-level seasonality — yearly peaks and troughs, weekly workday/weekend cycles, and intra-day consumption patterns (morning ramp-up, evening peak). Prophet's component plot mirrors the manual EDA visualisations closely, confirming the model learned the true underlying structure of the data.

> The confidence intervals widen over time, honestly reflecting growing uncertainty in long-horizon hourly forecasts — appropriate behaviour for a probabilistic model.

---

### Demand Forecasting (Retail)

Evaluated on the training period (in-sample):

| Metric | Score |
|---|---|
| **MSE** | — (printed in notebook) |
| **MPE** | ~13% |
| **Prediction Accuracy** | **~87%** |

The model was projected 24 months into the future with a 95% uncertainty interval. The component breakdown revealed clear yearly seasonality in retail sales, consistent with typical Q4 peak behaviour.

---

## Key Learnings

**Multi-scale EDA is essential for time series.** Zooming from 16 years down to a single day revealed patterns invisible at the macro level — intra-day demand curves, weekend dips, and holiday spikes — that directly informed the feature engineering strategy.

**Prophet's component plots are a built-in sanity check.** The decomposed trend, weekly, and yearly components matched the manually plotted average distributions almost exactly, confirming the model absorbed the real seasonality rather than overfitting noise.

**Holiday integration is non-trivial but high-impact.** Explicitly passing a `USFederalHolidayCalendar`-derived holiday dataframe to Prophet reduced error on peak anomaly days (Christmas, New Year, Thanksgiving) that pure seasonality models struggle to predict.

**Granularity choice shapes everything.** Hourly data (energy) demands different modelling decisions than monthly aggregates (retail). Resampling the retail dataset from daily transactions to monthly means before fitting Prophet was a deliberate trade-off — smoothing noise at the cost of sub-monthly resolution.

---

*Built with curiosity, multi-scale thinking, and a genuine interest in how patterns in time reveal structure in the real world.*
