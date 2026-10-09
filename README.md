# CellEvoTree

CellEvoTree builds phylogeny-inspired cell state evolution trees from
single-cell data. It is designed for cell state dynamics, not true genetic
lineage or mutation-based phylogeny.

Current release: **0.1.0**.

The `paper/` directory contains publication figures, compact source-data tables,
sample metadata, and reproducibility scripts. Manuscript drafts, large processed
objects, and raw sequencing files are maintained outside Git history. Public
accessions and deposition requirements are documented under `paper/`.

Before publishing a fork of this prepared repository, replace the identity
placeholders listed in `GITHUB_SETUP_REQUIRED.md`.

## Installation

After the repository owner is confirmed, install the development release from
GitHub with:

```r
install.packages("pak")
pak::pak("YOUR_GITHUB_USERNAME/CellEvoTree")
```

The checked source archive can also be installed directly:

```r
install.packages("CellEvoTree_0.1.0.tar.gz", repos = NULL, type = "source")
```

## Core Idea

```text
single-cell data
  -> metacells / cell states
  -> Gene expression + actual TF mRNA expression
  -> resampling-based primary weights and uncertainty
  -> sample-aware evidence audit (warn/downweight/demote)
  -> expression-primary average-linkage tree
  -> regulon/module refinement under protected primary splits
  -> Slingshot, velocity, UMAP, and CellRank direction/diagnostics
  -> sister-support audit and branch explanations
```

The package includes scalable topology evaluation and regulatory-character truth
tests while preserving the expression-first topology policy:

```text
constrained NNI candidates
  -> one topology-distance calculation per candidate
  -> cached primary ranks and neighbors
  -> primary-fidelity short circuit
  -> exact secondary scoring only for feasible candidates

SCENIC activity + target expression + TF-target edge prior
  -> state-specific edge weights
  -> 0/1/2 regulatory characters
  -> Gain/Loss/Strengthen/Weaken events
  -> known-truth event AUPRC and lift over prevalence
```

The publication-oriented benchmark layer provides:

```text
known-truth multimodal simulation or independent reference data
  -> CellEvoTree internal methods + Slingshot + Monocle3
  -> common graph geodesic and true-neighbor metrics
  -> paired bootstrap confidence intervals and Wilcoxon tests
  -> BH correction, method failure rates, and multi-metric ranks
```

Tree-only Robinson-Foulds scores are retained for binary tree comparisons but
are not used to judge principal-graph methods. Independent reference datasets
can be registered with `CreateCellEvoReferenceBenchmarkData()`.

The validation layer runs after the tree is built:

```text
rooted CellEvoTree result
  -> per-edge Slingshot/velocity direction support
  -> whole-sample leave-one-out topology stability
  -> leave-one-sample-out branch discrimination
  -> exact Shapley attribution across gene/regulon/module layers
  -> optional multi-start constrained NNI sensitivity analysis
```

The default topology is deliberately hierarchical. Gene expression and TF
mRNA define which states may be sisters. Regulon activity and module scores
may improve uncertain local topology only when the change stays within
empirical primary-distance and primary-neighborhood tolerances and preserves
bootstrap-supported primary splits. Slingshot and velocity orient or diagnose
the tree; they are not mixed into the primary sister distance.

Cell-type labels are annotation-only by default. They do not enter primary
distances, topology selection, or the default metacell boundary. With
`metacell.stratify = TRUE`, metacells are built within `group.by` strata.
Use `metacell.stratify = FALSE` for a fully unsupervised sensitivity run.
The legacy group-by-celltype boundary remains available only through the
explicit `celltype.role = "metacell_stratum"` option.

## Role-Separated Evidence Audit

Every data layer has a defined role. Expression is the protected reference
geometry. TF mRNA may join that geometry only after auditing. Regulons and
modules are secondary local refiners, pseudotime and RNA velocity orient the
tree, and genes/regulons/modules explain selected sister branches. The default
audit warns but does not silently change a result:

