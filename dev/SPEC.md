# SPEC: Target Credentialing Pipeline — Wnt Pathway & ERG/AR

## Overview

This project applies a target credentialing framework to public data. Wnt-pathway program and an ERG/AR-driven prostate cancer program. Each arm integrates
functional genomics (CRISPR dependency data) with patient genomics (mutation, fusion, and
pathway-alteration data) to evaluate the strength of evidence supporting each target
hypothesis.

## Data Sources

- **DepMap** (depmap.org/portal): CRISPR dependency (Chronos) scores and mutation classifications
  (Hotspot Mutations, Damaging Mutations), scoped to a custom gene list (APC, CTNNB1, RNF43,
  TCF7L2, AXIN1, AXIN2, ERG, AR, TMPRSS2), with cell line metadata included.
  - `CRISPR_(DepMap_Public_26Q1+Score,_Chronos)_subsetted.csv`
  - `Damaging_Mutations_(Public_26Q1)_subsetted.csv`
  - `Hotspot_Mutations_(Public_26Q1)_subsetted.csv`
- **TCGA Prostate Adenocarcinoma (Cell 2015)**, via cBioPortal: a curated dataset of 333 patients
  with molecular subtype classification (including ERG-fusion status), structural variant data,
  and AR-pathway variables (AR mRNA, AR Protein, AR Score, AR-V7 presence/ratio/reads).
  - `TCGA_PRAD_clinical_patient.txt`
  - `TCGA_PRAD_clinical_sample.txt`
  - `TCGA_PRAD_mutations.txt`
  - `TCGA_PRAD_structural_variants.txt`
- **MSK-IMPACT Clinical Sequencing Cohort**, via cBioPortal: a large real-world clinical
  sequencing dataset spanning many cancer types, used for the Wnt-pathway arm's patient
  genomics/indication-expansion component (Task 1.3), in place of generic TCGA pan-cancer data
  or AACR GENIE — GENIE requires an institutional Data Use Agreement not accessible without a
  current employer affiliation. MSK-IMPACT is a stronger choice than a curated research cohort
  for this specific question, since it reflects real-world clinical sequencing across a large,
  multi-cancer-type population.
  - `msk_impact_clinical_patient.txt`
  - `msk_impact_clinical_sample.txt`
  - `msk_impact_mutations.txt`
  - `msk_impact_structural_variants.txt`

## Phase 0 — Data Acquisition & Validation

1. Load DepMap CRISPR dependency, hotspot mutation, and damaging mutation files
   (`CRISPR_(DepMap_Public_26Q1+Score,_Chronos)_subsetted.csv`,
   `Hotspot_Mutations_(Public_26Q1)_subsetted.csv`,
   `Damaging_Mutations_(Public_26Q1)_subsetted.csv`); confirm cell line identifiers (`depmap_id`)
   are consistent across all three files for joining.
2. Load MSK-IMPACT clinical and mutation data (`msk_impact_clinical_patient.txt`,
   `msk_impact_clinical_sample.txt`, `msk_impact_mutations.txt`,
   `msk_impact_structural_variants.txt`) for the Wnt arm's indication-expansion analysis, and
   TCGA Prostate Adenocarcinoma (Cell 2015) clinical/mutation/structural variant data
   (`TCGA_PRAD_clinical_patient.txt`, `TCGA_PRAD_clinical_sample.txt`, `TCGA_PRAD_mutations.txt`,
   `TCGA_PRAD_structural_variants.txt`) for the ERG/AR arm.
3. Record sample sizes and confirm data completeness for each source before proceeding.

## Phase 1 — Wnt Pathway

### Task 1.1: Define Wnt-Activating Mutation Criteria
- APC: truncating/nonsense mutations only (exclude missense unless annotated pathogenic)
- CTNNB1: exon 3 hotspot mutations (canonical codons, e.g., S33, S37, S45, T41)
- RNF43: truncating mutations only
- Output: a per-cell-line (DepMap, `depmap_id`) and per-patient (MSK-IMPACT) binary "Wnt-activating mutation
  present" flag, with the underlying mutation definitions documented for transparency.

### Task 1.2: Functional Genomics — Dependency vs. Mutation Status
- Merge DepMap Chronos scores for CTNNB1, TCF7L2 (and optionally AXIN1/2) with the Task 1.1
  mutation flag, by cell line.
- Compare dependency scores between Wnt-activating-mutant and wild-type lines (Wilcoxon test).
- Visualize: dependency score by mutation status, faceted by gene.
- Report effect size and direction alongside significance, particularly given a likely modest n.

### Task 1.3: Patient Genomics — Prevalence and Co-Occurrence
- Using MSK-IMPACT clinical sequencing data, calculate prevalence of Wnt-activating mutations
  (Task 1.1 definition) across represented cancer types, with particular attention to
  indications beyond colorectal cancer.
- **Analysis unit:** MSK-IMPACT has 48,179 unique patients but 54,331 samples (some patients
  sequenced more than once across timepoints/panel versions). Deduplicate to one sample per
  patient (first sample per PATIENT_ID, or highest-coverage sample where easily determined)
  before calculating prevalence, and state the resulting analysis-unit n explicitly. Prevalence
  statistics assume independent observations; counting a patient twice would bias the result.
