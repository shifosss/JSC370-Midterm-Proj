# County-Level Modeling Dataset Documentation

**File:** `product/data/modified/county_modeling_final.csv`
**Produced by:** `product/option-c.ipynb`
**Dimensions:** 3221 rows x 42 columns
**Unit of analysis:** U.S. county (or county-equivalent)
**Reference date:** Clinical trials queried 2024-2025; ACS vintage 2022; CDC PLACES release 2023

---

## Overview

This is the current county-level modeling file used by the active Option C production workflow. Each row represents one county or county-equivalent and combines county trial access, distance-to-nearest-site, ACS socioeconomic context, CDC PLACES health measures, and infrastructure proxies.

The county file keeps `coverage_residual` only for schema compatibility with the state file; at the county level it is equal to `trials_per_100k` rather than a separate residualized target.

---

## Column-by-Column Reference

### Identifier

| Column | Type | Units | Missing (N) | Description |
| --- | --- | --- | --- | --- |
| `geo_id` | str | FIPS code | 0 | 5-digit county FIPS. Read with `dtype={"geo_id": str}` to preserve leading zeros. |
| `geo_level` | str | label | 0 | Always `county`. Included for compatibility with the state dataset. |

---

### Geography

| Column | Type | Units | Missing (N) | Description |
| --- | --- | --- | --- | --- |
| `lat` | float | decimal degrees | 0 | County centroid latitude. |
| `lon` | float | decimal degrees | 0 | County centroid longitude. |

---

### Clinical trial access

| Column | Type | Units | Missing (N) | Description |
| --- | --- | --- | --- | --- |
| `trials_per_100k` | float | trials per 100,000 population | 8 | Per-capita county trial density using `pop_total` as the denominator. |
| `coverage_residual` | float | trials per 100,000 population | 8 | Retained for state/county schema alignment. At county level it equals `trials_per_100k`. |
| `dist_nearest_trial_km` | float | kilometers | 0 | Great-circle distance from the county centroid to the nearest diabetes trial site; equals 0 when the county hosts a site. |
| `has_trial_site` | bool | bool | 0 | True if the county hosts at least one diabetes-related trial site. |
| `county_trial_count` | int | trials | 0 | Unique diabetes-related trials with at least one site geocoded to the county. |
| `county_site_count` | int | sites | 0 | Individual site-level records geocoded to the county. |

---

### ACS socioeconomic and demographic

| Column | Type | Units | Missing (N) | Description |
| --- | --- | --- | --- | --- |
| `pop_total` | float | persons | 8 | ACS 2022 total county population. |
| `pct_poverty` | float | percent | 8 | Population below the federal poverty line. |
| `median_hh_income` | float | USD | 9 | Median household income. |
| `gini_index` | float | 0-1 | 8 | Gini coefficient of income inequality. |
| `pct_nhblack` | float | percent | 8 | Population identifying as non-Hispanic Black alone. |
| `pct_hispanic` | float | percent | 8 | Population identifying as Hispanic or Latino (any race). |
| `pct_no_vehicle` | float | percent | 8 | Households with no vehicle available. |
| `pct_no_internet` | float | percent | 8 | Households without any internet subscription. |
| `pct_unemployed` | float | percent | 8 | Civilian unemployment rate. |
| `pct_uninsured` | float | percent | 8 | Population without health insurance. |
| `pct_bachelors_plus` | float | percent | 8 | Adults age 25+ with a bachelor's degree or higher. |
| `pop` | int | persons | 0 | Population from the county centroid reference file used in distance calculations. |
| `pct_nhwhite` | float | percent | 8 | Population identifying as non-Hispanic White alone. |

---

### CDC PLACES health measures

