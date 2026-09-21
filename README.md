# Multi-instance 3D-BPP revision experiments

This repository implements the approved reproducibility plan for the revised
multi-instance palletization manuscript. It connects the supplied January 2024 FLAP
optimizer to an event-driven warehouse experiment, robust runtime prediction,
resource-conflict coordination, Quad2Quad validation, exact MILP references, paired
statistics, and automated manuscript outputs.

## What is genuinely empirical

Mode B is enforced. Every row in packing_candidates is produced by the supplied
homogeneous layer builder, block builder, physical checks, and residual NSGA-II.
When a residual Pareto archive exists, candidates are drawn from its feasible
hall-of-fame solutions. Repeated controlled seeds provide additional genuine
optimizer outcomes. If the optimizer fails, the pipeline stops and records the error;
no packing quality is invented.

The warehouse CSV files do not include event timestamps, AMR logs, buffer histories,
or retrieval durations. Those inputs are explicitly labeled scenario assumptions and
live only in the YAML configuration. See docs/evidence_and_assumptions.md.

## Two main scripts

1. scripts/run_all_simulations.py prepares empirical data, fits out-of-fold runtime
   models, builds genuine packing candidates, runs all dynamic simulations, runs
   prediction-bias experiments, and runs selector/scalability benchmarks. It writes
   raw results only.
2. scripts/produce_figures_and_tables.py computes paired statistics and packing
   equivalence, then writes all figures and tables.

The user-editable variables are at the top of both scripts. In the first script:

- RESULTS_DIR controls where raw results are saved.
- NUM_ORDERS controls the number of real orders.
- NUM_STATIONS controls the number of stations.
- SCALABILITY_STATIONS controls the scalability sweep.

In the second script:

- RESULTS_DIR points to the raw run.
- FIGURES_AND_TABLES_DIR controls where manuscript outputs are saved.

## Step-by-step installation

### Windows PowerShell

1. Extract this project and open PowerShell in the extracted folder.
2. Create an isolated environment:

       py -3.11 -m venv .venv

3. Activate it:

       .\.venv\Scripts\Activate.ps1

4. Install the project:

       python -m pip install --upgrade pip
       python -m pip install -e ".[parquet,dev]"

5. Run the tests:

       python -m pytest

### Linux, WSL, or macOS

1. Open a terminal in the extracted project folder.
2. Create and activate an environment:

       python3.11 -m venv .venv
       source .venv/bin/activate

3. Install and test:

       python -m pip install --upgrade pip
       python -m pip install -e ".[parquet,dev]"
       python -m pytest

## First: run the quick verification

The quick configuration optimizes a four-order candidate pool and samples two orders
per episode, with two stations, two independent order sets, three layer-ordering
candidates per order, reduced GA parameters, and a compact scalability grid:

    python scripts/09_reproduce_all.py --config configs/quick.yaml

Inspect:

- outputs/quick_validation/audit/data_quality_report.md
- outputs/quick_validation/raw/
- outputs/quick_validation/figures/
- outputs/quick_validation/tables/

CSV companions are always written. Parquet files are also written when pyarrow is
installed.

## Run the requested 100-order, 6-station campaign

1. Open scripts/run_all_simulations.py.
2. Confirm or change:

       RESULTS_DIR = PROJECT_ROOT / "outputs" / "paper_2026_revision"
       NUM_ORDERS = 50
       NUM_STATIONS = 8
       ORDER_POOL_SIZE_PER_OVERLAP = 60
       ORDER_SET_REPLICATES = 10
       SCALABILITY_STATIONS = [2, 4, 6, 8, 10]

3. Review configs/paper.yaml. This is the authoritative location for AMRs, buffers,
   timing assumptions, random seeds, GA hyperparameters, replications, equivalence
   margins, prediction biases, and candidate counts.
4. Run:

       python scripts/run_all_simulations.py

The full Mode-B campaign executes the original packing optimizer repeatedly and may
take days. `NUM_ORDERS` is the number sampled into each simulated episode.
`ORDER_POOL_SIZE_PER_OVERLAP` controls how many orders are optimized so multiple
independent order sets can be sampled. With 60 pool orders, `[low, high]`, and three
candidates per order, the maximum base campaign is 120 unique orders and 360
optimizer attempts (overlap duplicates reduce this). Candidate files are
deterministic for the configured seed.

The candidates use `fill_first`, `weight_first`, and `group_first` full-layer
sorting. Their packing fingerprints, objective vectors, resource sets, and resource
sequences are audited after every order. Duplicate packing fingerprints remain in
the provenance file but are excluded from the selector's alternative set.

