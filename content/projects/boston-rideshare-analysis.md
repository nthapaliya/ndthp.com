---
title: "Boston Rideshare Price Analysis"
date: 2024-12-01
tags: ["data analysis", "scikit-learn", "regression", "jupyter"]
summary: "Full ML pipeline on 637K Uber/Lyft rides: data cleaning, model comparison, and feature importance."
---

**Course:** CSCI E-1090A (Introduction to Data Science), Harvard Extension School, Fall 2024

**Group:** Niraj Thapaliya, Jussan Da Silva Bahia Nascimento, Tim Niko

**Code:** [github.com/nthapaliya/boston-rideshare-analysis](https://github.com/nthapaliya/boston-rideshare-analysis)

**Notebook:** [View on Github](https://github.com/nthapaliya/boston-rideshare-analysis/blob/main/boston-rideshare-analysis.ipynb)

---

## The question

What are the most important factors determining Uber and Lyft ride prices in Boston:
distance, weather, time of day, neighborhood, or service tier?

## The pipeline

693,071 rides from late November to mid-December 2018, joined with hourly weather data.
After dropping Uber Taxi rides (all had missing prices — a structurally different product),
duplicates, and scrambled geo columns, the final dataset is 637,036 rows and 13 predictors.

Key cleaning decisions:
- Reduced 37 weather variables to 3 (temperature, wind speed, precipitation probability)
  after finding 26 of 37 are highly collinear and all have near-zero correlation with price
- Log-transformed the response variable for regression
- One-hot encoded 12 Boston neighborhoods (source and destination) and 13 product types

Six models compared by 5-fold cross-validation R², then the final model evaluated on a
held-out test set with permutation importance and bootstrap coefficient analysis to rank features.

## Results

| Model | Mean CV R² |
|-------|------------|
| Baseline (distance + surge only) | 0.1439 |
| Multilinear Regression | **0.9385** |
| Polynomial (degree 2) | 0.9410 |
| Interaction term | 0.9386 |
| Decision Tree | 0.9354 |
| Random Forest | 0.9373 |
| Gradient Boosting | 0.9155 |

Final model (**Multilinear Regression**, chosen for interpretability): **test R² = 0.9381**.

Feature importance ranking (permutation): **product type**, **distance**, **cab type**
(Uber vs. Lyft), **surge multiplier**. Weather contributes essentially nothing — the
dataset spans three uniformly cold winter weeks with almost no weather variation.

Uber rides are cheaper than Lyft on average. Theatre District and North End rides are
priced higher than Back Bay; Financial District and Boston University origins are cheaper.

## Dataset

[Uber & Lyft Dataset Boston, MA](https://www.kaggle.com/datasets/brllrb/uber-and-lyft-dataset-boston-ma)
on Kaggle — 693,071 rides across 12 neighborhoods near downtown Boston.

## Stack

Python, pandas, scikit-learn (LinearRegression, Lasso, DecisionTreeRegressor,
RandomForestRegressor, GradientBoostingRegressor, GridSearchCV), statsmodels (VIF),
folium, seaborn, uv
