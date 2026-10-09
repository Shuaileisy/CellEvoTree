# Contributing to CellEvoTree

Contributions should preserve the package's central distinction between a
regulatory cell-state tree and a genetic phylogeny.

## Before opening a pull request

1. Open an issue describing the scientific question, expected behavior, and
   assumptions.
2. Keep changes focused and avoid committing private single-cell objects,
   credentials, patient information, or machine-local paths.
3. Add focused `testthat` coverage for changed behavior.
4. Run:

```r
devtools::document()
devtools::test()
devtools::check(args = "--no-manual")
```

5. Explain whether the change affects primary expression geometry, secondary
   topology refinement, rooting, branch attribution, or visualization only.

## Reproducible examples

Use simulated or openly licensed data. Do not require unpublished local files
to reproduce a bug.

## Scientific claims

New biological interpretations must distinguish statistical association,
state separation, predicted regulation, and experimentally demonstrated
causality.

