# Top-Down vs Bottom-Up Forecast Reconciliation in FMCG Demand Planning

Advanced Time Series Analysis (ATSA) project, B.Tech Integrated, Semester VII, AY 2025-2026
Mukesh Patel School of Technology Management and Engineering (NMIMS, Mumbai)

**Team:** Alisha Rao (E016), Snigdha Mishra (E036), Pranati Bhoga (E012)

## What this project does

An FMCG company forecasts demand at several levels at once (total, state, store, SKU). Forecasts made separately at each level do not add up, which leads to conflicting purchasing and inventory decisions. This project compares forecast reconciliation strategies that make all levels consistent, and finds which works best at each level of the hierarchy.

Methods compared:

| Family | Variants |
|---|---|
| Baselines | Seasonal naive, unreconciled base forecast |
| Bottom-up | Forecast items, sum upwards |
| Top-down | Average proportions; proportion of averages |
| Middle-out | Forecast at Store level, sum up, split down to items |
| Optimal reconciliation | OLS, WLS (structural), MinT-shrink |

## Data

Real SKU-by-state FMCG data for India is proprietary, so the **M5 Forecasting (Walmart, USA)** dataset is used as a stand-in.

- Source: Kaggle, M5 Forecasting - Accuracy
- Files needed: `sales_train_evaluation.csv`, `calendar.csv` (not included in this repo; download from Kaggle)
- Subset: `FOODS_3` department, 50 randomly sampled items (`random_state=42`), across 10 stores in 3 states
- Daily sales are aggregated to months; the first and last partial months are dropped, leaving 63 months (Feb 2011 to Apr 2016)

Hierarchy (514 series in total):

| Level | Series |
|---|---|
| Total | 1 |
| State (CA, TX, WI) | 3 |
| Store | 10 |
| Item (item x store) | 500 |

## Method in brief

- **Base model:** additive Holt-Winters exponential smoothing (statsmodels, seasonal period 12, no trend) on every series. Falls back to seasonal naive if a series has fewer than 4 non-zero values or the fit fails. Forecasts are floored at zero.
- **Reconciliation:** all methods are built from the summing matrix `S` with numpy (`y = S b`).
- **Evaluation:** rolling-origin, 3 windows, 6-month horizon. Results are averaged over windows and series.
- **Metrics:** MASE (main), MAE and bias. Wilcoxon signed-rank test and win rate against the base forecast.

## Repository contents

| File | Description |
|---|---|
| `ATSA_Reconciliation_Project.ipynb` | Full experiment (Colab-ready) |
| `ATSA_Report_E012_E016_E036.docx` | Project report |
| `README.md` | This file |

Files generated when the notebook is run:

| File | Contents |
|---|---|
| `all_results.csv` | MAE, MASE and bias for every window, method and series |
| `table1_mase.csv` | Mean MASE by method and level |
| `table2_gain_pct.csv` | % improvement over the base forecast |
| `table3_intermittency.csv` | Item-level MASE by demand type |
| `table4_winrate.csv` | Win rate and Wilcoxon p-values |
| `insights.txt` | Auto-generated insight sentences |
| `fig1_gain_heatmap.png`, `fig2_total_forecast.png`, `fig3_intermittency.png` | Charts |

## How to run

1. Open the notebook in Google Colab (or any Python 3 environment).
2. Upload `sales_train_evaluation.csv` and `calendar.csv` to the working directory.
3. Run all cells in order. Runtime is about 4 minutes for 50 items and 3 windows.

Requirements: `pandas`, `numpy`, `matplotlib`, `statsmodels`, `scipy`. All are preinstalled on Colab.

Settings (Cell 1): `N_ITEMS = 50`, `H = 6`, `N_WINDOWS = 3`, `SEASON = 12`.

## Notebook structure

1. Settings and data loading
2. Build the hierarchy (summing matrix `S`)
3. Base forecasts and reconciliation methods
4. Rolling-origin evaluation
5. Results tables, significance tests and insights
6. Charts

## Key results (mean MASE, lower is better)

| Method | Total | State | Store | Item | Average |
|---|---|---|---|---|---|
| Seasonal naive | 2.255 | 2.040 | 2.147 | 4.373 | 2.704 |
| Base (unreconciled) | 1.670 | 1.812 | 1.802 | 3.592 | 2.219 |
| Bottom-up | 1.754 | 1.829 | 1.821 | 3.592 | 2.249 |
| Top-down (avg proportions) | 1.670 | 1.955 | 2.164 | 4.787 | 2.644 |
| Top-down (prop. of averages) | 1.670 | 1.906 | 2.068 | 4.582 | 2.556 |
| Middle-out (Store) | 1.612 | 1.780 | 1.802 | 4.811 | 2.501 |
| OLS | 1.663 | 1.827 | 1.824 | 3.714 | 2.257 |
| WLS | 1.666 | 1.798 | 1.802 | 3.624 | 2.222 |
| MinT-shrink | 1.706 | 1.823 | 1.818 | 3.593 | 2.235 |

Main findings:

- Middle-out (Store) is best at the Total, State and Store levels, but is poor at item level (MASE 4.811).
- Bottom-up is best at the Item level (3.592, equal to the base forecast).
- WLS has the best average MASE among reconciliation methods (2.222), but no method beats the unreconciled base forecast on average (2.219).
- Top-down methods lose accuracy at lower levels because fixed historical proportions cannot follow changes in the sales mix.
- Unreconciled forecasts disagree by about 1.5% (Total vs sum of items) and 0.6% (State vs sum of stores).
- Intermittent items (more than 40% zero-sales months) are hardest: MASE is about 7.3 for bottom-up and about 9.4 to 9.7 for top-down.
- The improvement from WLS and MinT-shrink over the base forecast is not statistically significant at the 5% level (Wilcoxon p = 0.081 and 0.157).

See the report for the full discussion, business interpretation and limitations.

## Limitations

- M5 is US grocery data, used as a proxy for Indian FMCG demand.
- Only 50 items from one department and 3 rolling windows, so results may vary with a different sample.
- A single base model (Holt-Winters) is used for all series.
- Reconciliation here changes accuracy only slightly, because the base forecasts are already close to coherent.

## References

See the References section of the report.
