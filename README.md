

### This code is associated with the paper from McCrone et al., "Stochastic processes constrain the within and between host evolution of influenza virus". eLife, 2018. http://dx.doi.org/10.7554/eLife.35962

# Influenza evolution within and between naturally occuring acute infections
This repository holds the analysis used in *citation,2018* it relies heavily on our other repository [variant_pipeline](https://github.com/lauringlab/variant_pipeline) and the [HIVEr R package] (https://github.com/jtmccr1/HIVEr).

#Overview
--------

    project
    |- README          # the top level description of content
    |
    |- data            # raw and primary data, are not changed once created
    |  |- reference/  # reference fasta files to be used in in sequence alignment
    |  |- raw/         # raw data, will not be altered. In practice this is where the raw fastq files go. They will need to be downloaded from the SRA. 
    |  |- process/     # cleaned data, will not be altered once created. During anlysis this includes intermediate files used in variant calling as well as consensus sequence processing
    |  |  |- secondary # The data made during the secondary analysis - everything after iSNV have been called.
    |- scripts/           # any programmatic code
    |  |- primary_analysis/    # The pbs scripts and option files used to process the sequencing data from fastq format to variant calls
    |  |- secondary_analysis/ 
    |  |  |- processing # R scripts used to do the heavily lifting finalizing variant calls, measuing genetic distance ect.
    |  |  |- Figures # R scripts used to make the figures in the paper
    |- results         # all output from workflows and analyses
    |  |- Figures/     #  manuscript figures
    |  |- *.Rmd  # exicutable R markdown file. In many cases these run the same analysis as that in the Figure*.R files but in a more exploritory manner - with more details 
    |  |- *ipynb and *py    # Python notebooks used to run a few steps of the analysis - identifying antigenic variants, calculating DnDs.
    +- Makefile        # executable Makefile for this study
    
  --------
# Dependencies    
The analysis expects the variant_pipeline repository to be your home directory. 
```
    cd ~/
    git clone https://github.com/lauringlab/variant_pipeline.git
```
This pipeline requires [fastqc](http://www.bioinformatics.babraham.ac.uk/projects/fastqc/), [bowtie2](http://bowtie-bio.sourceforge.net/bowtie2/index.shtml) and the R package [DeepSNV](https://www.bioconductor.org/packages/release/bioc/html/deepSNV.html).

More information on the dependencies needed to run the varaint\_pipline can be found [here](https://github.com/lauringlab/variant_pipeline)

The python code used in this analysis has been tested in version 2.7.11 (although it will probably work in other versions) and relies on the following modules. The version used for each module is also listed.

```
biopython -- 1.67
pandas -- 0.18.1
```


The consensus sequence analysis and antigenic analysis relies on [muscle](http://www.drive5.com/muscle/downloads.htm). It expects the exicutable to be named "muscle". Please change the path to this exicutable in the Makefile.

Also the R analysis relies heavily on the package HIVEr which contains functions that are commonly used in the analysis. That can be found [here](https://github.com/jtmccr1/HIVEr) and can be installed using devtools. 

# Reproducing the analysis

Reproducing the analysis involves 4 steps. 1) Downloading the fastq files from the SRA 2) Primary analsysis - Calling iSNV in each sample using the variant_pipeline repository referenced above, 3) Secondary analysis - maniputating the data and running the models 4) making the figures. All the intermediate files needed to run the secondary analysis are included in this repository. The Makefile can be used to run the secondary analysis. 

Due to space limitations, the raw fastq files and intermediate bam files are not included here but can be remade using the commands below.

Please note that in many places we refer to the genomic segment "NA" as "NR". This is to avoid complications in R as "NA" is a special term. 


## Downloading raw data

Becasue we use a plasmid control to estimate the lane specific error rates used to identify iSNV it is important that the fastq files from the SRA are split into separate directories for each Hiseq lane. We can download these files using the download pipeling present in the variant_calling_pipeline repository and the SRR.*.csv files located in data/raw/SRR_files. 

There is a help option that gives useful information regarding this pipeline
```
python PATH/TO/VARIANT_CALLING_PIPELINE/bin/download.fastq.pipe.py -h
```


Below is an an example of how to download the fastq files from the HK_1 lane

```
python PATH/TO/VARIANT_CALLING_PIPELINE/bin/download.fastq.pipe.py data/raw/SRR_files/SRR.HK_1.csv SRR.HK_1.branches ./data/raw/HK_1/
```



## Processing the raw data

The commands used to process the data from fastq format to preliminary iSNV identification can be found in the PBS scripts and option files located in scripts/primary_analysis/. (Note paths in the files may need to be updated.)These commands launch the analyis pipeline from https://github.com/lauringlab/variant_pipeline. This pipeline is based in [bpipe](http://bpipe-test-documentation.readthedocs.io/en/latest/) and is smart enough to remember what commands have been run in the event of failure. It also logs the commands that were run("./data/processed/\*/commandlog.txt"). These steps are time and memory intensive. These commands output a number of large intermediate files. The important files that are used in down stream analysis are the concatenated variant calls "./data/processed/\*/all.sum.csv". The concatenated coverage files "./data/processed/\*/all.coverage.csv", and the consensus sequences "./data/processed/\*/parsed_fa/\*.fa". The -bam option in some of the pbs scripts starts the analysis after the samples have been aligned, sorted, and duplicate reads removed. To run from fastq files simply delete this option. More information regarding the pipeline can be found by running the command below or reading the more about the pipeline [here](https://github.com/lauringlab/variant_pipeline)


```
python PATH/TO/VARIANT_CALLING_PIPELINE/bin/variantPipeline.py -h
```

Here is an example of the command for processing files from the HK_1 hiseq lane.

```
python PATH/TO/VARIANT_CALLING_PIPELINE/bin/variantPipeline.py ./scripts/primary_analysis/HK_1.options.yaml 
```




## Secondary analysis

The secondary analysis is broken up into 2 separate stages. The first finalizes variant calls and runs the time and memory intensive steps. The second make the figures. These stages can be run using the files located in ./scripts/secondary\_analysis. We have also included a Makefile located in the main directory. Note we have renamed file names to match those submitted and the name changes may not be reflected in the make files. We plan to update the names in the Makefile once the paper is accepted and file names are finalized. The most robust way to reproduce the analysis is to run the scripts directly without the Makefile.

Some of the processing steps are memory and time intensive. They also process some of the steps in parrallel so those parts of the code may need to be updated to reflect your setup. Note that the secondary analysis requires the raw iSNV calls to create the qual.snv.csv file (The file that contains all final iSNV calls - both minor and major allele). Those files must be generated from the steps above; they are not included here. You can comment out that rule if you wish to run any of the other rules in the secondary analysis.


```
make secondary_analysis
```

Here is a rough map of dependencies for this analysis made using [make2graph] (https://github.com/lindenb/makefile2graph)

![alt text](./secondary.png)

Reproducing the figures is easier as all the dependent files are included.

```
make figures
```

![alt text](./figures.png)


_Currently there is an issue with save.plot from cowplot and using arial fonts. The current work around is to redownload cowplot before running each rule._
