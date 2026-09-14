# Target Credentialing: Wnt Pathway & ERG/AR in Cancer

Good targets fail in the wrong patients. This project uses functional and patient genomics to ask: which patients actually carry the dependency? A demonstration of target credentialing methodology, using public data.

## Summary

This project applies one credentialing framework to two different oncology target
questions:

- **Wnt pathway:** Does Wnt-activating mutation status (APC/CTNNB1/RNF43) predict CRISPR
  dependency on Wnt-pathway genes, and how prevalent are these mutations outside colorectal
  cancer? Is there an indication-expansion opportunity?
- **ERG/AR:** Does TMPRSS2-ERG fusion status co-occur with AR-pathway alteration in prostate
  cancer, and does it inform patient stratification or combination strategy?

Each question is evaluated using public functional and patient genomics data.

## Key Findings

- **CTNNB1 in endometrial cancer is the lead credentialed hypothesis:** the same gene
  defines a prevalent (25.1%, n=2,353), literature-validated patient population outside
  colorectal cancer *and* shows the strongest functional dependency (rank-biserial r=0.70,
  FDR q≈10⁻³⁷) in Wnt-activating-mutant cell lines.
- **TCF7L2 is credentialed through a different line of evidence:** it's rarely mutated outside CRC,
  so its functional dependency is conferred by the composite APC/CTNNB1/RNF43 biomarker rather than
  by its own mutation status. A TCF7L2-directed therapy should select patients on composite
  biomarker status, not TCF7L2 mutation status.
- **TMPRSS2-ERG fusion status predicts AR transcriptional activity, not AR-V7 status:** fusion-positive
  tumors show significantly lower AR Score (p=4.9×10⁻⁸) but no difference in AR-V7 splice-variant
  presence (p=0.48). AR Score should be used as an independent patient-stratification factor
  alongside fusion status in combination trial design, not inferred from fusion status alone.
  Evaluated on patient genomics only, since no usable functional dependency data exists publicly
  for this fusion.

## Methods Overview

**Wnt pathway:** DepMap CRISPR dependency scores (Chronos) for Wnt-pathway genes were compared
between Wnt-activating-mutant and wild-type cell lines (Mann-Whitney U, Benjamini-Hochberg FDR
correction across genes). Mutation prevalence and cross cancer-type distribution were assessed
in a 48,179-patient MSK-IMPACT cohort, with explicit handling of patient deduplication and gene
panel coverage differences across sequencing platform versions.

**ERG/AR:** TMPRSS2-ERG fusion status (via TCGA's curated molecular subtype field, confirmed with structural variant calls) was tested against AR-pathway
alteration (AR-V7 presence, AR Score) in a 333-patient TCGA prostate adenocarcinoma cohort
(Fisher's exact test, Mann-Whitney U).

Full methodology, statistical detail, and interpretation are in the notebook.

## Data Sources

- [DepMap](https://depmap.org/portal/) — CRISPR dependency (Chronos) scores and mutation
  classifications
- [MSK-IMPACT Clinical Sequencing Cohort](https://www.cbioportal.org/) (via cBioPortal) — 
  real-world clinical sequencing data spanning many cancer types
- [TCGA Prostate Adenocarcinoma (Cell 2015)](https://www.cbioportal.org/) (via cBioPortal) — 
  curated prostate cancer cohort with molecular subtype and AR-pathway characterization

**Raw data files are not included in this repository** (excluded via `.gitignore` — one MSK-IMPACT
file alone is ~180MB, over GitHub's 100MB hard limit, and the rest were left out for consistency
rather than committing a partial dataset). To reproduce the analysis, download the following into
a local `data/` folder at the project root, keeping the original filenames:

- From [DepMap](https://depmap.org/portal/download/), the custom-gene-list downloads (CRISPR
  Chronos scores, Damaging Mutations, Hotspot Mutations) for the panel APC, CTNNB1, RNF43,
  TCF7L2, AXIN1, AXIN2, ERG, AR, TMPRSS2:
  `CRISPR_(DepMap_Public_26Q1+Score,_Chronos)_subsetted.csv`,
  `Damaging_Mutations_(Public_26Q1)_subsetted.csv`, `Hotspot_Mutations_(Public_26Q1)_subsetted.csv`
- From [cBioPortal](https://www.cbioportal.org/), the **MSK-IMPACT Clinical Sequencing Cohort**
  study files: `msk_impact_clinical_patient.txt`, `msk_impact_clinical_sample.txt`,
  `msk_impact_mutations.txt`, `msk_impact_structural_variants.txt`
- From [cBioPortal](https://www.cbioportal.org/), the **TCGA Prostate Adenocarcinoma (Cell 2015)**
  study files: `TCGA_PRAD_clinical_patient.txt`, `TCGA_PRAD_clinical_sample.txt`,
  `TCGA_PRAD_mutations.txt`, `TCGA_PRAD_structural_variants.txt`

See the notebook for the exact gene lists and any additional download parameters used.

## Repository Structure

```
├── README.md
├── analysis/
│   ├── target_credentialing.ipynb      # full analysis notebook
│   └── target_credentialing.html       # rendered export
├── data/
├── figures/
│   ├── wnt_dependency_by_gene.png
│   ├── wnt_prevalence_by_cancer_type.png
│   └── erg_ar_cooccurrence.png
├── slides/
│   └── target_credentialing_summary_slides.pdf
└── dev/
    ├── CLAUDE.md                        # project context for AI-assisted development
    └── SPEC.md                          # analysis specification
```

## Limitations

Full detail is in the notebook's Discussion section. Several cancer-type prevalence estimates
are from modest sample sizes. TCGA and MSK-IMPACT trade depth of curation against real-world scale
differently. ERG-fusion classification is from the TCGA curated subtype field rather than raw
structural variant calls. Wnt-activating mutation definition is assessed via precomputed classifications
in DepMap versus direct variant-level verification in MSK-IMPACT. These calls are conceptually
equivalent but not methodologically identical. The ERG/AR analysis is limited to patient genomics,
since no usable public functional dependency data exists for TMPRSS2-ERG fusion status.

## How This Was Built

This analysis was developed using an AI-assisted, spec-driven workflow (Claude Code), with a
persistent project specification (see `dev/`) guiding data validation, methodology, and
statistical rigor at each step. All analytical decisions, data source substitutions, and scope
limitations were made deliberately and are documented in the notebook itself.
