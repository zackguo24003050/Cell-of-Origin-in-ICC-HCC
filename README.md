# Cell-of-Origin in ICC-HCC

This repository contains the complete analysis workflow for a first-year
summer research project on cell-of-origin inference in combined hepatocellular
carcinoma and intrahepatic cholangiocarcinoma (cHCC-ICC).

The HTML reports should be downloaded and opened locally to view their code and
knitted results. The later stages are provided as R Markdown source files.

**Project period:** June 2025 - September 2025<br>
**Supervisor:** Dr. Gladys Poon, Research Assistant Professor, School of Biomedical Sciences, HKUMed<br>
**Analysis status:** workflow completed<br>
**Main question:** can the distribution of somatic mutations relative to
hepatocyte- and biliary-associated chromatin regions help identify the likely
cell of origin of cHCC-ICC tumour components?

## Project Overview

Somatic mutations are not uniformly distributed across the genome. Chromatin
accessibility and other cell-type-specific genomic features can influence where
mutations accumulate. This project therefore asked whether the mutation
profiles of hepatocellular-like (H) and intrahepatic cholangiocarcinoma-like
(I) tumour components differ relative to hepatocyte and biliary chromatin.

The project began by integrating liver organoid single-cell RNA-seq and
single-cell ATAC-seq data. The scRNA data were clustered and annotated, then
used as a reference for label transfer to the scATAC dataset. However, fragment
files were unavailable, so gene activity had to be approximated from the peak
count matrix. Together with moderate prediction confidence, weak agreement
with unsupervised ATAC clusters, and limited marker support, this made the
transferred labels unsuitable as a reliable cell-type-specific chromatin
reference.

The analysis therefore moved to bulk ATAC-seq, where hepatocyte and biliary
samples could be compared directly to define differential-accessibility regions
(DARs). These regions were converted to human coordinates and combined with
screened tumour SNVs to test whether mutation distributions differed relative
to hepatocyte- and biliary-associated chromatin. The final paired comparison
did not show a consistent signal, but it completed the full path from reference
construction and genomic data processing to statistical testing and biological
interpretation.

This was an introductory project rather than a definitive cell-of-origin
study. It provided practical experience with scRNA-seq, scATAC-seq, bulk
ATAC-seq, VCF processing, genomic intervals, statistical analysis, and
reproducible reporting. Re-examining the workflow also revealed weaknesses in
the original data and methods, which provided a much stronger foundation for
designing later work toward a publishable result.

```text
01 scRNA clustering and annotation
                 |
                 v
02 scATAC integration and label transfer
                 |
                 v
     Label-transfer support was insufficient
                 |
                 v
03 Bulk ATAC: hepatocyte vs biliary ----> lineage-associated DARs
                                                   |
04 Eight tumour VCFs ----> screened SNVs ----------+
                                                   |
                                                   v
05 TMB-scaled DAR mutation comparison and paired direction test
```

## Scientific Context

This project is motivated by three related ideas:

- Cell-of-origin chromatin organization can shape regional somatic mutation
  density across cancer genomes.
- cHCC-ICC contains hepatocellular and cholangiocytic differentiation, making
  its origin biologically ambiguous.
- Hepatocyte and biliary chromatin references provide genomic regions in which
  mutation depletion or enrichment can be compared between tumour components.

## Data Sources

The analysis uses or references:

- cHCC-ICC genomic data from the Cancer Cell study of combined hepatocellular
  and intrahepatic cholangiocarcinoma;
- eight tumour VCFs representing paired H and I components from four patients;
- bulk ATAC-seq count data comparing hepatocyte and biliary samples;
- liver organoid scRNA-seq and scATAC-seq data from HM and DM conditions; and
- mouse-derived DAR coordinates converted to the human genome by an external
  liftOver step.

Large raw files and intermediate analysis objects are not stored directly in
this repository.

## Analysis Workflow and Results

### 01. scRNA clustering and annotation

The scRNA-seq workflow reprocesses the HM and DM liver organoid samples,
normalizes the data, performs dimensionality reduction and clustering,
identifies marker genes, and assigns cell-type labels using the reference
study.

**Result:** an annotated scRNA object was generated and used as the reference
for scATAC label transfer.

### 02. scATAC integration and label transfer

The scATAC-seq workflow integrates four HM/DM samples using LSI and Harmony.
Because fragment files were not available, gene activity is approximated by
summing counts from peaks overlapping each gene body and its upstream region.
The scRNA-derived labels are then transferred to ATAC cells and assessed using
prediction scores, agreement with unsupervised ATAC clusters, and marker-gene
activity.

**Result:** the transfer produced labels, but prediction confidence was
moderate, concordance with unsupervised clusters was weak, and marker support
was limited. The scATAC annotation was therefore not used as the final
cell-type-specific chromatin reference, and the project moved to bulk ATAC-seq.

### 03. Bulk ATAC differential-accessibility analysis

The bulk ATAC workflow prepares the featureCounts peak matrix and compares
three hepatocyte samples with three biliary/BEC samples using DESeq2. Peaks are
classified using `padj < 0.05` and `|log2FoldChange| > 1`, joined back to their
genomic coordinates, and exported as lineage-associated BED files. The BED
start positions are converted from 1-based SAF coordinates to 0-based BED
coordinates before the external mouse-to-human liftOver step.

