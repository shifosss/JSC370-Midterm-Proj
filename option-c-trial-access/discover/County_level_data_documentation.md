# County-Level Modeling Dataset Documentation

**File:** `discover/modified_data/county_modeling_final.csv`
**Produced by:** `discover/data_acquiring/api_call_data_grabbing.ipynb`
**Dimensions:** 3,221 rows x 42 columns
**Unit of analysis:** U.S. county (or county-equivalent)
**Reference date:** Clinical trials queried 2024-2025; ACS vintage 2022; CDC PLACES release 2023

---

## Overview

This dataset is the primary analytic file for county-level modeling of clinical trial access equity for diabetes-related studies in the United States. Each row represents one U.S. county or county-equivalent (3,221 total). The dataset combines five source streams:

1. **ClinicalTrials.gov v2 API + geocoding** -- per-county trial and site counts, plus distance-to-nearest-trial for counties without a site
2. **CDC PLACES (SODA API)** -- age-adjusted prevalence estimates for chronic conditions and social determinants at the county level
3. **U.S. Census ACS 5-Year (2022)** -- socioeconomic and demographic characteristics at the county level
4. **NPI Registry** -- county-level endocrinologist counts
5. **Manually curated / CTSA lists** -- Academic Medical Center (AMC) presence and CTSA hub flags

The county dataset provides richer geographic detail than the state dataset but has more missing data for PLACES social deprivation measures and for counties in U.S. territories.

---

## Column-by-Column Reference

### Identifiers

| Column | Type | Description |
| -------- | ------ | ----------- |
| `geo_id` | int | 5-digit FIPS county code (e.g., `1003` for Baldwin County, AL). Primary key. Standard U.S. Census FIPS format (state FIPS x 1000 + county FIPS). |
| `geo_level` | str | Always `"county"` in this file. Included for compatibility with the state-level dataset. |

---

### Geographic Coordinates

| Column | Type | Description |
| -------- | ------ | ----------- |
| `lat` | float | Latitude of the county centroid (decimal degrees). Used for distance calculations and mapping. |
| `lon` | float | Longitude of the county centroid (decimal degrees). Used for distance calculations and mapping. |

---

### Clinical Trial Access Variables

| Column | Type | Units | Description |
| -------- | ------ | ------- | ----------- |
| `county_trial_count` | int | trials | Number of unique diabetes-related ClinicalTrials.gov trials with at least one site geocoded to this county. Zero for counties with no trial sites. |
| `county_site_count` | int | sites | Number of individual site-level records for diabetes-related trials located in this county. Zero for counties with no trial sites. |
| `has_trial_site` | bool | -- | True if the county has at least one diabetes-related trial site; False otherwise. 862 of 3,221 counties (26.8%) have at least one site. |
| `dist_nearest_trial_km` | float | kilometers | Great-circle distance (Haversine) from the county centroid to the nearest diabetes-related trial site centroid. Set to 0.0 for counties with `has_trial_site` = True. **Primary geographic access barrier variable** at the county level. |
| `trials_per_100k` | float | trials per 100,000 population | county_trial_count / (pop_total / 100,000). Per-capita trial density. Zero for counties with no sites. |
| `coverage_residual` | float | trials per 100,000 | At the county level this equals `trials_per_100k` directly (no regression-based adjustment applied at county level). Retained for structural consistency with state dataset. |

---
### Socioeconomic & Demographic Variables (ACS 2022)

All variables from the U.S. Census Bureau ACS 5-Year Estimates, 2022 vintage, at the county level. Retrieved via the Census API (`https://api.census.gov/data/2022/acs/acs5`).

| Column | Type | Units | ACS Variable(s) | Description |
| -------- | ------ | ------- | ----------------- | ----------- |
| `pop_total` | float | persons | `B01003_001E` | Total county population from ACS 2022. Float due to merge join; use `pop` (int) for integer counts from the geocoding source. |
| `pop` | int | persons | County centroid reference file | Population count from the county centroid reference file used in distance calculations. Highly correlated with `pop_total`. |
| `median_hh_income` | float | USD | `B19013_001E` | Median household income. |
| `gini_index` | float | 0-1 | `B19083_001E` | Gini coefficient of income inequality. Higher = more unequal. |
| `pct_poverty` | float | percent | `B17001_002E` / `B17001_001E` | Percentage of population below the federal poverty line. |
| `pct_nhwhite` | float | percent | `B03002_003E` / `B01003_001E` | Percentage of population identifying as non-Hispanic White alone. |
| `pct_nhblack` | float | percent | `B03002_004E` / `B01003_001E` | Percentage of population identifying as non-Hispanic Black alone. |
| `pct_hispanic` | float | percent | `B03002_012E` / `B01003_001E` | Percentage of population identifying as Hispanic or Latino (any race). |
| `pct_no_vehicle` | float | percent | `B08201_002E` / `B08201_001E` | Percentage of households with no vehicle available. Proxy for transportation access barrier. |
| `pct_no_internet` | float | percent | `B28002_013E` / `B28002_001E` | Percentage of households without any internet subscription. Proxy for digital access/connectivity. |
| `pct_unemployed` | float | percent | Civilian unemployed / civilian labor force | Civilian unemployment rate. |
| `pct_uninsured` | float | percent | SAHIE-derived within ACS | Percentage of population without health insurance coverage. |
| `pct_bachelors_plus` | float | percent | Educational attainment tables | Percentage of adults 25+ with a bachelor's degree or higher. |

