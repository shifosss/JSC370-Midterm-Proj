# State-Level Modeling Dataset Documentation

**File:** `discover/modified_data/state_modeling_final.csv`
**Produced by:** `discover/data_acquiring/api_call_data_grabbing.ipynb`
**Dimensions:** 51 rows x 57 columns
**Unit of analysis:** U.S. state (50 states + District of Columbia)
**Reference date:** Clinical trials queried 2024-2025; ACS vintage 2022; CDC PLACES release 2023; CDC BRFSS 2023

---

## Overview

This dataset is the primary analytic file for state-level modeling of clinical trial access equity for diabetes-related studies in the United States. Each row represents one U.S. state (or D.C.) and combines five source streams:

1. **ClinicalTrials.gov v2 API** -- diabetes-related trial and site counts, broken down by sponsor type, phase, and status
2. **CDC PLACES (SODA API)** -- age-adjusted prevalence estimates for chronic conditions and social determinants
3. **CDC BRFSS (Socrata)** -- diabetes prevalence with 95% confidence intervals (most recent year available per state)
4. **U.S. Census ACS 5-Year (2022)** -- socioeconomic and demographic characteristics
5. **NPI Registry + manually curated lists** -- endocrinologist supply, Academic Medical Center (AMC) presence, and Clinical and Translational Science Award (CTSA) hub presence

---

## Column-by-Column Reference

### Identifiers

| Column | Type | Description |
| -------- | ------ | ----------- |
| `geo_id` | str | Two-letter U.S. Postal Service state abbreviation (e.g., `AL`, `CA`). Primary key. |
| `geo_level` | str | Always `"state"` in this file. Included for compatibility with the county-level dataset. |

---

### Clinical Trial Outcome Variables

| Column | Type | Units | Description |
| -------- | ------ | ------- | ----------- |
| `trial_count` | int | trials | Total number of unique diabetes-related ClinicalTrials.gov studies with at least one U.S. site in this state. A trial is counted once per state regardless of the number of sites. |
| `site_count` | int | sites | Total number of individual site-level records for diabetes-related trials located in this state (one trial with three sites in the same state contributes 3). |
| `trials_per_100k` | float | trials per 100,000 population | Primary continuous outcome: trial_count / (pop_total / 100,000). Measures per-capita trial density. |
| `coverage_residual` | float | trials per 100,000 (residual) | Difference between observed `trials_per_100k` and expected trial density from an OLS regression on `places_diabetes`. Positive = more trials than expected given burden; negative = under-coverage. Missing for 2 states. |
| `industry_pct` | float | percent | Percentage of trials sponsored by industry. Derived from ClinicalTrials.gov `LeadSponsorClass` field. |

---
### Mismatch / Equity Metrics

| Column | Type | Description |
| -------- | ------ | ----------- |
| `mismatch_index` | float | Absolute difference between the state's normalized diabetes burden rank and its normalized trial density rank. Both ranks expressed on [0, 1] (min-max scaled across 51 states). A value near 1 indicates extreme misalignment. Missing for 2 states without BRFSS diabetes estimates. |
| `mismatch_quartile` | str | Categorical quartile of `mismatch_index`: Q1 (lowest mismatch), Q2, Q3, Q4 (highest mismatch). |
| `burden_rank` | float | Rank of state by diabetes prevalence (CDC BRFSS), normalized to [0, 1]. Higher = higher burden. Missing for states without BRFSS data. |
| `trial_rank` | float | Rank of state by `trials_per_100k`, normalized to [0, 1]. Higher = more trials per capita. |

---

### Socioeconomic & Demographic Variables (ACS 2022)

All ACS variables are derived from the U.S. Census Bureau American Community Survey 5-Year Estimates, 2022 vintage, at the state level. Retrieved via the Census API (`https://api.census.gov/data/2022/acs/acs5`).

| Column | Type | Units | ACS Variable(s) | Description |
| -------- | ------ | ------- | ----------------- | ----------- |
| `pop_total` | int | persons | `B01003_001E` | Total state population. Used as denominator for per-capita calculations. |
| `median_hh_income` | int | USD | `B19013_001E` | Median household income. |
| `gini_index` | float | 0-1 | `B19083_001E` | Gini coefficient of income inequality. Higher = more unequal. |
| `pct_poverty` | float | percent | `B17001_002E` / `B17001_001E` | Percentage of population below the federal poverty line. |
| `pct_nhblack` | float | percent | `B03002_004E` / `B01003_001E` | Percentage of population identifying as non-Hispanic Black alone. |
| `pct_hispanic` | float | percent | `B03002_012E` / `B01003_001E` | Percentage of population identifying as Hispanic or Latino (any race). |
| `pct_no_vehicle` | float | percent | `B08201_002E` / `B08201_001E` | Percentage of households with no vehicle available. Proxy for transportation access barrier. |
| `pct_no_internet` | float | percent | `B28002_013E` / `B28002_001E` | Percentage of households without any internet subscription. Proxy for digital access/connectivity. |
| `pct_unemployed` | float | percent | Civilian unemployed / civilian labor force | Civilian unemployment rate. |
| `pct_uninsured` | float | percent | SAHIE-derived within ACS | Percentage of population without health insurance. |
| `pct_bachelors_plus` | float | percent | Educational attainment tables | Percentage of adults 25+ with a bachelor's degree or higher. |

