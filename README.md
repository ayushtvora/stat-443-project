# Forecasting the U.S. Gasoline–Diesel Price Spread

A time series forecasting project for STAT 443 (Time Series and Forecasting) at the University of British Columbia, by Ayush Vora, Victor Liu, Elad Bromberg and Shahrukh Islam Prithibi.

## Question

Gasoline mostly fuels passenger vehicles and diesel mostly fuels freight, so the gap between their prices matters to shippers, policymakers and consumers. This project asks: **which forecasting method gives the most accurate one-week-ahead forecasts of the weekly U.S. gasoline–diesel price spread?**

The spread is the U.S. All Grades All Formulations retail gasoline price minus the diesel retail price, in dollars per gallon.

## Data

| File | Contents | Source |
|---|---|---|
| `PET_PRI_GND_DCUS_NUS_W.csv` | Weekly U.S. retail gasoline (by grade and formulation) and diesel prices, Jan 1995 – Dec 2021, 1,361 weeks | U.S. Energy Information Administration, via [Kaggle](https://www.kaggle.com/datasets/mruanova/us-gasoline-and-diesel-retail-prices-19952021) |
| `oilprices.csv` | Daily WTI crude oil price | Federal Reserve Bank of St. Louis (FRED, series `DCOILWTICO`) |
| `heatingoil.csv` | Daily heating oil price | U.S. Energy Information Administration |

## Method

All models are trained on the first 1,257 weeks and evaluated with **rolling one-step-ahead forecasts** over a **104-week holdout** (2020–2021). Coefficients are fixed at their training estimates, and models are compared by holdout RMSE.

1. **Exploratory analysis**: checked for gaps in the weekly series, compared gasoline grades and formulations (they move together, so the all-grades series is used), and found a structural shift around 2008, after which diesel has consistently cost more than gasoline. The ACF of the spread decays slowly, so the series is non-stationary. First differencing makes it stationary, and a log transform adds nothing.
2. **Baselines**: persistence (next week equals this week) and the training-set mean.
3. **Exponential smoothing**: Holt's linear method, and seasonal Holt-Winters with periods of 26 and 52 weeks.
4. **ARIMA**: started from `auto.arima()`'s suggestion of ARIMA(3,1,2), then searched 15 nearby non-seasonal orders and 17 seasonal (SARIMA) specifications by holdout RMSE. The best model's residuals were checked with an ACF plot and a Ljung-Box test.
5. **ARIMAX**: added crude oil and heating oil prices, lagged by one week (the lag the cross-correlations pointed to), as external regressors, and searched 27 combinations of regressors and ARIMA error structures.

## Results

| Model | Holdout RMSE ($/gal) |
|---|---|
| Training mean | 0.2460 |
| Persistence | 0.0291 |
| Holt-Winters, period 52 | 0.0309 |
| Holt-Winters, period 26 | 0.0305 |
| Holt linear | 0.0253 |
| **ARIMA(4,1,3)** | **0.0232** |
| ARIMAX: crude + heating oil (lag 1), ARIMA(2,0,1) errors | 0.0232 |

- **ARIMA(4,1,3) is the recommended model.** It cuts RMSE by about 20% compared with persistence, and its residuals look like white noise (Ljung-Box p = 0.365).
- **Seasonality did not help.** Seasonal Holt-Winters did worse than Holt linear, and seasonal ARIMA terms gave no improvement.
- **Oil prices did not help.** The ARIMAX model only tied ARIMA(4,1,3), so the spread's own history carries the useful short-term information, and the simpler model is preferred.

## Files

| File | What it does |
|---|---|
| `stat443-project-EDA.Rmd` | Data cleaning, summary statistics, plots, ACF/PACF and cross-correlation analysis |
| `stat443-project-persistence-iid.Rmd` | Persistence and training-mean baselines |
| `stat443-project-expsmo.Rmd` | Holt linear and seasonal Holt-Winters models |
| `stat443-project-arima.Rmd` | `auto.arima()`, non-seasonal and seasonal ARIMA search, residual diagnostics |
| `stat443-project-arimax.Rmd` | ARIMAX models with lagged crude oil and heating oil prices |

## Running it

Requires R with `tidyverse`, `forecast`, `zoo` and `gridExtra`:

```r
install.packages(c("tidyverse", "forecast", "zoo", "gridExtra"))
```

Open any `.Rmd` file in RStudio and knit it. The fuel price file is read from the team's original repository, [ShahrukhP15/Time-Series-Forecasting-of-U.S.-Fuel-Price-Spreads](https://github.com/ShahrukhP15/Time-Series-Forecasting-of-U.S.-Fuel-Price-Spreads), and a copy is included here. The ARIMAX file reads the oil price files from this folder, so run it with this folder as the working directory.

## My contribution

I formed the group, did the initial analysis for the project proposal, and led the exploratory data analysis: data cleaning, variable descriptions, summary statistics and visualizations. I also organized meetings and tracked tasks for the team.
