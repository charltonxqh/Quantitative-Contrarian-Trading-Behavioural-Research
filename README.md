# C22 veNTUre Stage 3 — Hidden Data Inference Guide

This README explains how to run the submitted frozen Stage 3 model on hidden trade data using `C22_Stage3.ipynb`.

## 1. Set up the environment

From the project directory, create and activate a virtual environment.

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

Install the required dependencies:

```bash
pip install -r requirements.txt
```

## 2. Prepare the trade data

Create these folders if they do not already exist:

```text
data/
├── User Trades/
└── Unseen User Trades/
```

Place all trade files that are historically available **before the hidden evaluation period** in:

```text
data/User Trades/
```

Place the new hidden-test trade files in:

```text
data/Unseen User Trades/
```

For example, if campaigns 67–76 are the hidden test and campaigns 33–66 are already available historically:

```text
data/
├── User Trades/
│   └── campaigns 33–66
└── Unseen User Trades/
    └── campaigns 67–76
```

Historical trades are used only to reconstruct chronological pre-trade behavioural features. They are **not** used to retrain the frozen model or reselect its decision threshold.

## 3. Frozen model files

The submitted frozen model artefacts are stored in:

```text
final_model/
├── exp10b_model.joblib
└── exp10b_model_metadata.json
```

The **Universal entry point** in `C22_Stage3.ipynb` should therefore use:

```python
FROZEN_MODEL_PATH = Path(
    "final_model/exp10b_model.joblib"
)

FROZEN_MODEL_METADATA_PATH = Path(
    "final_model/exp10b_model_metadata.json"
)
```

## 4. Run hidden-data inference

Open:

```text
C22_Stage3.ipynb
```

Navigate to:

```text
Universal entry point
```

Run **all cells under this section in order**.

The entry point will automatically:

1. load the historical and unseen trade files;
2. reconstruct chronological trade history;
3. generate the leakage-safe pre-trade features required by the frozen model;
4. assert that banned post-trade variables are not passed into the model;
5. load the frozen model and frozen numerical threshold;
6. produce one prediction and fade decision per unseen trade; and
7. export the results.

No retraining, hyperparameter search, threshold recalibration, or realised hidden-test outcomes are required.

## 5. Output

Predictions are exported to:

```text
outputs/stage3/unseen_trade_decisions.csv
```

The `outputs/stage3/` directory is created automatically when the results are saved and does not need to be prepared manually.

The output contains one row per unseen trade, including trade identifiers, predicted reverse profit per lot, the frozen decision threshold, and the final fade decision.

## 6. Submitted package

```text
C22_Stage1.ipynb
C22_Stage2.ipynb
C22_Stage3.ipynb
final_model/
backtest/
trials/
validation/
requirements.txt
README.md
```

`C22_Stage3.ipynb` contains the complete Stage 3 modelling workflow and the Universal entry point. The remaining folders contain the frozen model and the exported backtest, trials-log, and validation results.
