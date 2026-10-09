# CellEvoTree 0.1.0

Initial public release. This version relabels the established implementation for
publication; it does not change the tree-building algorithms or analysis results.

- Build sample-aware metacells and expression-first regulatory cell-state trees.
- Estimate primary gene and TF-mRNA distance weights and uncertainty by
  split-feature resampling, with reliability and sample-stability diagnostics.
- Permit local regulon and module refinement only within protected primary
  topology constraints; retain a balanced multi-view compatibility mode.
- Use Slingshot or Monocle3 pseudotime and RNA velocity as directional evidence
  without treating cell states as genetic lineage.
- Quantify whole-sample leave-one-out topology stability, sister-branch support,
  branch-level predictive contributions, and selected feature differences.
- Infer state-specific regulatory characters from explicit TF-target priors and
  summarize Gain, Loss, Strengthen, and Weaken events.
- Provide known-truth topology and GRN-character simulations, perturbation and
  ablation benchmarks, and external graph-scoring adapters.
- Visualize circular trees with multiple heatmap or bubble tracks; explore
  branches interactively, export figures, and annotate descendant cells in
  Seurat for downstream analysis.
- Include workflow examples, source-data tables, publication figures, tests,
  and manuscript materials in the public repository.