---

### CDC PLACES Health Measures

Pulled from CDC PLACES 2023 release via the Socrata SODA API (`data.cdc.gov`). All values are **age-adjusted prevalence estimates** as **percentages** of the adult population at the county level. Missing for 274 counties (core) and 931 counties (social deprivation).

| Column | PLACES Measure | Description |
| -------- | --------------- | ----------- |
| `places_access2` | ACCESS2 | Currently lacks health insurance (adults 18-64) |
| `places_bphigh` | BPHIGH | High blood pressure among adults |
| `places_chd` | CHD | Coronary heart disease among adults |
| `places_depression` | DEPRESSION | Current depression among adults |
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

**Missing data note:** Core health measures (`places_access2` through `places_stroke`, excluding social deprivation) are missing for 274 counties -- primarily very small-population counties and U.S. territory subdivisions. The four social deprivation measures (`places_foodinsecu`, `places_foodstamp`, `places_housinsecu`, `places_lacktrpt`) are missing for 931 counties.

---

### Healthcare Infrastructure Variables

| Column | Type | Source | Description |
| -------- | ------ | -------- | ----------- |
| `endo_count` | int | NPI Registry | Raw count of practicing endocrinologists in the county. Most counties have 0 (median = 0; 75th percentile = 1). |
| `endo_per_100k` | float | NPI Registry | Endocrinologists per 100,000 population (endo_count / (pop_total / 100,000)). Zero for counties with no registered endocrinologists. |
| `amc_count` | int | Manually curated | Number of Academic Medical Centers (AMCs) in the county. Values are 0, 1, or 2; most counties have 0. |
| `has_ctsa` | int (0/1) | NCATS CTSA Program list | Binary indicator for whether the county contains at least one NCATS-funded CTSA hub. |

---

### Derived / Composite Variables

| Column | Type | Description |
| -------- | ------ | ----------- |
| `burden_decile` | int | Decile rank (1-10, where 10 = highest burden) of county `places_diabetes` prevalence, computed across all counties with non-missing PLACES diabetes values. Counties missing PLACES diabetes receive `burden_decile` = 0. |

---## Summary Statistics

| Variable | N | Mean | SD | Min | Median | Max |
| --------- | --- | ------ | ---- | ----- | -------- | ----- |
| `county_trial_count` | 3,221 | 10.6 | 46.6 | 0 | 0 | 784 |
| `county_site_count` | 3,221 | 13.0 | 70.9 | 0 | 0 | 1,549 |
| `has_trial_site` | 3,221 | 26.8% | -- | False | False | True |
| `dist_nearest_trial_km` | 3,221 | 104.4 | 290.0 | 0.0 | 44.5 | 2,093.4 |
| `trials_per_100k` | 3,213 | 4.19 | 15.98 | 0.0 | 0.0 | 485.9 |
| `pop_total` | 3,213 | 102,944 | 329,724 | 50 | 25,839 | 9,936,690 |
| `median_hh_income` | 3,212 | $62,250 | $17,725 | $14,525 | $60,417 | $170,463 |
| `gini_index` | 3,213 | 0.448 | 0.038 | 0.274 | 0.446 | 0.721 |
| `pct_poverty` | 3,213 | 14.6% | 7.5% | 1.6% | 13.1% | 65.6% |
| `pct_nhblack` | 3,213 | 8.5% | 14.1% | 0.0% | 1.9% | 85.7% |
| `pct_hispanic` | 3,213 | 12.1% | 19.5% | 0.0% | 4.8% | 100.0% |
| `pct_no_internet` | 3,213 | 14.7% | 7.1% | 0.0% | 13.7% | 64.7% |
| `pct_uninsured` | 3,213 | 9.4% | 5.1% | 0.0% | 8.2% | 45.1% |
| `places_diabetes` | 2,947 | 11.2% | 2.3% | 6.0% | 10.8% | 23.8% |
| `places_obesity` | 2,947 | 37.6% | 4.7% | 17.1% | 38.2% | 54.0% |
| `endo_per_100k` | 3,213 | 1.54 | 5.58 | 0.0 | 0.0 | 107.3 |
| `burden_decile` | 3,221 | 4.50 | 2.87 | 0 | 4 | 9 |