```r
res <- CellEvoTree(
  object = scRNA,
  group.by = "group",
  sample.by = "sample",
  evidence.audit = "warn",
  features = list(
    expression = TRUE,
    regulon = scenic_auc,
    module = modules,
    pseudotime = monocle3_pseudotime,
    velocity = velocity_input
  )
)

res$evidence_audit$table
```

Use `evidence.audit = "downweight"` to multiply optional primary weights by
explicit audit factors, or `"demote"` to keep flagged optional layers for
interpretation without allowing them into topology. Because branch discovery
and branch testing can otherwise reuse the same target feature, publication
analyses should locate target-enriched microstates in one sample subset and
confirm the target and its neighbors in held-out samples or with a
leave-feature-out tree.

Regulon activity is derived from expression but need not preserve identical
local geometry. Quantify rather than assume this distinction:

```r
nonredundancy <- AssessLayerNonredundancy(
  res$features$expression,
  res$features$regulon,
  n.resamples = 100,
  n.states = floor(0.8 * nrow(res$features$expression)),
  k = 10
)
nonredundancy$summary
```

## Sample-Aware Validation

```r
direction_support <- ComputeLineageDirectionSupport(
  res,
  n.bootstrap = 500,
  sample.column = "sample_majority"
)

sample_stability <- AssessSampleTopologyStability(
  res,
  mode = "leave_one_sample_out",
  sample.column = "sample_majority"
)

branch_cv <- ComputeBranchContributionCV(
  result = res,
  node = selected_node,
  feature.layers = list(
    gene = gene_metacell_matrix,
    regulon = regulon_metacell_matrix,
    module = module_metacell_matrix
  ),
  sample.column = "sample_majority",
  score.unit = "sample",
  feature.selection = "sample_effect",
  n.bootstrap = 500,
  n.permutations = 999
)

multistart <- OptimizeTopologyMultistart(
  res,
  n.starts = 20,
  perturb.moves = 3
)

scalable <- OptimizeTopologyScalable(
  res,
  mode = "balanced",
  memory.limit.gb = 64
)
```

The Shapley percentages are held-out branch-discrimination contributions. They
are not causal effects, and zero or negative predictive information is not
silently converted into equal percentages.

## Regulatory Character Inference

```r
state_grn <- InferStateSpecificGRNEdges(
  expression = state_expression,
  regulon = state_regulon_auc,
  edge.prior = scenic_adjacency,
  method = "regulon_x_target_prior"
)

edge_events <- CallRegulatoryEdgeEvents(
  state_grn,
  state.parent = c(
    Healthy = NA,
    Inflammation = "Healthy",
    Fibrosis = "Inflammation"
  )
)
```

The edge prior should be a named TF-by-target matrix derived independently of
the event labels. For publication analyses, sensitivity to edge loss, false
positive edges, weight noise, thresholds, and the local-ridge alternative
should be reported.

## Minimal Example

```r
library(CellEvoTree)

res <- CellEvoTree(
  object = scRNA,
  assay = "RNA",
  slot = "data",
  group.by = "group",
  celltype.by = "celltype",
  celltype.role = "annotation",
  root.group = "control",
  stage.order = c("control", "day3", "day7"),
  method = "hybrid",
  features = list(
    expression = TRUE,
    tf_expression = "auto",
    regulon = scenic_auc,
    lineage = list(
      pseudotime = sling_pseudotime_matrix,
      weights = sling_curve_weights,
      root = "Fibroblasts_pro"
    ),
    velocity = list(
      vector = velocity_umap,
      position = umap,
      pseudotime = velocity_pseudotime,
      root_cells = root_cells,
      end_points = end_points
    ),
    module = list(
      Inflammation = c("IL6", "CXCL8", "NFKBIA"),
      ECM = c("COL1A1", "COL3A1", "FN1")
    ),
    pseudotime = NULL
  ),
  metacell.target.cells = 50,
  metacell.stratify = TRUE,
  knn.method = "auto",
  topology.priority = "expression_first",
  primary.tree.method = "average",
  primary.weight.method = "reliability",
  primary.resamples = 30,
  secondary.optimize = TRUE,
  secondary.residualize = TRUE,
  reconstruct.ancestry = FALSE,
  calculate.view.contribution = TRUE,
  verbose = TRUE,
  output.dir = "CellEvoTree_results"
)
```

