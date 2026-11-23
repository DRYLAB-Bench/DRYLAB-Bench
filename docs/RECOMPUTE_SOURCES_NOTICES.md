# Recompute source bundle — third-party notices

This notice covers the **recompute source bundle** produced by
`scripts/make_recompute_bundle.py`. It is not legal advice; review the upstream
terms before redistributing a modified bundle.

The recompute bundle contains source inputs used to regenerate the derived BT
evidence tables. It does **not** contain the evaluator's DMS/GT data, model
weights, FoldX, ToxinPred3, or BDT artifacts.

## Included sources

### RCSB PDB structures

- Source: RCSB PDB, https://www.rcsb.org/
- Files: experimental structures under `data/bt_evidence/*.pdb{,.gz}`
- Terms: RCSB PDB usage policy, https://www.rcsb.org/pages/usage-policy
- Use: ProteinMPNN structure scores and FoldX inputs. The derived runtime
  bundle excludes raw PDBs by default; this recompute bundle includes them
  explicitly.

### UniProt / Swiss-Prot

- Source: UniProt Consortium, https://www.uniprot.org/
- Files: `data/bt_evidence/*.gff`; optionally
  `data/recompute_inputs/uniprot_sprot.fasta.gz`
- License: Creative Commons Attribution 4.0 International (CC BY 4.0),
  https://www.uniprot.org/help/license
- Use: functional annotations, MSA/Neff statistics and the MMseqs2 Swiss-Prot
  search database. Attribution to UniProt is required.

### EVEscape

- Source: OATML-Markslab/EVEscape,
  https://github.com/OATML-Markslab/EVEscape
- Files: official per-mutation tables from
  `results/summaries_with_scores/` (for example `full_spike_evescape.csv`,
  `flu_h1_evescape.csv`, `hiv_env_evescape.csv`,
  `lassa_glycoprotein_evescape.csv`, `nipah_glycoprotein_evescape.csv`)
- License: MIT
- Use: per-task `phenotype.json` escape tables. Only the predictive
  `evescape` column is used; experimental columns are ignored (R1a).

### AlphaMissense (optional)

- Source: AlphaMissense predictions (Cheng et al., Science 2023), DeepMind
- File: `data/recompute_inputs/AlphaMissense_aa_substitutions.tsv.gz`
- License: Creative Commons Attribution 4.0 International (CC BY 4.0), as
  stated in https://github.com/google-deepmind/alphamissense
- Use: per-task `phenotype.json` pathogenicity tables. Attribution to
  DeepMind/AlphaMissense is required. The file is large (~1.2 GB compressed),
  so it is excluded unless `--include-alphamissense` is used.

## Not included

- **ProteinGym / ViroGym raw DMS data**: ProteinGym is MIT and is downloaded
  by the bootstrap helper; ViroGym currently has no explicit upstream license
  and is never redistributed. Users must obtain each dataset from upstream.
- **ESM-1v / ESM-2 weights**: downloaded from Hugging Face; not redistributed.
- **EVE TP53 predictions**: obtained from evemodel.org by the existing
  downloader; not redistributed.
- **ToxinPred3 trees**: GPL-3.0; exported locally after the user accepts the
  upstream terms; not redistributed.
- **FoldX**: commercial/academic-licensed software and its caches are excluded.
- **BDT oracle/zero-shot artifacts**: optional, GT-trained or separately
  trained; excluded.

## Attribution reminder

If you publish results or a modified source bundle, retain this notice and cite
the upstream resources: UniProt, RCSB PDB, EVEscape, AlphaMissense (if used),
ProteinMPNN, ProteinGym, and the relevant DMS studies.
