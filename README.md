# Retail Demand Forecasting for Small Businesses

Springboard Data Science Career Track, Capstone Two

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Python](https://img.shields.io/badge/Python-3.12-blue)](https://www.python.org/)
[![Dataset](https://img.shields.io/badge/Dataset-UCI%20Online%20Retail-green)](https://archive.ics.uci.edu/dataset/352/online+retail)

## Problem

A small retailer has to decide each week how much of every product to keep in stock. Too little means lost sales, too much ties up cash. This project forecasts **next week's units sold for each product** from the previous weeks' sales, and asks whether a tuned machine-learning model beats the simple rules a shop owner could apply in a spreadsheet.

## Result in brief

The final model is a histogram gradient boosting regressor (absolute-error loss), chosen by lowest validation MAE and then refit on the training and validation weeks. It was scored once on seven held-out test weeks (17 Oct to 28 Nov 2011, 23,511 product-weeks).

| Test metric | Gradient boosting | Four-week average | Last week | Product training average |
|---|---:|---:|---:|---:|
| Mean absolute error (units) | **27.66** | 28.97 | 33.00 | 35.48 |
| RMSE (units) | 116.85 | **98.39** | 117.76 | 122.20 |
| R² | 0.355 | **0.543** | 0.345 | 0.295 |
| Within ±20% of actual | **37.1%** | 34.3% | 36.4% | 19.2% |

- The model's typical error is 4.5% below the best simple baseline and 22% below the proposal's baseline (each product's training average).
- Most of the gain comes from recognizing weeks with no sales: the model forecast exactly zero in 75.3% of zero-sales weeks, against 64.6% for the four-week average.
- **Limitation:** the model under-forecasts the busiest products and weeks. Summed over all products, its weekly totals were 25% to 46% below actual sales in every test week. Its largest forecast for any product-week was 422 units, while 290 product-weeks sold more than 500. It does worse than the four-week average on RMSE and R², so it should not be used alone to set peak-season stock.

Full details are in the final report and the slides.

## Deliverables

| Item | File |
|---|---|
| Final report | [`reports/final_report.pdf`](reports/final_report.pdf) |
| Final slides | [`slides/Capstone_Final_Slides.pdf`](slides/Capstone_Final_Slides.pdf) |
| Model metrics | [`reports/modeling/model_metrics.csv`](reports/modeling/model_metrics.csv) |
| Project proposal | [`proposal/retail-demand-forecasting-capstone-Proposal.pdf`](proposal/retail-demand-forecasting-capstone-Proposal.pdf) |

## Data

UCI Online Retail data set (Chen, 2015), CC BY 4.0: invoice lines from a UK-based online retailer, December 2010 to December 2011.

- 541,909 raw lines, 522,685 after cleaning (cancellations, non-product codes and invalid rows removed).
- 3,815 distinct products in the story analysis; 3,434 products and 128,794 product-weeks in the modeling table.
- The workbook is expected at `data/Online Retail.xlsx` (or `data/raw/Online Retail.xlsx`). It can be downloaded from the [UCI repository](https://archive.ics.uci.edu/dataset/352/online+retail).

## What the data story showed

1. **Demand is seasonal.** November sold about 747,000 units, 2.1 times the January to June monthly average.
2. **A fifth of the products carry the business.** The top 20% of products (763 of 3,815) account for 77% of units.
3. **Most products sell in fits and starts.** The median established product had no sales in 15 of 42 follow-up weeks.

These findings shaped the design: split by date, report error by volume group, and check how well zero-sales weeks are handled.

## Method

- **Target:** units of one product in the coming Monday to Sunday week, forecast from information available through the prior Sunday (rolling one-week-ahead).
- **Splits by week:** train 10 Jan to 22 Aug 2011 (33 weeks, 83,092 rows); validation 29 Aug to 10 Oct (7 weeks, 22,191 rows); test 17 Oct to 28 Nov (7 weeks, 23,511 rows).
- **Predictors:** recent weekly sales, rolling summaries, product history and calendar features. Revenue and current-week information are excluded. See `reports/preprocessing/feature_dictionary.csv`.
- **Models:** Ridge regression, random forest and histogram gradient boosting, tuned with grid search over three expanding-window cross-validation folds (scored by MAE). XGBoost is an optional candidate that the notebook adds automatically when installed; it was not available in the reported run.
- **Baselines:** per-product training average, last week, four-week rolling mean.
- **Selection rule** (set in advance): lowest validation MAE. The chosen model is refit on training plus validation weeks and scored once on the test weeks.
- Forecasts are rounded to non-negative whole units before scoring.

## Repository structure

```
retail-demand-forecasting-capstone/
├── proposal/                      Project proposal
├── data/
│   ├── Online Retail.xlsx         Source workbook
│   └── processed/preprocessing_v1 Weekly training tables
├── notebooks/
│   ├── 02_exploratory_data_analysis.ipynb
│   ├── 03_preprocessing_and_training_data.ipynb
│   ├── 04_data_storytelling.ipynb
│   └── 05_modeling.ipynb
├── reports/
│   ├── final_report.pdf
│   ├── preprocessing/             Cleaning audit, splits, feature dictionary
│   ├── storytelling/              Charts and summary numbers
│   └── modeling/                  Metrics, tuning, predictions, figures
├── slides/
│   └── Capstone_Final_Slides.pdf
├── PREPROCESSING.md               Details of the training-data pipeline
├── requirements-preprocessing.txt
└── LICENSE
```

## Reproduce

Use Python 3.12.

```bash
python -m pip install -r requirements-preprocessing.txt
jupyter lab
```

Run the notebooks in order (02, 03, 04, 05) with Restart Kernel and Run All. Notebook 05 reads the tables written by notebook 03 and writes its outputs to `reports/modeling/`. Random seeds are fixed. `reports/modeling/run_manifest.json` records the library versions used for the reported results.

## Next steps

Compare XGBoost, test a selection rule or target that reduces the under-forecast of busy weeks, add holiday flags and a second year of data so the model can learn the autumn peak, and build the Streamlit dashboard described in the proposal.

## Author

**Jacob Paul**: [LinkedIn](https://www.linkedin.com/in/jacob25paul) · [GitHub](https://github.com/jacob25paul)

Data: Chen, D. (2015). Online Retail. UCI Machine Learning Repository. https://doi.org/10.24432/C5BW33