## Primary Weight Estimation

TF names are parsed from regulon rows and matched case-insensitively to the
Seurat expression matrix. These TF mRNA measurements are distinct from SCENIC
regulon AUC. Matched TF genes are removed from the ordinary expression layer
by default so the same measurement is not counted twice.

Gene and TF primary weights are not fixed at 70/30. For each resample, genes
within each layer are split, expression geometry is recomputed, and the
between-half distance error covariance is estimated. The selected
non-negative weights sum to one and minimize estimated primary-distance error
variance. Reported intervals are resampling uncertainty intervals. They are
precision weights, not causal biological percentages.

## Balanced Multi-view Compatibility

The package retains earlier balanced fusion algorithms behind an
explicit compatibility switch:

```r
# Nonlinear local-neighborhood fusion.
res_snf <- CellEvoTree(
  object = scRNA,
  group.by = "group",
  celltype.by = "celltype",
  root.group = "control",
  method = "hybrid",
  features = features,
  topology.priority = "balanced_multiview",
  fusion.method = "snf",
  distance.role.strategy = "separated"
)

# Geometry weights guided by pseudotime, Slingshot, and velocity distances.
res_mkl <- CellEvoTree(
  object = scRNA,
  group.by = "group",
  celltype.by = "celltype",
  root.group = "control",
  method = "hybrid",
  features = features,
  topology.priority = "balanced_multiview",
  fusion.method = "guided_mkl",
  distance.role.strategy = "separated"
)

comparison <- CompareFusionMethods(
  res_mkl,
  methods = c("robust_consensus", "equal", "snf", "guided_mkl")
)
comparison$comparison
comparison$recommended_method
```

The comparison score is an internal model diagnostic, not a claim of true
lineage accuracy. Candidate fusion methods are compared on rooted NJ trees so
that the diagnostic isolates the fusion layer. Rerun the selected method with
`method = "hybrid"` for the final hybrid-optimized topology. Final selection
should use sample-level resampling or an independent lineage-tracing reference.

CellRank 2 can be used without embedding Python in this package. Export its
cell-by-cell or metacell-by-metacell transition matrix and provide it as:

```r
features$transition <- cellrank_transition_matrix
```

High-probability transitions are then treated as a soft topology and direction
prior. The external matrix must have matching cell or metacell names.

## Important Outputs

- `res$metacells`: metacell membership and metadata.
- `res$metacells$dropped_strata`: group/cell-type strata below `min.cells`.
- `res$primary$weights`: Gene/TF precision weights, intervals, and split-half reliability.
- `res$evidence_audit$table`: reliability, sample-retention, batch-increment, and effective-role audit.
- `res$primary$bootstrap`: primary split support and Wilson lower bounds.
- `res$primary_sister_support`: supported, uncertain, conflicting, or untestable sister comparisons.
- `res$secondary_optimization`: accepted secondary moves and every primary guard.
- `res$distances`: all feature and direction distances, including layers excluded from primary fusion.
- `res$tree`: final expression-primary or primary-constrained cell-state tree.
- `res$ancestry`: optional model-dependent interpolation; disabled by default.
- `res$velocity_prior`: metacell-level velocity pseudotime, root score, and endpoint score.
- `res$topology_contribution`: lineage/velocity topology-constraint influence summaries.
- `res$view_contribution`: exact Shapley geometry-view contributions and subset diagnostics.
- `res$optimization`: NNI iterations, convergence flag, stop reason, and objective cost.
- `res$hybrid_selection`: NJ-versus-hybrid geometry-fidelity guard decision.
- `res$rooting`: requested root, root actually used, and rooting evidence source.
- `TraceEvolution(...)`: feature-level evolution along the tree.
- `ExplainBranch(...)`: branch split interpretation by statistical contribution.

