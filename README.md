# Quantitative Contrarian Trading & Behavioural Research

A quantitative research project investigating whether behavioural patterns in trader activity can identify trades where taking the opposite side has positive expected value.

The project combines behavioural finance, statistical hypothesis testing, leakage-safe feature engineering, walk-forward validation, and machine-learning models to develop and evaluate a selective contrarian trading strategy.

## Research Question

Can patterns in trading behaviour help identify trades that are more attractive to fade?

Rather than assuming that one behavioural feature provides a reliable edge, the project tests whether multiple weak pre-trade signals can be combined into a useful trading decision.

The modelling target is **reverse profit per lot (RP/lot)**. A positive RP/lot means that taking the opposite side of the original trade would have been profitable.

## Research Workflow

The project was developed in three research stages:

### Stage 1 — Exploratory Behavioural Research

- Audited and standardised campaign-level trading data
- Reconstructed chronological trade and trade-idea histories
- Investigated behavioural hypotheses such as holding duration, trading intensity, and direction switching

### Stage 2 — Feature Research

- Engineered leakage-safe pre-trade behavioural features
- Tested individual signals with walk-forward analysis
- Evaluated robustness using statistical tests, effect sizes, confidence intervals, and campaign-level diagnostics

Feature families included:

- post-loss behaviour
- historical and recent trading activity
- trade timing
- challenge state and drawdown pressure
- position sizing
- stop-loss / take-profit behaviour
- historical trading performance

### Stage 3 — Model Development

Classification, direct regression, and two-stage expected-value formulations were compared using **expanding-window walk-forward validation**.

The development process also tested:

- reconstructed SL/TP signals
- feature winsorisation
- target clipping
- feature redundancy reduction
- signed-log targets
- quantile regression
- neural-network regression

The strongest candidate was **Experiment 10B**, a reduced-feature LightGBM regression model using 22 leakage-safe behavioural features.

## Final Model

| Component | Final choice |
|---|---|
| Model | LightGBM regression |
| Features | 22 leakage-safe pre-trade behavioural features |
| Development scheme | Expanding-window walk-forward |
| Development campaigns | Up to Campaign 52 |
| Validation campaigns | Campaigns 53–66 |
| Selection | Development data only |
| Decision rule | Fade when predicted RP/lot exceeds the frozen threshold |

The selected numerical threshold and exact feature order are stored with the model metadata in `models/`.

## Validation Results

On campaigns 53–66, the selected model produced:

| Metric | Result |
|---|---:|
| Trades faded | 2,065 |
| Coverage | 9.67% |
| Mean reverse profit per lot | 25.74 |
| Lot-weighted reverse profit per lot | 16.32 |
| Positive campaign share | 64.3% |
| Median campaign RP/lot | 19.47 |

The strategy is intentionally selective: it acts on a relatively small high-score subset instead of attempting to fade most trades.

### Uncertainty

Account-cluster bootstrap was used because multiple trades from the same account are not independent.

| Metric | Bootstrap mean | 95% CI | Positive bootstrap samples |
|---|---:|---:|---:|
| Mean RP/lot | 27.24 | [-12.89, 66.12] | 91.0% |
| Lot-weighted RP/lot | 16.94 | [-11.44, 43.29] | 88.9% |

The confidence intervals still cross zero, so the observed edge should be interpreted as **promising rather than statistically established**.

A Bonferroni-adjusted bootstrap interval was also used as a multiple-testing robustness check for the final model-threshold search family.

## Repository Structure

```text
.
├── notebooks/
│   ├── 01_stage1_eda.ipynb
│   ├── 02_stage2_feature_research.ipynb
│   ├── 03_stage3_model_building.ipynb
│   └── 04_end_to_end_pipeline.ipynb
│
├── data/
│   ├── User Trades/
│   └── Unseen User Trades/
│
├── models/
│   ├── exp10b_model.joblib
│   └── exp10b_model_metadata.json
│
├── results/
│   ├── backtest/
│   ├── trials/
│   ├── validation/
│   └── inference/
│
├── requirements.txt
├── .gitignore
└── README.md
```

The first three notebooks preserve the research process. `04_end_to_end_pipeline.ipynb` is the clean inference entry point for running the frozen strategy from raw trade files to final fade decisions.

## Installation

Clone the repository and create a virtual environment.

### macOS / Linux

```bash
python3 -m venv venv
source venv/bin/activate
```

### Windows

```bash
python -m venv venv
venv\Scripts\activate
```

Install the dependencies:

```bash
pip install -r requirements.txt
```

## Running the End-to-End Pipeline

Place historically available trade files in:

```text
data/User Trades/
```

Place the new trade files to score in:

```text
data/Unseen User Trades/
```

Then open:

```text
notebooks/04_end_to_end_pipeline.ipynb
```

and run all cells in order.

The notebook performs:

```text
raw trade files
    ↓
chronological preprocessing
    ↓
behavioural feature engineering
    ↓
leakage and schema checks
    ↓
frozen LightGBM regression
    ↓
frozen decision threshold
    ↓
fade decisions
```

Predictions are exported to:

```text
results/inference/unseen_trade_decisions.csv
```

The output directory is created automatically if it does not already exist.

The historical files are used only to reconstruct chronological behavioural information available before each new trade. They do **not** retrain the model or recalibrate the decision threshold.

## Backtests and Research Outputs

Curated research outputs are stored under `results/`:

- `results/backtest/` — overall and campaign-level strategy comparisons
- `results/trials/` — experiment and model-search logs
- `results/validation/` — development-to-validation diagnostics
- `results/inference/` — generated end-to-end pipeline predictions

Temporary intermediate outputs should remain untracked.

## Key Research Findings

Several findings shaped the final model:

1. A large feature space generalised poorly despite containing potentially useful behavioural information.
2. Reconstructed SL/TP features did not improve the strategy enough to justify losing roughly ten campaigns of training history.
3. Outlier handling through feature winsorisation or target clipping did not consistently solve the instability.
4. Several behavioural feature families contained substantial redundancy.
5. Reducing the feature space from 40 to 22 representative features materially improved direct regression.
6. More historical data did not always improve generalisation, suggesting that behaviour may vary across campaigns.
7. The final model is better at identifying a selective high-score subset than at accurately ranking the entire trade distribution.

## Limitations

The final model does not produce a monotonic ranking of realised RP/lot across the full score distribution. Performance also varies between campaigns, and the account-cluster bootstrap confidence intervals include zero.

The results therefore provide evidence of a possible selective fade signal, but not definitive proof of a persistent trading edge.

## Future Work

Potential extensions include:

- rebuilding the SL/TP reconstruction pipeline using stronger models or alternative representations while preserving more training history;
- studying campaign or behavioural regime shifts explicitly;
- improving probability and score calibration;
- testing alternative ranking and selective-prediction objectives;
- evaluating the frozen strategy on additional genuinely unseen campaigns.

## Tech Stack

- Python
- pandas / NumPy
- scikit-learn
- LightGBM
- statistical hypothesis testing
- expanding-window walk-forward validation
- account-cluster bootstrap

## Acknowledgements

This project was developed as part of the C22 veNTUre programme. The trading dataset used for the research was provided by C22 for educational and research purposes.

## Data Availability

The underlying trading dataset was provided by C22 and is not included in this repository. The repository contains the research methodology, modelling pipeline and selected derived results only.

## Disclaimer

This repository is a research project for educational and analytical purposes. Reported historical or validation performance does not guarantee future trading performance.