---

### CDC PLACES Health Measures

Pulled from CDC PLACES 2023 release via the Socrata SODA API (`data.cdc.gov`). All values are **age-adjusted prevalence estimates** expressed as **percentages** of the adult population. Missing for 2 states (see note).

| Column | PLACES Measure | Description |
| -------- | --------------- | ----------- |
| `places_access2` | ACCESS2 | Currently lacks health insurance (crude/age-adjusted) |
| `places_bphigh` | BPHIGH | High blood pressure among adults |
| `places_chd` | CHD | Coronary heart disease among adults |
| `places_depression` | DEPRESSION | Depression among adults |
| `places_diabetes` | DIABETES | Diagnosed diabetes among adults. **Primary disease burden variable.** |
| `places_foodinsecu` | FOODINSECU | Food insecurity among adults |
| `places_foodstamp` | FOODSTAMP | Food stamp / SNAP participation |
| `places_ghlth` | GHLTH | Fair or poor general health status |
| `places_housinsecu` | HOUSINSECU | Housing insecurity |
| `places_lacktrpt` | LACKTRPT | Lack of reliable transportation |
| `places_lpa` | LPA | No leisure-time physical activity |
| `places_mhlth` | MHLTH | Poor mental health 14+ days in past 30 days |
| `places_obesity` | OBESITY | Obesity (BMI >= 30) among adults |
| `places_stroke` | STROKE | Stroke among adults |

**Missing data note:** `places_foodinsecu`, `places_foodstamp`, `places_housinsecu`, and `places_lacktrpt` are missing for 11 states (social deprivation sub-measures only available for a subset in PLACES 2023). Core health measures missing for 2 states (AK and HI).

---
### Healthcare Infrastructure Variables

| Column | Type | Source | Description |
| -------- | ------ | -------- | ----------- |
| `endo_per_100k` | float | NPI Registry | Number of practicing endocrinologists per 100,000 population in the state. Derived by querying the NPI Registry API for providers with taxonomy code `207RE0101X` (Endocrinology, Diabetes & Metabolism) and dividing by `pop_total`. |
| `amc_count` | int | Manually curated + NPI cross-reference | Number of Academic Medical Centers (AMCs) in the state, identified as hospitals affiliated with accredited medical schools with active research programs. |
| `has_ctsa` | int (0/1) | NCATS CTSA Program list | Binary indicator: state has at least one NCATS-funded Clinical and Translational Science Award (CTSA) hub. |

---

### Trial Density Breakdowns (per 100k population)

These columns decompose `trials_per_100k` by trial phase, sponsor type, and recruitment status. All values are **trials per 100,000 population**. NaN = zero trials of that category in the state.

#### By Phase

| Column | Description |
| -------- | ----------- |
| `trials_100k_phase_early_phase1` | Early Phase 1 trials per 100k |
| `trials_100k_phase_phase1` | Phase 1 trials per 100k |
| `trials_100k_phase_phase2` | Phase 2 trials per 100k |
| `trials_100k_phase_phase3` | Phase 3 trials per 100k |
| `trials_100k_phase_phase4` | Phase 4 / post-market trials per 100k |
| `trials_100k_phase_unknown` | Trials with unknown/unspecified phase per 100k |

#### By Sponsor Type

| Column | Description |
| -------- | ----------- |
| `trials_100k_sponsor_industry` | Industry-sponsored trials per 100k |
| `trials_100k_sponsor_non_industry` | Non-industry (NIH, academic, other) trials per 100k |

#### By Recruitment Status

| Column | Description |
| -------- | ----------- |
| `trials_100k_status_recruiting` | Actively recruiting trials per 100k |
| `trials_100k_status_not_yet_recruiting` | Not yet recruiting trials per 100k |
| `trials_100k_status_active_not_recruiting` | Active but not recruiting per 100k |
| `trials_100k_status_enrolling_by_invitation` | Enrolling by invitation only per 100k |
| `trials_100k_status_completed` | Completed trials per 100k |
| `trials_100k_status_terminated` | Terminated trials per 100k |
| `trials_100k_status_suspended` | Suspended trials per 100k |
| `trials_100k_status_withdrawn` | Withdrawn trials per 100k |
| `trials_100k_status_no_longer_available` | No longer available per 100k |
| `trials_100k_status_unknown` | Unknown status per 100k |

---

## Summary Statistics

