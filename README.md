# Heliconius Reference Gene Stability

Statistical analysis pipeline for evaluating **reference gene stability under thermal stress** in *Heliconius erato lativitta*. The pipeline combines qPCR Ct data with external geNorm / NormFinder / BestKeeper / RefFinder stability rankings to test how the choice of normalization gene set affects downstream relative expression results.

[![DOI](https://zenodo.org/badge/DOI/PENDING.svg)](https://doi.org/PENDING)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

---

## Overview

The analysis runs in two stages:

1. **Ct-level analysis** — for each candidate gene, a GLM family (Gamma(log), Gamma(inverse), or Gaussian(identity)) is auto-selected by AIC, then used consistently across all hypothesis-testing models for that stage. Additive (`Treatment + Sex + Tissue`) and full-interaction (`Treatment * Sex * Tissue`) models are compared by AIC (ΔAIC > 2 threshold), and Type II Likelihood-Ratio tests are reported for every term.
2. **Relative expression (EGR) analysis** — genes are split into `best` / `worst` reference-gene groups based on external geNorm/RefFinder stability rankings (computed upstream, outside this repo). The same GLM + Type II LR workflow is then applied to `log(EGR)` to test whether normalization-gene quality changes the detected Treatment × Sex × Tissue effects.

Every model is accompanied by Levene's test for homogeneity of variance, Q-Q plots, and residual histograms, so model fit can be checked rather than assumed.

The script is parametric in genes and groups: it loops over whatever genes/quality groups are present in the input data, so it can be re-run on a different gene panel or experiment without code changes.

## Repository structure

```
heliconius-reference-gene-stability/
├── analysis/
│   └── reference_gene_stability_analysis.Rmd   # main R Markdown analysis script
├── data/
│   ├── All_data.xlsx                           # Ct + EGR data
│   └── README.md                                # data dictionary
├── report/
│   └── reference_gene_stability_report.html    # full knitted report (all tables + figures)
├── LICENSE
├── CITATION.cff
└── README.md
```

`outputs/` (individual PNG figures) is generated on render but not tracked in git — the knitted HTML report already contains every figure at full resolution.

## How to run

Requirements: R (≥ 4.x) and the packages `knitr`, `rmarkdown`, `readxl`, `dplyr`, `ggplot2`, `ggpubr`, `rstatix`, `tidyr`, `car`, `broom`, `purrr`, `scales`, `kableExtra`, `patchwork`. The script installs any missing package on first run.

```r
rmarkdown::render("analysis/reference_gene_stability_analysis.Rmd")
```

`knitr` sets the working directory to the script's own location, so paths to `../data/All_data.xlsx` and `../outputs/` resolve correctly without any manual `setwd()`.

## Methods summary

- **Family selection by AIC** — rather than assuming a single error distribution for every gene, the script fits all valid candidate families (Gamma requires strictly positive values; Gaussian is the fallback for response variables that can be zero or negative, e.g. `log(EGR)`) and picks a consensus family per analysis stage.
- **Model comparison by AIC** — additive vs. interaction models are compared per group, with |ΔAIC| > 2 as the threshold for preferring the more complex model, balancing fit against parsimony.
- **Type II Likelihood-Ratio tests** — used for all GLM terms, appropriate for unbalanced designs with no implied ordering between Treatment, Sex, and Tissue.
- **Diagnostics** — Levene's test, Q-Q plots, and residual histograms are produced automatically for every fitted model.

## Case study: *H. erato lativitta* thermal-stress reference-gene panel

The dataset in this repository comes from a qPCR reference-gene validation study in *Heliconius erato lativitta* (Treatment: Cold / Heat / Control; Tissue: Head / Thorax / Abdomen; Sex: Male / Female), carried out to identify the most stable normalization genes ahead of downstream HSP40/HSP90 thermal-stress expression analyses. Full results, figures, and the automatically generated AIC interpretation are in [`report/reference_gene_stability_report.html`](report/reference_gene_stability_report.html).

This analysis supports a manuscript currently in preparation (citation to be added once available as a preprint).

## Citation

If you use this pipeline, cite it via the metadata in [`CITATION.cff`](CITATION.cff) (also exposed by GitHub's "Cite this repository" button) or directly via the Zenodo DOI above.

## Acknowledgments

This pipeline was developed for a manuscript in preparation led by Paola Calderón and Joshué Ruiz, with Leo Camino and Caroline Bacquet (Universidad Regional Amazónica Ikiam). Shane Wright (LMU Munich) also contributed methodological input during the development of the underlying experiment.

## License

MIT — see [LICENSE](LICENSE).
