# 📌 Abalone Age Prediction — Statistical Modeling Project (Instructed by Professor Jie Peng)

This repository explores statistical modeling techniques for predicting the **age of abalone** using physical measurements from the UCI Abalone dataset.
We investigate multiple modeling strategies to address **nonlinear relationships**, **influential points**, **multicollinearity**, and the trade-off between **predictive performance**, **model robustness**, and **interpretability**.

The analysis is implemented in **R**, combining classical statistical theory with modern feature selection and model diagnostics.

---

## 🔍 Background & Motivation

The age of an abalone is determined by counting **rings inside its shell**.
Unlike tree rings, abalone rings are not visible externally and can only be observed **after cutting or grinding the shell**, which is destructive and costly.

A **non-destructive, model-based** approach can:

* preserve the animal
* reduce labor cost
* accelerate ecological surveys
* enable large-scale population monitoring

By using **shell dimensions and weight measurements**, we build statistical models that estimate age with minimal measurement effort.

---

## 📊 Data Description

**Source:** UCI Machine Learning Repository
[https://archive.ics.uci.edu/ml/datasets/Abalone](https://archive.ics.uci.edu/ml/datasets/Abalone)

* **4,177 observations**
* **9 variables** (8 predictors + 1 response)

| Variable      | Type        | Description                    |
| ------------- | ----------- | ------------------------------ |
| Sex           | Categorical | M, F, I                        |
| Length        | Numeric     | Longest shell measurement (mm) |
| Diameter      | Numeric     | Perpendicular diameter (mm)    |
| Height        | Numeric     | Height with meat in shell (mm) |
| WholeWeight   | Numeric     | Whole abalone weight (grams)   |
| ShuckedWeight | Numeric     | Meat weight (grams)            |
| VisceraWeight | Numeric     | Gut weight (grams)             |
| ShellWeight   | Numeric     | Dried shell weight (grams)     |
| Rings         | Numeric     | Target variable (age proxy)    |

---

# 🧪 Research Questions

We study several key methodological questions:

1. **Transformation & Linearity:**
   How to induce linearity and normal residuals when the response is strongly skewed?

2. **Influential Observations:**
   How does model performance change after removing outliers identified via Cook’s distance, studentized residuals, and leverage?

3. **Interaction Terms:**
   Do two-way interactions meaningfully improve fit?
   How do we manage multicollinearity and overfitting risks?

4. **Model Comparison:**
   Among all candidate models:

   * which is most interpretable?
   * which is most robust?
   * which gives the best in-sample fit?
   * which gives the best predictive accuracy?

---

# 🛠️ Analysis Workflow

We fit a sequence of models from simple OLS to complex interaction structures, followed by LASSO reduction.
The progression is:

| Model    | Key Feature                                             |
| -------- | ------------------------------------------------------- |
| **fit1** | Raw OLS using `rings`                                   |
| **fit2** | Box–Cox → log(rings)                                    |
| **fit3** | Remove influential points                               |
| **fit4** | Nonlinear transforms of predictors + stepwise selection |
| **fit5** | Refit after additional influential removal              |
| **fit6** | Add two-way interactions + stepwise                     |
| **fit7** | Refit interactions after removal                        |
| **fit8** | LASSO reduction from fit7                               |

---

# 🔍 Exploratory Data Analysis

* No missing values
* Many predictors are **strongly right-skewed**
* `rings` is **left-skewed**
* Male and female adults show **similar size distributions**
* Infants are significantly smaller → merged adult categories
* Scatterplots show **diminishing returns** in size–age trend
* **Strong multicollinearity** among weight and size measures

**Outliers:**
Detected using Tukey’s rule, Cook’s distance, leverage, and studentized residuals.
Two zero-height values are removed.

---

# 🧭 Model Selection Principles

### Normalized Information Criteria

Since removing influential observations changes sample size, we compare models using per-observation criteria for AIC/N and BIC/N

Also:

* **Avg Log-Likelihood**
* **Training MSE**
* **MSPE**
* **Generalization Gap**: ( MSPE - MSE(train) )

This follows statistical learning theory: smaller gap → better robustness.

---

# 📈 Model Comparison (Training)

| Model    | Avg LL     | AIC/n      | BIC/n      | MSPE       | Train MSE  | Gap         |
| -------- | ---------- | ---------- | ---------- | ---------- | ---------- | ----------- |
| fit2     | 0.1829     | -0.3597    | -0.3414    | 0.0409     | 0.0406     | 0.00028     |
| fit3     | 0.2205     | -0.4350    | -0.4166    | 0.0379     | 0.0377     | 0.00024     |
| fit4     | 0.2828     | -0.556     | -0.527     | 0.0337     | 0.0333     | 0.00040     |
| fit5     | 0.3040     | -0.601     | -0.579     | 0.0321     | 0.0319     | 0.00024     |
| fit6     | 0.3197     | -0.627     | -0.588     | 0.0313     | 0.0309     | 0.00045     |
| **fit7** | **0.3402** | **-0.660** | **-0.599** | **0.0303** | **0.0296** | 0.00065     |
| fit8     | 0.2549     | -0.504     | -0.488     | 0.0354     | 0.0352     | **0.00020** |

* **fit7** is the best in-sample candidate
* **fit8** is the most robust (smallest gap)

---

# 🎯 Final Model Evaluation (Test Set)

Back-transformation uses **Duan’s smearing estimator** to correct bias from log-scale predictions

To avoid extreme predictions, log(y) is truncated at the **99th percentile**.

| Model    | MSE      | RMSE     | MAE      | MAPE      | R²        |
| -------- | -------- | -------- | -------- | --------- | --------- |
| fit1     | 4.53     | 2.13     | 1.53     | 15.68     | 0.569     |
| fit4     | 4.12     | 2.03     | 1.45     | 14.56     | 0.608     |
| **fit6** | **4.02** | **2.00** | **1.41** | **14.17** | **0.618** |
| fit7     | 4.03     | 2.01     | 1.42     | 14.27     | 0.617     |
| fit8     | 4.72     | 2.17     | 1.52     | 15.47     | 0.552     |

* **fit6** and **fit7** are the **best predictive models**
* **fit8** reach a balance between robustness and fitting, predictive performance


# 🏁 Conclusion

1. **Transformations matter:**
   Box–Cox/log improves residual shape and linearity.

2. **Outliers are important:**
   Removing influential observations improves both fit and robustness.

3. **Interactions capture flexible effects:**
   `fit6–7` give the **best prediction**, but can overfit and exhibit large generalization gaps.

4. **LASSO is effective:**
   `fit8` provides a **parsimonious model**, removing redundant interactions and improving interpretability.

> **Trade-off:**
> complex models improve accuracy but reduce robustness.
> simple models generalize better but may underfit relevant structure.
