# Setup

This guide starts after your access request has been approved. It installs the
gated bundle and prepares the data and model assets needed by the benchmark.

## 1. Download the gated bundle

```bash
pip install -U huggingface_hub
hf auth login
hf download drylab-bench/benchmark \
  --repo-type dataset --local-dir drylab-bench
cd drylab-bench
```

The bundle contains a complete source tree, a built wheel, documentation, and
the evidence manifest.

## 2. Python environment

```bash
conda env create -f environment.yml
conda activate drylab-bench
python -m pip install -e .
```

Alternatively, install the wheel from `dist/`:

```bash
python -m pip install dist/drylab_bench-1.0.0-py3-none-any.whl
```

Run commands from the bundle root so the CLI can find `configs/`.

Requirements: Python 3.10+, at least 16 GB RAM for the full tool stack, and
enough disk for ProteinGym, ViroGym, ESM weights, and the evidence bundle.

## 3. API configuration

```bash
cp .env.example .env
# edit .env
source .env
```

The default provider is an OpenAI-compatible endpoint:

```bash
export OPENAI_BASE_URL=https://your-endpoint.example/v1
export OPENAI_API_KEY=...
```

Provider profiles live in `configs/providers/`. API keys are referenced from
the environment; never put literal keys in YAML.

## 4. Data and assets

### ProteinGym (required)

```bash
python -m drylab_bench.download_datasets --data-dir ./data
```

### ViroGym (required)

ViroGym's upstream repository does not carry an explicit redistribution
license. Obtain it from the upstream project and place it under
`data/ViroGym/` according to its terms.

### Biological-tool evidence

The gated bundle provides the evidence manifest. Install the canonical bundle
with:

```bash
python scripts/bootstrap_assets.py --profile runtime \
  --install-evidence /path/to/bt_evidence_bundle.tar.gz --yes
```

The bundle includes Swiss-Prot/MMseqs, GFF annotations, ProteinMPNN structure
scores, phenotype tables, and `wt_sequences.json`. FoldX caches, raw PDBs, and
GT-training artifacts are not included; see `THIRD_PARTY_NOTICES.md`.

### ESM weights and in-silico assets

```bash
python scripts/download_insilico_models.py
```

This fetches the ESM checkpoints and EVE TP53 predictions from their upstream
sources. Review the upstream terms before use.

### MMseqs2

```bash
python scripts/install_mmseqs.py
# or place a static build under .tools/mmseqs/
bash scripts/prepare_mmseqs_db.sh
export DRYLAB_MMSEQS_DB=<destination>/sprotDB
```

### ToxinPred3 (optional, GPL-3.0)

The derived tree file is not redistributed. If you need the toxin tool,
obtain ToxinPred3 under its GPL terms and export the trees locally:

```bash
bash scripts/install_toxinpred3.sh
```

### FoldX (optional, commercial/academic license)

FoldX is not redistributed. Users with a license can install it under
`.tools/foldx/` (macOS) or `.tools/foldx_linux/` (Linux). The `phenotype` tool
falls back to precomputed tables when FoldX is absent.

## 5. Preflight

```bash
python scripts/preflight.py --preset benchmark --provider default \
  --models <model-id> --strict
```

`FAIL` blocks a run; `WARN` is reported but does not block unless `--strict`
is used. The report is written under `runs/benchmark/_preflight/`.

To check only the provider configuration (no large data):

```bash
python scripts/preflight.py --preset benchmark --provider default \
  --models <model-id> --skip-core
```

## 6. Smoke run

```bash
drylab-bench --preset benchmark --provider default \
  --models <model-id> --scenarios <scenario-id> --step conditions
```

Each run writes a timestamped directory under `runs/` with:

```text
run_manifest.json
resolved_config.yaml          # secrets redacted
conditions_results.json
outputs.jsonl
units/
experiment.log
```

## 7. Analyze

```bash
drylab-bench --preset benchmark --step analyze \
  --output-dir runs/run_<timestamp>

python scripts/report_brs.py \
  --run-dir runs/run_<timestamp> \
  --config configs/generations/config_benchmark.yaml
```

## 8. Recompute profile

To rebuild evidence from public upstream sources instead of using the
precomputed bundle, follow [RECOMPUTE.md](RECOMPUTE.md).

## Troubleshooting

- **`HTTP 429`**: provider concurrency or quota limit; reduce
  `concurrency.max_workers` or probe the endpoint.
- **`HTTP 401`**: invalid key; check `.env`.
- **`HTTP 403`**: upstream quota or policy block; it is recorded as a refusal
  or provider block, not a model failure.
- **MMseqs index errors on a shared filesystem**: copy the database to local
  disk and set `DRYLAB_MMSEQS_DB`.
- **Missing FoldX**: the phenotype tool degrades to precomputed tables; this
  is expected.
- **No `EsmForMaskedLM`**: use a compatible `transformers` version; the
  sequence-PLM tool reports degraded status in preflight.