- **Gene panel coverage adjustment:** MSK-IMPACT samples were sequenced on four gene panel
  versions (IMPACT341/410/468/505) that expanded over time. TCF7L2 is absent from the earliest
  panel, IMPACT341 (2,599 samples). Exclude IMPACT341 samples from the TCF7L2 prevalence
  denominator specifically, and state this adjustment explicitly rather than leaving an
  uncorrected undercount. All other target genes are present across all four panel versions.
- Identify the cancer types with the highest prevalence outside CRC — the indication-expansion
  signal.
- **Rationale for MSK-IMPACT over TCGA (empirically confirmed, not just an access-driven
  substitution):** TCGA PRAD's mutation data is too sparse to support this analysis — across
  333 patients, mutation record counts are APC=3, CTNNB1=8, RNF43=0, confirming that TCGA is
  not a viable source for Wnt-pathway prevalence. MSK-IMPACT, a real-world clinical sequencing
  cohort spanning many cancer types, is the appropriate source for this task; TCGA PRAD's role
  stays confined to the ERG/AR arm (Task 2.1), where its data is adequate.

### Task 1.4: Arm 1 Synthesis
- Integrate Tasks 1.2 and 1.3: genes or mutation patterns supported by both functional
  dependency evidence and patient-population prevalence represent the highest-credentialed
  candidates for indication-expansion hypotheses.

## Phase 2 — ERG/AR

**Scope note (updated):** this arm is scoped to patient genomics only (Task 2.1 below). The
functional genomics comparison originally planned here (DepMap CRISPR dependency by
TMPRSS2-ERG fusion status) was cut given time constraints and its low evidentiary value — DepMap
has no usable fusion calls for this gene pair, and the only available comparison was a single
fusion-positive cell line (VCaP) against three fusion-negative lines, too small to support even
descriptive reporting worth the space. This arm now rests entirely on TCGA's well-powered,
curated patient cohort, which is the stronger analysis regardless of time constraints.

### Task 2.1: Patient Genomics — ERG Fusion x AR Alteration Co-Occurrence
- TCGA Prostate Adenocarcinoma (Cell 2015) includes a curated molecular subtype field, with the
  "1-ERG" subtype (n=152/333, 45.6%) corresponding directly to ERG-fusion-positive tumors —
  consistent with the dataset's structural variant calls.
- Because ETV1 and ETV4 subtypes in this cohort are also fusion-driven (with different fusion
  partners than ERG), the fusion-negative comparison group is defined as non-fusion-driven
  subtypes only (e.g., "other" and "SPOP"), explicitly excluding ETV1/ETV4 to preserve a clean
  fusion-vs-non-fusion contrast.
- AR-pathway status is assessed using AR-V7 presence (binary) and/or AR Score (continuous),
  both available directly in this dataset — a more specific and clinically meaningful choice
  than a generic amplification/mutation flag, given AR-V7's established role in treatment
  resistance.
- Statistical approach: Fisher's exact test for the binary AR-V7 comparison, or a Mann-Whitney
  test if using continuous AR Score.
- Visualize: contingency table or mosaic plot (binary) or boxplot by subtype group (continuous).

### Task 2.2: Arm 2 Synthesis
- Since this arm now rests on a single, well-powered patient genomics analysis (Task 2.1) rather
  than two converging evidence lines, synthesis here is a direct statement of the co-occurrence
  finding and its patient-stratification/combination-rationale implication, rather than a
  cross-list integration exercise like Arm 1's Task 1.4.
- State clearly that this arm's evidence base is narrower in kind (patient genomics only) than
  Arm 1's (functional + patient genomics), and note this as a scope decision driven by time
  constraints and the low evidentiary value of the only available functional comparison — not
  as a gap in rigor.

## Phase 3 — Integrated Write-Up

- Present both arms under a single target credentialing framework: problem framing, Arm 1
  (methods, results, synthesis), Arm 2 (methods, results, synthesis), and a closing reflection
  on what credentialing evidence looks like across two different target and modality contexts.
- Output as a rendered notebook (Jupyter via nbconvert, or R Markdown), suitable for sharing as
  a standalone portfolio artifact.
- Include a limitations section covering sample size constraints, the distinction between
  curated research cohorts and real-world clinical data, and the fusion-status classification
  approach used in Arm 2.

## Statistical Methods

- Spearman correlation for continuous dependency-versus-mutation relationships
- Wilcoxon/Mann-Whitney for two-group comparisons where sample size supports formal testing
- Fisher's exact test for categorical co-occurrence
- Benjamini-Hochberg FDR correction wherever multiple genes or pathways are tested simultaneously
- Arm 2's functional genomics comparison (fusion-positive vs. fusion-negative dependency) was
  scoped out of this project due to time constraints; see Phase 2 scope note for rationale

## Scope

This project focuses on functional and patient genomics evidence integration. It does not
include structural or protein-protein interaction modeling, de novo CRISPR screen design or
hit-calling (DepMap's pre-computed dependency scores are used throughout, not raw screen data),
or real-world clinical/EHR outcomes data, which is not publicly accessible without an
institutional affiliation.