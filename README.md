# Liver Disease Prediction — a Clinician's Evaluation of a Simple ML Model

**Mariana Rodrigues, MD** · Critical Care Physician · Medical Affairs · Clinical Research (GCP) · AI in HealthTech

> A physician-led evaluation of a machine-learning model: not just *"how accurate is it?"* but *"would it be safe to use, and for whom?"*

## Summary

I trained a logistic regression model to predict liver disease from age and routine liver tests (Indian Liver Patient Dataset, 583 patients). The first version looked reasonable on paper: **91% specificity** and **64% accuracy**. But it **missed 47% of patients with liver disease** (sensitivity 53%). For a screening or triage tool, that is the failure mode that matters most.

This project documents how I diagnosed and addressed that problem, and why I still would not deploy the model.

## What I did

| Step | Why it matters clinically |
|---|---|
| Compared 4 model variants with 5-fold cross-validation | Avoids judging a model on one lucky split |
| Log-transformed bilirubin, ALP, ALT and AST | Lab values span orders of magnitude; extreme results otherwise dominate |
| Showed how `class_weight="balanced"` *hurt* sensitivity | In this dataset the healthy group is the minority, so "balancing" favours it |
| Tuned the decision threshold for ≥90% sensitivity (on training data only) | The cut-off is a clinical decision, not a default of 0.5 |
| Evaluated AUC, sensitivity, specificity, PPV, NPV and missed cases on an untouched test set | Accuracy alone hides missed diagnoses |
| Checked calibration (Brier score, calibration curve) | Predicted probabilities must mean what they say |
| Interpreted predictors as odds ratios, and flagged collinearity | Coefficients must make physiological sense |
| Assessed performance by age band | Models can fail silently in subgroups; here, younger patients were under-detected |

## Key results (test set, n = 116)

| Model | AUC | Sensitivity | Specificity | Missed cases |
|---|---|---|---|---|
| Original (raw values, balanced weights, threshold 0.50) | 0.77 | 53% | 91% | 39 of 83 |
| Final (log-transformed, no class weighting, threshold 0.49) | 0.74 | 90% | 18% | 8 of 83 |

**What this means:**
- Missed cases fell from 39 to 8. The gain came mainly from **removing class weighting**, not from moving the threshold (0.49 is almost the default).
- The price is steep: specificity fell to 18%, so about 8 in 10 people *without* liver disease would be flagged. The model performs only marginally better than labelling every patient as diseased, which in this dataset already gives 100% sensitivity.
- Discrimination is modest (AUC 0.74–0.77).
- **Age bias:** in patients over 60 the model flags almost everyone (specificity 7%), while in patients under 40 it misses more cases (sensitivity 82%).

## Would I deploy it? No.

- Single regional dataset, with no external validation
- Disease prevalence of ~71%, far above any real screening population, so predictive values do not transfer
- Outcome ("liver patient") not defined by a reference standard
- No history, serology, imaging or sex variable, which is information any clinician would use
- Small sample, so wide uncertainty, and subgroup findings are exploratory

**What it would take:** a clearly defined outcome, external and prospective validation, calibration in the target population, and a defined clinical pathway for flagged patients.

## Repository

- `liver_model_clinical_evaluation.ipynb`: full analysis with clinical commentary
- Data: Indian Liver Patient Dataset (ILPD), UCI Machine Learning Repository. Download it there and place it next to the notebook.

**Tools:** Python, pandas, scikit-learn, matplotlib, seaborn

*Educational project. Not intended for clinical use.*
