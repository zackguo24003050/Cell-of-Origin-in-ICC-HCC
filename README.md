# Cell-of-Origin in ICC-HCC

This repository contains analysis scripts for a summer research project on cell-of-origin inference in combined hepatocellular carcinoma and intrahepatic cholangiocarcinoma (cHCC-ICC).

The HTML reports should be downloaded and opened locally to view the full source document; they contain both the analysis code and the knitted results.

**Project period:** June 2025 - September 2025  
**Supervisor:** Dr. Gladys Poon, Research Assistant Professor, School of Biomedical Sciences, HKUMed  
**Current stage:** exploratory integration of single-cell, bulk ATAC, and tumour mutation data  
**Main question:** can genome-wide mutation-density patterns help infer the likely cell-of-origin of different cHCC-ICC components or subtypes?

## Project Overview

Cancer mutations are not uniformly distributed across the genome. Regional mutation density is influenced by chromatin organization, replication timing, and other cell-type-specific genomic features. This project applies that concept to cHCC-ICC, asking whether tumor mutation-density profiles are more consistent with hepatocyte-like or cholangiocyte-like chromatin states.

The long-term goal is to compare cHCC-ICC mutation-density landscapes with normal liver and liver-lineage chromatin references, then use those comparisons to support cell-of-origin interpretation.

```text
cHCC-ICC tumor mutations
        |
        v
Genome-wide mutation-density profiles
        |
        v
Comparison with liver-lineage chromatin references
        |
        v
Candidate cell-of-origin interpretation
```

## Scientific Context

This project is motivated by three related ideas:

- Cell-of-origin chromatin organization can shape regional somatic mutation density across cancer genomes.
- cHCC-ICC contains hepatocellular and cholangiocytic differentiation, making its origin biologically ambiguous.
- Liver organoid and liver-lineage epigenomic data may provide useful reference states, but their quality needs to be evaluated before downstream interpretation.

## Data Sources

The analysis uses or references:

- cHCC-ICC genomic data from the Cancer Cell study of combined hepatocellular and intrahepatic cholangiocarcinoma.
- Tumor VCF files for mutation-density profiling.
- Bulk ATAC-seq data comparing hepatocyte and biliary-accessible regions.
- Liver organoid scRNA-seq and scATAC-seq data from HM and DM conditions.

Large raw files and intermediate analysis objects are not intended to be stored directly in this repository.

## Current Analysis Stage

### Single-Cell RNA Reference

The scRNA-seq workflow reprocesses HM and DM liver organoid samples, performs clustering, identifies marker genes, and assigns cell-type labels based on the reference paper.

The annotated scRNA object is used as a reference for scATAC label transfer.

### Single-Cell ATAC Integration

The scATAC-seq workflow integrates four HM/DM ATAC samples, applies LSI and Harmony correction, computes a gene activity matrix, and transfers scRNA-derived labels onto ATAC cells.

The current analysis focuses on quality checking this transferred annotation.

### Label Transfer Quality Assessment

Three checks are used to evaluate whether transferred scATAC labels are reliable:

- prediction score distribution
- agreement between unsupervised ATAC clusters and transferred scRNA labels
- marker gene activity across transferred labels

Current interpretation:

```text
The transferred scATAC labels show moderate prediction confidence, weak concordance with unsupervised ATAC clusters, and limited marker support. These labels should therefore be treated as exploratory rather than as high-confidence cell-type annotations.
```

This supports the decision to be cautious about using this scATAC dataset as a cell-type-specific chromatin reference.

## Downstream Analysis Workflow

The downstream analysis now follows five documented steps. The final three
notebooks are exploratory and are deliberately kept separate from the
single-cell reference work.

### 01. scRNA reference

`scRNA clustering and annotating.html` reprocesses the HM/DM liver organoid
scRNA-seq data, clusters cells, identifies marker genes, and assigns the
reference cell-type labels used later for ATAC label transfer.

**Result:** a labelled scRNA reference was produced for the scATAC analysis.

### 02. scATAC integration and annotation

`scATAC-integration.html` integrates four HM/DM scATAC samples using LSI and
Harmony, constructs a peak-based gene-activity assay, and transfers the scRNA
labels to ATAC cells.

**Result:** label transfer was possible, but prediction scores, cluster
agreement, and marker support were only moderate/limited. The labels are
therefore exploratory rather than a high-confidence cell-type reference.

