# Project: Target Credentialing Pipeline: Wnt Pathway & ERG/AR 

## Purpose
Portfolio project demonstrating target credentialing methods using publicly available functional genomic and patient genomic data. 

## Overview

### Wnt Pathway 
- Functional genomics: DepMap CRISPR dependency (Chronos) scores for Wnt pathway genes, stratified by APC/CTNNB1/RNF43 mutation status (DepMap join key: `depmap_id`). 
    Question: Does Wnt-activating mutation status predict pathway dependency?
- Patient genomics: MSK-IMPACT clinical sequencing data (real-world, multi-cancer-type cohort) including prevalence and co-occurrence of Wnt-activating mutations. TCGA PRAD's mutation data is confirmed too sparse for this purpose (APC=3, CTNNB1=8, RNF43=0 records across 333 patients) — MSK-IMPACT is the correct source, not just an access-driven substitute. Analysis deduplicates to one sample per patient (48,179 unique patients vs. 54,331 samples); TCF7L2 prevalence excludes IMPACT341-panel samples, which don't test this gene.
    Question: what is the broader indication-expansion opportunity for a Wnt-pathway therapeutic?

### ERG/AR 
- **Scope note:** the functional genomics comparison originally planned here (DepMap CRISPR dependency by TMPRSS2-ERG fusion status) was cut due to time constraints and low evidentiary value — DepMap has no usable fusion calls for this gene pair, and the only available comparison was one fusion-positive line (VCaP) vs. three fusion-negative lines, too small to be worth the space even as a descriptive note. This arm now rests entirely on the well-powered TCGA patient genomics analysis below.
- Patient genomics: TCGA-PRAD co-occurrence of TMPRSS2-ERG fusion status (via the curated "1-ERG" subtype field, n=152/333) with AR-V7 presence/AR Score. 
    Question: what does this suggest about combination or patient-stratification strategy?

## Analytical Framework 
Use a cross-list evidence integration approach: classify findings as 
    (1) functional-genomics-supported
    (2) patient-genomics-supported (prevalent/co-occurring in the relevant population)
    (3) supported by both (the strongest candidates supported by multiple independent lines of evidence)

## Data Sources (see also SPEC.md)
- DepMap: https://depmap.org/portal/download/ — 
        `CRISPR_(DepMap_Public_26Q1+Score,_Chronos)_subsetted.csv`, `Damaging_Mutations_(Public_26Q1)_subsetted.csv`, `Hotspot_Mutations_(Public_26Q1)_subsetted.csv`
- TCGA Prostate Adenocarcinoma (Cell 2015), via cBioPortal (https://www.cbioportal.org/) — used for the ERG/AR arm's patient genomics component:
        `TCGA_PRAD_clinical_patient.txt`, `TCGA_PRAD_clinical_sample.txt`, `TCGA_PRAD_mutations.txt`, `TCGA_PRAD_structural_variants.txt`
- MSK-IMPACT Clinical Sequencing Cohort, via cBioPortal — used for the Wnt-pathway arm's patient genomics/indication-expansion component:
        `msk_impact_clinical_patient.txt`, `msk_impact_clinical_sample.txt`, `msk_impact_mutations.txt`, `msk_impact_structural_variants.txt`

## Statistical Methods
- Spearman correlation for continuous dependency-vs-mutation-burden relationships
- Mann-Whitney/Wilcoxon for two-group comparisons where sample size supports it
- Fisher's exact test for categorical co-occurrence 
- Benjamini-Hochberg FDR correction wherever testing multiple genes/pathways simultaneously

## Tone/Rigor Standard
State sample sizes plainly, flag underpowered comparisons explicitly rather than hiding them, and distinguish exploratory signal from decision-grade evidence at every step. Do not overstate findings. Small-n limitations are a feature of honest reporting, not something to paper over.

## Output Goal
A single integrated write-up (Jupyter notebook or R Markdown, output as both code and a rendered
report) presenting both pathway arms under one "target credentialing" narrative, suitable as a portfolio
piece for translational data science roles.