# Cell-of-Origin in ICC-HCC

This repository contains analysis scripts for a summer research project on cell-of-origin inference in combined hepatocellular carcinoma and intrahepatic cholangiocarcinoma (cHCC-ICC).

The HTML reports should be downloaded and opened locally to view the full source document; they contain both the analysis code and the knitted results.

**Project period:** June 2025 - September 2025  
**Supervisor:** Dr. Gladys Poon, Research Assistant Professor, School of Biomedical Sciences, HKUMed  
**Current stage:** single-cell RNA/ATAC integration and quality assessment  
**Main question:** can genome-wide mutation-density patterns help infer the likely cell-of-origin of different cHCC-ICC components or subtypes?

## Project Overview

Cancer mutations are not uniformly distributed across the genome. Regional mutation density is influenced by chromatin organization, replication timing, and other cell-type-specific genomic features. This project applies that concept to cHCC-ICC, asking whether tumor mutation-density profiles are more consistent with hepatocyte-like, cholangiocyte-like, progenitor-like, or other liver-lineage chromatin states.

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
- Normal liver or hepatocyte epigenomic references, including histone modification and methylome tracks.
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

## Downstream Direction

Bulk ATAC and mutation-density analyses are part of the next stage of the project. Some scripts and preliminary outputs are already included, but this section is intentionally left open for later updates.

To be added:

- bulk ATAC processing summary
- hepatocyte versus biliary DAR definition
- mutation-density binning strategy
- comparison between mutation density and chromatin/DAR regions
- final cell-of-origin interpretation

## Repository Structure

```text
.
+-- scRNA clustering and annotating.Rmd
+-- scATAC integration.Rmd
+-- future bulk ATAC analysis file
+-- future mutation-density analysis file
+-- future integrative interpretation file
+-- README.md
```

## Main Analysis Files

| File | Purpose |
|---|---|
| `scRNA clustering and annotating.Rmd` | Reprocesses HM/DM liver organoid scRNA-seq data, annotates clusters, and creates the scRNA reference for ATAC label transfer |
| `scATAC integration.Rmd` | Integrates HM/DM scATAC-seq data, transfers scRNA labels, and evaluates label quality |
| Future bulk ATAC file | Placeholder for bulk ATAC processing and DAR-based chromatin reference analysis |
| Future mutation-density file | Placeholder for tumor mutation-density binning and comparison with chromatin references |
| Future integration file | Placeholder for final cell-of-origin interpretation |

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
| Bulk ATAC comparison | To be added | Will document hepatocyte- and biliary-associated chromatin references |
| Mutation-density comparison | To be added | Will document tumor mutation-density binning and comparison |
| Final cell-of-origin model | To be added | Will integrate mutation-density and chromatin evidence |

## References

- Polak et al. *Cell-of-origin chromatin organization shapes the mutational landscape of cancer.*
- Xue et al. *Genomic and Transcriptomic Profiling of Combined Hepatocellular and Intrahepatic Cholangiocarcinoma Reveals Distinct Molecular Subtypes.*
- Kim et al. *Integrative analysis of single-cell RNA-seq and ATAC-seq reveals heterogeneity of induced pluripotent stem cell-derived hepatic organoids.*

## Notes

This repository is an active research workspace. Some scripts still contain local file paths and may require path updates before running on another machine.