### 03. Bulk ATAC differential analysis

`03 bulk ATAC analysis.Rmd` prepares the bulk ATAC count matrix, models the
Hepatocyte (H) versus biliary (I/BEC) contrast with DESeq2, annotates peaks,
and exports significant differential-accessibility regions (DARs) as BED
files for downstream overlap analysis.

**Result:** hepatocyte-associated and biliary-associated DAR sets were defined
for use as chromatin references. This step provides the regions, not a final
cell-of-origin call.

### 04. Tumour VCF screening

`04 VCF QC and screening.Rmd` reads the eight tumour samples (`Com01-04H/I`).
Each VCF contains tumour and matched-normal columns, but the records lack a
reliable Mutect2 `FILTER` field, so the notebook applies an explicit
exploratory screen for standard biallelic SNVs, tumour/normal allele evidence,
mapping/base-quality support, and common/low-quality exclusions.

**Result:** a screened SNV table and QC summary were generated for the eight
labelled tumour samples. These are candidate-like WES calls; no capture or
callable-region BED was supplied, so mutation densities should not be treated
as calibrated absolute rates.

### 05. DAR mutation comparison

`05 DAR enrichment and classification.Rmd` overlaps the screened SNVs with
the human-coordinate hepatocyte and biliary DAR BED files. Counts are scaled
to each sample's total screened mutation burden (`per_10000_mutations`) and
also normalized by the total size of each DAR set. A paired H-versus-I
direction test is then applied across the four matched patient pairs.

Mix02T is not part of the current analysis, and no external validation or
classifier is reported. The notebook is now a descriptive comparison only.

**Result:** the expected direction was not consistent enough to support a
reliable H/I separation. The result should be interpreted as exploratory,
given the small number of pairs, candidate-like VCF calls, and missing
callable-region normalization.

## Repository Structure

```text
.
+-- scRNA-clustering-and-annotating.html
+-- scATAC-integration.html
+-- 03 bulk ATAC analysis.Rmd
+-- 04 VCF QC and screening.Rmd
+-- 05 DAR enrichment and classification.Rmd
+-- README.md
```

## Main Analysis Files

| File | Purpose |
|---|---|
| `scRNA clustering and annotating.Rmd` | Reprocesses HM/DM liver organoid scRNA-seq data, annotates clusters, and creates the scRNA reference for ATAC label transfer |
| `scATAC integration.Rmd` | Integrates HM/DM scATAC-seq data, transfers scRNA labels, and evaluates label quality |
| `03 bulk ATAC analysis.Rmd` | Differential bulk ATAC analysis and export of hepatocyte- and biliary-associated DAR BED files |
| `04 VCF QC and screening.Rmd` | Exploratory screening and QC of the eight tumour VCFs |
| `05 DAR enrichment and classification.Rmd` | TMB-scaled DAR overlap counts and paired H/I direction test; no validation classifier |

## Software

The analysis is mainly written in R. Packages used across the project include:

- dplyr
- tidyr
- ggplot2
- data.table
- GenomicRanges
- GenomeInfoDb
- rtracklayer
- VariantAnnotation
- DESeq2
- edgeR
- Seurat
- Signac
- Harmony

## Current Status

| Module | Status | Notes |
|---|---:|---|
| scRNA reference annotation | Completed | Used to create the scRNA reference for ATAC label transfer |
| scATAC integration | Current stage | Includes label transfer and quality assessment |
| scATAC label quality check | Current stage | Results suggest caution in using transferred labels |
| Bulk ATAC comparison | Completed, exploratory | DAR reference sets exported for hepatocyte versus biliary comparison |
| Mutation-density comparison | Completed, exploratory | Eight tumour VCFs screened and overlapped with DAR sets after per-sample scaling |
| Final cell-of-origin model | Not supported yet | Current paired result does not justify a reliable H/I classifier |

## References

- Polak et al. *Cell-of-origin chromatin organization shapes the mutational landscape of cancer.*
- Xue et al. *Genomic and Transcriptomic Profiling of Combined Hepatocellular and Intrahepatic Cholangiocarcinoma Reveals Distinct Molecular Subtypes.*
- Kim et al. *Integrative analysis of single-cell RNA-seq and ATAC-seq reveals heterogeneity of induced pluripotent stem cell-derived hepatic organoids.*

## Notes

This repository is an active research workspace. Some scripts still contain local file paths and may require path updates before running on another machine.
