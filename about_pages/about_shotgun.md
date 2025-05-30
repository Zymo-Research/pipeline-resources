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

## Pipeline parameters: Trimming
### `--shortread_qc_tool`
Options: 

         --shortread_qc_tool fastp

         --shortread_qc_tool adapterremoval
         
         --shortread_qc_tool DO_NOT_RUN
         
Specify which tool to use for short-read quality control. The tool chosen will remove adapters, trim low quality bases, remove reads that are too short, etc. Choose 'DO_NOT_RUN' if you don't want this step performed. Default is `--shortread_qc_tool fastp`.

### `--shortread_qc_skipadaptertrim`
Options:

         --shortread_qc_skipadaptertrim true

         --shortread_qc_skipadaptertrim false

Skip the removal of sequencing adapters. 

This often can be useful to speed up run-time of the pipeline when analysing data downloaded from public databases such as the ENA or SRA, as adapters should already be removed (however, we recommend to check FastQC results to ensure this is the case).

### `--shortread_qc_mergepairs`
Options:

         --shortread_qc_mergepairs true

         --shortread_qc_mergepairs false

Turn on the merging of read-pairs of paired-end short read sequencing data. 

> Modifies tool parameter(s):
> > - AdapterRemoval: `--collapse`
> > - fastp: `-m --merged_out`",

### `--shortread_qc_includeunmerged`   
Options:

         --shortread_qc_includeunmerged true

         --shortread_qc_includeunmerged false

Turns on the inclusion of unmerged reads in resulting FASTQ file from merging paired-end sequencing data when using `fastp` and/or `AdapterRemoval`. For `fastp` this means the unmerged read pairs are directly included in the output FASTQ file. For `AdapterRemoval`, additional output files containing unmerged reads are all concatenated into one file by the workflow.

Excluding unmerged reads can be useful in cases where you prefer to have very short reads (e.g. aDNA), thus excluding longer-reads or possibly faulty reads where one of the pair was discarded.

> Adds `fastp` option: `--include_unmerged`
         
### `--shortread_qc_adapter1`
Options:

         --shortread_qc_adapter1 [string]
         
Specify a custom forward or R1 adapter sequence to be removed from reads.

If not set, the selected short-read QC tool's defaults will be used.\n\n> Modifies tool parameter(s):
> - fastp: `--adapter_sequence`. fastp default: `AGATCGGAAGAGCACACGTCTGAACTCCAGTCA`
> - AdapterRemoval: `--adapter1`. AdapteRemoval2 default: `AGATCGGAAGAGCACACGTCTGAACTCCAGTCACNNNNNNATCTCGTATGCCGTCTTCTGCTTG`
         
### `--shortread_qc_adapter2` 
Options:

         --shortread_qc_adapter2 [string]

Specify a custom reverse or R2 adapter sequence to be removed from reads.

If not set, the selected short-read QC tool's defaults will be used.\n\n> Modifies tool parameter(s):
> - fastp: `--adapter_sequence`. fastp default: `AGATCGGAAGAGCGTCGTGTAGGGAAAGAGTGT`
> - AdapterRemoval: `--adapter1`. AdapteRemoval2 default: `AGATCGGAAGAGCGTCGTGTAGGGAAAGAGTGTAGATCTCGGTGGTCGCCGTATCATT`"

### `--shortread_qc_adapterlist`
Options:

         --shortread_qc_adapterlist [filepath string]

Allows to supply a file with a list of adapter (combinations) to remove from all files.

Overrides the --shortread_qc_adapter1/--shortread_qc_adapter2 parameters . 

For AdapterRemoval this consists of a two column table with a `.txt` extension: first column represents forward strand, second column for reverse strand. You must supply all possible combinations, one per line, and this list is applied to all files. See AdapterRemoval documentation for more information.

For fastp this consists of a standard FASTA format with a `.fasta`/`.fa`/`.fna`/`.fas` extension. The adapter sequence in this file should be at least 6bp long, otherwise it will be skipped. fastp trims the adapters present in the FASTA file one by one.\n\n> Modifies AdapterRemoval parameter: --adapter-list\n> Modifies fastp parameter: --adapter_fasta",

### `--shortread_qc_minlength`
Options:

         --shortread_qc_minlength [int]

Specifying a mimum read length filtering can speed up profiling by reducing the number of short unspecific reads that need to be match/aligned to the database.

> Modifies tool parameter(s):
> - removed from reads `--length_required`
> - AdapterRemoval: `--minlength`

### BBDuk parameters

### Sourmash parameters

## Sourmash default databases
