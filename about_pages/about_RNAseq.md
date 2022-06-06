# About bulk RNAseq pipeline 
This pipeline is designed to process bulk RNAseq data in general. It is compatible with various library preparation methods such as total RNAseq or mRNA sequencing. It can help the user evaluate the quality of their samples or libraries, quantify reads to genes, find differentially-expressed genes and pathways, and more. Please find the sample report [here](https://zymo-research.github.io/pipeline-resources/reports/RNAseq_sample_report.html)

## Source of the pipeline
This pipeline is originally adapted from community-developed [nf-core/rnaseq pipeline](https://github.com/nf-core/rnaseq) version 1.4.2. [Zymo Research](https://www.zymoresearch.com) made significant contributions in the adaption effort. This mainly include the addition of downstream analyses such as differential gene expression and pathway enrichment analysis, and the improvement of the report and its documentation.

## What is in the pipeline
This pipeline is built using [Nextflow](https://www.nextflow.io/). A brief summary of pipeline:

1. Raw read QC ([`FastQC`](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/))
2. UMI extraction when applicable ([`UMI-tools`](https://github.com/CGATOxford/UMI-tools))
3. Adapter trimming ([`Trim Galore!`](https://www.bioinformatics.babraham.ac.uk/projects/trim_galore/))
4. Alignment ([`STAR`](https://github.com/alexdobin/STAR))
5. UMI deduplication when applicable ([`UMI-tools`](https://github.com/CGATOxford/UMI-tools))
6. Sample level QC:
    * Various RNAseq related QC plots ([`RSeQC`](http://rseqc.sourceforge.net/))
    * 5' or 3' bias plot ([`Qualimap`](http://qualimap.bioinfo.cipf.es/))
    * Library complexity estimation ([`Preseq`](http://smithlabresearch.org/software/preseq/))
    * Mark duplicate reads ([`Picard`](https://broadinstitute.github.io/picard/))
    * Duplication rate QC ([`dupRadar`](https://bioconductor.org/packages/release/bioc/html/dupRadar.html))
    * Biotype composition plot ([`featureCounts`](http://bioinf.wehi.edu.au/featureCounts/))
    * ERCC spike-in plot ([[`STAR`](https://github.com/alexdobin/STAR) and [`featureCounts`](http://bioinf.wehi.edu.au/featureCounts/))
7. Read counting ([`featureCounts`](http://bioinf.wehi.edu.au/featureCounts/))
8. Sample comparison
    * Sample similarity matrix, sample MDS plot, heatmap of expression patterns of top genes ([`DESeq2`](https://bioconductor.org/packages/release/bioc/html/DESeq2.html)) 
    * Differential gene expression analysis ([`DESeq2`](https://bioconductor.org/packages/release/bioc/html/DESeq2.html))
    * Gene set enrichment analysis ([`g:Profiler`](https://biit.cs.ut.ee/gprofiler/gost))
9. Present all QC, analysis results in an interactive, comprehensive report ([`MultiQC`](http://multiqc.info/))

For details, please find the source code [here](https://github.com/Zymo-Research/aladdin-rnaseq).

## Citations
> **Comming soon ...**<br>
> A list of publications where this pipeline and its predecessors were used to analyze the data.