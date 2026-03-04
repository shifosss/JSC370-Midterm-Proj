# Four Updated Research Options (EDA + ML Runway)

These options are written to be **EDA-friendly for the midterm** while still having a clear **machine-learning extension** for the final project.

---

## Option A — Post‑market drug safety “signals” vs public attention (time‑series + lead/lag)

### Research questions (ML‑ready, not obvious at a glance)
1. **Detection delay:** For each drug, what is the time lag between a sustained increase in *serious* adverse event reporting and a sustained increase in *public attention* (news + Wikipedia)?
2. **Selectivity:** Does attention track **seriousness‑weighted** safety signals (severity composition) more than raw report volume?
3. **Heterogeneity:** Do lag patterns differ systematically by drug class (e.g., psychiatric vs metabolic)?

### APIs + exactly what to pull
**1) openFDA drug adverse events (FAERS-derived)**  
Pull at minimum:
- Report/time: `safetyreportid`, `receivedate`, `receiptdate`
- Seriousness flags: `serious`, `seriousnessdeath`, `seriousnesshospitalization`, `seriousnesslifethreatening`, `seriousnessdisabling`, `seriousnesscongenitalanomali`, `seriousnessother`
- Geography/source: `occurcountry`, `primarysourcecountry`
- Patient container (extract within): `patient` → drug names, reactions, basic demographics

**2) GDELT DOC API (news attention proxy)**  
Pull:
- Article list: `url`, `title`, `seendate`, `domain`, `language`, `sourcecountry`
- Time series: `timelinevol` / `timelinevolraw`, `timelinetone`

**3) Wikimedia Pageviews API (public interest proxy)**  
Pull:
- Daily pageviews time series per article (drug page) for the same date range

### Midterm EDA deliverables
- Weekly alignment of: adverse event counts, serious-event counts, seriousness share, news volume, wiki pageviews
- Lead/lag exploration with cross-correlation and change-point style visualizations

### Final ML extension
- Forecasting / anomaly detection on seriousness‑weighted series and models predicting “attention spikes” from safety-signal features

---

## Option B — ED timeliness vs provider supply + socioeconomic context (geo joins + prediction)

### Research questions (ML‑ready, not obvious at a glance)
1. **Effect modification:** Is the association between poverty (or uninsured rate) and ED delays weaker in counties with higher primary‑care density?
2. **Outlier profiling:** Which hospitals are unexpectedly fast/slow given their county context (residual analysis)?
3. **Predictability:** How well can ED timeliness be predicted from county socioeconomic variables + provider density, and which features matter most?

### APIs + exactly what to pull
**1) CMS hospital timeliness measures (API via dataset endpoints)**  
Pull:
- Hospital ID/name and location: provider/hospital ID, name, address/ZIP, state (county if present)
- Measure identifiers: measure ID/name for ED timeliness
- Measure values: score/rate and date/year fields

**2) US Census ACS API (county context)**  
Pull county-level:
- Population denominators and age structure
- Poverty rate / income measures
- Insurance coverage measures (uninsured rate)
- Any additional covariates you justify (e.g., urban/rural proxies)

**3) NPI Registry (provider density)**  
Pull:
- `npi` (identifier)
- `taxonomies` (filter primary care)
- `addresses` (practice location)
Aggregate to county/ZIP/state to compute provider density.

### Midterm EDA deliverables
- Choropleths / binned plots: ED timeliness vs poverty quartiles, overlays of provider density
- Baseline modeling as supporting evidence (keep focus on EDA and interpretation)

### Final ML extension
- Tree-based models to predict timeliness; interpretable feature importance; interactive hospital/county dashboard

---

## Option C — Clinical trial site distribution vs disease burden (geography + modeling)

### Research questions (ML-ready, not obvious at a glance)
1. **Nonlinear alignment:** How does **trial site density** change with **disease burden** across geographies (linear vs thresholded vs saturating)?
2. **Sponsor effects:** After controlling for burden and basic SES context, do **industry vs academic** trials show different geographic concentration patterns?
3. **Coverage outliers (model-based, no composite index):** Which areas have **higher/lower trial site density than expected** given burden + covariates (identified via model residuals rather than a handcrafted “mismatch” score)?
4. **Topic segmentation:** Do trial topics/subtypes (from outcomes/eligibility text) cluster into groups with distinct geographic footprints?

