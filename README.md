# 🌀 Bantay-Bagyo — Typhoon Impact Modelling

**Predicting typhoon impact severity for Metro Manila cities by fusing storm-track physics, flood hazard, and socioeconomic vulnerability into a single classifier.**

![Python](https://img.shields.io/badge/Python-3776AB?style=flat&logo=python&logoColor=white)
![pandas](https://img.shields.io/badge/pandas-150458?style=flat&logo=pandas&logoColor=white)
![scikit-learn](https://img.shields.io/badge/scikit--learn-F7931E?style=flat&logo=scikit-learn&logoColor=white)
![XGBoost](https://img.shields.io/badge/XGBoost-337AB7?style=flat)

> **Scope & attribution.** This repository contains the **impact-modelling component** I built within the broader Bantay-Bagyo project. The upstream storm-cluster assignments and rapid-intensification flags were produced by the Bantay-Bagyo team and enter this work as input features — I consumed them, I did not build them. Everything in this notebook (the data fusion, the modelling, the evaluation) is my contribution.

---

## The question

When a typhoon approaches Metro Manila, *which* cities get hit hardest — and why? Wind speed alone doesn't answer it. The same storm produces very different outcomes across cities depending on flood exposure and how vulnerable the population is. This component frames impact as **hazard × vulnerability**, not meteorology alone: it learns to classify a (city, typhoon, year) into one of three impact-severity bands from the storm's physical characteristics combined with where it lands and who lives there.

---

## What I built

An end-to-end pipeline that integrates four independent data sources into one modelling table, then runs a disciplined model-selection process on top of it.

### Data fusion (the hard part)

The real work is joining messy, mismatched sources on keys that don't line up cleanly:

| Source | Contribution | Join key |
|---|---|---|
| **IBTrACS** storm tracks | Wind (max/ave), pressure (min), wind radii (R34/R50/R64), gusts, storm speed & direction — aggregated from track points to **storm-level** features | `(year, typhoon_intl == NAME)` |
| **Rapid-intensification** *(upstream input)* | RI occurrence + first-RI-window flags | `SID` |
| **Flood hazard (5-yr)** | Per-city share of area at low/medium/high flood exposure (ADM3) | `city_std` |
| **Poverty / vulnerability** | Poverty rate and poor-household counts (2018) | `city_std` |

Reconciling these meant standardizing city keys, mapping **PAGASA local typhoon names → international names**, and resolving the `(city ↔ typhoon ↔ year)` grain so hazard and socioeconomic features attach to the right impact records.

### Modelling

After consistency checks, null handling, distribution review, and a multicollinearity pass, I compared five classifiers — **Perceptron, AdaBoost, SVM (RBF), Naive Bayes, and XGBoost** — selected on cross-validated macro-F1, then tuned the winner (XGBoost) and inspected feature importance.

---

## Results — read these honestly

XGBoost was the clear winner. On the held-out test set, the tuned model reaches:

| Metric | Test set | 5-fold CV |
|---|---|---|
| Accuracy | 0.85 | — |
| Macro F1 | **0.82** | **0.47 ± 0.06** |

**The gap between the test and CV numbers is the real story, and I'm surfacing it deliberately.** The test set is small (34 records; the most severe band has only 5), so a single split flatters the model. The 5-fold CV is the more honest estimate of how it generalizes, and it says the signal is real but modest — meaningfully better than chance across a 3-class problem, not a solved one. This is a **small-data regime**: the number of well-recorded NCR typhoon-impact events is genuinely limited, and no amount of tuning manufactures records that don't exist.

I'd rather present a model I understand the limits of than a headline accuracy I can't defend. The value here is the **fusion methodology and the evaluation discipline**, not a leaderboard number.

---

## Repository structure

```
bantay_bagyo/
├── typhoon_impact_model.ipynb   # full pipeline: fusion → cleaning → modelling → evaluation
└── typhoon_impact.pdf           # write-up
```

## Tech stack

**Data:** pandas · NumPy — multi-source merging, key reconciliation, feature engineering
**ML:** scikit-learn (Perceptron, AdaBoost, SVM, Naive Bayes) · XGBoost · cross-validated model selection
**Viz:** matplotlib

---

## Limitations & next steps

- **More storm-years.** The single highest-leverage improvement is simply more labelled impact events — the current ceiling is data volume, not modelling.
- **City-specific hazard.** Storm-level features are currently shared across NCR cities for a given typhoon; distance-to-track would let hazard vary by city.
- **Spatial / grouped cross-validation.** With so few storms, grouping CV folds by storm (so the same event can't leak across train/test) would give an even more conservative — and more trustworthy — generalization estimate.
- **Calibration & uncertainty** on the severity probabilities, so the output supports prioritization rather than a hard label.

---

*Impact-modelling component of the Bantay-Bagyo project. Built in Python — multi-source data fusion to a typhoon impact-severity classifier for Metro Manila.*
