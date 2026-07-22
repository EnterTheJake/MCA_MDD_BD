# Drug Type and Presence encode information about drug–cognition interaction in Major Depression and Bipolar Disorder: A Multiple Correspondence Analysis approach

Code and analysis notebook accompanying the manuscript of the same title (currently under review).

The study proposes an interpretable framework — **Multiple Correspondence Analysis (MCA)** combined with **regularized (ridge) regression** under nested cross-validation — to characterize **Drug–Cognition Interactions (DCIs)** from real-world baseline pharmacological data, and to predict longitudinal change in cognitive and clinical outcomes in patients with Major Depressive Disorder (MDD, *n* = 28) and Bipolar Disorder (BD, *n* = 24).

## Repository contents

| File | Description |
|------|-------------|
| `Analysis.ipynb` | Main Jupyter notebook. Reproduces the entire analysis and every figure/table reported in the manuscript (see *Notebook structure* below). |
| `pyproject.toml` | Project metadata and pinned Python dependencies. |
| `README.md` | This file. |

> ⚠️ **The data files are not included in this repository.** `T0.csv` and `T1.csv` must be downloaded separately and placed in the repository root before the notebook can run (see *Data availability*).

## Data availability

The de-identified data supporting these analyses are available at:

**https://figshare.com/s/00ac49c9b681a44aa2e3**

Download the two files into the repository root:

| File | Content |
|------|---------|
| `T0.csv` | Baseline (T0) assessment |
| `T1.csv` | Follow-up (T1) assessment, ~3 months after baseline |

Both files are comma-separated (UTF-8) and share the same layout. Columns used by the notebook:

- **`Name`** — anonymized patient identifier, used to match T0 and T1 records.
- **`Diagnosis`** — `1` = Major Depressive Disorder (MDD); `2` = Bipolar Disorder (BD).
- **Medication columns** (`Benzodiazepines` … `Aripiprazole(6)`) — baseline pharmacological exposure, encoded both as **type** (pharmacological-class level) and **presence** (individual-drug, binary) variables.
- **Test columns** (`MMSE` … `FAS`) — the 11 neuropsychological / psychopathological outcomes: MMSE, MoCA, FAB, SPAN-A, SPAN-I, REY_I, REY_D, Vocab, BDI, HDRS, FAS.

Only patients with valid records at **both** time points are retained; each outcome is analysed as a change score **Δ = T1 − T0**.

## Requirements

- **Python ≥ 3.12**
- Packages (pinned in `pyproject.toml`): `numpy`, `pandas`, `matplotlib`, `seaborn`, `prince` (MCA), `scikit-learn`, `jupyterlab`, `ipykernel`.

### Setup

```bash
# with uv (recommended — reads pyproject.toml):
uv sync

# or with pip, in a fresh virtual environment:
python -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install numpy pandas matplotlib seaborn prince scikit-learn jupyterlab ipykernel
```

## How to reproduce

1. Install the requirements (above).
2. Download `T0.csv` and `T1.csv` from figshare (link above) into the repository root.
3. Open and run the notebook from top to bottom:
   ```bash
   jupyter lab Analysis.ipynb
   ```

All analyses use **fixed random seeds** (both the outer/inner `KFold` splits and the MCA solver set a `random_state`), so re-running the notebook reproduces the reported numbers and figures deterministically.

## Notebook structure

| Section (cells) | What it does | Corresponds to |
|-----------------|--------------|----------------|
| Imports & data loading | Load `T0`/`T1`, match patients, split by diagnosis, compute Δ = T1 − T0 change scores | Methods |
| MDD — MCA + ridge (nested CV) | Full model for MDD (selected configuration: **5 components, α = 1.0**); ridge-coefficient and scaled-impact plots | Table 4 (MDD); Figures 1, 3 |
| BD — MCA + ridge (nested CV) | Full model for BD (selected configuration: **5 components, α = 10.0**); same plots | Table 4 (BD); Figures 1, 4 |
| Type-only / presence-only re-evaluation (MDD & BD) | Refit each model using **only type** or **only presence** features | Table 3 |
| Full vs Type vs Presence comparison | Normalized-MSE comparison across encoding strategies | Figure 2 |

### Modelling details

- **Pipeline:** `prince.MCA` (sklearn engine, fixed seed; 5000 iterations for the final refit) → `sklearn.linear_model.Ridge`.
- **Nested cross-validation:** outer and inner **5-fold** `KFold` (shuffled, fixed seeds), grid search over hyperparameters in the inner loop.
- **Hyperparameter grid:** number of MCA components ∈ **{5, 15, 30}** (restricted to values below the number of input features) and ridge regularization strength **α ∈ {0.1, 1, 10}**.
- **Metrics:** mean absolute error (MAE) and Pearson R² per outcome; mean squared error (MSE), with 95% bootstrap confidence intervals, for the encoding-strategy comparison.
- **Interpretation:** for each outcome, medication-level *scaled impact* is obtained by combining each medication's MCA loading with the ridge coefficient of the corresponding component.

## Citation

If you use this code, please cite the accompanying manuscript:

> Catania, G., Guerrera, C. S., et al. *Drug Type and Presence encode information about drug–cognition interaction in Major Depression and Bipolar Disorder: A Multiple Correspondence Analysis approach.* (Manuscript under review.)