| Variable | N | Mean | SD | Min | Median | Max |
| --------- | --- | ------ | ---- | ----- | -------- | ----- |
| `trial_count` | 51 | 400.1 | 290.2 | 1 | 359 | 1,346 |
| `site_count` | 51 | 923.9 | 1,313.2 | 1 | 581 | 6,378 |
| `trials_per_100k` | 51 | 7.99 | 3.79 | 0.17 | 7.38 | 18.64 |
| `coverage_residual` | 49 | ~0.00 | 3.48 | -8.58 | -0.21 | 7.48 |
| `industry_pct` | 51 | 87.2% | 14.5% | 0% | 90.2% | 99.7% |
| `pop_total` | 51 | 6.49M | 7.39M | 578K | 4.50M | 39.4M |
| `median_hh_income` | 51 | $74,805 | $12,432 | $52,985 | $72,458 | $101,722 |
| `gini_index` | 51 | 0.467 | 0.019 | 0.429 | 0.466 | 0.517 |
| `pct_poverty` | 51 | 12.0% | 2.6% | 7.1% | 11.7% | 18.5% |
| `places_diabetes` | 49 | 10.2% | 1.5% | 7.0% | 10.1% | 13.6% |
| `places_obesity` | 49 | 34.1% | 4.0% | 25.1% | 34.6% | 40.8% |
| `endo_per_100k` | 51 | 3.73 | 1.89 | 1.35 | 3.32 | 12.08 |
| `mismatch_index` | 49 | ~0.00 | 0.44 | -0.90 | 0.07 | 0.86 |

---

## Missing Data Summary

| Column Group | N Missing | Reason |
| ------------- | ----------- | -------- |
| `coverage_residual`, `mismatch_index`, `mismatch_quartile`, `burden_rank` | 2 | States without CDC BRFSS diabetes prevalence estimates (used in regression/ranking) |
| `places_*` (core health measures) | 2 | AK and HI excluded from some PLACES 2023 state-level estimates |
| `places_foodinsecu`, `places_foodstamp`, `places_housinsecu`, `places_lacktrpt` | 11 | Social deprivation sub-measures available only for subset of states in PLACES 2023 |
| `trials_100k_phase_early_phase1` | 33 | Most states have zero Early Phase 1 diabetes trials; NaN encodes zero density |
| `trials_100k_status_no_longer_available` | 41 | Rare status; most states have no trials in this category |
| `trials_100k_status_enrolling_by_invitation` | 26 | Uncommon status |
---

## Data Lineage

```
ClinicalTrials.gov v2 API
  --> ct_trials_flat.csv           (one row per trial)
  --> ct_sites_flat.csv            (one row per site)
  --> ct_trial_state_pairs.csv     (trial x state deduplication)
  --> ct_state_aggregates.csv      (trial_count, site_count per state)
  --> state_density_by_phase.csv   (phase breakdown)
  --> state_density_by_sponsor.csv (sponsor breakdown)
  --> state_density_by_status.csv  (status breakdown)

CDC BRFSS / Socrata
  --> cdc_diabetes_incidence_state.csv
  --> cdc_diabetes_state_latest.csv (most recent year per state)

CDC PLACES / SODA
  --> cdc_places_state_wide.csv   (pivoted to wide format)
  --> cdc_places_wide_state.csv   (final wide)

US Census ACS 5-Year 2022
  --> acs_state_2022_full.csv
  --> acs_state_2022_features.csv (selected + derived variables)

US Census 2020 Decennial (rural pop)
  --> rural_pct_state_2020.csv

NPI Registry API
  --> npi_state_infrastructure.csv (endo_count, amc_count, has_ctsa)

Manually curated
  --> medicaid_expansion_status.csv

Merge and feature engineering
  --> state_coverage_alignment.csv  (intermediate merged dataset, 44 cols)
  --> state_mismatch_index.csv      (mismatch metrics added)
  --> state_modeling_final.csv      (FINAL: 51 x 57)
```

---

## Construction Notes

- **Trial filtering:** Only trials with query.cond = "diabetes" returned by ClinicalTrials.gov v2 and at least one U.S. site are included. Sponsor type derived from `LeadSponsorClass`: INDUSTRY = industry, all others = non-industry.
- **State assignment:** Each site matched to a state via the `LocationState` field. A trial is attributed to a state if any site is in that state; counted once per state regardless of site count.
- **`trials_per_100k` denominator:** Uses ACS 2022 `pop_total`. Consistent across state and county levels.
- **`coverage_residual` construction:** OLS regression of `trials_per_100k` on `places_diabetes` across 49 states (2 excluded with missing PLACES diabetes). Residual captures trial density unexplained by disease burden alone.
- **`mismatch_index` construction:** abs(burden_rank - trial_rank), where both ranks are min-max normalized to [0, 1] across all 51 states using BRFSS diabetes prevalence and `trials_per_100k` respectively.
- **Endocrinologist supply:** NPI Registry queries used taxonomy code `207RE0101X`. Active NPI holders only; no part-time adjustment.
- **AMC / CTSA:** AMC count from cross-referencing NPI-registered hospital facilities with AAMC member lists. CTSA is a binary flag based on the NCATS CTSA hub roster.
