# Cell Type of Origin in ICC-HCC

This repository contains the complete analysis workflow for a first-year
summer research project on cell type of origin inference in combined
hepatocellular carcinoma and intrahepatic cholangiocarcinoma (cHCC-ICC).

The five numbered HTML reports contain the complete code, output, figures, and
interpretation for each analysis stage. Download and open them locally for the
best viewing experience.

**Project period:** June 2025 - September 2025<br>
**Supervisor:** Dr. Gladys Poon, Research Assistant Professor, School of Biomedical Sciences, HKUMed<br>
**Analysis status:** workflow completed<br>
**Main question:** can the distribution of somatic mutations relative to
hepatocyte- and biliary-associated chromatin regions help identify the likely
cell type of origin of cHCC-ICC tumour components?

## Project Overview

Somatic mutations are unevenly distributed across the genome and can retain a
signal of the tumour's cell type of origin through cell-type-specific chromatin
accessibility. This project asked whether paired hepatocellular-like (H) and
intrahepatic cholangiocarcinoma-like (I) tumour components differ relative to
hepatocyte and biliary chromatin. Identifying their cell type of origin may help
distinguish biologically different tumour subtypes with different prognosis or
treatment response.

The project first integrated liver organoid scRNA-seq and scATAC-seq data.
Because fragment files were unavailable and the transferred labels had limited
support, bulk ATAC-seq was used instead to define hepatocyte- and
biliary-associated DARs. Within the 10% of 1 Mb bins showing the largest scaled
H-versus-I mutation difference, all four paired tumours followed the expected
relative mutation-depletion direction.

