# State-Level Modeling Dataset Documentation

**File:** `product/data/modified/state_modeling_final.csv`
**Produced by:** `product/option-c.ipynb`
**Dimensions:** 51 rows x 53 columns
**Unit of analysis:** U.S. state (50 states + District of Columbia)
**Reference date:** Clinical trials queried 2024-2025; ACS vintage 2022; CDC PLACES release 2023; BRFSS burden benchmark used only to construct `coverage_residual`

---

## Overview

This is the current state-level modeling file used by the active Option C production workflow. Each row represents one state or the District of Columbia and combines clinical trial counts, ACS socioeconomic context, CDC PLACES health measures, infrastructure proxies, and per-100k trial breakdowns by phase, sponsor, and status.

The state file no longer includes the earlier mismatch-rank variables. The current schema is centered on trial density, coverage residual, burden-related health measures, demographic / socioeconomic context, and infrastructure availability.

---

## Column-by-Column Reference

### Identifier

| Column | Type | Units | Missing (N) | Description |
| --- | --- | --- | --- | --- |
| `geo_id` | str | State postal abbreviation | 0 | Two-letter U.S. Postal Service abbreviation; primary key. |
| `geo_level` | str | label | 0 | Always `state`. Included for compatibility with the county dataset. |

---

### Outcome

| Column | Type | Units | Missing (N) | Description |
| --- | --- | --- | --- | --- |
| `trial_count` | int | trials | 0 | Unique diabetes-related trials with at least one site in the state. |
| `site_count` | int | sites | 0 | Individual site-level records located in the state. |
| `trials_per_100k` | float | trials per 100,000 population | 0 | Per-capita trial density using `pop_total` as the denominator. |
| `coverage_residual` | float | trials per 100,000 population | 2 | Observed `trials_per_100k` minus the mean trial density within the same state diabetes-burden decile. Negative values indicate under-coverage relative to peers with similar burden. |
| `industry_pct` | float | percent | 0 | Share of state trials sponsored by industry. |

---

### ACS socioeconomic and demographic

| Column | Type | Units | Missing (N) | Description |
| --- | --- | --- | --- | --- |
| `pop_total` | int | persons | 0 | Total state population. |
| `median_hh_income` | int | USD | 0 | Median household income. |
| `gini_index` | float | 0-1 | 0 | Gini coefficient of income inequality. |
| `pct_poverty` | float | percent | 0 | Population below the federal poverty line. |
| `pct_nhblack` | float | percent | 0 | Population identifying as non-Hispanic Black alone. |
| `pct_hispanic` | float | percent | 0 | Population identifying as Hispanic or Latino (any race). |
| `pct_no_vehicle` | float | percent | 0 | Households with no vehicle available. |
| `pct_no_internet` | float | percent | 0 | Households without any internet subscription. |
| `pct_unemployed` | float | percent | 0 | Civilian unemployment rate. |
| `pct_uninsured` | float | percent | 0 | Population without health insurance. |
| `pct_bachelors_plus` | float | percent | 0 | Adults age 25+ with a bachelor's degree or higher. |

---

### CDC PLACES health measures

| Column | Type | Units | Missing (N) | Description |
| --- | --- | --- | --- | --- |
| `places_access2` | float | percent | 2 | Currently lacks health insurance among adults. |
| `places_bphigh` | float | percent | 2 | High blood pressure among adults. |
| `places_chd` | float | percent | 2 | Coronary heart disease among adults. |
| `places_depression` | float | percent | 2 | Depression among adults. |
| `places_diabetes` | float | percent | 2 | Diagnosed diabetes among adults. Primary disease-burden variable. |
| `places_foodinsecu` | float | percent | 11 | Food insecurity among adults. |
| `places_foodstamp` | float | percent | 11 | Food stamp / SNAP participation. |
| `places_ghlth` | float | percent | 2 | Fair or poor general health status. |
| `places_housinsecu` | float | percent | 11 | Housing insecurity. |
| `places_lacktrpt` | float | percent | 11 | Lack of reliable transportation. |
| `places_lpa` | float | percent | 2 | No leisure-time physical activity. |
| `places_mhlth` | float | percent | 2 | Poor mental health 14+ days in the past 30 days. |
| `places_obesity` | float | percent | 2 | Obesity (BMI >= 30) among adults. |
| `places_stroke` | float | percent | 2 | Stroke among adults. |

---

### Infrastructure

| Column | Type | Units | Missing (N) | Description |
| --- | --- | --- | --- | --- |
| `endo_per_100k` | float | providers per 100,000 population | 0 | Endocrinologists per 100,000 population. |
| `amc_count` | int | count | 0 | Academic Medical Center count in the state. |
| `has_ctsa` | int | 0/1 flag | 0 | Indicator that the state contains at least one NCATS CTSA hub. |

---

### Trial density breakdowns

