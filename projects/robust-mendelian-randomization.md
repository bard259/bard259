# Robust Mendelian Randomization

Applies robust optimization to construct tighter, more reliable confidence intervals for Mendelian Randomization (MR-SPI) causal effect estimates. Rather than assuming all selected instrumental variables (IVs) are valid, the method finds the tightest interval that remains valid under a worst-case subset of invalid IVs.

Preprint: `2023.02.20.23286200v3.full.pdf`

## Background

Mendelian Randomization uses genetic variants (SNPs) as instrumental variables to estimate the causal effect of an exposure (e.g., LDL cholesterol) on an outcome (e.g., coronary artery disease) using summary-level GWAS data. The MR-SPI method selects valid IVs automatically, but its confidence intervals can be sensitive to the boundary cases.

This work applies the robust optimization philosophy: instead of assuming the selected IV set is exactly correct, it computes the smallest confidence interval that would be valid for any subset of size ≥ ⌊n/2⌋ + 1 of the selected IVs (loose) or ≥ ⌊n/2⌋ (strict).

## Contents

- `MR.SPI Robust CI .Rmd` — R Markdown with the full analysis, simulation studies, and Monte Carlo coverage experiments
- `MR.SPI-Robust-CI-.pdf` — knitted PDF output
- `MR_SPI_Robust_CI.pdf` — polished version
- `2023.02.20.23286200v3.full.pdf` — bioRxiv preprint (baseline MR-SPI paper)

## Running the analysis

```r
# Install dependencies
install.packages(c("CVXR", "bigstatsr", "igraph", "intervals", "matrixStats", "svMisc"))
devtools::install_github("MinhaoYaooo/MR-SPI")

# Knit the report
rmarkdown::render("MR.SPI Robust CI .Rmd")
```

> **Note:** Update the `setwd()` path in the Rmd to your local directory, or replace it with `here::here()` after installing the `here` package.

## Key method: `robust_ci_temp()`

Given GWAS summary statistics `(gamma, Gamma, se_gamma, se_Gamma)` for `pz` SNPs:

1. Constructs covariance matrices for the exposure and outcome effect estimates
2. Iterates over all subsets of size ≥ ⌊pz/2⌋+1 of the selected IVs
3. For each subset, computes a two-sample TSHT estimator and its variance
4. Returns the union of all resulting CIs — the tightest interval valid under worst-case IV invalidity

## Simulation results

Monte Carlo experiments (1,000 iterations, n₁=10,000 exposure, n₂=20,000 outcome, 10 SNPs) compare:
- Default MR-SPI CI
- Robust CI (this work)
- Target coverage: ≥ 95%

## Roadmap

- [ ] **Fix hardcoded `setwd` path** — replace with `here::here()` for portability
- [ ] **Add `renv` lockfile** — pin R and package versions for reproducibility
- [ ] **Package as R package** — exportable functions with `roxygen2` documentation
- [ ] **End-to-end reproducibility script** — one command installs dependencies and replicates the main table and figure
- [ ] **Real data example** — demonstrate on a published GWAS dataset (e.g., from IEU Open GWAS)

## License

MIT
