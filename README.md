# Causal Effect of Payment Protection Plans on Credit Default

Causal inference analysis of the effect of **Payment Protection Plan (PPP) participation** on credit default using observational data.

The project compares several strategies for confounder adjustment and two causal-effect estimators:

- outcome regression / g-computation;
- stabilized inverse-probability weighting (IPW);
- full-covariate adjustment;
- LASSO-based variable selection;
- Rank-PC causal structure learning;
- local IAMB Markov-blanket selection.

This project was developed for **Applied Statistics (MATH-516) at EPFL**.

## Overview

The goal is to estimate the causal effect of participation in a Payment Protection Plan on the probability of credit default.

The analysis distinguishes:

- **Treatment `A`**: participation in the Payment Protection Plan;
- **Outcome `Y`**: credit default;
- **22 candidate covariates** describing demographic, credit, billing and payment information.

Because treatment was not randomly assigned, the observed difference in default rates cannot automatically be interpreted as causal.

The project therefore investigates how the estimated effect changes under different approaches to confounder adjustment.

## Research Questions

The analysis focuses on three main questions:

1. What is the observed association between PPP participation and credit default?
2. How does adjustment for observed confounding affect the estimated causal effect?
3. How sensitive is the estimated effect to the method used to select adjustment variables?

## Data

The analysis uses an observational credit dataset contained in:

```text
low2036upd.csv
```

The dataset contains:

- treatment indicator `A`;
- default outcome `Y`;
- 22 candidate covariates.

Examples of available variables include:

- credit limit;
- age;
- education;
- repayment history;
- bill amounts;
- previous payment amounts.

In the analyzed sample:

| Group | Observed default rate | N |
|---|---:|---:|
| No PPP (`A = 0`) | 22.74% | 299 |
| PPP (`A = 1`) | 34.33% | 201 |

The corresponding crude risk difference is approximately **11.59 percentage points**.

This difference is descriptive rather than automatically causal because treatment assignment may be confounded.

## Causal Estimation

Two complementary estimators are implemented.

### Outcome Regression / G-Computation

A logistic outcome model is fitted for default conditional on treatment and the selected adjustment variables.

For every individual, the fitted model is then used to predict:

```text
Y(A = 1)
```

and

```text
Y(A = 0)
```

The average difference between these counterfactual predictions provides an estimate of the **average causal effect (ACE)** on the risk-difference scale.

### Stabilized Inverse-Probability Weighting

A propensity-score model estimates:

```text
P(A = 1 | X)
```

where `X` contains the selected adjustment variables.

Stabilized inverse-probability weights are then used to construct a weighted pseudo-population in which treatment assignment is less associated with the measured covariates.

The difference in weighted default risk between treated and untreated observations provides a second estimate of the ACE.

## Adjustment Strategies

A central part of the project is comparing different ways of selecting the variables used for causal adjustment.

### 1. Full Covariate Adjustment

All **22 candidate covariates** are included.

This provides a broad adjustment strategy without performing variable selection.

### 2. Outcome-Based LASSO

Cross-validated logistic LASSO is used to identify variables predictive of the outcome.

The selected adjustment variables were:

```text
PAY_0
PAY_2
PAY_6
```

### 3. Rank-PC Neighborhood

A Rank-PC procedure is used to estimate the local dependence structure around the outcome.

The resulting adjustment set was:

```text
PAY_0
```

### 4. Local IAMB Markov Blanket

A local Incremental Association Markov Blanket procedure is used to identify variables conditionally associated with the outcome.

The selected adjustment set was:

```text
PAY_0
PAY_2
BILL_AMT1
```

## Results

The estimated average causal effects are shown below.

All effects are expressed as **risk differences**.

| Adjustment strategy | Estimator | Variables | ACE | 95% bootstrap CI |
|---|---|---:|---:|---:|
| Unadjusted | Crude difference | 0 | 0.1159 | — |
| All covariates | Outcome regression | 22 | 0.1189 | [0.0299, 0.2069] |
| All covariates | Stabilized IPW | 22 | 0.1388 | [0.0461, 0.2434] |
| LASSO | Outcome regression | 3 | 0.1162 | [0.0456, 0.1920] |
| LASSO | Stabilized IPW | 3 | 0.1216 | [0.0388, 0.1983] |
| Rank-PC | Outcome regression | 1 | 0.1145 | [0.0431, 0.1870] |
| Rank-PC | Stabilized IPW | 1 | 0.1183 | [0.0376, 0.1979] |
| Local IAMB | Outcome regression | 3 | 0.1216 | [0.0473, 0.1918] |
| Local IAMB | Stabilized IPW | 3 | 0.1283 | [0.0648, 0.1961] |

