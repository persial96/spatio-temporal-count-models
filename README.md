# Quarterly Burglary Prediction in Zug and St. Gallen

This repository contains the code used to model and predict quarterly burglary counts across spatial grid cells in the Swiss cantons of Zug and St. Gallen. Burglary forecasting is reformulated as a quarterly spatial count prediction problem. The objective is to identify areas that consistently exhibit elevated risk and may therefore deserve greater preventive attention from police patrols.

## Approach

Three increasingly flexible specifications are compared:

1. **MDL0**: covariates only (benchmark);
2. **MDL1**: MDL0 plus seasonal fixed effects and a linear time trend;
3. **MDL2**: MDL1 plus a quadratic spatial trend in the cell coordinates.

Each specification is estimated with a Poisson and a Negative Binomial likelihood (six candidate models per canton, `_NB` denoting the Negative Binomial version). The models are estimated separately for each canton and evaluated with AIC/BIC, crime capture (hit) rates, the Prediction Accuracy Index (PAI), and quarter-to-quarter hotspot stability (Jaccard index).

## Data and aggregation

The original data contain daily observations for spatial RELI grid cells (100 m × 100 m) in both cantons. They are aggregated by cell $i$ and calendar quarter $t$:

```math
Y_{it} = \sum_{d \in t} \mathrm{Burglary}_{id}
```

The quarterly panel contains burglary counts, cell coordinates, population density, employment and business density, temperature, precipitation, and calendar variables. Covariates are averaged within the quarter (only precipitation is summed), and the number of observed days $n_{it}$ is kept as the exposure.

## Model specifications

For cell $i$ in quarter $t$:

```math
Y_{it}\mid X_{it}\sim \mathrm{Poisson}(\mu_{it}) \;\text{ or }\; \mathrm{NB}(\mu_{it}, \theta), \qquad
\log(\mu_{it}) = \eta_{it} + \log(n_{it}),
```

where $\log(n_{it})$ is an offset and, under the Negative Binomial, $\mathrm{Var}(Y_{it}) = \mu_{it} + \mu_{it}^2/\theta$ (smaller $\theta$ means stronger overdispersion). The covariates $X_{it}$ are population density, Swiss population, female population, number of businesses, employment density, average temperature, and cumulative precipitation.

| Model | Linear predictor $\eta_{it}$ |
|---|---|
| MDL0 | $\beta_0 + X_{it}'\beta$ |
| MDL1 | $\beta_0 + \delta_{q(t)} + \beta_t t + X_{it}'\beta$ |
| MDL2 | $\beta_0 + \delta_{q(t)} + \beta_t t + X_{it}'\beta + s_{\mathrm{quad}}(E_i, N_i)$ |

Here $\delta_{q(t)}$ are quarter-of-year fixed effects, $\beta_t t$ is a linear time trend, and

```math
s_{\mathrm{quad}}(E_i,N_i) = \gamma_1E_i + \gamma_2N_i + \gamma_3E_i^2 + \gamma_4N_i^2 + \gamma_5E_iN_i
```

is a quadratic surface in the standardized coordinates, which captures broad east–west and north–south gradients and central high- or low-risk areas.

## Train-test design

Chronological split: the first 80% of quarters are used for training and the final 20% for testing.

## Evaluation metrics

- **Hit rate** at $k$: share of test-period burglaries that fall in the top $k$ share of cells ranked by predicted risk.
- **PAI** at $k$: hit rate divided by the share of cells selected, i.e. how concentrated burglaries are in the predicted hotspots relative to a uniform allocation.
- **Jaccard index**: overlap of the hotspot sets in consecutive quarters, $J_t = |H_t \cap H_{t-1}| / |H_t \cup H_{t-1}|$ (0 = no overlap, 1 = identical sets).

## Main results

Best model per canton (selected by AIC) and its out-of-sample hotspot performance on the test quarters:

| Canton | Best model | $\theta$ | Dev. explained | AIC | BIC | Hit rate 5% | Hit rate 10% | PAI 5% | PAI 10% | Jaccard 5% | Jaccard 10% |
|---|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Zug | MDL2_NB | 0.338 | 17.4% | 16222.49 | 16392.38 | 33.7% | 45.6% | 6.74 | 4.55 | 0.959 | 0.962 |
| St. Gallen | MDL2_NB | 0.304 | 18.0% | 61061.37 | 61265.84 | 27.9% | 42.5% | 5.58 | 4.25 | 0.918 | 0.927 |

In both cantons the preferred specification is MDL2_NB: the Negative Binomial with seasonal, trend, and quadratic spatial terms. The estimated $\theta$ (about 0.30–0.34) indicates strong overdispersion, which a Poisson likelihood cannot accommodate. Under tight resource constraints, targeting only the top 5% of cells captures 33.7% of test-period burglaries in Zug and 27.9% in St. Gallen, corresponding to burglary concentrations about 6.7 and 5.6 times higher than under a uniform allocation. Extending coverage to 10% of cells raises the hit rate to 45.6% and 42.5%, respectively. Hotspot sets are highly stable from one quarter to the next (Jaccard above 0.91 in both cantons), pointing to a persistent core of high-risk areas.

## Relative-risk maps

The map shows the predicted relative-risk percentile of each cell for one test quarter, for Zug (left) and St. Gallen (right).

![Relative-risk maps for Zug and St. Gallen](figures/relative_risk_maps.png)

## Repository structure

```text
.
├── README.md
├── LICENSE
├── code
│   ├── 01_data_aggregation.R
│   ├── 02_covariate_creation.R
│   ├── 03_model_estimation.R
│   ├── 04_model_evaluation.R
│   ├── 05_hotspot_metrics.R
│   └── 06_figures.R
├── figures
    └── relative_risk_maps.png

```

## Requirements

```r
required_packages <- c("dplyr", "tidyr", "purrr", "lubridate", "ggplot2", "scales",
                       "MASS", "mgcv", "spdep", "stargazer", "knitr")
install.packages(setdiff(required_packages, rownames(installed.packages())))
```

## Data availability

The data are not disclosed, as they are subject to a non-disclosure agreement (NDA). A synthetic dataset with the same structure can be provided upon request, so that the full pipeline can be run.

## Citation

A citation entry will be added after publication.

## License

The code is released under the [MIT License](LICENSE).
