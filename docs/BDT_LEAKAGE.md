# BDT leakage and arm boundaries

The BDT assets contain two arms with different leakage properties.

## `bdt_gt` — oracle arm

- Trained on the benchmark's DMS ground truth.
- Manifests set `labels_confirmed_non_gt: false` and
  `oracle_gt_training: true`.
- The harness accepts these artifacts only when the preset/config sets
  `conditions.tools.bdt.allow_gt_training: true`.
- Every evidence card and `meta_summary` carries the oracle/leakage flag.
- **Use only for a ceiling/sensitivity analysis.** Never merge oracle scores
  with the fair benchmark, and never describe the oracle as a tool-quality
  measurement.

## `bdt_zeroshot` — no-training arm

- Uses no labels; the runner performs masked-marginal PLM scoring.
- Manifests set `labels_confirmed_non_gt: true` and `zero_shot: true`.
- It is related to the standard `sequence_plm` tool (ESM-1v) but uses a
  different PLM (default ESM-2 650M). Do not treat it as an independent
  evidence source.

## Rules

1. Keep oracle and zero-shot outputs in separate run roots and tables.
2. Do not train a "fair" BDT artifact on the benchmark GT. Use an orthogonal
   assay and record the true `label_source`; the runner rejects GT-derived
   labels.
3. For non-oracle artifacts, prefer embeddings from ESM-1b or ProtT5 rather
   than the evaluator's ESM-2 (component isolation).
4. Never publish per-scenario oracle scores as if they were benchmark results.

## Manifest examples

Fair / zero-shot:

```json
{
  "labels_confirmed_non_gt": true,
  "label_source": "orthogonal assay, not the benchmark GT",
  "zero_shot": true
}
```

Oracle:

```json
{
  "labels_confirmed_non_gt": false,
  "oracle_gt_training": true,
  "label_source": "BENCHMARK GT DMS ... (ORACLE — leakage arm)"
}
```

The harness and runners enforce these fields; do not remove or rewrite them.
