# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

JSC370 midterm project exploring clinical/health research questions using programmatic data acquisition (APIs), EDA, and reproducible Quarto pipelines. The deliverable is a rendered `.qmd` report per selected option.

## Common Commands

```bash
# Render a production QMD to HTML
quarto render option-a-drug-safety/product/option-a.qmd

# Preview with live reload
quarto preview option-a-drug-safety/product/option-a.qmd

# Run a discovery script
python option-a-drug-safety/discover/fetch_raw.py
```

## Repository Structure

```
<option-name>/
  discover/               # Exploration, API prototyping, EDA notebooks/scripts
    modified_data/        # Cleaned data produced during exploration (not committed if large)
  product/
    data/
      raw/                # Raw API responses — never modified after download
      modified/           # Cleaned data used by the production QMD (commit if small)
    <option>.qmd          # Exactly ONE production Quarto document per option
```

**Data flow rule:** `discover/` scripts read from `product/data/raw/` and write to `discover/modified_data/`. The production `product/<option>.qmd` reads from `product/data/raw/` and may write intermediate outputs to `product/data/modified/`.

## Four Research Options

| Dir | Option | Primary APIs |
|-----|--------|-------------|
| `option-a-drug-safety/` | A — Post-market drug safety signals vs public attention | openFDA (FAERS), GDELT DOC API, Wikimedia Pageviews |
| `option-b-ed-timeliness/` | B — ED timeliness vs provider supply + SES | CMS Hospital Timeliness, US Census ACS, NPI Registry |
| `option-c-trial-access/` | C — Clinical trial site density vs disease burden | ClinicalTrials.gov v2, CDC PLACES (Socrata/SODA), US Census ACS |
| `option-d-conflicts-of-interest/` | D — Industry payments vs trial results dissemination | ClinicalTrials.gov v2, CMS Open Payments, PubMed E-utilities, OpenAlex |

## API Patterns

**openFDA** — Elasticsearch-backed REST. Max 1000 records per call; use `skip` for pagination. Adverse event endpoint: `https://api.fda.gov/drug/event.json`. Reports are unverified passive surveillance; never imply causality.

**GDELT DOC API** — Query with `query`, `mode=artlist` or `mode=timelinevol`; filter by `startdatetime`/`enddatetime`.

**ClinicalTrials.gov v2** — `https://clinicaltrials.gov/api/v2/studies`; use `query.cond` for condition, `pageSize` (max 1000), `pageToken` for cursor-based pagination. Check `/api/v2/version` for freshness.

**Socrata/SODA (CDC PLACES, CMS)** — Page with `$limit` + `$offset`; always add `$order` for stable pagination. CDC endpoint pattern: `https://data.cdc.gov/resource/<dataset-uid>.json`.

**US Census ACS** — `https://api.census.gov/data/<year>/acs/acs5`; requires `key` param (free API key). Use `get` for variable list and `for`/`in` for geography filters.

**NPI Registry** — `https://npiregistry.cms.hhs.gov/api/`; filter by `taxonomy_description` for primary care; paginate with `skip` + `limit` (max 200).

**CMS Open Payments** — Socrata endpoint on `openpaymentsdata.cms.gov`; filter by `Nature_of_Payment_or_Transfer_of_Value` and year.

**PubMed E-utilities** — `esearch.fcgi` to find PMIDs by NCT ID query, then `efetch.fcgi` for metadata. Respect the 3 req/s unauthenticated limit (10/s with API key from NCBI).

**OpenAlex** — Requires API key as of Feb 2026; credit-based rate limiting. Cache responses to avoid re-fetching.

## Conventions

- Store API keys in `.env` (gitignored); load via `python-dotenv` or `Sys.getenv()` in R.
- Save raw API responses with a timestamped filename (e.g., `faers_20240101.json`) in `product/data/raw/`.
- Document pagination strategy and download date in the methods section of the QMD.
- All analysis must cite official API disclaimers in the Limitations section (especially openFDA).
- Keep scope controlled: one condition area, one geographic level, focused time window.
