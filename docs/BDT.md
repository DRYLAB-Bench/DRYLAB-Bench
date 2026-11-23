# BDT tools: EVOLVEpro and VIRAL

DRYLAB-Bench includes an optional BDT (biological design tool) layer with two
supervised protein-design tools:

- **EVOLVEpro** — PLM embeddings plus a regression head;
- **VIRAL** — PLM embeddings plus a Gaussian-process/active-learning scorer.

Both are external tools. The main `drylab_bench` environment only sends JSON
to a runner; the tools run in separate conda environments through `subprocess`.

## 1. Assets

The gated bundle contains:

```text
evidence/bdt/bdt_assets_2026-11-20.tar.gz
  data/bt_evidence/bdt_gt/          # oracle arm, GT-trained (leakage)
  data/bt_evidence/bdt_zeroshot/    # zero-shot arm, no labels
evidence/bdt/bdt_assets_2026-11-20.tar.gz.manifest.json
evidence/bdt/bdt_assets_2026-11-20.tar.gz.contents.tsv
evidence/bdt/bdt_assets_2026-11-20.tar.gz.sha256
```

Install them from the bundle root:

```bash
python scripts/bootstrap_assets.py --profile bdt \
  --install-bdt evidence/bdt/bdt_assets_2026-11-20.tar.gz --yes
```

`bdt_gt` is a **GT-trained oracle**. It is provided only to reproduce the
ceiling/leakage analysis. Never report it as a fair benchmark score.
`bdt_zeroshot` uses no labels and is the no-training PLM baseline.

See [BDT_LEAKAGE.md](BDT_LEAKAGE.md) for the full boundary.

## 2. Environment

The external repositories are cloned from their upstream projects; they are
not redistributed in this bundle.

```bash
bash scripts/setup_bdt_env.sh
```

The script:

1. clones `EvolvePro` and `VIRAL` into `.tools/bdt/`;
2. creates three conda environments under `.tools/bdt/envs/`:
   - `bdt_evolvepro` — EVOLVEpro core;
   - `bdt_plm` — PyTorch/ESM/ProtT5 embedding environment;
   - `bdt_viral` — PyTorch/GPyTorch VIRAL environment;
3. prints the absolute Python paths and the remaining model/data downloads.

Useful options:

```bash
bash scripts/setup_bdt_env.sh --dry-run
bash scripts/setup_bdt_env.sh --skip-plm
bash scripts/setup_bdt_env.sh --verify
```

### Model weights and VIRAL data

- ESM/ProtT5 weights are downloaded by the upstream PLM code on first use.
  Set `HF_ENDPOINT=https://hf-mirror.com` if needed.
- VIRAL's official datasets are on Zenodo record `15511858`; place
  `data_bloom/` and `data_desai/` under `.tools/bdt/VIRAL/`.
- Follow the upstream licenses. The VIRAL data and EVOLVEpro/VIRAL code are
  not redistributed by DRYLAB-Bench.

The public presets expect the environments at:

```text
./.tools/bdt/envs/bdt_plm/bin/python
./.tools/bdt/envs/bdt_viral/bin/python
```

If your setup uses different paths, edit the `conditions.tools.bdt.python`
block in the preset.

## 3. Presets and arms

| Preset | Artifacts | Training | Tool menu |
|---|---|---|---|
| `bdt_oracle` | `data/bt_evidence/bdt_gt` | benchmark GT (leakage) | BDT + six standard BT tools |
| `bdt_zeroshot` | `data/bt_evidence/bdt_zeroshot` | none | BDT + six standard BT tools |
| `bdt_oracle_only` | `data/bt_evidence/bdt_gt` | benchmark GT (leakage) | BDT only |
| `bdt_zeroshot_only` | `data/bt_evidence/bdt_zeroshot` | none | BDT only |

All four presets inherit the 30-scenario benchmark configuration:

```bash
drylab-bench --list-presets
drylab-bench --preset bdt_oracle --dump-config
```

Run one arm:

```bash
drylab-bench --preset bdt_oracle --provider default \
  --models <model-id> --scenarios <scenario-id> --step conditions

drylab-bench --preset bdt_zeroshot --provider default \
  --models <model-id> --scenarios <scenario-id> --step conditions
```

Each run writes to its own root:

```text
runs/bdt/oracle/
runs/bdt/zeroshot/
runs/bdt/oracle_only/
runs/bdt/zeroshot_only/
```

## 4. Leakage and component-isolation rules

### Oracle arm (R1a)

