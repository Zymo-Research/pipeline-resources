# About Aladdin Genomics pipeline
Aladdin Genomics is a Nextflow analysis pipeline designed to detect germline or somatic variants from whole genome or targeted sequencing data. The pipeline was originally adapted from **nf-core/sarek**, a workflow designed to detect variants in whole genome or targeted sequencing data. It focuses on human reference genomes and can handle tumor/normal pairs, as well as additional relapses. Please find the sample report [here](https://zymo-research.github.io/pipeline-resources/reports/aladdin_genomics_sample_report.html)

## Source of the pipeline
This pipeline was originally adapted from the community-developed [nf-core/sarek pipeline](https://github.com/nf-core/sarek) version 3.2.3. [Zymo Research](https://www.zymoresearch.com) made significant contributions to the adaptation effort. These contributions mainly include modifications of variant callers used for somatic and germline samples, the integration of [nf-pcgr](https://github.com/BarryDigby/nf-pcgr), and improvements to the report and its documentation.

## What is in the pipeline
This pipeline is built using [Nextflow](https://www.nextflow.io/). A brief summary of pipeline:

Depending on the options and samples provided, the pipeline can currently perform the following:

- Form consensus reads from UMI sequences ([`fgbio`](https://fulcrumgenomics.github.io/fgbio/))
- Sequencing quality control and trimming ([`FastQC`](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/), [`fastp`](https://github.com/OpenGene/fastp))
- Map Reads to Reference ([`BWA-mem2`](https://github.com/bwa-mem2/bwa-mem2))
- Process BAM file ([`GATK MarkDuplicates`](https://gatk.broadinstitute.org/hc/en-us/articles/360037052812-MarkDuplicates-Picard), [`GATK BaseRecalibrator`](https://gatk.broadinstitute.org/hc/en-us/articles/360035890531-Base-Quality-Score-Recalibration-BQSR), [`GATK ApplyBQSR`](https://gatk.broadinstitute.org/hc/en-us/articles/360036856671-ApplyBQSR))
- Summarise alignment statistics ([`samtools stats`](https://www.htslib.org/doc/samtools-stats.html), [`mosdepth`](https://github.com/brentp/mosdepth))
- Variant caller:

  - `main_caller`: Activate the main variant caller. If set to `germline`, [`DeepVariant`](https://github.com/google/deepvariant) is used to detect germline variants, applicable only for normal samples (status 0 in `metadata.csv`). If set to `somatic`, [`Mutect2`](https://gatk.broadinstitute.org/hc/en-us/articles/360037593851-Mutect2) is used to detect somatic variants, applicable only for tumor samples (status 1). If no Panel of Normals (PON) is provided, the default GATK PON is used.
  - `structural_caller`: Activate [`Manta`](https://github.com/Illumina/manta) to detect structural variants.
  - `copy_number_caller`: Activate [`CNVkit`](https://cnvkit.readthedocs.io/en/stable/) to detect copy number variations.
  - `msi_caller` : Activate [`MSIsensorpro`](https://github.com/xjtu-omics/msisensor-pro) to assess microsatellite instability (MSI) using paired tumor-normal samples (defined by status in 'metadata.csv').

- Variant filtering and annotation ([`Ensembl VEP`](https://github.com/Ensembl/ensembl-vep))
- Categorize Single Nucleotide Variants (SNVs) and Insertions/Deletions (InDels) detected in somatic ([PCGR](https://github.com/sigven/pcgr)) and germline samples ([CPSR](https://sigven.github.io/cpsr/))
- Summarise and represent QC ([`MultiQC`](http://multiqc.info/))


For details, please find the source code [here](https://github.com/Zymo-Research/aladdin-genomics).

## Citations
> Garcia M, Juhos S, Larsson M et al. **Sarek: A portable workflow for whole-genome sequencing analysis of germline and somatic variants [version 2; peer review: 2 approved]** _F1000Research_ 2020, 9:63 [doi: 10.12688/f1000research.16665.2](http://dx.doi.org/10.12688/f1000research.16665.2).

> **The nf-core framework for community-curated bioinformatics pipelines.**
>
> Philip Ewels, Alexander Peltzer, Sven Fillinger, Harshil Patel, Johannes Alneberg, Andreas Wilm, Maxime Ulysse Garcia, Paolo Di Tommaso & Sven Nahnsen.
>
> _Nat Biotechnol._ 2020 Feb 13. doi: [10.1038/s41587-020-0439-x](https://dx.doi.org/10.1038/s41587-020-0439-x).