## Interactive Current-versus-Sister Analysis

After fitting the tree, prepare only the features needed in the circular rings,
then pass the original object separately so the explorer can search a broader
variable-gene layer:

```r
tracks <- PrepareCircularTreeData(
  result = res,
  object = scRNA,
  genes = c("COL1A1", "FN1", "ACTA2"),
  modules = list(
    ECM = c("COL1A1", "COL3A1", "FN1"),
    Contractile = c("ACTA2", "TAGLN", "MYL9")
  ),
  regulons = c("STAT3", "SMAD3", "CEBPB")
)

CellEvoTreeExplorer(
  result = res,
  circular.data = tracks,
  object = scRNA,
  max.genes = 2000
)
```

The default comparison is the selected current branch versus the subtree that
shares its parent. The focal subtree is red and the sister subtree is blue.
Absolute Hedges' g is calculated for every feature. Per-layer top-feature
scores are calibrated against branch-label permutations before gene, SCENIC
regulon, and module percentages are reported. PCA/UMAP, lineage, velocity, and
other topology or direction constraints are deliberately excluded from the
Explorer contribution denominator. Selecting a dominant layer then reveals
the individual features carrying that layer's contribution. The original
child-versus-child behavior remains available in the interface.

The **Feature differences** tab displays the actual metacell values behind the
same Top features ranking. It offers:

- a heatmap across focal and sister metacells;
- a bubble plot whose color is the branch mean and whose size is the fraction
  of metacells above the pooled feature median;
- per-feature boxplots; and
- per-feature violin plots.

Use pooled feature z-scores to compare patterns across genes, regulons, and
modules with different scales, or raw values for direct within-feature
interpretation. The summary table and all four figures can be downloaded.

The selected comparison can be written back to Seurat metadata:

```r
DS_Fibro_v <- AnnotateSeuratBranch(
  object = DS_Fibro_v,
  result = res,
  node = 390,
  comparison = "sister",
  column = "CellEvo_branch_390"
)

Seurat::Idents(DS_Fibro_v) <- "CellEvo_branch_390"
markers <- Seurat::FindMarkers(
  DS_Fibro_v,
  ident.1 = "Current_branch",
  ident.2 = "Sister_branch"
)
```

The explorer exposes the same operation as a button and can download both a
cell annotation CSV and an annotated Seurat RDS. Cells omitted from the tree
are explicitly labeled `Not_in_tree`.

The **Figure export** panel downloads the current highlighted circular tree,
the layer-contribution barplot, the top-feature barplot, or a top-feature
branch-difference plot as PNG or vector PDF. Width and height may be entered
in centimeters, inches, or millimeters; PNG also accepts a DPI. The tree
export retains the selected focal and sister branches, all feature rings,
branch geometry, title, and legend choices.

The same export is available without Shiny:

```r
ExportCellEvoTreeFigure(
  result = res,
  circular.data = tracks,
  file = "node_390_tree.png",
  figure = "tree",
  width = 30,
  height = 24,
  units = "cm",
  dpi = 300,
  plot.args = list(
    branch.display = "bifurcation",
    highlight.edge = focal_edges,
    contrast.edge = sister_edges
  )
)
```

## Known-Truth Method Benchmark

The package includes a truth-aware stress benchmark. It evaluates whether
the method recovers a stored tree, root direction, branch drivers, and TF-target
network after controlled dropout, batch, velocity, and regulon errors.

Start with the main-effects screen. It covers every requested level while
changing one factor at a time, giving 17 scenarios and 340 runs at 20
replicates:

```r
design <- CreateCellEvoBenchmarkDesign(
  design = "main_effects",
  n.replicates = 20,
  seed = 202608
)

EstimateCellEvoBenchmarkResources(design)

benchmark <- RunCellEvoTreeBenchmarkSuite(
  design,
  output.dir = "benchmark_main_effects",
  workers = 4,
  resume = TRUE
)

benchmark_summary <- SummarizeCellEvoTreeBenchmark(benchmark)
PlotCellEvoTreeBenchmark(
  benchmark_summary,
  metric = "normalized_RF",
  stress.factor = "dropout"
)
```