The oracle artifacts were trained on the benchmark's DMS ground truth. They
are a **leakage ceiling**, not a fair measurement of tool quality. Every
evidence card and `meta_summary` carries the oracle/leakage flag. Keep oracle
results in a separate table and never merge them with the main benchmark.

### Zero-shot arm

The zero-shot arm performs masked-marginal PLM scoring:

```text
log p(mutant) − log p(wild type)
```

It uses no labels. It is related to the existing `sequence_plm` tool
(ESM-1v), but uses a different PLM (default ESM-2 650M); do not present it as
an independent evidence source.

### Component isolation (R3)

The evaluator uses ESM-2; the standard `sequence_plm` tool uses ESM-1v. For
non-oracle BDT artifacts, prefer embeddings from ESM-1b or ProtT5 and record
the embedding model in the manifest.

### Manifest guards

The harness and both runners refuse artifacts unless the manifest declares a
valid label source:

```json
{
  "labels_confirmed_non_gt": true,
  "label_source": "orthogonal assay, not the benchmark GT",
  "artifact": "model.pkl",
  "embedding_model": "facebook/esm2_t33_650M_UR50D",
  "tool_version": "evolvepro"
}
```

Oracle manifests instead declare:

```json
{
  "labels_confirmed_non_gt": false,
  "oracle_gt_training": true,
  "label_source": "BENCHMARK GT DMS ... (ORACLE — leakage arm)"
}
```

The oracle is accepted only when the config sets
`conditions.tools.bdt.allow_gt_training: true`.

## 5. Verification

Offline wiring and leakage guard:

```bash
python tests/test_bdt_tools.py
```

Asset and environment check:

```bash
python scripts/verify_bdt_artifacts.py --preset bdt_oracle
python scripts/verify_bdt_artifacts.py --preset bdt_zeroshot
```

The check covers the external Python paths, runner files, manifest fields, and
the leakage guard. It does not run a model API call.

A real smoke run can use a single scenario and model:

```bash
drylab-bench --preset bdt_oracle --provider default \
  --models <model-id> --scenarios <scenario-id> --step conditions
```

Inspect `units/*/final.json` and `trajectory.jsonl` for `tool: evolvepro` /
`tool: viral` evidence and `meta_summary.bdt`.

## 6. Training your own artifacts

### Zero-shot

```bash
python scripts/make_bdt_zeroshot_artifacts.py \
  --config configs/generations/config_benchmark.yaml \
  --out-dir data/bt_evidence/bdt_zeroshot
```

This writes only manifests; the runner performs the PLM scoring at inference
time.

### Oracle / custom supervised artifacts

```bash
python scripts/bdt_export_gt_training_set.py \
  --config configs/generations/config_benchmark.yaml \
  --out-dir data/bt_evidence/bdt_gt

# In the external environments:
./.tools/bdt/envs/bdt_plm/bin/python scripts/bdt_runners/train_evolvepro_gt.py \
  --gt-dir data/bt_evidence/bdt_gt --artifact-dir data/bt_evidence/bdt_gt
./.tools/bdt/envs/bdt_viral/bin/python scripts/bdt_runners/train_viral_gt.py \
  --gt-dir data/bt_evidence/bdt_gt --artifact-dir data/bt_evidence/bdt_gt
```

The export step uses benchmark GT and is therefore oracle-only. For a fair
tool, train on an orthogonal assay and set `labels_confirmed_non_gt: true`
with an accurate `label_source`.

### VIRAL scoring hook

A VIRAL artifact can provide a `viral_scorer.py` and declare:

```json
{"scoring_hook": "viral_scorer:score"}
```

The hook receives `(mutations, sequence, artifact_dir)` and returns a mapping
from mutation to score.

## 7. Troubleshooting

- **`python for evolvepro not configured`**: edit the preset's
  `conditions.tools.bdt.python` paths.
- **`manifest missing`**: install the BDT assets or point
  `conditions.tools.bdt.artifact_dir` at the correct directory.
- **`label_source looks GT-derived`**: the zero-shot/oracle boundary is being
  violated. Use the correct arm or fix the manifest.
- **`ModuleNotFoundError: torch/esm`**: the external conda environment is
  incomplete; rerun `setup_bdt_env.sh` and inspect its output.
- **VIRAL data missing**: download Zenodo record `15511858` under the upstream
  VIRAL directory. This is optional unless you are training a VIRAL artifact.

## 8. Licenses

- EVOLVEpro and VIRAL code/data: follow the upstream repositories and their
  licenses.
- ESM/ProtT5 weights: follow the model terms.
- The BDT assets in this bundle are gated by the DRYLAB-Bench Responsible Use
  Agreement; the oracle artifacts are especially sensitive because they were
  trained on the benchmark ground truth.