Each optimizer attempt runs in an isolated subprocess. The paper configuration sets
`max_optimizer_runtime_minutes: 30`; a process exceeding that limit is terminated,
the entire order is discarded, and the next replacement order is attempted. Ordinary
optimizer exceptions are handled the same way and recorded rather than terminating
the campaign. Ten replacement orders per overlap level are available. The overall
candidate-generation budget is 6,000 minutes, leaving 1,200 minutes of a 7,200-minute
allocation for dynamic simulations and scalability. Results, attempts, progress,
and diversity diagnostics are atomically checkpointed after every order; a matching
interrupted run resumes without recomputing completed orders.

Key audit outputs are:

- `audit/candidate_diversity.csv`
- `raw/selected_decisions.csv`
- `derived/statistics/selector_disagreement.csv`
- `derived/statistics/packing_pair_audit.csv`
- `logs/candidate_checkpoint.json`

For a first timing estimate, set `NUM_ORDERS` to 10 and replications to 2.

## Create figures and tables

1. Open scripts/produce_figures_and_tables.py.
2. Set RESULTS_DIR to the completed raw run.
3. Set FIGURES_AND_TABLES_DIR to the desired output directory.
4. Run:

       python scripts/produce_figures_and_tables.py

Each figure is saved as editable SVG, manuscript-ready PDF, and high-resolution PNG.
The exact source data for each figure is saved beside the plots. Each table is saved
as CSV, Markdown, and LaTeX.

Reviewer 1 Comment 3 is addressed with:

    Figure_D_quad2quad_conflict_2station.pdf

The figure is generated from a real two-station decision record, not hand-drawn data.

## Run scalability separately

The main script now calls the raw simulations and scalability analysis sequentially:

    if __name__ == "__main__":
        run_all_simulations()
        run_scalability_analysis()

The scalability stage reuses the completed packing candidate pool and therefore does
not rerun the original packing optimizer. To run scalability independently from Python:

    from scripts.run_all_simulations import run_scalability_analysis
    run_scalability_analysis(station_counts=[2, 4, 6, 8, 10])

Or edit SCALABILITY_STATIONS and call the function from your IDE. The dedicated
outputs are Figure H and Table I.

## Run stages individually

All thin stage scripts accept --config and an optional --results-dir:

    python scripts/00_validate_data.py --config configs/paper.yaml
    python scripts/01_prepare_features.py --config configs/paper.yaml
    python scripts/02_fit_runtime_models.py --config configs/paper.yaml
    python scripts/03_build_candidate_pool.py --config configs/paper.yaml
    python scripts/04_run_simulations.py --config configs/paper.yaml
    python scripts/05_run_selector_benchmarks.py --config configs/paper.yaml
    python scripts/06_compute_statistics.py --config configs/paper.yaml
    python scripts/07_make_figures.py --config configs/paper.yaml
    python scripts/08_make_tables.py --config configs/paper.yaml

For one-command reproduction:

    python scripts/09_reproduce_all.py --config configs/paper.yaml

## Output structure

    outputs/<run_name>/
      manifest.json
      resolved_config.yaml
      audit/
      raw/
        empirical_orders
        prediction_oof
        packing_candidates
        episodes
        orders
        decisions
        events
        conflicts
        prediction_robustness
        selector_benchmarks
      derived/
        statistics/
        figure_data/
      figures/
      tables/
      logs/

The manifest records hashes, environment versions, seed policy, hardware, and the
empirical/scenario distinction.

## Scientific cautions

- The supplied result file contains 995 matched orders; Dataset1000 contains one
  additional order without a legacy result. The audit preserves this discrepancy.
- Optimization-time prediction uses only pre-optimization information. Residual
  boxes, GA runs, blocks, compactness, and final utilization are excluded to prevent
  leakage.
- Ordinary MAPE is not used because measured runtime includes zeros.
- Quad2Quad is tested as a preference mechanism. Resource feasibility is imposed by
  explicit conflict constraints, and exact MILP solutions provide the reference.
- Equivalence margins in paper.yaml are research decisions. Confirm them with the
  industrial partner before interpreting Table H.
- The simulator continuously serializes exclusive product-pallet access and records
  zero committed asset conflicts. Raw proposal conflicts remain visible in the
  conflict log.

## Original optimizer provenance

The adapted source is in src/mibpp/legacy_optimizer. Compatibility changes are
documented in its ORIGIN.md. The layer construction, block construction, constraint
checks, residual evaluation, crossover, mutation, and survivor logic remain connected
to the supplied project.
