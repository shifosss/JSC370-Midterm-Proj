# Repository Guidelines

## Project Structure & Module Organization

This repository is now centered on `option-c-trial-access/`. Treat Option C as the only active workstream unless the user explicitly asks to revive archived material.

Active Option C structure:

- `option-c-trial-access/discover/data_acquiring/`: exploratory acquisition notebooks, including `api_call_data_grabbing.ipynb`
- `option-c-trial-access/discover/modified_data/`: discovery-stage modeling tables such as `county_modeling_final.csv` and `state_modeling_final.csv`
- `option-c-trial-access/discover/results/`: exploratory figures and maps used to assess coverage, burden, SES, and sponsor patterns
- `option-c-trial-access/discover/County_level_data_documentation.md`: county-level variable and source notes
- `option-c-trial-access/discover/State_level_data_documentation.md`: state-level variable and source notes
- `option-c-trial-access/product/option-c.qmd`: production Quarto report
- `option-c-trial-access/product/data/raw/`: immutable raw API pulls for the production pipeline
- `option-c-trial-access/product/data/modified/`: cleaned datasets consumed by the Quarto report
- `option-c-trial-access/product/results/`: rendered/report-ready outputs tied to the production workflow

Archived reference-only work:

- `option-a-drug-safety/`
- `option-b-ed-timeliness/`
- `option-d-conflicts-of-interest/`

Do not add new analysis, pipelines, or documentation to archived options unless explicitly requested. Use them only as historical reference for patterns or prior decisions.

Shared project documents live in `docs/`:

- `docs/clinical_midterm_options.md`: original option specification
- `docs/deep-research-report.md`: background framing and research motivation

Keep secrets in `.env` (gitignored).

## Build, Test, and Development Commands

- `quarto preview option-c-trial-access/product/option-c.qmd`: live preview while editing the active report
- `quarto render option-c-trial-access/product/option-c.qmd`: execute and render the Option C report
- `jupyter notebook option-c-trial-access/discover/data_acquiring/api_call_data_grabbing.ipynb`: run the discovery/data-acquisition notebook
- `Get-ChildItem option-c-trial-access\\discover\\results`: quickly inspect exploratory output artifacts

Avoid repo-wide render commands by default; the archived options are no longer part of the active lifecycle.

## Coding Style & Naming Conventions

Use clear, reproducible analysis code and keep `option-c.qmd` sectioned around the current report flow (`Introduction`, `Data Sources`, `Methods`, `EDA`, `Coverage Outlier Analysis`, `Limitations`, `Conclusion and Final-Project Plan`).

Prefer:

- 4-space indentation in Python code cells
- `snake_case` for variables and filenames
- descriptive Option C filenames such as `county_modeling_final.csv` or `map_trial_density_tile.png`

Do not rewrite raw data files after download. New transformed datasets belong in `discover/modified_data/` or `product/data/modified/`. New figures belong in `discover/results/` for exploratory work or `product/results/` for production outputs.

## Testing Guidelines

There is no dedicated automated test suite yet. Treat Option C rendering and reproducibility checks as the required validation step:

1. Run `quarto render option-c-trial-access/product/option-c.qmd` after report or production-data changes.
2. Confirm there are no execution errors and that tables/figures resolve from the expected Option C paths.
3. For notebook edits, rerun changed cells from a clean kernel and verify outputs land in the intended `discover/` or `product/` data directory.
4. If you update documentation references or filenames, make sure the linked paths still exist.

## Commit & Pull Request Guidelines

Existing history uses short, sentence-case messages. Keep that style, scoped to the active workstream:

- `option-c: refine county coverage residual plots`
- `option-c: update trial access data documentation`
- `docs: archive inactive options in repo guidance`

PRs should include: purpose, affected Option C components, changed data sources or APIs, reproduction commands, and a screenshot or rendered HTML snippet when report output changes. Mention any new environment variables explicitly.
