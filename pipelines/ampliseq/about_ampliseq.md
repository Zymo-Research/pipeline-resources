---
layout: default
title: About page
parent: amplicon sequencing(16S, 18S, ITS)
nav_order: 1
---

# About ampliseq pipeline

This pipeline is designed to process microbial amplicon sequencing data, performing taxonomic assignment of the prokaryote 16S ribosomal RNA gene, the eukaryote 18S ribosomal RNA gene and the ITS (internal transcribed spacer). Supported data types include single- or paired-end Illumina and PacBio reads.

## Source of the pipeline

This pipeline was originally adapted from community-developed [nf-core/ampliseq pipeline](https://nf-co.re/ampliseq) version 2.4.0.

## What is in this pipeline

This pipeline was built using [Nextflow](https://www.nextflow.io/). A brief summary of pipeline:

1. Raw read QC ([`FastQC`](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/))
2. Read trimming using ([`Cutadapt`](https://journal.embnet.org/index.php/embnetjournal/article/view/200))
3. Infer Amplicon Sequence Variants (ASVs) ([`DADA2`](https://doi.org/10.1038/nmeth.3869))
4. Taxonomic classification using [`QIIME2`](https://www.nature.com/articles/s41587-019-0209-9)
5. Excludes unwanted taxa, produces absolute and relative feature/taxa count tables and plots, plots alpha rarefaction curves, computes alpha and beta diversity indices and plots thereof ([`QIIME2`](https://www.nature.com/articles/s41587-019-0209-9))
6. Calls differentially abundant taxa ([`ANCOM-BC`](https://www.nature.com/articles/s41467-020-17041-7))
7. Presents all QC and analysis results in a comprehensive report ([`MultiQC`](https://multiqc.info/))

For details, please find the source code [here](https://github.com/Zymo-Research/aladdin-ampliseq).
