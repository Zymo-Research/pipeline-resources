# About aladdin-shotgun pipeline 
This pipeline is designed to conduct taxonomy profiling and diversity analysis on metagenome shotgun sequencing data. Please find the sample report [here](https://zymo-research.github.io/pipeline-resources/reports/shotgun_sample_report.html).

## Source of the pipeline
This pipeline is originally adapted from community-developed [nf-core/taxprofiler pipeline](https://github.com/nf-core/taxprofiler) version 1.0.0. [Zymo Research](https://www.zymoresearch.com) made significant contributions in the adaption effort. This mainly include the addition of downstream analyses such as diversity analysis, addition of sourmash as the profiler, and the improvement of the report.

## What is in the pipeline
This pipeline is built using [Nextflow](https://www.nextflow.io/). A brief summary of pipeline:

1. Read QC ([`FastQC`](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/) or [`falco`](https://github.com/smithlabcode/falco) as an alternative option)
2. Performs optional read pre-processing (*code for long-read inherited from nf-core/taxprofiler, but not separately tested by us yet*)
   - Adapter clipping and merging (short-read: [`fastp`](https://github.com/OpenGene/fastp), [`AdapterRemoval2`](https://github.com/MikkelSchubert/adapterremoval); long-read: [`porechop`](https://github.com/rrwick/Porechop))
   - Low complexity and quality filtering (short-read: [`bbduk`](https://jgi.doe.gov/data-and-tools/software-tools/bbtools/), [`PRINSEQ++`](https://github.com/Adrian-Cantu/PRINSEQ-plus-plus); long-read: [`Filtlong`](https://github.com/rrwick/Filtlong))
3. Perform Host-read removal
   - Host-read removal (short-read: [`BowTie2`](http://bowtie-bio.sourceforge.net/bowtie2/); long-read: [`Minimap2`](https://github.com/lh3/minimap2)). This is not performed when `sourmash-zymo` is selected as the database because it already contains host sequences. 
   - Statistics for host-read removal ([`Samtools`](http://www.htslib.org/))
4. Run merging when applicable
5. Identifies antimicrobial resistance genes in samples from database [MEGARes version 3](https://academic.oup.com/nar/article/51/D1/D744/6830666)
   - Reads are aligned to MEGARes reference ([`bwa mem`](https://bio-bwa.sourceforge.net/bwa.shtml))
   - Resistome statistics are quantified and compiled for each sample ([`AMRplusplus`](https://github.com/Microbial-Ecology-Group/AMRplusplus/tree/master))
7. Performs taxonomic profiling using one of: (*nf-core/taxprofiler has more choices for this step, if there are tools you'd like for this step, please let us know.*)
   - [`sourmash`](https://github.com/sourmash-bio/sourmash)
   - [`MetaPhlAn4`](https://huttenhower.sph.harvard.edu/metaphlan/)
8. Merge all taxonomic profiling results into one table ([`Qiime2`](https://qiime2.org/)) and generate composition plot ([`Krona`](https://github.com/marbl/Krona))
9. Perform alpha/beta diversity analysis ([`Qiime2`](https://qiime2.org/)) and beta diversity differential expression ([`ANCOM-BC`](https://www.bioconductor.org/packages/release/bioc/html/ANCOMBC.html))
10. Present all results in above steps in a report ([`MultiQC`](http://multiqc.info/))
    
For details, please find the source code [here](https://github.com/Zymo-Research/aladdin-shotgun).

## Pipeline parameters: fastp trimming

The **adapter** is specified by the user. If no adapter is specified, fastp will automatically detect adapter sequences.

The **minimum length** filter is set by default to **15**.

## Pipeline parameters: BBDuk complexity filtering

BBDuk offers a **minimum entropy** read threshold option between 0 to 1, with 0 being a read with only one repeated base and 1 being completely random. The default for this pipeline is **0.3**.

Entropy is calcuated with a **sliding window** of **50** basepairs by default.

Sequences that do not meet entropy thresholds are **automatically discarded**.

## Pipeline parameters: Sourmash taxonomic classification

In this pipeline, sourmash uses a default **k-mer length** of **31**.

Sourmash generates a **scaled** MinHash from a provided reference with kmers sampled at 1 per **1000** basepairs.

Matches of k-mers to reference hashes are weighted by the **abundance** of the hash in the sample metagenome.

By default, sourmash matches k-mers to **database sourmash-zymo-2024**.

The minimum estimated **basepair overlap** between a sample and the reference for a match to be reported in sourmash is **5,000**.

## Sourmash default databases

## Pipeline filtering criteria

Under the following criteria, a sample may be dropped from the pipeline:

1. Samples with a file size <1 kb after trimming and read length filtering are excluded.
2. Samples with no identified microbial genomes by sourmash after host genome filtering are excluded.
3. Samples with less than 1,000 reads assigned to non-host organisms cannot pass filtering.
