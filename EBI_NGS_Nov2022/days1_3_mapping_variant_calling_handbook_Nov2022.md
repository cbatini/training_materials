# NGS data: from initial QC to a ready-to-analyse variant set.  
### Chiara Batini  
### Genome bioinformatics: resequencing and variant calling  
### 14-18 November 2022  

## Summary  

Over the next three days, during the practical sessions, we will take an Illumina paired end data set and perform the following steps:  

* initial data QC 
* removal of adapter and primer sequences and low quality bases/reads from the data set
* alignment to a reference sequence
* BAM refinement 
* BAM QC and visualisation
* variant calling and filtering   


## Dataset

The data used here is a subset of re-sequencing data from *Saccharomyces cerevisciae*.
Characteristics of the experiment:  

* Yeast genome: 12.5 Mbp; 16 chromosomes
* Whole genome sequencing
* Paired-end reads, 108bp, one library, 2 lanes

## Getting the data  

A tar archive containing all the files needed for this practical (EBI_NGSBioinfo_Nov2022.tar) 
is available in the shared directory (/media/penelopeprime/GenomeBioinformaticsNov2022/Day1/).  
Create a directory to use for this practical, move into it using `cd` and copy the tar archive there using this command:  
```
cp /media/penelopeprime/chiara/EBI_NGSBioinfo_Feb2021.tar .
```

**Please DO NOT work in the shared directory. If unsure, use `pwd` to check in which directory you are.**  
You can then open this file using the command:  
```
tar -xvf EBI_NGSBioinfo_Feb2021.tar
```

You should now find a folder called **VariantCalling** containing read data 
(in subfolders lane1, lane2), a reference genome (Saccharomyces_cerevisiae.EF4.68.dna.toplevel.fa) 
and a file with the sequences of some of Illumina primers and adapters (primers_adapters.fa).  

:question: **Question:** Can you recognize the read data? Which is read 1 and which is read 2?   


## Assess the quality of the data using FastQC
**FastQC** is a quality control tool for high throughput sequence data.  

You can launch FastQC with a graphical interface as follows:  
```
fastqc &
```
and upload your fastq files by using the File tab.  

> NOTE: The ‘&’ at the end of the command puts FastQC into the background, 
this means that the FastQC process will run independently of the shell, 
leaving the terminal free and allowing you to continue to use it.  

Otherwise you can use the command line interface and make an html report of the results as follows:  
```
fastqc <fastq_input_file>
```

Work your way through the analysis modules on the left hand side of the FastQC window/html, 
using the FastQC documentation, familiarize yourself with what each module is showing. 
Pay particular attention to the modules in the list below as they 
will help direct the downstream trimming and adapter removal steps:  

* Basic Statistics
* Per Base Sequence Quality
* Per Sequence Quality Scores
* Overrepresented Sequences


:question: **Questions**  

* What is the total number of sequences in each of the paired end fastq files? Do both lanes provide the same sequencing output (same number of reads)?
* What type of encoding is used in the fastq files?
* What is the length of the sequences in the fastq files? Is the length the same for both lanes?
* How is the quality of the reads across the pair? Is one read better than the other?
* Is there any issue with the data?
























