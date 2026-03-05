# Repository Guidelines

## Project Structure & Module Organization

This repository is organized by research option:

- `option-a-drug-safety/`
- `option-b-ed-timeliness/`
- `option-c-trial-access/`
- `option-d-conflicts-of-interest/`

Each option follows the same layout:

- `discover/`: exploratory work (notebooks/scripts) and `modified_data/`
- `product/`: final deliverable and reproducible pipeline
- `product/data/raw/`: immutable API pulls
- `product/data/modified/`: cleaned/intermediate datasets used by Quarto
- `product/option-<letter>.qmd`: one production report per option
- `PLAN.md`: Plan for API Call, EDA, and modeling for that option

Use `docs/` for shared project documentation and `.env` (gitignored) for API keys.

## Documentation Specification

- `clinical_midterm_options.md`: The specification of the latest version of the four options
- `deep-research-report.md`: Background information and research question motivations

## Build, Test, and Development Commands

- `quarto preview option-a-drug-safety/product/option-a.qmd`: live preview while editing.
- `quarto render option-b-ed-timeliness/product/option-b.qmd`: execute and render a final HTML report.
- `Get-ChildItem option-*\\product\\option-*.qmd | ForEach-Object { quarto render $_.FullName }`: render all option reports in PowerShell.
- `jupyter notebook option-b-ed-timeliness/discover/data_acquiring/api_call_data_grabbing.ipynb`: run discovery/data-acquisition workflow.

## Coding Style & Naming Conventions

Use clear, reproducible analysis code and keep Quarto documents sectioned (`Introduction`, `Data Sources`, `Methods`, `EDA`, `Limitations`, `Conclusion`). Prefer:

- 4-space indentation in Python code cells
- `snake_case` for variables/files (for example, `cms_ed_meta_yv7e-xc69.json`)
- descriptive, option-scoped filenames (`option-b.qmd`, `faers_semaglutide_2021_2024.json`)

Do not rewrite raw data files after download; create transformed outputs in `modified/` or `modified_data/`.

## Testing Guidelines

There is no dedicated automated test suite yet. Treat report rendering as the required validation step:

1. Run `quarto render` for the affected option.
2. Confirm no execution errors and that figures/tables load from expected paths.
3. For notebook edits, rerun changed cells from a clean kernel and verify outputs written to the correct data directory.

## Commit & Pull Request Guidelines

Existing history uses short, sentence-case messages (for example, `Initial commit`). Follow that style with scope:

- `option-c: refine mismatch index calculation`
- `option-b: update CMS raw extract metadata`

PRs should include: purpose, affected option(s), changed data sources/APIs, reproduction command(s), and a screenshot or rendered HTML snippet when report output changes. Mention any new environment variables explicitly.
