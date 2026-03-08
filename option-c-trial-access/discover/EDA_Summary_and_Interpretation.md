# EDA Summary and Interpretation for Modeling

**Source notebook:** `option-c.ipynb`
**Datasets produced:** `discover/modified_data/state_modeling_final.csv` (51 x 57), `discover/modified_data/county_modeling_final.csv` (3,221 x 42)

---

## 1. Data Acquisition Overview

| Source | Records | Notes |
|--------|---------|-------|
| ClinicalTrials.gov v2 | 3,646 US Type 2 diabetes studies; 47,118 US site rows; 20,407 unique state-trial pairs | Cursor-paginated; condition = diabetes |
| CDC PLACES 2023 | 15 age-adjusted prevalence measures per geography | Socrata SODA API |
| CDC BRFSS 2023 | State-level diabetes prevalence with 95% CIs | Most recent year per state |
| US Census ACS 5-Year 2022 | 11 socioeconomic/demographic variables | State + county levels |
| Census 2020 Decennial | Rural population share by state | P2 table |
| NPI Registry | Endocrinologist counts by state and county | Taxonomy `207RE0101X` |
| AMC / CTSA | Academic Medical Center org counts; CTSA hub indicators | NPI-2 orgs + curated list |

**Missing data notes:**
- Kentucky and Pennsylvania are missing state-level PLACES diabetes burden.
- 458 city-state pairs remain unmatched after county FIPS resolution (89.0% match rate overall).

---

## 2. State-Level EDA Findings

### 2.1 Trial Density Distribution

- **Mean trials per 100k:** ~8.0 (wide spread across states).
- **Top density states:** DC, Montana, Idaho, North Dakota, Nebraska.
- The distribution is right-skewed: a few states have very high per-capita trial density while many cluster at lower values.

### 2.2 Coverage Residual (Burden vs. Trial Density)

The coverage residual is defined as observed `trials_per_100k` minus expected density from an OLS regression on `places_diabetes`. It measures whether a state has more or fewer trials than its disease burden would predict.

| Direction | Top States | Residual |
|-----------|-----------|----------|
| Most under-covered | Wyoming (-8.58), Alaska (-7.66), New Hampshire (-5.11) | Negative |
| Also under-covered | California, Illinois, New Jersey, New York (large-population states) | Negative |
| Most over-covered | Idaho (+7.48), North Dakota (+6.57), DC (+6.43), Nebraska (+6.19) | Positive |

**Key takeaway:** Trial density does not rise cleanly with diabetes burden. The relationship is lumpy and appears institution-driven rather than burden-responsive.

### 2.3 SES Correlations with Coverage Residual

A correlation heatmap of ~20 SES and health variables against `coverage_residual` shows:
- **All correlations are weak** -- the largest absolute correlation is only ~0.22.
- No single socioeconomic variable (poverty, income, race/ethnicity, insurance, education, rurality, Medicaid expansion) is a strong univariate predictor of coverage gaps.

**Implication for modeling:** SES variables alone will not explain state-level access gaps. They should be included as controls but are unlikely to be the primary drivers.

### 2.4 PCA Biplot (SES Feature Space)

- PCA on the SES feature set does not reveal a clear separation of over-covered vs. under-covered states along the first two principal components.
- This reinforces the weak SES-residual relationship and suggests the access pattern is driven by factors outside the standard SES gradient.

### 2.5 Portfolio Composition

| Dimension | Finding |
|-----------|---------|
| **Sponsor mix** | Mean industry share = 87.2%; national density ~5.39/100k (industry) vs. ~0.78/100k (non-industry) |
| **Phase mix** | Phase 3 studies dominate |
| **Status mix** | Completed studies dominate the status distribution |

**Implication for modeling:** The portfolio is overwhelmingly industry-sponsored. Industry site selection criteria (existing investigator networks, patient volume at sites) likely drive geographic concentration more than public health need.

### 2.6 State Mismatch Index

- `mismatch_index` = |burden_rank - trial_rank|, both normalized to [0,1].
- Quartile bins (Q1-Q4) show a gradient from well-aligned to severely misaligned states.
- This provides an alternative categorical outcome variable for modeling.

---

## 3. County-Level EDA Findings

### 3.1 Trial Site Presence

- **Only 862 of 3,221 counties (26.8%) host at least one diabetes trial site.**
- The vast majority of US counties have zero trial sites, making this a highly zero-inflated outcome.

### 3.2 Distance to Nearest Trial Site

Among counties **without** a trial site:
- **Median distance:** 58.8 km
- **>100 km away:** 24.1% of site-less counties
- **>200 km away:** 8.2% of site-less counties

The distance histogram is right-skewed with a long tail extending past 400 km for the most isolated counties.

### 3.3 Distance vs. Diabetes Burden

- A scatter plot of `dist_nearest_trial_km` vs. `places_diabetes` shows **no clear linear relationship**.
- The OLS fit line is near-flat, meaning high-burden counties are not systematically closer to or farther from trial sites.
- **Implication:** Distance to trials is driven by geography and infrastructure, not by disease prevalence.

