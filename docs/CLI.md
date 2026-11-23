# CLI reference

The entry point is `drylab-bench` (or `python -m drylab_bench`). The root
`run_experiment.py` file is a compatibility shim.

## Inspect configuration

```bash
drylab-bench --list-presets
drylab-bench --preset benchmark --dump-config
drylab-bench --preset benchmark --provider default \
  --models <model-id> --dump-config
```

`--dump-config` prints the resolved preset, catalog, and provider layers;
API keys are redacted.

## Preflight and provider probe

```bash
python scripts/preflight.py --preset benchmark --provider default \
  --models <model-id> --strict

python scripts/probe_provider.py --preset benchmark --provider default \
  --models <model-id> --dry-run

python scripts/probe_provider.py --preset benchmark --provider default \
  --models <model-id> --allow-api --concurrency-levels 1,2,4
```

`probe_provider.py` is offline unless `--allow-api` is passed.

## Run the benchmark

```bash
# One scenario
drylab-bench --preset benchmark --provider default \
  --models <model-id> --scenarios <scenario-id> --step conditions

# The full model × scenario matrix
drylab-bench --preset benchmark --provider default \
  --models <model-id> --step conditions

# Multiple models or scenarios (comma-separated)
drylab-bench --preset benchmark --provider default \
  --models model-a,model-b --scenarios scenario-a,scenario-b \
  --step conditions
```

`--step conditions` runs the two parent conditions `S0-iter` and `S2`.
`S0` and `S1` are derived snapshots from the same `S2` trajectory. Reporting
always covers all four conditions.

## BDT reproduction presets

The gated bundle adds optional presets:

```bash
drylab-bench --preset bdt_oracle --provider default \
  --models <model-id> --step conditions
drylab-bench --preset bdt_zeroshot --provider default \
  --models <model-id> --step conditions
drylab-bench --preset bdt_oracle_only --provider default \
  --models <model-id> --step conditions
drylab-bench --preset bdt_zeroshot_only --provider default \
  --models <model-id> --step conditions
```

The oracle presets use GT-trained BDT artifacts and must be analyzed as a
leakage ceiling. See [BDT.md](BDT.md) and [BDT_LEAKAGE.md](BDT_LEAKAGE.md).

## Resume and merge

```bash
# Reuse completed units and fill only what is missing
drylab-bench --preset benchmark --provider default --models <model-id> \
  --step conditions --resume-from runs/run_<timestamp>

# Plan only, no API calls
drylab-bench --preset benchmark --models <model-id> \
  --step conditions --resume-from runs/run_<timestamp> --resume-dry-run

# Offline row-level merge of several runs
drylab-bench --preset benchmark --step merge \
  --resume-from runs/run_A,runs/run_B --output-dir runs
```

`--resume-mode valid|measured|all` controls whether refusals and partial
outputs are reused. `measured` (the default) reuses refusals; `api_error`
units are never reused.

## Analyze

```bash
drylab-bench --preset benchmark --step analyze \
  --output-dir runs/run_<timestamp>

python scripts/report_brs.py \
  --run-dir runs/run_<timestamp> \
  --config configs/generations/config_benchmark.yaml
```

## Output layout

```text
runs/
  run_YYYYMMDD_HHMMSS/
    run_manifest.json
    resolved_config.yaml
    conditions_results.json
    gains_summary.{json,csv}
    coverage_report.{json,csv}
    brs_report.{json,csv,md}
    units/<model>__<scenario>__P<variant>__<condition>/
      manifest.json
      trajectory.jsonl
      final.json
      evidence.jsonl
      snapshots/
      prompts/
```
