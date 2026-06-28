# Data dictionary: `All_data.xlsx`

> **Draft.** Column names and value categories below are inferred directly from the analysis script (`analysis/reference_gene_stability_analysis.Rmd`). Confirm against the actual file once it's in place, and correct anything that doesn't match: in particular the exact gene names/IDs in `Gene`, and whether `Quality` has only `best`/`worst` or additional categories.

## Sheet `Cq_tagged`

| Column      | Type    | Description                                                  |
|-------------|---------|----------------------------------------------------------------|
| `Gene`      | factor  | Candidate reference gene ID                                   |
| `Treatment` | factor  | `Cold` / `Heat` / `Control`                                    |
| `Sex`       | factor  | `Male` / `Female`                                              |
| `Tissue`    | factor  | `H` (Head) / `T` (Thorax) / `A` (Abdomen)                       |
| `Ct`        | numeric | qPCR cycle threshold value                                     |

## Sheet `EGR`

| Column      | Type    | Description                                                              |
|-------------|---------|---------------------------------------------------------------------------|
| `Gene`      | factor  | Candidate reference gene ID                                              |
| `Quality`   | factor  | Gene-stability classification (`best` / `worst`) from external geNorm / NormFinder / BestKeeper / RefFinder ranking, computed upstream and not by this repo |
| `Treatment` | factor  | `Cold` / `Heat` / `Control`                                               |
| `Sex`       | factor  | `Male` / `Female`                                                        |
| `Tissue`    | factor  | `H` / `T` / `A`                                                          |
| `EGR`       | numeric | Relative gene expression ratio (target gene normalized by reference gene set) |
| `Log_EGR`   | numeric | Natural-log-transformed `EGR`                                            |

## Notes

- Both sheets are read with `na.omit()` applied right after typing, so rows with missing values in any used column are dropped silently, not flagged.
- The stability ranking that produces `Quality` is **not computed in this repository**. It comes from a separate geNorm/RefFinder step run upstream; this pipeline only tests the downstream statistical consequences of that ranking.