### APIs + exactly what to pull
**1) ClinicalTrials.gov v2 API**  
Pull:
- Trial identifiers/titles: `protocolSection.identificationModule.nctId`, `protocolSection.identificationModule.officialTitle`
- Locations: `protocolSection.contactsLocationsModule.locations` (extract state/country; optionally city)
- Trial descriptors: phase, status (recruiting/completed), enrollment (if available), sponsor type (if available)
- Text fields for topic modeling (if available in returned modules): brief summary, eligibility, primary outcomes
- Filter using condition query parameter: `query.cond`

**2) CDC PLACES (burden proxy; Socrata API)**  
Pull:
- Geography keys: `locationid`, `locationname`, `stateabbr`
- Measure key: `measureid` (choose measures aligned with your condition)
- Outcome and uncertainty: `data_value`, `low_confidence_limit`, `high_confidence_limit`
- Value type: `datavaluetypeid` (prefer age-adjusted where appropriate)

**3) US Census ACS (optional, explanatory covariates)**  
Pull SES features you will explicitly use (e.g., poverty rate, median income, insurance coverage), at the same geographic granularity.

### Midterm EDA deliverables
- Trial site density per 100k (state or county) by: sponsor type, phase, status
- Burden prevalence estimates with confidence intervals
- Scatter + binned plots of site density vs burden (with uncertainty bands)
- Maps of **site density** and **burden** side-by-side
- A baseline “expected density” model (e.g., Poisson/negative binomial or tree model) and visualization of **residuals** to highlight over/under-covered areas (without defining a bespoke index)

### Final ML extension
- Predict site density (or probability of having any site) from burden + SES + infrastructure proxies
- Topic modeling/embeddings for trial text and stratified geographic analyses by topic cluster
- Interactive explorer: filter by condition, sponsor type, phase, topic cluster, and geography

---

## Option D — Conflicts of interest vs evidence dissemination (record linkage + supervised models)

### Research questions (ML‑ready, not obvious at a glance)
1. **Results posting:** Are higher industry payments associated with higher/lower probability that a trial posts results, controlling for sponsor/phase/condition?
2. **Time-to-publication:** Among trials with linked publications, is payment level associated with publication delay?
3. **Confounding audit:** How sensitive are conclusions to match-confidence tiers in your record linkage?

### APIs + exactly what to pull
**1) ClinicalTrials.gov v2 API**  
Pull:
- Trial ID/title and key dates (e.g., completion)
- Results status (posted vs not, where available)
- Investigator names/locations (where available)
- Condition filter: `query.cond`

**2) CMS Open Payments**  
Pull:
- Amount/date: `Total_Amount_of_Payment_USDollars`, `Date_of_Payment`
- Payment details: `Form_of_Payment_or_Transfer_of_Value`, `Nature_of_Payment_or_Transfer_of_Value`
- Company fields: `Submitting_Applicable_Manufacturer_or_Applicable_GPO_Name`, `Applicable_Manufacturer_or_Applicable_GPO_Making_Payment_Name`
- Linkage identifiers: `Covered_Recipient_Profile_ID`, `Covered_Recipient_NPI`
- Transaction identifier: `Record_ID`

**3) PubMed (NCBI E‑utilities)**  
Pull:
- Search by NCT identifiers (e.g., `NCT#######`) appearing in abstracts/full metadata
- Return: PMID, publication date, journal, title, extracted NCT IDs

**4) OpenAlex (optional)**  
Pull citations/venue metadata if you decide to incorporate impact-style outcomes (note: API-key/limits may apply).

### Midterm EDA deliverables
- Payment distributions by specialty/company
- Share of trials with results posted; share with linked publications
- Linkage QA: manual sample checks + match-confidence strata

### Final ML extension
- Supervised models predicting results posted/published; improved probabilistic record linkage; interactive exploration by sponsor/condition

---

## Recommended scope controls (to keep this feasible)
- Choose **one condition area** and one geographic level (state OR county) for the midterm.
- Use a focused time window and a small curated drug/hospital/trial list initially, then scale up if time allows.
