# Reproducing the benchmark

This guide starts from the gated bundle and reproduces:

1. the 30-scenario benchmark (four conditions);
2. the optional EVOLVEpro / VIRAL BDT arms;
3. the analysis and BRS reports.

Evaluation results are not included. Reproduction means running the same
protocol with your own model endpoints and comparing your runs.

## 1. Install the bundle

```bash
hf auth login
hf download drylab-bench/benchmark \
  --repo-type dataset --local-dir drylab-bench
cd drylab-bench

conda env create -f environment.yml
conda activate drylab-bench
python -m pip install -e .
cp .env.example .env
source .env
```

The bundle root contains `evidence/`, `configs/`, `scripts/`, `tests/`, and a
built wheel under `dist/`. Run commands from the bundle root so the CLI finds
`configs/`.

## 2. Install the data packs

### Runtime biological-tool evidence

```bash
python scripts/bootstrap_assets.py --profile runtime \
  --install-evidence evidence/bt_evidence/bt_evidence_2026-11-20.tar.gz --yes
```

### Recompute sources (for custom scenarios)

```bash
python scripts/bootstrap_assets.py --profile recompute \
  --recompute-bundle evidence/recompute_sources/recompute_sources_2026-11-20.tar.gz --yes
```

### BDT assets (oracle + zero-shot)

```bash
python scripts/bootstrap_assets.py --profile bdt \
  --install-bdt evidence/bdt/bdt_assets_2026-11-20.tar.gz --yes
```

The BDT archive installs:

```text
data/bt_evidence/bdt_gt/          # oracle, GT-trained (leakage)
data/bt_evidence/bdt_zeroshot/    # zero-shot, no labels
```

Read [BDT_LEAKAGE.md](BDT_LEAKAGE.md) before using the oracle.

### External data and weights

Follow [SETUP.md](SETUP.md) for ProteinGym, ViroGym, ESM/EVE weights, MMseqs,
ToxinPred3, and FoldX. ViroGym and FoldX have their own upstream terms.

## 3. Verify the installation

```bash
drylab-bench --list-presets
python scripts/preflight.py --preset benchmark --provider default \
  --models <model-id> --strict

python tests/test_benchmark_config.py
python tests/test_bdt_tools.py
python scripts/verify_bdt_artifacts.py --preset bdt_oracle
python scripts/verify_bdt_artifacts.py --preset bdt_zeroshot
```

`test_bdt_tools.py` is offline: it uses fake runners and artifacts and does not
require the external BDT environments.

## 4. Main benchmark

The default preset is `benchmark`: 30 scenarios, 3 prompt variants, and the
two parent conditions `S0-iter` and `S2`. The S2 trajectory derives `S0` and
`S1`, so a full run scores four conditions.

The full model matrix is 15 models × 30 scenarios × 3 variants × 2 parent
conditions = 2700 parent units (5400 scored rows across four conditions).
Start with one model and one scenario:

```bash
drylab-bench --preset benchmark --provider default \
  --models <model-id> --scenarios <scenario-id> --step conditions
```

Run a full model × scenario matrix:

```bash
drylab-bench --preset benchmark --provider default \
  --models <model-id> --step conditions
```

Use `--models` and `--scenarios` to select a subset. Model definitions live in
`configs/models/catalog.yaml`; provider profiles live in `configs/providers/`.

## 5. BDT arms

Set up the external environments once:

```bash
bash scripts/setup_bdt_env.sh
```

The presets use `./.tools/bdt/envs/bdt_plm/bin/python` and
`./.tools/bdt/envs/bdt_viral/bin/python`. If your setup differs, edit the
`conditions.tools.bdt.python` block.

```bash
# Oracle: GT-trained; ceiling/leakage analysis only
drylab-bench --preset bdt_oracle --provider default \
  --models <model-id> --step conditions

# Zero-shot: no labels
drylab-bench --preset bdt_zeroshot --provider default \
  --models <model-id> --step conditions

# BDT-only variants (standard BT tools disabled)
drylab-bench --preset bdt_oracle_only --provider default \
  --models <model-id> --step conditions
drylab-bench --preset bdt_zeroshot_only --provider default \
  --models <model-id> --step conditions
```

Output roots:

```text
runs/bdt/oracle/
runs/bdt/zeroshot/
runs/bdt/oracle_only/
runs/bdt/zeroshot_only/
```

The BDT presets inherit all 30 scenarios and the 15-model catalog. The
archived BDT study used a smaller effort-ladder subset; to mirror it, add the
model IDs to `configs/models/catalog.yaml` with your provider's native names
and select them with `--models`. For example:

```yaml
kimi-k3-low:
  default_provider: default
  native_names:
    default: kimi-k3
  params:
    temperature_override: 1.0
    reasoning_effort: low

deepseek-v4-pro-low:
  default_provider: default
  native_names:
    default: deepseek-v4-pro-0813
  params:
    temperature_override: 0.3
    reasoning_effort: low
```

Then:

```bash
drylab-bench --preset bdt_oracle --provider default \
  --models kimi-k3-low,deepseek-v4-pro-low --step conditions
```

The oracle arm is not a fair benchmark result. Keep its tables separate and
label every comparison as a leakage ceiling.

## 6. Analyze

```bash
drylab-bench --preset benchmark --step analyze \
  --output-dir runs/run_<timestamp>

python scripts/report_brs.py \
  --run-dir runs/run_<timestamp> \
  --config configs/generations/config_benchmark.yaml
```

For BDT runs, use the same analysis command with the BDT run directory and the
benchmark config (the scenario set is identical):

```bash
python scripts/report_brs.py \
  --run-dir runs/bdt/zeroshot/run_<timestamp> \
  --config configs/generations/config_benchmark.yaml
```

Keep oracle and zero-shot reports separate.

## 7. Reproducibility notes

- The protocol is reproducible; exact model outputs depend on the provider,
  model version, temperature/reasoning settings, and upstream safety filters.
- Refusals, provider blocks, and API errors are recorded separately. See
  [SCORING.md](SCORING.md).
- Calibration and bootstrap intervals use fixed seeds from the config.
- The full tool stack (ESM-1v 5 seeds, ESM-2, MMseqs, FoldX) is memory-heavy.
  Use a machine with sufficient RAM; on memory-constrained machines reduce
  ESM-1v seeds for wiring tests only, not for reported production runs.
- `--resume-from` reuses completed units and is safe for long runs.

## 8. Extend the benchmark

See [EXTENDING.md](EXTENDING.md) for adding a new DMS scenario and generating
its biological-tool evidence from the recompute sources.