| Column | Type | Units | Missing (N) | Description |
| --- | --- | --- | --- | --- |
| `trials_100k_phase_early_phase1` | float | trials per 100,000 population | 33 | Early Phase 1 trials per 100k population. |
| `trials_100k_phase_phase1` | float | trials per 100,000 population | 9 | Phase 1 trials per 100k population. |
| `trials_100k_phase_phase2` | float | trials per 100,000 population | 2 | Phase 2 trials per 100k population. |
| `trials_100k_phase_phase3` | float | trials per 100,000 population | 1 | Phase 3 trials per 100k population. |
| `trials_100k_phase_phase4` | float | trials per 100,000 population | 1 | Phase 4 trials per 100k population. |
| `trials_100k_phase_unknown` | float | trials per 100,000 population | 4 | Trials with unknown or unspecified phase per 100k population. |
| `trials_100k_sponsor_industry` | float | trials per 100,000 population | 1 | Industry-sponsored trials per 100k population. |
| `trials_100k_sponsor_non_industry` | float | trials per 100,000 population | 0 | Non-industry trials per 100k population. |
| `trials_100k_status_active_not_recruiting` | float | trials per 100,000 population | 3 | Active, not recruiting trials per 100k population. |
| `trials_100k_status_completed` | float | trials per 100,000 population | 1 | Completed trials per 100k population. |
| `trials_100k_status_enrolling_by_invitation` | float | trials per 100,000 population | 26 | Enrolling-by-invitation trials per 100k population. |
| `trials_100k_status_not_yet_recruiting` | float | trials per 100,000 population | 13 | Not-yet-recruiting trials per 100k population. |
| `trials_100k_status_no_longer_available` | float | trials per 100,000 population | 41 | No-longer-available trials per 100k population. |
| `trials_100k_status_recruiting` | float | trials per 100,000 population | 2 | Recruiting trials per 100k population. |
| `trials_100k_status_suspended` | float | trials per 100,000 population | 20 | Suspended trials per 100k population. |
| `trials_100k_status_terminated` | float | trials per 100,000 population | 2 | Terminated trials per 100k population. |
| `trials_100k_status_unknown` | float | trials per 100,000 population | 9 | Trials with unknown status per 100k population. |
| `trials_100k_status_withdrawn` | float | trials per 100,000 population | 9 | Withdrawn trials per 100k population. |

---

## Summary Statistics

| Variable | N | Mean | SD | Min | Median | Max |
| --- | --- | --- | --- | --- | --- | --- |
| `trial_count` | 51 | 400.2 | 290.3 | 1 | 359 | 1,347 |
| `site_count` | 51 | 923.9 | 1,313.3 | 1 | 581 | 6,379 |
| `trials_per_100k` | 51 | 7.99 | 3.79 | 0.17 | 7.38 | 18.64 |
| `coverage_residual` | 49 | 0.00 | 3.47 | -8.58 | -0.21 | 7.48 |
| `industry_pct` | 51 | 87.2% | 14.5% | 0.0% | 90.2% | 99.7% |
| `pop_total` | 51 | 6,492,109.7 | 7,392,321.1 | 577,929 | 4,502,935 | 39,356,104 |
| `median_hh_income` | 51 | $74,805 | $12,432 | $52,985 | $72,458 | $101,722 |
| `gini_index` | 51 | 0.47 | 0.02 | 0.43 | 0.47 | 0.52 |
| `pct_poverty` | 51 | 12.0% | 2.6% | 7.1% | 11.7% | 18.5% |
| `places_diabetes` | 49 | 10.2% | 1.5% | 7.0% | 10.1% | 13.6% |
| `places_obesity` | 49 | 34.1% | 4.0% | 25.1% | 34.6% | 40.8% |
| `endo_per_100k` | 51 | 3.73 | 1.89 | 1.35 | 3.32 | 12.08 |

---

## Missing Data Summary

