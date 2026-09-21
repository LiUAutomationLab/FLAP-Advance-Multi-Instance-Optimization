# Candidate-diversity and validation revision

## Implemented

- Exactly three candidate executions per order: `fill_first`, `weight_first`, and
  `group_first` full-layer sorting.
- The legacy fill-first priority is preserved exactly; the full-layer loop no
  longer retests the current block.
- Hard per-attempt subprocess timeout, logged skip/continue behavior, atomic
  per-order checkpoints, configuration-safe resume, and global progress tracking.
- Candidate diversity audit covering executions, packing fingerprints, objective
  vectors, resource-requirement sets, resource sequences, and distinct alternatives.
- Duplicate packing fingerprints are retained for provenance but excluded from
  selector alternatives.
- Candidate-specific block-handling time and resource-use windows so a different
  selection can propagate through reservations, retrievals, timing, and episode
  results.
- Selected-decision table and Q2Q disagreement summary against GREEDY, GC-P90, and
  ORACLE.
- Controlled scalability instances with distinct low/medium/high conflict density,
  meaningful exact-objective gaps, explicit deferral semantics, and runtime
  quantiles.
- Multiple independently sampled order sets; paired inference uses the order set,
  rather than timing repetitions, as its experimental unit.
- Packing-pair audit and separate reporting of identical outcomes versus statistical
  equivalence.
- Robust P90 methods now consume the cross-fitted conformal upper bound.
- Expanded Tables D-I and a readable faceted selector-scalability figure.

## Verification

- 15 unit and integration tests pass.
- The fresh quick reproduction completed 12/12 optimizer attempts and resumed
  without recomputation.
- The four quick-run orders each produced one unique packing fingerprint and three
  resource sequences. This is reported transparently; no duplicate is counted as a
  distinct packing alternative.
- Controlled mean conflict counts were ordered low (3.0), medium (19.375), and high
  (33.875).
- Maximum objective gaps relative to exact were nonzero for ASF, greedy, Q2Q, and
  weighted selectors, while exact remained zero.
