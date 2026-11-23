# Recompute profile

The recompute profile rebuilds biological-tool evidence from public upstream
sources instead of using the precomputed runtime bundle. Use it when you need
to change the evidence, audit provenance, or extend the scenario set.

## 1. Runtime vs recompute

| profile | inputs | outputs |
|---|---|---|
| `runtime` | precomputed evidence bundle | `data/bt_evidence/` ready for a run |
| `recompute` | PDB, UniProt GFF/FASTA, EVEscape tables, optional AlphaMissense | regenerated evidence tables |

Neither profile changes the DMS ground truth. ViroGym must be obtained from
its upstream project under its own terms.

## 2. One-command flow (with a source bundle)

```bash
# 1. Raw DMS data
python -m drylab_bench.download_datasets --data-dir ./data
# ViroGym: obtain from upstream and place under data/ViroGym/

# 2. Install the recompute source bundle
python scripts/bootstrap_assets.py --profile recompute \
  --recompute-bundle evidence/recompute_sources/recompute_sources_2026-11-20.tar.gz --yes

# 3. Tools: MMseqs2 + ProteinMPNN
python scripts/install_mmseqs.py
# ProteinMPNN: clone under .tools/ProteinMPNN per its MIT license

# 4. Swiss-Prot MMseqs database (skip if the runtime bundle already has one)
bash scripts/prepare_mmseqs_db.sh

# 5. Generate evidence
python scripts/precompute_bt_evidence.py \
  --config configs/generations/config_benchmark.yaml \
  --data-dir ./data --evidence-dir ./data/bt_evidence

# 6. Check
python scripts/preflight.py --preset benchmark --skip-models
```

## 3. Source inputs

The recompute source bundle contains PDB structures, UniProt GFF/FASTA, and
EVEscape tables. AlphaMissense is optional and large; the raw TSV is deleted
after extraction into per-scenario `phenotype.json` files.

```bash
# Download upstream sources
python scripts/fetch_recompute_inputs.py --verify

# Optional AlphaMissense
python scripts/fetch_recompute_inputs.py --with-alphamissense --verify
```

Review [RECOMPUTE_SOURCES_NOTICES.md](RECOMPUTE_SOURCES_NOTICES.md) before
redistributing any source input.

## 4. What each precompute step does

| script | output |
|---|---|
| `precompute_bt_evidence.py` | ProteinMPNN structure scores, MSA statistics, UniProt annotations, WT sequences |
| `precompute_phenotype.py` | AlphaMissense / EVEscape phenotype tables |
| `precompute_foldx.py` | optional local FoldX ΔΔG tables |
| `precompute_interface_evidence.py` | structure-derived functional-site evidence |
| `precompute_metal_sites.py` | FoldX applicability notes |

## 5. Verification

```bash
python scripts/preflight.py --preset benchmark --skip-models
```

A strict preflight checks the evidence directory, Swiss-Prot database,
structure/phenotype/GFF coverage, MMseqs runtime, and model assets. Missing
coverage is reported as `WARN`; the corresponding tool degrades explicitly.
