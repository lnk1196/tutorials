# Differential Gene Expression

## Trinity Documentation
[Trinity Documentation](https://github.com/trinityrnaseq/trinityrnaseq)

1. **Create conda environment**
    ```bash
    conda create -n <envr name>
    conda activate <envr name>
    ```

2. **Check raw sequence quality with fastQC**
    1. Enter directory with raw data
        ```bash
        cd <dir name>
        ```
    2. Create submission script
        ```bash
        nano <script name>
        ```
    3. Paste following and change as necessary
        ```bash
        #!/bin/bash
        #$ -S /bin/sh
        . /etc/profile
        #$ -cwd
        #$ -pe threads 50

        PATH="/home-dir/home/mbrown/software/fastqc"

        fastqc -o <output directory name> *.fastq.gz
        ```
    4. While in raw data folder, submit script to the cluster
        ```bash
        qsub <script name>
        ```

3. **Trim raw sequences with trimmomatic**
    1. Concatenate forward sequences together
        ```bash
        cat *R1* > forward.gz
        ```
    2. Concatenate reverse sequences together
        ```bash
        cat *R2* > reverse.gz
        ```
    3. Create submission script
        ```bash
        #!/bin/bash
        #$ -S /bin/sh
        . /etc/profile
        #$ -cwd
        #$ -pe threads 70

        PATH="/home-dir/home/mbrown/software/trinityrnaseq- v2.13.2:$PATH"

        Trinity --seqType fq --max_memory 100G  --CPU 70 --trimmomatic --quality_trimming_params "ILLUMINACLIP://home-dir/home/mbrown/software/trinityrnaseq-v2.13.2/Trimmomatic/adapters/NexteraPE-PE.fa:2:30:10 SLIDINGWINDOW:4:5 LEADING:5 TRAILING:5 MINLEN:25" --bflyHeapSpaceMax 100G --full_cleanup --output salt_trinity_out --left /home-dir/home/lkirsch/salt_experiment/foward.gz --right /home-dir/home/lkirsch/salt_experiment/reverse.gz
        ```
    4. Submit script

4. **Transcript quantification with RSEM**
    1. Create samples_ID.txt
        - Tab delimited
        - Columns => condition sample_ID full path to forward full path to reverse
        - Headers not needed
        - Need to list out each individual sample with the full path
    2. Create submission script
        ```bash
        #!/bin/bash
        #$ -S /bin/bash
        . /etc/profile
        #$ -cwd
        #$ -pe threads 50
        #$ -V

        align_and_estimate_abundance.pl --transcripts /home/lkirsch/salt_experiment/raw_data/Arcevulg-NoBact.outRef_Transcriptome.fas --est_method RSEM --aln_method bowtie2 --prep_reference --trinity_mode --samples_file salt_IDs.txt --seqType fq
        ```
    4. Build transcript and gene expression matrices
        ```bash
        <path to Trinity utils>/abundance_estimates_to_matrix.pl --est-method RSEM <list files> --gene_trans_map ‘none’
        ```
        - A ‘quant file’ listing all target files can be used, but sometimes doesn’t work
        - You can list each file, it’s annoying but will for sure work
        - HAVE TO SPECIFY NO GENE TRANS MAP
    5. Counting number of expressed genes
        1. Run in command line
            ```bash
            <Trinity path>/util/misc/count_matrix_features_given_MIN_TPM_threshold.pl genes_matrix.TPM.not_cross_norm | tee genes_matrix.TPM.not_cross_norm.counts_by_min_TPM
            ```
            and
            ```bash
            <Trinity path>/util/misc/count_matrix_features_given_MIN_TPM_threshold.pl trans_matrix.TPM.not_cross_norm | tee trans_matrix.TPM.not_cross_norm.counts_by_min_TPM
            ```
        2. Open R from command line
            ```R
            %R

            Plot(expressed gene counts, xlim = c(-100,0), ylim = c(0,100000), t=’b’)
            ```
    6. Filter transcripts based on expression value
        ```bash
        <Trinity path>/util/filter_low_expr_transcripts.pl –matrix <string> --transcripts <string> --trinity_mode
        ```

5. **QC Samples and Biological Replicates**
    1. Create a new samples.txt
        - Tab delimited
        - Columns => condition sample_ID
        - Header not needed

6. **Compare reps for each sample**
    ```bash
    <Trinity path>/Analysis/DifferentialExpression/PtR --matrix counts.matrix --samples samples.txt --log2 --CPM --min_rowSums 10 --compare_replicates
    ```

7. **Compare reps across samples**
    1. Create heat map
        ```bash
        <Trinity path>/Analysis/DifferentialExpression/PtR --matrix counts.matrix --min_rowSums 10 --s samples.txt --log2 --CPM --sample_cor_matrix
        ```
    2. Plot PCA
        ```bash
        <Trinity path>/Analysis/DifferentialExpression/PtR --matrix counts.matrix -s samples.txt --min_rowSums 10 --log2 --CPM --center_rows --prin_comp 3
        ```

## Differential Expression Analysis
9. **DE Analysis**
    1. Open R in command line
        ```R
        %R

        BiocManager::install(c(“edgeR”, “limma”, “DESeq2”, “ctc”, “Biobase”, “gplots”, “ape”, “argparse”))
        ```

    2. Run DESeq2 and edgeR
        ```bash
        <Trinity path>/Analysis/DifferentialExpression/run_DE_analysis.pl --matrix counts.matrix --method <edgeR | DESeq2> --samples_file samples.txt 
        ```
        - Use the 2nd samples.txt file you made with the conditions and IDs only
        - For edgeR, need to add --dispersion <float> argument
        - If you don’t use a samples.txt file, the program will compare every sample to every other sample

10. **Extract and cluster DE transcripts**
    ```bash
    <Trinity path>/Analysis/DifferentialExpression/analyze_diff_expr.pl --matrix <TMM.EXPR.matrix> --samples samples.txt
    ```
11. **Partition genes into clusters**
    ```bash
    <Trinity path>/Analysis/DifferentialExpression/define_clusters_by_cutting_tree.pl -R <string> --Ptree 60
    ```

## Annotation

## Enrichment
12. **For each of the DE output .subset file, create a .txt with only the transcript IDs**
    1. Open file in TextEdit
        - Right click file name -> open with -> TextEdit
    2. Command + shift + down arrow to highlight entire text
    3. Command + c to copy
    4. Open excel document and paste text
    5. Delete all but the first column with the transcript IDs
        - You can also remove the sampleA header
    6. Save as tab delimited .txt
        - You’ll need to add an additional line at the end
        - Make sure to name appropriately so you will know to what each file corresponds

13. **Format eggNOG annotation**
    1. Open eggNOG annotation with TextEdit
    2. Remove all text above the line beginning with “#query” and save
    3. Make sure you are in the directory with you annotation file and the transcript ID files
    4. Run format.py
        ```bash
        python format.py <filename> <filename.formatted>
        ```

    5. Open R in command line
        ```R
        %R

        Open cluster_profile.r in a separate IDE window
        ```
        - You will need to change this file to correspond with your data
            - Read each transcript ID file into R
            - Check enrichment
        ```R
        <variable> <- enricher(<filename>, pvalueCutoff = 0.05, pAdjustMethod = “fdr”, qvalueCutoff = 0.05, TERM2GENE = term2gene, TERM2NAME = NA)
        ```

    6. In R, install the ‘clusterProfiler’ library
        ```R
        library(“clusterProfiler”)
        ```
        - If the first time using, you will need to install from Bioconductor
        ```R
        if (!require(“BiocManager”, quietly = TRUE))
        install.packages(“BiocManager”)

        BiocManager::install(“clusterProfiler”)
        ```

    7. Run cluster_profile.r line by line in groups per transcript ID file
        - Readlines
        - Enricher
        - If enriched, write.table to save output
