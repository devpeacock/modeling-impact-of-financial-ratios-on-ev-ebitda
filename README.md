# Modeling the Impact of Financial Ratios on EV/EBITDA

This project combines econometrics, machine learning, and clustering to explain cross-sectional differences in EV/EBITDA multiples.

The workflow moves from raw financial data and rigorous cleaning, through regression-based inference and predictive modeling, to company archetype discovery. The goal is not only to improve fit, but also to understand which financial profiles the market rewards with higher valuation multiples.

## Executive Summary

- Data quality is a first-order issue: the baseline linear model improves from about `R^2=0.028` on a minimally cleaned sample to about `R^2=0.188` after structured cleaning.
- Correcting for heteroskedasticity with WLS/FGLS-like estimation lifts fit further to about `R^2=0.220`.
- The best out-of-sample predictive model is **tuned XGBoost** with `R^2=0.3313`, `MAE=3.5877`, `RMSE=4.6996`, and `MAPE=47.93%`.
- Clustering identifies four distinct company archetypes with statistically significant valuation differences (`Kruskal H=69.23`, `p<0.001`).
- The highest EV/EBITDA median is observed in the **innovation + growth** cluster (`14.079`), while the lowest is found in the **market core** cluster (`9.730`).

## Research Scope

The analysis starts from [data/raw/dane_finansowe.csv](data/raw/dane_finansowe.csv) and progresses through cleaned and modeling-ready datasets stored in [data/processed](data/processed).

The main explanatory variables include:

- growth: `Revenue_Growth`
- profitability: `Profit_Margin`, `Gross_Profit_Margin`, `Return_on_Equity`
- efficiency: `Asset_Turnover`, `Fixed_Asset_Turnover`
- leverage: `Debt_to_Equity`
- investment intensity: `CAPEX_to_Revenue`, `R&D_to_Revenue`
- liquidity: `Current_Ratio`
- shareholder payout: `Dividend_Yield`

## Project Structure

### 1. Data and Problem Setup

Notebook: [notebooks_new/01_data_and_problem_setup.ipynb](notebooks_new/01_data_and_problem_setup.ipynb)

This stage prepares the modeling dataset by:

- checking data completeness and identifying implausible values
- filtering EV/EBITDA and ratio outliers using economic thresholds and percentile rules
- moving from a minimally cleaned sample to a modeling-ready dataset

Main outcome: a more economically consistent and statistically stable base for downstream modeling.

### 2. Econometric Analysis

Notebook: [notebooks_new/02_econometric_analysis.ipynb](notebooks_new/02_econometric_analysis.ipynb)

This notebook develops the regression framework through:

- baseline OLS estimation
- residual diagnostics: Breusch-Pagan, White, Jarque-Bera, and RESET tests
- influential-point filtering using Cook's distance
- WLS/FGLS-like estimation to address heteroskedasticity
- sector-effect and structural-stability checks

Key results:

- OLS on the cleaned benchmark: `R^2=0.188`
- WLS/FGLS-like specification: `R^2=0.220`
- heteroskedasticity is statistically significant
- sector controls are economically and statistically important

### 3. Machine Learning Models

Notebook: [notebooks_new/03_machine_learning_models.ipynb](notebooks_new/03_machine_learning_models.ipynb)

This stage compares predictive models using out-of-sample `R^2` as the primary ranking metric.

Models and procedures include:

- WLS benchmark on a consistent train/test split
- baseline XGBoost
- tuned XGBoost with `RandomizedSearchCV`
- LightGBM, CatBoost, Random Forest, and Ridge
- ensemble variants and SHAP-based interpretability

Key results:

- WLS test performance: `R^2=0.2472` and weighted test `R^2=0.2734`
- XGBoost baseline: `R^2=0.3081`
- tuned XGBoost: `R^2=0.3313`
- LightGBM: about `R^2=0.3178`

Interpretation: non-linear effects and feature interactions matter, which is why tree-based models outperform the linear benchmark in predictive terms.

### 4. Clustering Companies

Notebook: [notebooks_new/04_clustering_comapnies.ipynb](notebooks_new/04_clustering_comapnies.ipynb)

This notebook segments firms into business archetypes using K-means clustering.

The workflow includes:

- feature standardization
- cluster-count validation
- PCA and silhouette-based diagnostics
- statistical testing of between-cluster differences
- export of cluster company lists and descriptive statistics

Key results:

- mean silhouette: `0.3175`
- EV/EBITDA differs significantly across clusters: `H=69.23`, `p<0.001`
- EV/EBITDA medians by cluster:
  - Cluster 4: `14.079`
  - Cluster 1: `11.726`
  - Cluster 2: `9.785`
  - Cluster 3: `9.730`