After screening, create the full factorial design to estimate interactions.
With the default levels and 20 replicates it contains 46,080 runs, so execute it
in deterministic chunks on a workstation or cluster:

```r
full_design <- CreateCellEvoBenchmarkDesign(
  design = "full_factorial",
  n.replicates = 20
)

chunk_1 <- RunCellEvoTreeBenchmarkSuite(
  full_design,
  output.dir = "benchmark_full",
  chunk.id = 1,
  n.chunks = 20,
  workers = 4,
  resume = TRUE
)
```

Every run is checkpointed separately. Core metrics include normalized PH85
Robinson-Foulds distance, shared-split recall, cophenetic Spearman correlation,
true-neighbor retention, driver-split Jaccard, root distance, velocity direction
accuracy, driver AUPRC, regulon-edge precision/recall, adaptive batch-selection
decisions, learned weights, failures,
and runtime. The regulon-refined method is skipped above the configured
`secondary.max.metacells` limit unless that limit is explicitly raised.

The benchmark identified overcorrection from unconditional within-group
centering when sample effects were absent or weak. Real-data runs can request
permutation-gated empirical-Bayes shrinkage:

```r
res <- CellEvoTree(
  object = DS_Fibro_v,
  group.by = "group",
  sample.by = "sample",
  primary.batch.adjust = "adaptive",
  batch.adaptive.threshold = 0.05,
  batch.permutations = 199,
  features = features
)
```

The decision table is returned in `res$batch_selection` and exported as
`06pb_adaptive_batch_selection.csv`.

## Method Wording

Use this wording in manuscripts:

> CellEvoTree constructs a phylogeny-inspired cell state evolution tree rather
> than a true genetic lineage tree.

The branch contribution percentages are statistical feature-importance scores
for separating tree branches. They are not experimental causal contributions.

In expression-first mode, distance percentages are Gene/TF measurement-
precision weights. Topology penalties for UMAP, Slingshot, velocity, and
CellRank are external diagnostics unless explicitly using the compatibility
mode. Neither global weights nor branch-separation percentages are causal
effect sizes.

## Detailed Methodology

See the Chinese methodology note with mathematical and biological rationale:

`inst/docs/CellEvoTree_Methodology_CN.md`

For a full Chinese end-to-end tutorial, see:

`inst/docs/CellEvoTree_Complete_Tutorial_CN.md`

For current-branch versus sister-branch mathematics and usage, see:

`inst/docs/CellEvoTree_Sister_Branch_Analysis_CN.md`

For the expression-first hierarchy, data-driven Gene/TF weights, protected
splits, and validation rules, see:

`inst/docs/CellEvoTree_Expression_First_CN.md`

For the annotation-only cell-type policy and label-free sensitivity
validation, see:

`inst/docs/CellEvoTree_Celltype_Annotation_Policy_CN.md`

For mapping selected branches back to Seurat cells, see:

`inst/docs/CellEvoTree_Seurat_Branch_Annotation_CN.md`

For interactive PNG/PDF figure export, see:

`inst/docs/CellEvoTree_Figure_Export_CN.md`

For robust consensus, SNF, guided MKL, CellRank transition priors, and fusion
model comparison, see:

`inst/docs/CellEvoTree_Multiview_Fusion_CN.md`

For development-stage performance fixes and updated mathematical derivations, see:

`inst/docs/CellEvoTree_v0.3_Performance_and_Method_Update_CN.md`

The updated Word methods document is:

`inst/docs/CellEvoTree_v0.3_GRN_Evolution_Methods_CN.docx`

For a real-data script template, see:

`inst/examples/run_cellevotree_template.R`

For the known-truth stress benchmark and reporting criteria, see:

`inst/docs/CellEvoTree_v0.3.0_Top_Method_Benchmark_CN.md`