| Column | Missing (N) | Missing (%) | Reason |
| --- | --- | --- | --- |
| `coverage_residual` | 2 | 3.9% | Missing in Kentucky and Pennsylvania because the intermediate burden benchmark used to construct the residual was unavailable. |
| `places_access2` | 2 | 3.9% | PLACES core health measure unavailable for AK and HI in the assembled state table. |
| `places_bphigh` | 2 | 3.9% | PLACES core health measure unavailable for AK and HI in the assembled state table. |
| `places_chd` | 2 | 3.9% | PLACES core health measure unavailable for AK and HI in the assembled state table. |
| `places_depression` | 2 | 3.9% | PLACES core health measure unavailable for AK and HI in the assembled state table. |
| `places_diabetes` | 2 | 3.9% | PLACES core health measure unavailable for AK and HI in the assembled state table. |
| `places_foodinsecu` | 11 | 21.6% | PLACES social-deprivation sub-measure only available for a subset of states. |
| `places_foodstamp` | 11 | 21.6% | PLACES social-deprivation sub-measure only available for a subset of states. |
| `places_ghlth` | 2 | 3.9% | PLACES core health measure unavailable for AK and HI in the assembled state table. |
| `places_housinsecu` | 11 | 21.6% | PLACES social-deprivation sub-measure only available for a subset of states. |
| `places_lacktrpt` | 11 | 21.6% | PLACES social-deprivation sub-measure only available for a subset of states. |
| `places_lpa` | 2 | 3.9% | PLACES core health measure unavailable for AK and HI in the assembled state table. |
| `places_mhlth` | 2 | 3.9% | PLACES core health measure unavailable for AK and HI in the assembled state table. |
| `places_obesity` | 2 | 3.9% | PLACES core health measure unavailable for AK and HI in the assembled state table. |
| `places_stroke` | 2 | 3.9% | PLACES core health measure unavailable for AK and HI in the assembled state table. |
| `trials_100k_phase_early_phase1` | 33 | 64.7% | Sparse trial-breakdown category; NaN indicates no trials in that category for many states. |
| `trials_100k_phase_phase1` | 9 | 17.6% | Sparse trial-breakdown category; NaN indicates no trials in that category for many states. |
| `trials_100k_phase_phase2` | 2 | 3.9% | Sparse trial-breakdown category; NaN indicates no trials in that category for many states. |
| `trials_100k_phase_phase3` | 1 | 2.0% | Sparse trial-breakdown category; NaN indicates no trials in that category for many states. |
| `trials_100k_phase_phase4` | 1 | 2.0% | Sparse trial-breakdown category; NaN indicates no trials in that category for many states. |
| `trials_100k_phase_unknown` | 4 | 7.8% | Sparse trial-breakdown category; NaN indicates no trials in that category for many states. |
| `trials_100k_sponsor_industry` | 1 | 2.0% | Sparse trial-breakdown category; NaN indicates no trials in that category for many states. |
| `trials_100k_status_active_not_recruiting` | 3 | 5.9% | Sparse trial-breakdown category; NaN indicates no trials in that category for many states. |
| `trials_100k_status_completed` | 1 | 2.0% | Sparse trial-breakdown category; NaN indicates no trials in that category for many states. |
| `trials_100k_status_enrolling_by_invitation` | 26 | 51.0% | Sparse trial-breakdown category; NaN indicates no trials in that category for many states. |
| `trials_100k_status_not_yet_recruiting` | 13 | 25.5% | Sparse trial-breakdown category; NaN indicates no trials in that category for many states. |
| `trials_100k_status_no_longer_available` | 41 | 80.4% | Sparse trial-breakdown category; NaN indicates no trials in that category for many states. |
| `trials_100k_status_recruiting` | 2 | 3.9% | Sparse trial-breakdown category; NaN indicates no trials in that category for many states. |
| `trials_100k_status_suspended` | 20 | 39.2% | Sparse trial-breakdown category; NaN indicates no trials in that category for many states. |
| `trials_100k_status_terminated` | 2 | 3.9% | Sparse trial-breakdown category; NaN indicates no trials in that category for many states. |
| `trials_100k_status_unknown` | 9 | 17.6% | Sparse trial-breakdown category; NaN indicates no trials in that category for many states. |
| `trials_100k_status_withdrawn` | 9 | 17.6% | Sparse trial-breakdown category; NaN indicates no trials in that category for many states. |

---

## Data Lineage

```
ClinicalTrials.gov v2 API
  --> ct_trials_flat.csv
  --> ct_sites_flat.csv
  --> ct_trial_state_pairs.csv
  --> ct_state_aggregates.csv
  --> state_density_by_phase.csv
  --> state_density_by_sponsor.csv
  --> state_density_by_status.csv

CDC BRFSS / Socrata
  --> cdc_diabetes_state_latest.csv  (used only in the intermediate burden benchmark)

CDC PLACES / SODA
  --> cdc_places_state_wide.csv

US Census ACS 5-Year 2022
  --> acs_state_2022_full.csv

NPI Registry + curated infrastructure lists
  --> npi_state_infrastructure.csv

Merge and feature engineering
  --> state_coverage_alignment.csv
  --> state_modeling_final.csv
```

---

## Construction Notes

- `trials_per_100k` uses ACS 2022 `pop_total` as the denominator.
- `coverage_residual` is carried from the intermediate state alignment table and reflects trial density relative to states in the same diabetes-burden decile.
- `places_*` variables are CDC PLACES age-adjusted prevalence estimates expressed as percentages.
- Sparse `trials_100k_*` columns use `NaN` when a state has no trials in that category.
- `has_ctsa` is a binary flag; `amc_count` and `endo_per_100k` are infrastructure proxies rather than direct burden measures.