| Column | Type | Units | Missing (N) | Description |
| --- | --- | --- | --- | --- |
| `places_access2` | float | percent | 274 | Currently lacks health insurance among adults. |
| `places_bphigh` | float | percent | 274 | High blood pressure among adults. |
| `places_chd` | float | percent | 274 | Coronary heart disease among adults. |
| `places_depression` | float | percent | 274 | Depression among adults. |
| `places_diabetes` | float | percent | 274 | Diagnosed diabetes among adults. Primary disease-burden variable. |
| `places_foodinsecu` | float | percent | 931 | Food insecurity among adults. |
| `places_foodstamp` | float | percent | 931 | Food stamp / SNAP participation. |
| `places_ghlth` | float | percent | 274 | Fair or poor general health status. |
| `places_housinsecu` | float | percent | 931 | Housing insecurity. |
| `places_lacktrpt` | float | percent | 931 | Lack of reliable transportation. |
| `places_lpa` | float | percent | 274 | No leisure-time physical activity. |
| `places_mhlth` | float | percent | 274 | Poor mental health 14+ days in the past 30 days. |
| `places_obesity` | float | percent | 274 | Obesity (BMI >= 30) among adults. |
| `places_stroke` | float | percent | 274 | Stroke among adults. |

---

### Infrastructure

| Column | Type | Units | Missing (N) | Description |
| --- | --- | --- | --- | --- |
| `endo_per_100k` | float | providers per 100,000 population | 8 | Endocrinologists per 100,000 population. |
| `amc_count` | int | count | 0 | Academic Medical Center count in the county. |
| `has_ctsa` | int | 0/1 flag | 0 | Indicator that the county contains at least one NCATS CTSA hub. |
| `endo_count` | int | count | 0 | Raw count of endocrinologists matched to the county. |

---

### Derived variables

| Column | Type | Units | Missing (N) | Description |
| --- | --- | --- | --- | --- |
| `burden_decile` | int | 0-9 decile code | 0 | Decile rank of `places_diabetes` across counties with observed burden; 0 marks counties missing the burden input. |

---

## Summary Statistics

| Variable | N | Mean | SD | Min | Median | Max |
| --- | --- | --- | --- | --- | --- | --- |
| `county_trial_count` | 3,221 | 10.61 | 46.65 | 0 | 0 | 784 |
| `county_site_count` | 3,221 | 13.01 | 70.91 | 0 | 0 | 1,550 |
| `dist_nearest_trial_km` | 3,221 | 104.4 | 290.0 | 0 | 44.51 | 2,093.4 |
| `trials_per_100k` | 3,213 | 4.19 | 15.98 | 0 | 0 | 485.9 |
| `pop_total` | 3,213 | 102,943.9 | 329,723.7 | 50 | 25,839 | 9,936,690 |
| `median_hh_income` | 3,212 | $62,250 | $17,725 | $14,525 | $60,417 | $170,463 |
| `gini_index` | 3,213 | 0.45 | 0.04 | 0.27 | 0.45 | 0.72 |
| `pct_poverty` | 3,213 | 14.6% | 7.5% | 1.6% | 13.1% | 65.6% |
| `pct_nhblack` | 3,213 | 8.5% | 14.1% | 0.0% | 1.9% | 85.7% |
| `pct_hispanic` | 3,213 | 12.1% | 19.5% | 0.0% | 4.8% | 100.0% |
| `places_diabetes` | 2,947 | 11.2% | 2.3% | 6.0% | 10.8% | 23.8% |
| `places_obesity` | 2,947 | 37.6% | 4.7% | 17.1% | 38.2% | 54.0% |
| `endo_per_100k` | 3,213 | 1.54 | 5.58 | 0 | 0 | 107.3 |

---

## Missing Data Summary

