# Capstone Two: Preprocessing and Training Data Development

Add `notebooks/03_preprocessing_and_training_data.ipynb` to the existing retail-demand-forecasting-capstone repository. It follows the newer `02_exploratory_data_analysis.ipynb` and the supplied Springboard rubric.

## Run

Use Python 3.12. In the repository root:

```bash
python -m pip install -r requirements-preprocessing.txt
jupyter lab
```

Open the new notebook and select Restart Kernel and Run All Cells. The workbook must be at `data/Online Retail.xlsx` or `data/raw/Online Retail.xlsx`. Run from the root or notebooks folder. The workbook is already present in the reviewed repository. Reading the Excel file is the slowest step; the rest runs on the completed weekly panel. There are no machine-specific data paths or automatic network downloads.

## Forecasting contract

Forecast one product's units for the coming Monday–Sunday week from information available through the prior Sunday. Later holdout weeks may use actuals from earlier completed holdout weeks. This is rolling one-week-ahead forecasting, not a multi-week forecast made at one fixed origin.

- Same cleaning as current EDA: 522,685 retained merchandise rows; missing customer IDs are retained.
- Whole boundary weeks are excluded because the source begins and ends midweek.
- Zero-sales rows continue after introduction through the end of coverage; ending at the eventual last sale would use future information.
- Eligibility uses four prior selling weeks, not full-year product totals.
- Lags, rolling summaries and last price use prior weeks only.
- Revenue, current-week transactions/price, descriptions and customer IDs are excluded from predictors.

## Splits and results

| Split | Week starts | Rows |
|---|---|---:|
| Train | 2011-01-10–2011-08-22 | 83,092 |
| Validation | 2011-08-29–2011-10-10 | 22,191 |
| Test | 2011-10-17–2011-11-28 | 23,511 |

The unscaled development table has 128,794 rows across 3,434 products. Processed matrices have 3,107 columns: 11 numeric, 3,084 product indicators and 12 calendar-month indicators. Product vocabulary, imputation and scaling are learned on training rows only. The 12-month vocabulary comes from calendar knowledge; autumn categories absent from training have no learned effect yet.

All 11 code cells executed sequentially twice, each in a fresh Python process. All 15 leakage/alignment checks passed, including current/future-outcome perturbation, and saved artifacts passed reload checks. Both plots were inspected. Local Jupyter kernel execution was not tested.

## Artifacts and row order

`data/processed/preprocessing_v1/` contains:

- `development_dataset.csv.gz`: unscaled features, UnitsSold target, WeekStart, RowID and Split. Use this for fold-specific preprocessing during tuning.
- `X_train.npz`, `X_validation.npz`, `X_test.npz`: sparse CSR feature matrices.
- `y_train.npy`, `y_validation.npy`, `y_test.npy`: unscaled target arrays.
- `rows_train.csv.gz`, `rows_validation.csv.gz`, `rows_test.csv.gz`: row keys in the corresponding matrix/target order.
- `preprocessor.joblib`, `feature_names.json`, `manifest.json`: fitted transformation, column order and reproducibility metadata.

Never independently shuffle the metadata, target and feature matrix. Each split is sorted by WeekStart then StockCode. Do not convert the full sparse matrix to a dense array.

```python
from pathlib import Path
import pandas as pd
import numpy as np
from scipy import sparse

artifacts = Path('data/processed/preprocessing_v1')
development = pd.read_csv(
    artifacts / 'development_dataset.csv.gz',
    dtype={'StockCode': str, 'Month': str, 'RowID': str},
    parse_dates=['WeekStart'],
)
X_train = sparse.load_npz(artifacts / 'X_train.npz')
y_train = np.load(artifacts / 'y_train.npy', allow_pickle=False)
```

The explicit Month string dtype preserves values such as `01`. The transformer expects engineered history features, not raw invoice lines. Load joblib files only from trusted sources and with matching package versions.

Small reports in `reports/preprocessing/` document cleaning, splits, schema, scale ranges, unseen products, cross-validation folds and validation checks. Large reproducible data are excluded from Git through the supplied `.gitignore` addition.

## GitHub integration

Extract the update ZIP into the root of your existing clone, preserving folders. Review README and .gitignore changes before replacing any locally edited copies. Keep your existing EDA notebooks and workbook. Commit the new notebook, PREPROCESSING.md, requirements file, reports and the reviewed README/.gitignore updates. Do not commit generated data/processed files.

The notebook itself is the mentor submission. After you upload it, open its GitHub page while signed out and submit that URL. No remote commit or push has been performed by this deliverable.

## Modeling next

Use date-block cross-validation on the raw development table with preprocessing inside each fold's model pipeline. The notebook supplies a unique-week splitting helper, avoiding same-week train/validation overlap across different products. Tune on training folds, choose on validation, and reserve the final test for one final evaluation. A train+validation refit must refit preprocessing on that combined past partition only.

Sales may understate demand because stockouts are not observed. Continuing inactive products assumes they remain in the assortment, since discontinuation dates are unavailable. Check these assumptions with your mentor before modeling.

Source: Chen, D. (2015), UCI Online Retail, https://doi.org/10.24432/C5BW33, CC BY 4.0. Prepared with AI assistance.

## Rolling standard deviation correction

The four-week population standard deviation is recomputed directly within each window with NumPy. This avoids a small positive residual that incremental rolling variance can leave for constant histories on some environments. Validation checks every complete window against direct lag-matrix arithmetic and requires constant-window standard deviations to be exactly zero. Replace the notebook, restart the kernel, then run all cells to regenerate transformed data and reports. Jupyter kernel execution could not be tested here because this execution environment blocks communication sockets; the two full verification runs used fresh Python processes. Your Python 3.14/macOS environment was not reproduced.