This first-year project was a small proof-of-concept success. It produced a
complete reproducible workflow and a consistent 4/4 paired pattern, while the
small sample size limited statistical power (`p = 0.125`). It also provided a
practical foundation for designing later publishable work.

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
05 Top-10% bins: DAR mutation-density score and paired direction test
```

## Scientific Context

This project is motivated by three related ideas:

- Accessible DNA regions tend to undergo more effective DNA repair and
  therefore accumulate fewer somatic mutations [1].
- Because chromatin accessibility is cell-type-specific, regional mutation
  density can retain information about a tumour's cell type of origin [2].
- cHCC-ICC contains hepatocellular and cholangiocytic differentiation, making
  its cell type of origin biologically ambiguous. Hepatocyte and biliary
  chromatin references make it possible to compare lineage-associated mutation
  depletion between its tumour components.

## Data Sources

The analysis uses or references:

- cHCC-ICC genomic data from the Cancer Cell study of combined hepatocellular
  and intrahepatic cholangiocarcinoma [3];
- eight tumour VCFs representing paired H and I components from four patients;
- bulk ATAC-seq count data comparing hepatocyte and biliary samples;
- liver organoid scRNA-seq and scATAC-seq data from HM and DM conditions [4].

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
genomic coordinates, and exported as lineage-associated BED files. 

**Result:** the comparison identified 1,217 biliary-associated and 2,951
hepatocyte-associated peaks. The lifted human-coordinate files contained 1,020
biliary and 2,704 hepatocyte intervals and became the chromatin references for
the mutation analysis.

### 04. Tumour VCF QC and screening

The VCF workflow reads the eight `Com01-04H/I` tumour samples. All eight are
tumour samples; within each VCF, the tumour genotype is compared with its
matched-normal genotype. Because the supplied Mutect2 records have `FILTER=.`,
the workflow applies a documented screening rule to standard biallelic SNVs
using tumour and normal read depth, alternate-allele support, allele fraction,
and the Mutect2 tumour log-odds score (`TLOD`). The criteria are tumour depth
`>= 30`, tumour alternate reads `>= 5`, tumour allele fraction `>= 0.08`, normal
depth `>= 15`, normal alternate reads `<= 1`, normal allele fraction `<= 0.01`,
and `TLOD >= 6.3`. The same thresholds are applied to every sample.

**Result:** the workflow generated one screened somatic-SNV table and a QC
summary for all eight samples. Depending on the sample, 1,419 to 44,394 SNVs
passed the screening criteria.

### 05. DAR mutation comparison

The final workflow divides chromosomes 1-22 and X into 3,053 1 Mb bins. Within
each sample, bin mutation counts are scaled per 10,000 screened SNVs. The H and
I group means are compared in each bin, and the 306 bins with the largest
absolute difference—the top 10%—are retained. Hepatocyte and biliary DARs are
then restricted to these bins.

For each tumour, mutation density is calculated separately in the restricted
hepatocyte and biliary DARs. The lineage score is the log2 ratio of
hepatocyte-DAR to biliary-DAR mutation density, with a 0.5 pseudocount. This
accounts for the different total lengths of the two DAR sets; scaling by total
screened mutation burden is equivalent within this within-sample ratio because
the common denominator cancels.

Under the chromatin-associated mutation-depletion hypothesis, an H sample
should have a lower score than its paired I sample. This direction is tested
within each of the four patients.

**Result:** all four patient pairs followed the expected direction: the H
component had a lower lineage score than its paired I component. The paired
differences (`I - H`) were 1.585, 1.585, 0.652, and 0.330. The two-sided sign
test was `p = 0.125`; the direction was completely consistent in this dataset,
but the four analysed pairs provide insufficient power for conventional
statistical significance.

## Overall Conclusion

The project completed the full path from single-cell reference construction to
bulk chromatin comparison and tumour mutation analysis. The single-cell ATAC
labels were not sufficiently well supported to serve as the final reference,
but bulk ATAC-seq provided hepatocyte- and biliary-associated DAR sets for a
direct test of the hypothesis. After VCF screening, mutation-burden scaling,
and top-bin selection, all four H/I pairs showed the predicted relative
mutation-depletion direction. The project can therefore be considered a small
proof-of-concept success: it recovered a coherent signal across the available
pairs and connected chromatin accessibility with regional tumour mutation
patterns in one reproducible workflow.

The result remains preliminary rather than definitive. With only four paired
patients, even 4/4 concordance gives a two-sided sign-test `p = 0.125`.
Nevertheless, this first complete bioinformatics project provided useful
biological evidence, practical experience across several genomic data types,
and a stronger foundation for later publishable work.

## Limitations

- Only four paired patients were analysed, giving very low power for the
  paired direction test; 4/4 concordance still yields `p = 0.125`.
- The input VCFs did not include variant-level filtering status (`FILTER=.`),
  so a common set of predefined quality thresholds was applied to all samples.
- The samples appear to be whole-exome data, but no capture or callable-region
  BED was supplied.
- Fragment files were unavailable, so the scATAC-derived clusters and labels
  were not used as the final chromatin reference. A complete single-cell
  analysis could potentially define cell-type-specific chromatin more precisely
  than the bulk comparison.
- The DARs were converted from mouse to human coordinates by liftOver, which
  may introduce mapping inaccuracies.

## Repository Structure

```text
.
+-- 01-scRNA-clustering-and-annotating.html
+-- 02-scATAC-integration.html
+-- 03-bulk-ATAC-analysis.html
+-- 04-VCF-QC-and-screening.html
+-- 05-DAR-enrichment-and-classification.html
+-- README.md
```

## Main Analysis Files

| Step | File | Purpose | Outcome |
|---:|---|---|---|
| 01 | [01-scRNA-clustering-and-annotating.html](01-scRNA-clustering-and-annotating.html) | Cluster and annotate liver organoid scRNA-seq data | Annotated scRNA reference |
| 02 | [02-scATAC-integration.html](02-scATAC-integration.html) | Integrate scATAC samples and transfer scRNA labels | Label support was insufficient for the final chromatin reference |
| 03 | [03-bulk-ATAC-analysis.html](03-bulk-ATAC-analysis.html) | Compare hepatocyte and biliary bulk ATAC-seq and export DARs | Human-coordinate hepatocyte and biliary DAR references |
| 04 | [04-VCF-QC-and-screening.html](04-VCF-QC-and-screening.html) | Screen and summarize the eight paired-component tumour VCFs | Screened SNV table and per-sample QC summary |
| 05 | [05-DAR-enrichment-and-classification.html](05-DAR-enrichment-and-classification.html) | Select the top 10% bins and compare paired DAR mutation-density scores | Expected direction in 4/4 pairs; sign-test `p = 0.125` |

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

## References

1. Polak P, Lawrence MS, Haugen E, et al. *Reduced local mutation density in
   regulatory DNA of cancer genomes is linked to DNA repair.* Nature
   Biotechnology. 2014;32:71-75. [doi:10.1038/nbt.2778](https://doi.org/10.1038/nbt.2778)
2. Polak P, Karlić R, Koren A, et al. *Cell-of-origin chromatin organization
   shapes the mutational landscape of cancer.* Nature. 2015;518:360-364.
   [doi:10.1038/nature14221](https://doi.org/10.1038/nature14221)
3. Xue R, Chen L, Zhang C, et al. *Genomic and Transcriptomic Profiling of
   Combined Hepatocellular and Intrahepatic Cholangiocarcinoma Reveals Distinct
   Molecular Subtypes.* Cancer Cell. 2019;35:932-947.e8.
   [doi:10.1016/j.ccell.2019.04.007](https://doi.org/10.1016/j.ccell.2019.04.007)
4. Kim J-H, Mun SJ, Kim J-H, et al. *Integrative analysis of single-cell RNA-seq
   and ATAC-seq reveals heterogeneity of induced pluripotent stem cell-derived
   hepatic organoids.* iScience. 2023;26:107675.
   [doi:10.1016/j.isci.2023.107675](https://doi.org/10.1016/j.isci.2023.107675)

## Notes

The reports preserve the local Windows file paths used in the original code;
reproducing the workflow on another computer requires updating those paths.