Confidence intervals were estimated using **300 bootstrap samples** for each adjusted estimator.

Despite substantial differences in adjustment-set size, the estimated effects remain relatively similar across the approaches considered.

Interpretation as a causal effect nevertheless depends on the standard identification assumptions of observational causal inference, including adequate control of confounding, positivity and correct model specification.

## Diagnostics

The project does not rely only on point estimates.

Several diagnostics are used to evaluate the plausibility and behavior of the adjustment procedures.

### Propensity-Score Overlap

For the full adjustment model, estimated propensity scores are inspected across treatment groups to assess whether comparable treated and untreated observations exist throughout the relevant covariate space.

The fitted full model produced propensity scores ranging approximately from:

```text
0.006 to 0.942
```

### Weight Diagnostics

The stabilized IPW weights are inspected for extreme values.

For the full adjustment model, the observed weight range was approximately:

```text
0.427 to 11.651
```

### Covariate Balance

Standardized mean differences are computed before and after weighting.

This allows the analysis to assess whether IPW reduces systematic differences in measured covariates between the treated and untreated groups.

### Bootstrap Uncertainty

Each adjusted estimate is accompanied by a bootstrap confidence interval rather than reporting only a point estimate.

## Experimental Pipeline

```text
Observational credit data
          |
          v
 Treatment / outcome definition
          |
          v
 Baseline imbalance analysis
          |
          +--------------------------------+
          |               |                |
          v               v                v
       LASSO          Rank-PC          Local IAMB
          |               |                |
          +---------------+----------------+
                          |
                          v
                 Adjustment sets
                          |
               +----------+----------+
               |                     |
               v                     v
      Outcome regression      Propensity model
        / g-computation               |
               |                      v
               |               Stabilized IPW
               |                     |
               +----------+----------+
                          |
                          v
                  ACE estimation
                          |
                          v
               Bootstrap uncertainty
                          |
                          v
          Overlap and balance diagnostics
```

## Technical Components

### Statistical Modeling

The project uses logistic generalized linear models for both:

- outcome modeling;
- propensity-score estimation.

### Penalized Regression

Cross-validated LASSO is used as a data-driven variable-selection strategy.

### Causal Structure Learning

Rank-PC and local IAMB provide alternatives to purely predictive variable selection by investigating conditional-dependence structure around the outcome.

### Resampling

Bootstrap resampling is used to quantify uncertainty in the causal-effect estimates.

### Diagnostics

The analysis includes:

- propensity-score overlap;
- stabilized-weight inspection;
- standardized mean differences;
- before/after covariate-balance comparisons.

## Main Takeaways

- Implemented two complementary causal estimators: **g-computation/outcome regression** and **stabilized IPW**.
- Compared causal estimates across four different adjustment strategies.
- Applied **LASSO, Rank-PC and IAMB** for data-driven covariate selection.
- Used bootstrap resampling to quantify uncertainty.
- Evaluated propensity-score overlap, weight stability and covariate balance.
- Found broadly consistent positive effect estimates across adjustment strategies, subject to the identification assumptions required for observational causal inference.

## Repository Structure

```text
causal-inference-credit-default/
├── .github/
├── figures/
├── Code.ipynb
├── harris13a.pdf
├── low2036upd.csv
└── report.pdf
```

### `Code.ipynb`

Contains the complete analysis pipeline, including:

- preprocessing;
- exploratory diagnostics;
- outcome regression;
- stabilized IPW;
- LASSO selection;
- Rank-PC;
- local IAMB;
- bootstrap inference;
- balance and overlap diagnostics;
- figures and result summaries.

### `figures/`

Contains figures generated during the analysis.

### `report.pdf`

Contains the accompanying project report and methodological discussion.

## Running the Analysis

A minimal Python environment requires:

```bash
pip install numpy pandas scipy matplotlib networkx statsmodels scikit-learn jupyter
```

Then launch:

```bash
jupyter notebook Code.ipynb
```

The notebook expects `low2036upd.csv` to be available in the repository root and writes generated plots to the `figures/` directory.

## Context

This project was completed as part of:

**MATH-516 — Applied Statistics**  
École polytechnique fédérale de Lausanne (EPFL), 2026.

The project focuses on observational causal inference, confounder adjustment, causal variable selection, propensity-score methods and statistical diagnostics.