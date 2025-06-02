# About aladdin-shotgun pipeline 
This pipeline is designed to conduct taxonomy profiling and diversity analysis on metagenome shotgun sequencing data. Please find the sample report [here](https://zymo-research.github.io/pipeline-resources/reports/shotgun_sample_report.html).

## Source of the pipeline
This pipeline is originally adapted from community-developed [nf-core/taxprofiler pipeline](https://github.com/nf-core/taxprofiler) version 1.0.0. [Zymo Research](https://www.zymoresearch.com) made significant contributions in the adaption effort. This mainly include the addition of downstream analyses such as diversity analysis, addition of sourmash as the profiler, addition of antimicrobial resistance gene identification and the improvement of the report.

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

## Default pipeline parameters

#### Trimming (fastp)
`fastp` automatically detects and trims adapter sequences. Reads shorter than **15bp** after trimming are discarded.

#### Low complexity filtering (BBDuk)
`BBDuk` discards any read that has less than **0.3** in average entropy in a sliding window of **50bp**.

#### Taxonomic classification (sourmash)
`sourmash sketch` is run with the parameter of `k=31,scaled=1000,abund`. `sourmash gather` is run with the parameter of `--ksize 31 --threshold-bp 5000`. This means that **k-mers of length 31** are profiled at 1:1000 downsampling with abundance of each k-mer consdiered. A match/genome will only be reported if the estimated overlap between a sample and the reference genome is at least **5000bp**.

#### Sourmash database
By default, the database option `sourmash-zymo-2024` include the following reference genomes.
1. [GTDB R08-RS214 genomic representatives](https://sourmash.readthedocs.io/en/latest/databases.html#gtdb-r08-rs214-genomic-representatives-85k)
2. [Genbank viral genomes(2022)](https://sourmash.readthedocs.io/en/latest/databases.html#genbank-viral)
3. [Genbank archael genomes(2022)](https://sourmash.readthedocs.io/en/latest/databases.html#genbank-archaeal)
4. [Genbank protozoa genomes(2022)](https://sourmash.readthedocs.io/en/latest/databases.html#genbank-protozoa), with one genome GCA_002893375.1 removed due to contamination
5. [Genbank fungi genomes(2022)](https://sourmash.readthedocs.io/en/latest/databases.html#genbank-fungi)
6. A list of 29 genomes realted to [ZymoBIOMICS Microbial Community Standards](https://www.zymoresearch.com/collections/zymobiomics-microbial-community-standards)
7. A list of 16 common host genomes such as human, mouse, rat, etc. These genomes are included for host reads removal purposes. Their reads are not included downstream analysis. 

#### K-mer to read approximation
We estimate the numbers of reads belonging to each microbial genome by multiplying `f_unique_weighted` values of that genome from `sourmash` to the total number of reads after trimming and low complexity filtering.

#### Sample filtering criteria
The following samples not removed from downstream analysis at various steps:
1. Samples with a file size <1 kb after trimming and read length filtering
2. Samples with no identified microbial genomes by sourmash after host genome removal
3. Samples with less than 1,000 total reads assigned to non-host genomes
4. Samples that fail to produce a sourmash result after 3 tries with increased compute resources, up to 180GB of memory. This rarely happens and only happens to samples with extremely complex profile.
