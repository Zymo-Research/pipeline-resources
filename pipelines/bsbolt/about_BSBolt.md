---
layout: default
title: About page
parent: bisulfite sequencing (BSBolt)
nav_order: 1
---

# About BSBolt pipeline

This pipeline is designed to process bisulfite sequencing data. It is compatible with various library preparation methods such as targeted methylation sequencing, RRBS, or WGBS. It can help the user evaluate the quality of their samples or libraries, quantify reads, find differentially-methylated sites and regions, and more. Please find the sample report [here](https://zymo-research.github.io/pipeline-resources/reports/BSBolt_sample_report.html)

## Source of the pipeline

This pipeline is originally adapted from published [BiSulfite Bolt pipeline](https://github.com/NuttyLogic/BSBolt/releases) v1.5.0. [Zymo Research](https://www.zymoresearch.com) and Pacific Informatics made significant contributions in the adaption effort. This mainly include making a Nextflow pipeline running the BSBolt software, and the improvement of the report and its documentation.

## What is in the pipeline

This pipeline is built using [Nextflow](https://www.nextflow.io/). A brief summary of pipeline:

1. Raw read QC ([`FastQC`](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/))
2. Reads trimming ([`Cutadapt`](https://cutadapt.readthedocs.io/en/stable/))
3. Alignment ([`BiSulfite Bolt`](https://bsbolt.readthedocs.io/en/latest/align/))
4. Methylation calling ([`BiSulfite Bolt`](https://bsbolt.readthedocs.io/en/latest/methylation_calling/))
