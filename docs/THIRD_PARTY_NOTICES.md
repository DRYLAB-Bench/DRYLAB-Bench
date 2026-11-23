# Third-party notices for the BT evidence bundle

This notice covers the **public BT evidence bundle** produced by
`scripts/make_evidence_bundle.py`. It does not cover raw datasets or model
weights that the bootstrap script downloads directly from their upstream
sources, and it is not legal advice. Always review the upstream license before
redistributing a derived artifact.

The separate **recompute source bundle** (raw PDB/GFF/EVEscape, optional
AlphaMissense/UniProt FASTA) has its own notice:
[RECOMPUTE_SOURCES_NOTICES.md](RECOMPUTE_SOURCES_NOTICES.md).

## Included in the public evidence bundle

### UniProt / Swiss-Prot (sprotDB, GFF, sequences, annotations)

- Source: UniProt Consortium, https://www.uniprot.org/
- License: Creative Commons Attribution 4.0 International (CC BY 4.0)
- The evidence bundle includes a Swiss-Prot-derived MMseqs database and
  UniProt-derived GFF/annotation/sequence files. Attribution to UniProt is
  required. See https://www.uniprot.org/help/license.

### PDB structures and PDB-derived structure metadata

- Source: RCSB PDB, https://www.rcsb.org/
- PDB entry data are openly available; see the RCSB PDB usage policy
  (https://www.rcsb.org/pages/usage-policy). The public bundle excludes raw PDB
  files by default; use `--include-pdbs` only if the downstream use is
  compatible with the PDB terms.

### ProteinMPNN-derived structure scores

- Method: ProteinMPNN (Dauparas et al., Science 2022)
- Code license: MIT (https://github.com/dauparas/ProteinMPNN)
- The bundle contains per-mutant scores derived from PDB coordinates and
  ProteinMPNN; it does not contain ProteinMPNN code or weights.

### AlphaMissense-derived phenotype tables

- Source: AlphaMissense predictions (Cheng et al., Science 2023),
  DeepMind; hosted at
  https://console.cloud.google.com/storage/browser/dm_alphamissense
- License: **Creative Commons Attribution 4.0 International (CC BY 4.0)**
  as stated in the AlphaMissense README
  (https://github.com/google-deepmind/alphamissense).
- The bundle contains per-task `phenotype.json` tables derived from
  AlphaMissense predictions; attribution to DeepMind/AlphaMissense is required.

### EVEscape-derived phenotype tables

- Source: OATML-Markslab/EVEscape, https://github.com/OATML-Markslab/EVEscape
- License: MIT
- The bundle contains per-task phenotype tables derived from EVEscape
  precomputed escape tables.

### structure_site_importance / MSA / annotation tables

- Derived from PDB coordinates and UniProt annotations/sequences; covered by
  the RCSB PDB usage policy and the UniProt CC BY 4.0 terms above.

### ProteinGym

- Source: OATML-Markslab/ProteinGym,
  https://github.com/OATML-Markslab/ProteinGym
- License: MIT
- ProteinGym raw DMS data is **not** included in the evidence bundle; the
  bootstrap script directs users to the upstream download.

## Deliberately NOT included in the public evidence bundle

### ViroGym

- The current upstream repository has **no explicit license** in its GitHub
  metadata. Raw ViroGym data is therefore not redistributed in the evidence
  bundle. Users must obtain it from the upstream project and comply with its
  terms.

### ToxinPred3 trees

- Source: https://github.com/raghavagps/toxinpred3
- License: **GPL-3.0**
- The exported `toxinpred3_trees.npz` contains a derived representation of the
  GPL-licensed model. It is not included in the public bundle. Users who need
  the toxin tool must obtain/install ToxinPred3 under its GPL terms and export
  the trees locally (see `docs/SETUP.md`).

### ESM-1v / ESM-2 model weights

- ESM code is MIT (https://github.com/facebookresearch/esm), but model weights
  are obtained from Hugging Face and are **not redistributed** in the evidence
  bundle. The bootstrap script downloads them from the upstream model
  repositories; users are responsible for accepting the model terms.

### EVE TP53 predictions

- The existing downloader fetches these from evemodel.org. The file is not
  included in the public evidence bundle; use
  `scripts/download_insilico_models.py` and review the upstream terms.

### FoldX and FoldX-derived caches

- FoldX is commercial/academic-licensed software. The public bundle excludes
  `data/bt_evidence/foldx_ondemand/` and per-task `foldx_ddg.json`.
  Users with a FoldX license can install FoldX and regenerate these caches
  locally.

### BDT artifacts (EVOLVEpro / VIRAL)

- The oracle/zero-shot BDT artifacts are optional rebuttal assets and are not
  part of the public evidence bundle. See `scripts/setup_bdt_env.sh` and the
  VIRAL Zenodo terms if reproducing those arms.

## Attribution reminder

If you publish results or a derived bundle, retain this notice and cite the
upstream methods/datasets (UniProt, RCSB PDB, ProteinMPNN, AlphaMissense,
EVEscape, ProteinGym, and the relevant DMS studies).