| Column | Missing (N) | Missing (%) | Reason |
| --- | --- | --- | --- |
| `pop_total` | 8 | 0.2% | ACS population is missing for eight counties, so dependent rates and ACS-derived percentages are also missing there. |
| `trials_per_100k` | 8 | 0.2% | ACS population is missing for eight counties, so dependent rates and ACS-derived percentages are also missing there. |
| `coverage_residual` | 8 | 0.2% | ACS population is missing for eight counties, so dependent rates and ACS-derived percentages are also missing there. |
| `places_access2` | 274 | 8.5% | PLACES core health measure unavailable for very small counties and territory subdivisions. |
| `places_bphigh` | 274 | 8.5% | PLACES core health measure unavailable for very small counties and territory subdivisions. |
| `places_chd` | 274 | 8.5% | PLACES core health measure unavailable for very small counties and territory subdivisions. |
| `places_depression` | 274 | 8.5% | PLACES core health measure unavailable for very small counties and territory subdivisions. |
| `places_diabetes` | 274 | 8.5% | PLACES core health measure unavailable for very small counties and territory subdivisions. |
| `places_foodinsecu` | 931 | 28.9% | PLACES social-deprivation sub-measure only available for a subset of counties. |
| `places_foodstamp` | 931 | 28.9% | PLACES social-deprivation sub-measure only available for a subset of counties. |
| `places_ghlth` | 274 | 8.5% | PLACES core health measure unavailable for very small counties and territory subdivisions. |
| `places_housinsecu` | 931 | 28.9% | PLACES social-deprivation sub-measure only available for a subset of counties. |
| `places_lacktrpt` | 931 | 28.9% | PLACES social-deprivation sub-measure only available for a subset of counties. |
| `places_lpa` | 274 | 8.5% | PLACES core health measure unavailable for very small counties and territory subdivisions. |
| `places_mhlth` | 274 | 8.5% | PLACES core health measure unavailable for very small counties and territory subdivisions. |
| `places_obesity` | 274 | 8.5% | PLACES core health measure unavailable for very small counties and territory subdivisions. |
| `places_stroke` | 274 | 8.5% | PLACES core health measure unavailable for very small counties and territory subdivisions. |
| `pct_poverty` | 8 | 0.2% | ACS population is missing for eight counties, so dependent rates and ACS-derived percentages are also missing there. |
| `median_hh_income` | 9 | 0.3% | Eight counties are missing ACS coverage and one additional county has a suppressed ACS income estimate. |
| `gini_index` | 8 | 0.2% | ACS population is missing for eight counties, so dependent rates and ACS-derived percentages are also missing there. |
| `pct_nhblack` | 8 | 0.2% | ACS population is missing for eight counties, so dependent rates and ACS-derived percentages are also missing there. |
| `pct_hispanic` | 8 | 0.2% | ACS population is missing for eight counties, so dependent rates and ACS-derived percentages are also missing there. |
| `pct_no_vehicle` | 8 | 0.2% | ACS population is missing for eight counties, so dependent rates and ACS-derived percentages are also missing there. |
| `pct_no_internet` | 8 | 0.2% | ACS population is missing for eight counties, so dependent rates and ACS-derived percentages are also missing there. |
| `pct_unemployed` | 8 | 0.2% | ACS population is missing for eight counties, so dependent rates and ACS-derived percentages are also missing there. |
| `pct_uninsured` | 8 | 0.2% | ACS population is missing for eight counties, so dependent rates and ACS-derived percentages are also missing there. |
| `pct_bachelors_plus` | 8 | 0.2% | ACS population is missing for eight counties, so dependent rates and ACS-derived percentages are also missing there. |
| `endo_per_100k` | 8 | 0.2% | ACS population is missing for eight counties, so dependent rates and ACS-derived percentages are also missing there. |
| `pct_nhwhite` | 8 | 0.2% | ACS population is missing for eight counties, so dependent rates and ACS-derived percentages are also missing there. |

---

## Data Lineage

```
ClinicalTrials.gov v2 API + geocoding
  --> ct_trials_flat.csv
  --> ct_sites_flat.csv
  --> ct_sites_county.csv
  --> county_trial_aggregates.csv

County centroid reference / distance calculation
  --> county_distance_to_trial.csv

US Census ACS 5-Year 2022
  --> acs_county_2022_features.csv

CDC PLACES / SODA
  --> cdc_places_county_wide.csv

NPI Registry + curated infrastructure lists
  --> npi_endocrinologist_county.csv
  --> amc_presence_county.csv

Merge and feature engineering
  --> county_coverage_alignment.csv
  --> county_modeling_final.csv
```

---

## Construction Notes

- `dist_nearest_trial_km` is a Haversine great-circle distance from the county centroid to the nearest diabetes trial site centroid.
- `trials_per_100k` uses ACS 2022 `pop_total` as the denominator; when `pop_total` is missing, dependent rates are also missing.
- `coverage_residual` equals `trials_per_100k` at county level and is retained only for state/county schema compatibility.
- `burden_decile` is derived from `places_diabetes`; counties missing the burden input receive `burden_decile = 0`.
- `geo_id` should be read as a string FIPS code when joining to shapefiles or other county reference data.
