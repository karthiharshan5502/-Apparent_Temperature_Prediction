# -Apparent_Temperature_Prediction
Bayesian and frequentist analysis of apparent temperature (heat index) in Szeged, Hungary (2006–2016), using SAS and R on 96,453 hourly weather observations.

## 📌 Overview

This project investigates the **meteorological factors that influence apparent temperature** (heat index) — the temperature as perceived by humans, combining air temperature, humidity, wind speed, and other factors. A **full Bayesian regression analysis** was conducted using SAS PROC MCMC, followed by a **frequentist analysis** to predict days with apparent temperature below 0°C.

---

## 🔬 Research Questions

1. Which meteorological variables are statistically significant predictors of apparent temperature?
2. How do interaction effects (e.g., temperature × humidity) influence apparent temperature?
3. How well can a frequentist model predict days with apparent temperature below 0°C?

---

## 📂 Repository Structure

```
├── Project_Report_final.docx         # Full research report (Bayesian + Frequentist analysis)
├── FINAL_Codes_Memba.docx            # Complete SAS and R code
├── weatherHistory.csv                # Dataset — hourly weather data, Szeged Hungary 2006–2016
```

---

## 📊 Dataset

- **Source**: [Kaggle](https://www.kaggle.com) — `weatherHistory.csv`
- **Coverage**: Szeged, Hungary — hourly observations, 2006–2016
- **Size**: 96,453 observations

| Variable | Description | Mean | Std Dev |
|---|---|---|---|
| Temperature | Air temperature (°C) | 11.93 | 9.55 |
| Apparent Temperature | Perceived temperature (°C) — **response** | 10.86 | 10.70 |
| Humidity | Relative humidity (0–1) | 0.735 | 0.195 |
| Wind Speed | Wind speed (km/h) | 10.81 | 6.91 |
| Pressure | Atmospheric pressure (millibars) | 1003.24 | 116.97 |
| Wind Bearing | Wind direction (degrees) | 187.51 | 107.38 |
| Visibility | Visibility (km) | 10.35 | 4.19 |
| Precip Type | Precipitation type (categorical) | — | — |

---

## ⚙️ Methodology

### Part 1 — Bayesian Analysis (SAS PROC MCMC)

**Model:**
```
ApparentTemperature ~ Normal(μ, σ²)
μ = β0 + β1·Temperature + β2·WindSpeed
    + β3·(Temperature × Humidity)
    + β4·(Temperature × WindSpeed)
```

**Prior distributions** (non-informative):
```
βⱼ ~ N(0, 1000²)   for j = 0, 1, ..., p
```

**MCMC Configuration:**

| Setting | Value |
|---|---|
| Chains | 3 |
| Iterations per chain | 50,000 |
| Burn-in | 10,000 |
| Thinning | Every 10th sample |
| Total posterior samples | 12,000 |

**Convergence diagnostics**: Trace plots, Gelman-Rubin statistic, Effective Sample Size (ESS).

---

### Part 2 — Frequentist Analysis (SAS PROC GLM / GLMSELECT)

- **Training set**: First 86,453 observations
- **Test set**: Next 10,000 observations
- **Goal**: Predict days where apparent temperature < 0°C
- **Evaluation**: Mean Absolute Error (MAE)

---

## 📈 Key Results

### Parsimonious Bayesian Model

After removing insignificant predictors, the final model retains:

| Parameter | Mean | 95% HPD Interval | Interpretation |
|---|---|---|---|
| β0 (Intercept) | 0.0266 | [0.004, 0.051] | Baseline apparent temperature |
| β1 (Temperature) | 1.014 | [0.992, 1.035] | Strong positive effect — 1°C rise → ~1°C increase in apparent temp |
| β2 (Wind Speed) | -0.0603 | [-0.080, -0.041] | Negative effect — higher wind → lower apparent temp |
| β3 (Temp × Humidity) | 0.0437 | [0.023, 0.063] | Humidity amplifies temperature's effect |
| β4 (Temp × Wind Speed) | 0.0780 | [0.057, 0.101] | Wind speed amplifies temperature's effect |

### MCMC Diagnostics

| Parameter | ESS | Efficiency |
|---|---|---|
| β0 | 1522 | 30.4% |
| β1 | 2546 | 50.9% |
| β2 | 2465 | 49.3% |
| β3 | 2134 | 42.7% |
| β4 | 2029 | 40.6% |

### Predictive Posterior

- At **Temperature = 10°C, Humidity = 50%, Wind Speed = 5 km/h** → predicted apparent temperature ≈ **14°C**
- Increasing temperature to **15°C** shifts the distribution right, demonstrating the model's sensitivity to temperature changes

---

## 🛠️ Requirements

**SAS** (with PROC MCMC and PROC GLMSELECT)

**R packages:**
```r
install.packages(c("ggplot2", "coda", "bayesplot"))
```

---

## 🚀 How to Run

```sas
/* 1. Import the dataset */
proc import datafile="/path/to/weatherHistory.csv" out=weather dbms=csv; run;

/* 2. Run full Bayesian model */
proc mcmc data=weather nmc=50000 nbi=10000 thin=10 seed=42;
    parms beta0 0 beta1 0 beta2 0 beta3 0 beta4 0 sigma 1;
    prior beta: ~ normal(0, var=1000000);
    mu = beta0 + beta1*Temperature + beta2*WindSpeed
       + beta3*temp_humidity + beta4*temp_windSpeed;
    model ApparentTemperature ~ normal(mu, var=sigma*sigma);
run;

/* Full SAS and R code available in FINAL_Codes_Memba.docx */
```

---

## 💡 Key Findings

- **Temperature** is the strongest predictor of apparent temperature, with a near 1:1 relationship
- **Wind speed** has a significant negative effect — higher wind speed reduces perceived temperature
- **Interaction terms** (temperature × humidity, temperature × wind speed) are significant, showing these variables do not act independently
- **Humidity, pressure, visibility, and wind bearing alone** are not statistically significant predictors
- The Bayesian approach provided well-calibrated posterior distributions with good MCMC convergence

---

## ⚠️ Limitations

- Dataset limited to one geographic location (Szeged, Hungary) — results may not generalise globally
- `LoudCover` variable had zero variance and was excluded
- Frequentist model for sub-zero prediction may have limited accuracy due to class imbalance

---

## 📄 References

- van de Schoot, R., et al. (2014). A gentle introduction to Bayesian analysis: Applications to developmental research. *Child Development*, 85(3), 842–860.

---
