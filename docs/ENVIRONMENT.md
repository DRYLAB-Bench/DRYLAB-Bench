# Environment and resource checks

The runtime evidence bundle and the recompute profile are described in
[SETUP.md](SETUP.md) and [RECOMPUTE.md](RECOMPUTE.md).

## Configuration entry points

```bash
drylab-bench --list-presets
drylab-bench --preset benchmark --dump-config
```

- `configs/presets/benchmark.yaml` — the benchmark preset;
- `configs/models/catalog.yaml` — logical model IDs, native names, request
  parameters;
- `configs/providers/*.yaml` — endpoint profiles;
- `configs/assets/core.yaml` — required data and model assets.

Add a model in the catalog; add a provider profile for a new endpoint. Every
run writes `run_<ts>/resolved_config.yaml` with API keys redacted.

## Preflight

```bash
# Full check
python scripts/preflight.py --preset benchmark --provider default \
  --models <model-id> --strict

# Provider configuration only (no large data)
python scripts/preflight.py --preset benchmark --provider default \
  --models <model-id> --skip-core

# Let a run preflight automatically
drylab-bench --preset benchmark --provider default \
  --models <model-id> --step conditions
```

Checks include:

- provider `base_url` / `api_key` environment variables;
- optional endpoint probes (`--probe-api`, `--probe-chat`,
  `--probe-concurrency 1,2,4`);
- ProteinGym, ViroGym, biological-tool evidence, MMseqs/Swiss-Prot, ESM-1v;
- `torch` and `transformers.EsmForMaskedLM` importability;
- a tiny MMseqs `easy-search` for the runtime MSA tool;
- ESM-2 cache, EVE TP53, ToxinPred3, and FoldX availability;
- per-scenario structure/phenotype/GFF coverage;
- in-silico calibration cache presence (it is rebuilt when missing).

Reports are written to `runs/benchmark/_preflight/preflight_report.{json,txt}`.
`FAIL` blocks a run. `WARN` does not block unless `--strict` is used.

## Provider probe

Use the probe before running a full matrix on a new endpoint or model. It
resolves the same preset/catalog/provider layers as a real run.

```bash
# Offline: show the resolved request shape
python scripts/probe_provider.py --preset benchmark --provider default \
  --models <model-id> --dry-run

# One minimal chat + GET /models
python scripts/probe_provider.py --preset benchmark --provider default \
  --models <model-id> --allow-api --concurrency-levels none

# Concurrency and timeout sweep
python scripts/probe_provider.py --preset benchmark --provider default \
  --models <model-id> --allow-api --concurrency-levels 1,2,4 \
  --requests-per-level 3 --timeout-levels 30,60
```

Reports are written to `runs/benchmark/_probe/provider_probe.{json,md}` and
include chat status, HTTP error classes, P50/P95 latency, per-level
success/429/timeout counts, and recommended concurrency/timeout. API keys and
authorization headers are never written.

The probe can also be attached to preflight with `--probe-api`,
`--probe-chat`, and `--probe-concurrency`.

## BDT assets

The optional EVOLVEpro / VIRAL assets are checked separately from the core
preflight:

```bash
python scripts/verify_bdt_artifacts.py --preset bdt_oracle
python scripts/verify_bdt_artifacts.py --preset bdt_zeroshot
```

The check covers the external Python paths, runner files, manifest fields, and
the leakage guard. The BDT environments are created by
`bash scripts/setup_bdt_env.sh`. See [BDT.md](BDT.md).

## Phenotype backends

The `phenotype` tool selects a backend per scenario:

- **AlphaMissense** — precomputed pathogenicity tables for human proteins;
- **EVEscape** — precomputed escape tables;
- **FoldX-live** — on-demand physics-based ΔΔG for scenarios with a PDB and a
  local FoldX installation.

Preflight reports `bt.phenotype_alphamissense`, `bt.phenotype_backends`,
`bt.phenotype_effective`, and `bt.phenotype_no_backend`. A missing precomputed
table does not mean FoldX is unavailable. Do not force AlphaMissense onto
non-human proteins merely to silence a warning; coverage limitations are
reported explicitly and the corresponding tool degrades with a label.

If FoldX reports a missing `molecules/` directory:

```bash
cp -a .tools/foldx_linux/molecules data/bt_evidence/foldx_ondemand/
# macOS installs may use .tools/foldx/molecules
```

## Not covered by preflight

- full SHA256 verification of large model weights (size/loadability only);
- long stress tests (the probe is a small-sample estimate);
- mutation of input data (`data/` is read-only to the benchmark).