### 3.4 Endocrinologist Density vs. Trial Density

- The county scatter plot of `endo_per_100k` vs. `trials_per_100k` shows a **positive but weak and noisy** association.
- Counties with endocrinologists are more likely to have trials, but the relationship has very high variance.
- Endocrinologist presence is sparse at the county level (most counties have zero).

### 3.5 AMC Presence vs. Trial Density

- Counties with more AMC NPI-2 organizations tend to have higher median trial density.
- The relationship is step-like: the biggest jump is from 0 to 1 AMC, with diminishing incremental effects.
- AMC presence may be a useful binary or low-cardinality predictor.

### 3.6 County Correlation Heatmap

Key correlation patterns across ~17 variables:
- `trials_per_100k` correlates positively with `endo_per_100k` and `amc_count`.
- `dist_nearest_trial_km` correlates negatively with infrastructure variables.
- Health burden variables (`places_diabetes`, `places_obesity`, `places_bphigh`) form a tight inter-correlated cluster.
- SES disadvantage variables (`pct_poverty`, `pct_uninsured`, `pct_no_vehicle`, `pct_no_internet`) form another correlated cluster.
- **Burden and SES clusters are correlated with each other** but neither cluster is strongly correlated with trial density.

---

## 4. Modeling Implications

### 4.1 Outcome Variables

| Level | Primary Outcome | Alternative Outcomes |
|-------|----------------|---------------------|
| State | `trials_per_100k` (continuous) | `coverage_residual`, `mismatch_index`, `mismatch_quartile` |
| County | `dist_nearest_trial_km` (continuous) | `has_trial_site` (binary), `trials_per_100k` (zero-inflated) |

### 4.2 Predictor Groups

Based on EDA, predictors should be organized into these conceptual blocks:

1. **Disease burden:** `places_diabetes` and related comorbidities (obesity, hypertension, kidney disease, stroke, CHD)
2. **SES / demographics:** poverty, income, Gini, race/ethnicity, insurance, education, vehicle/internet access
3. **Infrastructure:** `endo_per_100k`, `amc_count`, `has_ctsa`
4. **Policy:** `medicaid_expanded` (state-level only)
5. **Portfolio composition:** `industry_pct`, phase/status/sponsor breakdowns (state-level only)

### 4.3 Key Modeling Considerations

1. **Burden alone is insufficient.** The EDA consistently shows that disease prevalence is a weak predictor of trial access. Infrastructure and institutional variables should be given equal or greater weight in model specification.

2. **Multicollinearity within predictor blocks.** Health burden measures and SES measures are internally correlated. Consider PCA, variable selection, or regularization to handle collinearity.

3. **County-level zero inflation.** 73.2% of counties have no trial site. Models for `trials_per_100k` at county level should account for excess zeros (e.g., zero-inflated Poisson/NB, hurdle models, or two-stage approaches).

4. **Right-skewed continuous outcomes.** Both `trials_per_100k` and `dist_nearest_trial_km` are right-skewed. Log transformations or GLMs with appropriate link functions are warranted.

5. **State vs. county modeling.** State-level models have only 51 observations, limiting model complexity. County-level models (n=3,221) allow richer specifications but require careful handling of spatial autocorrelation.

6. **Ecological fallacy.** All associations are aggregate-level. Results should not be interpreted as individual-level causal effects.

7. **Missing data.** KY and PA lack state PLACES diabetes burden; 11% of site rows are unmatched at county level; `places_kidney` is missing from the county modeling table.

### 4.4 Suggested Modeling Approaches

| Question | Suggested Model | Outcome | Key Predictors |
|----------|----------------|---------|----------------|
| What predicts state trial density? | OLS / Elastic Net regression | `trials_per_100k` | Burden + SES + infrastructure + portfolio |
| Which states are most misaligned? | Logistic / ordinal regression | `mismatch_quartile` | Burden + SES + infrastructure |
| What predicts county trial site presence? | Logistic regression / Random Forest | `has_trial_site` (binary) | Burden + SES + infrastructure + distance features |
| What predicts county distance to nearest site? | GLM (Gamma or log-normal) on site-less counties | `dist_nearest_trial_km` | Burden + SES + infrastructure |
| County trial density (zero-inflated)? | Zero-inflated NB / Hurdle model | `trials_per_100k` | Burden + SES + infrastructure |

---

## 5. Summary Statement

The EDA reveals that U.S. diabetes clinical trial access is characterized by **geographic and institutional concentration** rather than alignment with disease burden. Industry sponsorship dominates the portfolio, trial sites cluster around academic medical centers and specialist providers, and neither disease prevalence nor socioeconomic disadvantage strongly predicts where trials are located. County-level analysis exposes sharper access gaps than state averages, with nearly three-quarters of counties lacking any trial site and substantial distance barriers for rural and under-resourced areas. These patterns argue for models that treat infrastructure and institutional presence as primary predictors, with burden and SES as contextual controls rather than expected drivers.
