# Brain-Age Prediction from Multi-Site Clinical MRI Data

This repository contains the machine-learning experiments developed for my MSc thesis, **“Which Brain-Age Model Predicts Age Best Across the Clinical Dataset?”** The project compares Support Vector Regression (SVR), Lasso Regression, and XGBoost for predicting chronological age from structural MRI-derived cortical measurements.

The analysis also examines whether the resulting brain-age gap can distinguish healthy controls from participants with schizophrenia, bipolar disorder, ADHD, dementia, or mild cognitive impairment. Models are evaluated separately for male and female participants to explore variation in predictive performance across the two groups.

> **Research use only:** This project is an academic analysis and is not a clinical diagnostic tool.

## Research question

**Which brain-age model predicts chronological age most reliably across a multi-site clinical dataset?**

The study addresses two related objectives:

1. Compare the age-prediction performance of SVR, Lasso Regression, and XGBoost.
2. Evaluate how well age-adjusted brain-age gaps separate healthy controls from individual diagnostic groups.

## Dataset

The analysis uses a multi-site neuroimaging dataset derived from the work of Kia et al. (2020), *Hierarchical Bayesian Regression for Multi-Site Normative Modeling of Neuroimaging Data*.

The thesis analysis dataset contains:

- 6,676 participants and 156 variables
- Ages ranging from 8 to 97 years
- 3,068 male and 3,608 female participants
- 6,190 healthy controls and 486 participants with a diagnosis
- Data from 32 scanner/site identifiers
- Structural MRI-derived measurements, including regional cortical thickness and summary thickness measures

Diagnostic labels are encoded as follows:

| Code | Group |
|---:|---|
| 0 | Healthy control |
| 1 | Schizophrenia (SZ) |
| 2 | Bipolar disorder (BD) |
| 3 | ADHD |
| 4 | Dementia |
| 5 | Mild cognitive impairment (MCI) |

The analysis-ready data files are not included in this repository. Access to the source data should be arranged through the original study and its associated data providers, subject to their terms of use.

## Methodology

The experimental workflow is implemented separately for male and female participants:

1. **Data exploration** – inspect participant demographics, diagnosis distributions, missing values, and cortical-thickness patterns.
2. **Sex-stratified analysis** – divide the dataset into male and female subsets.
3. **Multi-site harmonization** – use NeuroCombat to reduce scanner and site effects while retaining age as a biological covariate.
4. **Preprocessing** – impute missing numerical values and scale model features.
5. **Training strategy** – train age-prediction models primarily on healthy controls.
6. **Grouped validation** – use 10-fold `GroupKFold`, grouped by `site_id`, to reduce leakage between scanner sites.
7. **Brain-age gap analysis** – calculate the difference between predicted and chronological age and adjust it for age-related bias using ordinary least squares regression.
8. **Clinical-group evaluation** – calculate ROC AUC scores for each diagnosis versus healthy controls using the adjusted brain-age gap.

## Models

| Model | Role in the study | Notebook implementation |
|---|---|---|
| Support Vector Regression | Non-linear baseline | Polynomial kernel |
| Lasso Regression | Linear, regularized comparison model | `LassoCV` with cross-validated regularization |
| XGBoost | Tree-based ensemble model | 100 estimators, learning rate 0.1, maximum depth 3, squared-error objective |

## Results

### Brain-age prediction

The following MAE values are reported in the thesis. Lower values indicate more accurate age predictions.

| Dataset | SVR | Lasso Regression | XGBoost |
|---|---:|---:|---:|
| Male | 9.169 | **7.261** | 8.651 |
| Female | **7.812** | 12.260 | 8.807 |

No single model achieved the lowest MAE for both groups. Lasso produced the lowest reported male MAE, while SVR produced the lowest female MAE. XGBoost was the most consistent model across the two groups, with only a small difference between male and female MAE, supporting its selection as the most reliable overall model in the thesis.

### Healthy controls versus diagnostic groups

ROC AUC scores reported in the thesis are shown below. These scores evaluate the discriminative signal in the adjusted brain-age gap; they should not be interpreted as the performance of a deployable diagnostic classifier.