## Cross-Sectional Findings

1. **Data cleaning materially changes model quality.**
Structured filtering removes economically inconsistent observations and substantially improves signal quality.

2. **Machine learning outperforms the linear benchmark in out-of-sample prediction.**
The strongest result comes from tuned XGBoost (`R^2=0.3313`) versus WLS (`R^2=0.2472`).

3. **Heteroskedasticity is a real feature of the data, not a technical footnote.**
Weighted estimation produces a measurable gain and supports more credible inference.

4. **Clustering reveals valuation archetypes that are not visible in a single average regression.**
The innovation-oriented growth profile commands a clear valuation premium over the neutral core profile.

## Business Interpretation

- The market rewards firms that combine growth and development intensity, especially when innovation spending is not accompanied by excessive leverage.
- High profitability alone does not guarantee a premium multiple if balance-sheet risk remains elevated.
- A combined framework using econometrics, machine learning, and clustering provides a fuller view than any single method used in isolation.

## Reproducibility

The project was developed and run locally with a Python virtual environment. To reproduce the analysis, create a fresh `venv`, install the notebook dependencies from [requirements.txt](requirements.txt), and use that environment as the notebook kernel.

### Recommended setup on Windows

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
python -m ipykernel install --user --name modeling-ev-ebitda --display-name "Python (modeling-ev-ebitda)"
```

Then open the notebooks in VS Code and select the `Python (modeling-ev-ebitda)` kernel.

### Notes on reproducibility

- The repository includes the processed datasets used by the notebooks, which makes the workflow reproducible without reconstructing every intermediate file from scratch.
- `requirements.txt` captures the core Python dependencies referenced in the notebooks.
- The notebooks are intended to be run in order, from notebook `01` through `04`.
- Some outputs may vary slightly across environments because of package-version differences and model stochasticity, but the reported ranking and main conclusions should remain stable if the same versions are used.

## Limitations

- The analysis is cross-sectional rather than causal.
- Results depend on the adopted cleaning logic and outlier treatment.
- Some relevant valuation drivers are not observed directly, such as management quality, macro exposure, or one-off corporate events.
- Cluster separation is meaningful but not perfectly sharp, so firm-level boundaries between archetypes should not be treated as absolute.

## Repository Contents

Core outputs:

- [data/processed/df_cleaned.csv](data/processed/df_cleaned.csv)
- [data/processed/df_sectors_cook2.csv](data/processed/df_sectors_cook2.csv)
- [data/processed/ml_train_dataset_used_for_models.csv](data/processed/ml_train_dataset_used_for_models.csv)
- [data/processed/ml_test_dataset_used_for_models.csv](data/processed/ml_test_dataset_used_for_models.csv)
- [data/processed/clusters/companies/cluster_company_list_1.csv](data/processed/clusters/companies/cluster_company_list_1.csv)
- [data/processed/clusters/companies/cluster_company_list_2.csv](data/processed/clusters/companies/cluster_company_list_2.csv)
- [data/processed/clusters/companies/cluster_company_list_3.csv](data/processed/clusters/companies/cluster_company_list_3.csv)
- [data/processed/clusters/companies/cluster_company_list_4.csv](data/processed/clusters/companies/cluster_company_list_4.csv)
- [data/processed/clusters/statistics/cluster_statistics_1.csv](data/processed/clusters/statistics/cluster_statistics_1.csv)
- [data/processed/clusters/statistics/cluster_statistics_2.csv](data/processed/clusters/statistics/cluster_statistics_2.csv)
- [data/processed/clusters/statistics/cluster_statistics_3.csv](data/processed/clusters/statistics/cluster_statistics_3.csv)
- [data/processed/clusters/statistics/cluster_statistics_4.csv](data/processed/clusters/statistics/cluster_statistics_4.csv)

Main notebooks:

- [notebooks_new/01_data_and_problem_setup.ipynb](notebooks_new/01_data_and_problem_setup.ipynb)
- [notebooks_new/02_econometric_analysis.ipynb](notebooks_new/02_econometric_analysis.ipynb)
- [notebooks_new/03_machine_learning_models.ipynb](notebooks_new/03_machine_learning_models.ipynb)
- [notebooks_new/04_clustering_comapnies.ipynb](notebooks_new/04_clustering_comapnies.ipynb)

## Conclusion

The project delivers an end-to-end valuation workflow: from data cleaning and econometric diagnostics to predictive modeling and business archetype discovery.

The central practical conclusion is that valuation premia are linked not only to current profitability, but also to a firm's growth profile, innovation intensity, and risk structure.