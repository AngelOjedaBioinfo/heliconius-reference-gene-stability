# Heliconius Reference Gene Stability

Statistical analysis pipeline for evaluating reference gene stability under thermal stress in *Heliconius erato lativitta*. The pipeline combines qPCR Ct data with external geNorm / NormFinder / BestKeeper / RefFinder stability rankings to test how the choice of normalization gene set affects downstream relative expression results.

[![DOI](https://zenodo.org/badge/DOI/PENDING.svg)](https://doi.org/PENDING)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

---

## Overview

The analysis runs in two stages.

1. **Ct-level analysis.** For each candidate gene, a GLM family (Gamma(log), Gamma(inverse), or Gaussian(identity)) is auto-selected by AIC and used across all hypothesis-testing models in this stage. Additive (`Treatment + Sex + Tissue`) and full-interaction (`Treatment * Sex * Tissue`) models are compared by AIC (ΔAIC > 2 threshold), and Type II Likelihood-Ratio tests are reported for every term.
2. **Relative expression (EGR) analysis.** Genes are split into `best` / `worst` reference-gene groups based on external geNorm/RefFinder stability rankings, computed upstream and outside this repo. The same GLM and Type II LR workflow is then applied to `log(EGR)` to test whether normalization-gene quality changes the detected Treatment × Sex × Tissue effects.

Every model is checked, not just fitted: Levene's test for homogeneity of variance, Q-Q plots, and residual histograms are produced for each one.

The script is parametric in genes and groups. It loops over whatever genes and quality groups are present in the input data, so it can be re-run on a different gene panel or experiment without code changes.

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

`outputs/` (individual PNG figures) is generated on render but isn't tracked in git. The knitted HTML report already contains every figure at full resolution, so the loose PNGs would just duplicate it.

## How to run

**Recommended: conda environment.** For a reproducible setup, create and activate the provided conda environment:

```bash
conda env create -f environment.yml
conda activate ref-gene-stability
```

**Manual/fallback alternative:** R (≥ 4.x) and the packages `knitr`, `rmarkdown`, `readxl`, `dplyr`, `ggplot2`, `ggpubr`, `rstatix`, `tidyr`, `car`, `broom`, `purrr`, `scales`, `kableExtra`, `patchwork`. The script installs any missing package on first run.

```r
rmarkdown::render("analysis/reference_gene_stability_analysis.Rmd")
```

`knitr` sets the working directory to the script's own location, so paths to `../data/All_data.xlsx` and `../outputs/` resolve correctly without any manual `setwd()`.

## Methods summary

Rather than assuming one error distribution for every gene, the script fits all valid GLM families per gene (Gamma needs strictly positive values; Gaussian is the fallback when a response can be zero or negative, like `log(EGR)`) and picks whichever wins on AIC most often as the consensus family for that stage.

Additive and interaction models are then compared per group by AIC, using |ΔAIC| > 2 as the cutoff for preferring the more complex model. Below that threshold, the simpler additive model wins on parsimony.

All GLM terms are tested with Type II Likelihood-Ratio tests, which don't assume an ordering between Treatment, Sex, and Tissue, so they fit the unbalanced design used here. Levene's test, Q-Q plots, and residual histograms are generated automatically for every fitted model, so the fit can be checked rather than assumed.

## Case study: *H. erato lativitta* thermal-stress reference-gene panel

The dataset in this repository comes from a qPCR reference-gene validation study in *Heliconius erato lativitta* (Treatment: Cold / Heat / Control; Tissue: Head / Thorax / Abdomen; Sex: Male / Female), carried out to identify the most stable normalization genes ahead of downstream HSP40/HSP90 thermal-stress expression analyses. Full results, figures, and the automatically generated AIC interpretation are in [`report/reference_gene_stability_report.html`](report/reference_gene_stability_report.html).

This analysis supports a manuscript currently in preparation (citation to be added once available as a preprint).

## Citation

If you use this pipeline, cite it via the metadata in [`CITATION.cff`](CITATION.cff) (also exposed by GitHub's "Cite this repository" button) or directly via the Zenodo DOI above.

## Acknowledgments

This pipeline was developed for a manuscript in preparation led by Paola Calderón and Joshué Ruiz, with Leo Camino and Caroline Bacquet (Universidad Regional Amazónica Ikiam). Shane Wright (LMU Munich) also contributed methodological input during the development of the underlying experiment.

## License

MIT. See [LICENSE](LICENSE).