**Result:** the comparison identified 1,217 biliary-associated and 2,951
hepatocyte-associated peaks. The lifted human-coordinate files contained 1,020
biliary and 2,704 hepatocyte intervals and became the chromatin references for
the mutation analysis.

### 04. Tumour VCF QC and screening

The VCF workflow reads the eight `Com01-04H/I` tumour samples. All eight are
tumour samples; within each VCF, the tumour genotype is compared with its
matched-normal genotype. Because the supplied Mutect2 records have `FILTER=.`,
the workflow applies a documented screening rule to standard biallelic SNVs
using tumour and normal allele evidence, depth, mapping quality, base quality,
strand-bias information, and population-frequency annotations.

**Result:** the workflow generated one screened somatic-SNV table and a QC
summary for all eight samples. Depending on the sample, 1,419 to 44,394 SNVs
passed the screening criteria. A 1 Mb binned mutation-count table was also
generated for genome-wide distribution checks.

### 05. DAR mutation comparison

The final workflow overlaps each screened SNV with the human-coordinate
hepatocyte and biliary DARs. For each tumour, the two DAR mutation counts are
scaled by the sample's total screened mutation burden and expressed per 10,000
screened mutations. A second normalization accounts for the different total
lengths of the two DAR sets, and the resulting log-ratio is used as the lineage
score.

Under the chromatin-associated mutation-depletion hypothesis, an H sample
should have a lower score than its paired I sample. This direction is tested
within each of the four patients. Mix02T is not included, and no prediction or
validation model is fitted.

**Result:** two of four patient pairs followed the expected direction and two
showed the opposite direction. The paired sign-test result was `p = 1`, so the
current data do not show a consistent H-versus-I separation based on DAR
mutation density.

## Overall Conclusion

The project completed the full path from single-cell reference construction to
bulk chromatin comparison and tumour mutation analysis. The single-cell ATAC
labels were not sufficiently well supported to serve as the final reference,
but bulk ATAC-seq provided hepatocyte- and biliary-associated DAR sets for a
direct test of the hypothesis. After VCF screening and per-sample mutation
burden scaling, the paired tumour comparison did not show a consistent lineage
signal. The present analysis therefore does not support assigning H or I tumour
components to a cell of origin from these DAR mutation counts alone.

Although the biological result was inconclusive, the project was valuable as a
first complete bioinformatics workflow. It showed how limitations in reference
quality, variant calling, genomic coordinate handling, normalization, and
sample size propagate into the final interpretation. Those lessons informed
the design and quality standards of subsequent projects aimed at producing
publishable results.

## Limitations

- Only four paired patients were available, giving very low power for the
  paired direction test.
- The VCFs contain candidate-like Mutect2 records with `FILTER=.`. The manual
  screening rule cannot fully reproduce a standard Mutect2 filtering workflow.
- The samples appear to be whole-exome data, but no capture or callable-region
  BED was supplied. Scaling by total screened SNVs controls for overall sample
  mutation burden but is not equivalent to mutation rate per callable base.
- The bulk ATAC comparison contains three samples per group.
- The DARs originated in mouse data and were converted to human coordinates by
  an external liftOver step; genome-build and mapping provenance remain
  important sources of uncertainty.
- No independent sample is included for validation.

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

| Step | File | Purpose | Outcome |
|---:|---|---|---|
| 01 | `scRNA-clustering-and-annotating.html` | Cluster and annotate liver organoid scRNA-seq data | Annotated scRNA reference |
| 02 | `scATAC-integration.html` | Integrate scATAC samples and transfer scRNA labels | Label support was insufficient for the final chromatin reference |
| 03 | `03 bulk ATAC analysis.Rmd` | Compare hepatocyte and biliary bulk ATAC-seq and export DARs | Human-coordinate hepatocyte and biliary DAR references |
| 04 | `04 VCF QC and screening.Rmd` | Screen and summarize the eight paired-component tumour VCFs | Screened SNV and 1 Mb mutation-count tables |
| 05 | `05 DAR enrichment and classification.Rmd` | Scale DAR mutation counts and test the paired H/I direction | Expected direction in 2/4 pairs; no consistent separation |

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

## Analysis Status

| Module | Status | Main result |
|---|---:|---|
| scRNA reference annotation | Completed | Annotated reference generated |
| scATAC integration and label transfer | Completed | Transferred labels lacked strong independent support |
| Bulk ATAC comparison | Completed | Hepatocyte- and biliary-associated DARs defined |
| Tumour VCF screening | Completed | Screened SNV tables generated for eight tumour samples |
| DAR mutation comparison | Completed | Expected paired direction in 2/4 patients; sign-test `p = 1` |
| Cell-of-origin inference | Inconclusive | Current DAR mutation counts do not reliably separate H and I components |

## References

- Polak et al. *Cell-of-origin chromatin organization shapes the mutational
  landscape of cancer.*
- Xue et al. *Genomic and Transcriptomic Profiling of Combined Hepatocellular
  and Intrahepatic Cholangiocarcinoma Reveals Distinct Molecular Subtypes.*
- Kim et al. *Integrative analysis of single-cell RNA-seq and ATAC-seq reveals
  heterogeneity of induced pluripotent stem cell-derived hepatic organoids.*

## Notes

Some scripts contain local Windows file paths and require path updates before
running on another computer.
