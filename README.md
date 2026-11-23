# DRYLAB-Bench

**DRYLAB-Bench** is a biosecurity benchmark for evaluating whether large
language models can design protein mutations that achieve a stated design
goal. It uses real deep mutational scanning (DMS) measurements from
ProteinGym and ViroGym as ground truth, and evaluates models under four
experimental conditions:

| Condition | Tools | Iteration |
|---|---|---|
| `S0` | none | single-shot |
| `S0-iter` | none | multi-round self-iteration |
| `S1` | biological tools | one static tool pass |
| `S2` | biological tools | adaptive multi-round iteration |

Each scenario is run with three prompt variants. A model's proposal is scored
against the DMS assay when it matches a measured mutation; unmatched
proposals are scored by a calibrated in-silico model and weighted by task
reliability.

> **Access is gated.** The benchmark code, the 30 dual-use scenarios, and the
> runtime evidence are distributed together as a manually approved Hugging
> Face dataset. This repository is the public landing page and documentation.
> It contains no benchmark code, scenarios, or results.

## Access

1. Open the gated dataset:
   <https://huggingface.co/datasets/drylab-bench/benchmark>
2. Sign in with a Hugging Face account, read the Responsible Use Terms, and
   submit the access request.
3. After manual approval, authenticate and download:

```bash
pip install -U huggingface_hub
hf auth login
hf download drylab-bench/benchmark \
  --repo-type dataset --local-dir drylab-bench
```

Approval is manual. Paper reviewers can select **Paper reviewer** in the
request form for expedited review. See [ACCESS.md](ACCESS.md) for the approval
criteria and [SCENARIO_TERMS.md](SCENARIO_TERMS.md) for the terms.

## What the benchmark measures

- **30 scenarios** spanning immune escape, receptor binding, antimicrobial
  resistance, and human-cell phenotypes.
- **Real DMS ground truth** from ProteinGym and ViroGym; the benchmark does
  not use a model to define the target phenotype.
- **Four conditions** that isolate the effect of biological tools and
  multi-round iteration.
- **Three scoring conventions**: raw composite, winsorized z-score, and
  empirical rank. The z-score is the primary scale; raw and rank are
  reported for robustness.
- **Refusal-aware reporting**: safety refusals, upstream provider blocks, and
  infrastructure errors are tracked separately from valid measurements.
- **Optional BDT arms**: EVOLVEpro / VIRAL oracle and zero-shot reproduction
  packs for the supervised-tool analysis.

## Quick start

After approval, the gated bundle contains a complete source tree, a built
wheel, documentation, and the runtime evidence manifest.

```bash
# 1. Create the environment
conda env create -f environment.yml
conda activate drylab-bench
python -m pip install -e .

# 2. Configure one OpenAI-compatible endpoint
cp .env.example .env
# edit .env: OPENAI_BASE_URL / OPENAI_API_KEY
source .env

# 3. Inspect the resolved configuration and assets
drylab-bench --list-presets
python scripts/preflight.py --preset benchmark --provider default \
  --models <model-id> --strict

# 4. Run one scenario
drylab-bench --preset benchmark --provider default \
  --models <model-id> --scenarios <scenario-id> --step conditions

# 5. Generate the unified report
python scripts/report_brs.py \
  --run-dir runs/run_<timestamp> \
  --config configs/generations/config_benchmark.yaml
```

Full installation and asset instructions are in [docs/SETUP.md](docs/SETUP.md).
The CLI reference is in [docs/CLI.md](docs/CLI.md).

## Configuration

The default preset is `benchmark`; optional BDT reproduction presets are
included in the gated bundle:

```text
configs/presets/benchmark.yaml          # default benchmark preset
configs/presets/bdt_*.yaml              # optional BDT reproduction presets
configs/generations/config_benchmark.yaml  # scenarios, tools, budgets
configs/models/catalog.yaml             # logical model id -> native name/params
configs/providers/default.yaml          # OpenAI-compatible endpoint template
configs/assets/core.yaml                # required data/model assets
configs/assets/bdt.yaml                 # optional BDT assets
```

Add a model by editing `configs/models/catalog.yaml`. Add a provider by
creating a profile under `configs/providers/`; all provider profiles use
environment-variable references, never literal keys.

## Scoring

Every scored row is a `(model, scenario, prompt variant, condition)` unit.
The benchmark reports:

- `valid`: at least one parsed final candidate and no refusal/error;
- `refused`: the model refused or the upstream provider blocked the request;
- `api_error`: transport/quota failure, not a model refusal;
- `parse_failed` / `empty_pool`: malformed output or no final candidate.

The unified BRS report includes four variants (`valid`, `refused`,
`valid_no_insilico`, `refused_no_insilico`), model/scenario scopes, and
paired condition contrasts (`S2 − S0`, `S2 − S1`, `S1 − S0`,
`S2 − S0-iter`). The precise definitions are in
[docs/SCORING.md](docs/SCORING.md).

## Reproduction packs

The gated bundle is self-contained for reproduction:

- runtime biological-tool evidence and the source bundle for rebuilding it;
- optional EVOLVEpro / VIRAL assets (oracle and zero-shot);
- the precompute inputs for adding custom scenarios;
- setup scripts and offline tests.

Start with [docs/REPRODUCE.md](docs/REPRODUCE.md). To add a scenario, see
[docs/EXTENDING.md](docs/EXTENDING.md). BDT setup, presets, and leakage rules
are in [docs/BDT.md](docs/BDT.md) and
[docs/BDT_LEAKAGE.md](docs/BDT_LEAKAGE.md).

Each run writes its resolved configuration (secrets redacted), a run
manifest, per-unit trajectories, and a coverage report. The benchmark uses
fixed random seeds for calibration and bootstrap intervals. Runtime evidence
is versioned in the gated bundle; the recompute profile rebuilds it from
public upstream sources. See [docs/RECOMPUTE.md](docs/RECOMPUTE.md).

## Responsible use

DRYLAB-Bench is a dual-use research artifact. Access is gated and subject to
the [Responsible Use Terms](SCENARIO_TERMS.md). Do not use the scenarios for
wet-lab synthesis, pathogen enhancement, or any application outside safety
evaluation and defensive research. Report misuse or vulnerabilities through
the process in [SECURITY.md](SECURITY.md).

## Citation

If you use DRYLAB-Bench, please cite the software and the accompanying paper.
Citation metadata is provided in [CITATION.cff](CITATION.cff).

## License

- **Code**: MIT. See [LICENSE](LICENSE).
- **Scenarios**: Responsible Use Agreement, not MIT. See
  [SCENARIO_TERMS.md](SCENARIO_TERMS.md).
- **Third-party data and evidence**: see
  [docs/THIRD_PARTY_NOTICES.md](docs/THIRD_PARTY_NOTICES.md) and
  [docs/RECOMPUTE_SOURCES_NOTICES.md](docs/RECOMPUTE_SOURCES_NOTICES.md).

## Repository scope

This repository is intentionally a landing page. The runnable benchmark,
scenario definitions, tests, and runtime evidence live in the gated bundle.
Bug reports and access questions can be filed in this repository's issue
tracker; code contributions are handled privately.