| Comparison | Female SVR | Female Lasso | Female XGBoost | Male SVR | Male Lasso | Male XGBoost |
|---|---:|---:|---:|---:|---:|---:|
| Healthy vs SZ | 0.51 | 0.50 | 0.52 | 0.27 | 0.38 | **0.46** |
| Healthy vs BD | 0.44 | 0.01 | **0.61** | 0.32 | 0.34 | **0.50** |
| Healthy vs ADHD | **0.60** | 0.16 | 0.44 | 0.49 | **0.52** | 0.49 |
| Healthy vs Dementia | 0.55 | **0.65** | 0.45 | 0.50 | **0.53** | 0.48 |
| Healthy vs MCI | 0.57 | **0.62** | 0.50 | 0.63 | 0.68 | **0.74** |

The strongest result was obtained by XGBoost for male healthy controls versus MCI (AUC = 0.74). Performance varied substantially by diagnosis, model, and participant group, reinforcing the exploratory nature of this analysis.

## Repository contents

| Notebook | Analysis |
|---|---|
| `Male_SVR_Final_Thesis_May20th (1).ipynb` | Male SVR experiment |
| `Male_Lasso_Final_Thesis_May20th (1).ipynb` | Male Lasso experiment |
| `Male_XG_Boost_Final_Thesis_May20th (1).ipynb` | Male XGBoost experiment |
| `Female_SVR_Final_Thesis_May20th (1).ipynb` | Female SVR experiment |
| `Female_Lasso_Final_Thesis_May20th (1) (1).ipynb` | Female Lasso experiment |
| `Female_XG_Boost_Final_Thesis_May20th (1).ipynb` | Female XGBoost experiment |

Each notebook covers data loading, NeuroCombat harmonization, preprocessing, model fitting, grouped cross-validation, error analysis, age-gap adjustment, and ROC AUC evaluation for its corresponding model and participant group.

## Technologies

- Python
- Jupyter Notebook / Google Colab
- pandas and NumPy
- scikit-learn
- XGBoost
- NeuroCombat
- statsmodels
- Matplotlib

## Running the notebooks

### 1. Clone the repository

```bash
git clone https://github.com/<your-username>/brain-age-prediction-clinical-mri.git
cd brain-age-prediction-clinical-mri
```

### 2. Create and activate a virtual environment

```bash
python -m venv .venv
```

On macOS or Linux:

```bash
source .venv/bin/activate
```

On Windows PowerShell:

```powershell
.venv\Scripts\Activate.ps1
```

### 3. Install the dependencies

```bash
pip install jupyter pandas numpy scikit-learn xgboost neuroCombat statsmodels matplotlib
```

### 4. Add the data

The notebooks expect the following analysis-ready files:

```text
male_data_updated.csv
female_data_updated.csv
```

The original notebooks were developed in Google Colab and contain paths such as:

```python
/content/drive/MyDrive/Thesis_May20th/male_data_updated.csv
```

Update each `pd.read_csv(...)` call to match the location of the data in your environment. If running locally, remove or skip the Google Drive mounting cell.

### 5. Start Jupyter

```bash
jupyter notebook
```

Run a notebook from top to bottom after updating its data path. The notebooks retain intermediate exploratory cells and saved outputs from the thesis workflow, so a clean rerun is recommended when reproducing results.

## Limitations

- The dataset is highly imbalanced: healthy controls represent approximately 92.7% of participants.
- Clinical subgroup sample sizes are substantially smaller than the healthy-control group.
- Scanner, acquisition, and site differences may remain even after harmonization.
- Results vary by model, diagnosis, and participant group and may not generalize to other populations.
- The analysis uses structural MRI-derived features rather than raw MRI volumes.
- The notebooks represent academic research code and would require additional refactoring, validation, and independent testing before any production or clinical use.

## Thesis citation

If you use or discuss this repository, please cite:

```text
Kelesi, N. (2024). Which Brain-Age Model Predicts Age Best Across the Clinical Dataset?
MSc thesis, Tilburg University.
```

## Author

**Nikoleta Kelesi**  
MSc Data Science & Society, Tilburg University

