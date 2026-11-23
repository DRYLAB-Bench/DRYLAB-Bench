# Extending the benchmark with a new scenario

A scenario needs five things:

1. a DMS assay with a measurable phenotype;
2. a scenario block in the experiment config;
3. a verbatim TASK_SPEC and a tool/reward note;
4. biological-tool evidence (structure, MSA, annotation, phenotype);
5. a preflight pass and a small smoke run.

The recompute source bundle contains the public upstream inputs (PDB, UniProt
GFF/FASTA, EVEscape tables) needed for step 4.

## 1. Install the recompute sources

```bash
python scripts/bootstrap_assets.py --profile recompute \
  --recompute-bundle evidence/recompute_sources/recompute_sources_2026-11-20.tar.gz --yes
```

AlphaMissense and the Swiss-Prot FASTA are optional/large; see
[RECOMPUTE.md](RECOMPUTE.md). The source bundle does not contain ProteinGym or
ViroGym raw DMS data; obtain those from their upstream projects.

## 2. Add the DMS assay

- ProteinGym: the assay is read from `data/ProteinGym/DMS_substitutions.csv`
  by `assay_id`.
- ViroGym: the assay is read from `data/ViroGym/DMS/benchmark.csv` and the
  per-assay CSV.
- Set `polarity` correctly (`higher_is_worse` or `lower_is_worse`).
- If the assay's residue numbering differs from the protein sequence, set
  `residue_offset` or `residue_map`.

The benchmark does not redistribute ViroGym; follow its upstream terms.

## 3. Add the scenario block

Create a custom config, for example:

```bash
cp configs/generations/config_benchmark.yaml configs/generations/config_custom.yaml
```

Copy the scenario template from `templates/scenario_template.yaml` and append
it under `scenarios:`. The key fields are:

```yaml
- id: my_scenario
  name: "My scenario"
  data_source: proteingym        # or virogym
  protein_id: ...
  protein_name: ...
  data_file: DMS_substitutions.csv
  assay_id: ...
  mutation_col: mutant
  score_cols:
    fitness_score: DMS_score
  wildtype_sequence: ''
  ground_truth:
    primary_field: fitness_score
    secondary_field: null
    primary_threshold: 0.0
    polarity: higher_is_worse
  min_mutations_suggest: 3
  max_mutations_suggest: 3
  num_candidates_rank: 50
  rank_strata: [12, 26, 12]
```

Also add the scenario id to `mvp.tasks` so the default matrix includes it.

## 4. Add the TASK_SPEC and tool note

The iterative benchmark pipeline renders a verbatim task description from
`drylab_bench/v2_prompts._PREFIXES` and a tool/reward note from
`_SCORING_DIMENSION_NOTES`.

1. Add `my_scenario` to `_PREFIXES` with the exact prompt text: goal, protein,
   mutation-count guidance, output format, and context. Do not include GT
   numbers or a "use this tool in reverse" conclusion.
2. Add `my_scenario` to `_SCORING_DIMENSION_NOTES` describing what each tool
   measures and which dimension is the reward versus a constraint.
3. If the scenario is also needed by the legacy single-shot path, add it to
   `drylab_bench/scenarios.py`.

A configuration can also inject TASK_SPEC overrides through
`experiment.task_spec_overrides: my_module:MY_SPECS` without editing the core
prompts; see `register_task_spec_overrides` in `drylab_bench/v2_prompts.py`.

## 5. Generate biological-tool evidence

```bash
# Structure / MSA / annotation / WT sequence
python scripts/precompute_bt_evidence.py \
  --config configs/generations/config_custom.yaml \
  --data-dir ./data --evidence-dir ./data/bt_evidence \
  --proteins my_scenario

# Phenotype table (AlphaMissense or EVEscape), if applicable
python scripts/precompute_phenotype.py \
  --config configs/generations/config_custom.yaml \
  --data-dir ./data --evidence-dir ./data/bt_evidence \
  --scenario my_scenario --am-tsv /path/to/AlphaMissense.tsv.gz

# FoldX / structure-derived evidence: use the runtime configuration
# conditions.tools.phenotype.foldx.pdb / .chain
```

Notes:

- The legacy `precompute_foldx.py` / `precompute_interface_evidence.py`
  fixed-scenario drivers are not part of the public bundle. Configure the
  runtime FoldX backend instead.
- The precompute scripts read WT/PDB/annotation data, never the benchmark GT.
  Keep it that way (R1a).
- AlphaMissense is a human-pathogenicity proxy and is not appropriate for
  every protein. EVEscape is for viral escape tasks. A missing table degrades
  the tool explicitly; do not fabricate one.
- PDB, UniProt, AlphaMissense, EVEscape, and FoldX have their own licenses.
  See [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md) and
  [RECOMPUTE_SOURCES_NOTICES.md](RECOMPUTE_SOURCES_NOTICES.md).

## 6. Validate

```bash
python scripts/preflight.py --config configs/generations/config_custom.yaml \
  --skip-models

drylab-bench --config configs/generations/config_custom.yaml \
  --provider default --models <model-id> \
  --scenarios my_scenario --step conditions
```

Check that:

- the scenario resolves and the WT residue validation passes;
- the expected tool evidence files exist;
- derived S0/S1 snapshots are produced from the S2 trajectory;
- the coverage report includes the new scenario;
- a fresh `report_brs.py` run includes the scenario in `scope_all`.

## 7. Calibrate the in-silico fallback

The evaluator calibrates the in-silico fallback on a held-out split of the DMS
assay. A new small assay may be rejected by the reliability threshold; it is
still reported, with `accepted=false`. Inspect the calibration log and the
`reliability_aligned` weight before interpreting miss scores.

## 8. Templates

- `templates/scenario_template.yaml` — scenario block skeleton.
- `templates/README.md` — where each piece of the scenario definition goes.
- `precompute/README.md` — the evidence-generation workflow.