---

## Missing Data Summary

| Column(s) | N Missing | Reason |
| ---------- | ----------- | -------- |
| `pop_total` and most ACS columns | 8 | County-level ACS 2022 not available for 8 county-equivalents (primarily U.S. territory subdivisions) |
| `median_hh_income` | 9 | 1 additional county with suppressed ACS income estimate |
| Core PLACES measures (`places_access2` through `places_stroke`) | 274 | Very small counties and territory subdivisions excluded from PLACES 2023 county-level estimates |
| Social deprivation (`places_foodinsecu`, `places_foodstamp`, `places_housinsecu`, `places_lacktrpt`) | 931 | Social deprivation measures available only for subset of counties in PLACES 2023 |
| `pct_nhwhite` and other ACS percentage columns | 8 | Same 8 counties as ACS missingness |

Counties with `has_trial_site` = False have `dist_nearest_trial_km` > 0; counties with a site have `dist_nearest_trial_km` = 0.0. There are no missing values in `dist_nearest_trial_km`, `has_trial_site`, `county_trial_count`, `county_site_count`, `amc_count`, `has_ctsa`, `endo_count`, `lat`, `lon`, `pop`, or `burden_decile`.

---

## Data Lineage

```
ClinicalTrials.gov v2 API
  --> ct_trials_flat.csv           (one row per trial)
  --> ct_sites_flat.csv            (one row per site, with state/city/facility)
  --> ct_sites_county.csv          (sites geocoded to county FIPS via city+state lookup)
  --> county_trial_aggregates.csv  (county_trial_count, county_site_count per FIPS)

County centroid reference (Census / geocoding)
  --> county_distance_to_trial.csv (lat, lon, pop, has_trial_site, dist_nearest_trial_km)

NPI Registry API
  --> npi_endocrinologist_county.csv (endo_count per county FIPS)

Manually curated / CTSA roster
  --> amc_presence_county.csv       (amc_count, has_ctsa per county FIPS)

US Census ACS 5-Year 2022
  --> acs_county_2022_features.csv  (SES / demographic variables per county FIPS)

CDC PLACES 2023 / SODA API
  --> cdc_places_county_wide.csv    (pivoted wide, one row per county FIPS)

Merge and feature engineering
  --> county_coverage_alignment.csv (intermediate merge of all sources, 41 cols)
  --> county_modeling_final.csv     (FINAL: 3,221 x 42)
```

---

## Construction Notes

- **Trial-to-county geocoding:** Site records from `ct_sites_flat.csv` were matched to county FIPS using a city|state lookup key against a reference table of U.S. city centroids. Unmatched records were dropped. The `ct_sites_county.csv` file contains 47,118 geocoded site records across 862 unique county FIPS codes.
- **Distance calculation:** Haversine great-circle distance was computed between each county centroid (`lat`, `lon` from the Census county centroid file) and all trial site county centroids. The minimum distance was assigned to `dist_nearest_trial_km`. Counties containing a trial site receive distance = 0.
- **`trials_per_100k` denominator:** Uses ACS 2022 `pop_total`. For counties where `pop_total` is missing (8 counties), `trials_per_100k` and `coverage_residual` are also missing.
- **`coverage_residual` at county level:** Unlike the state dataset, no regression-based residualization was performed at the county level. The column equals `trials_per_100k` directly, retained for structural compatibility with the state file.
- **`burden_decile`:** Computed using pd.qcut with 10 equal-frequency bins on `places_diabetes` across all counties with non-missing values. Counties with missing `places_diabetes` receive `burden_decile` = 0.
- **`endo_per_100k`:** Counties with no NPI-registered endocrinologists receive `endo_count` = 0 and `endo_per_100k` = 0.0 (not missing). Very high values (e.g., 107 per 100k) reflect small-population counties containing specialist practices.
- **`pop` vs `pop_total`:** `pop` is from the county centroid reference file used for distance calculations. `pop_total` is from ACS 2022. They are highly correlated but may differ slightly due to vintage differences. Use `pop_total` for per-capita rate calculations when possible.
- **FIPS code format:** Stored as integer without leading zeros. To reconstruct the 5-digit string FIPS (e.g., for map joins), use: str(geo_id).zfill(5).
