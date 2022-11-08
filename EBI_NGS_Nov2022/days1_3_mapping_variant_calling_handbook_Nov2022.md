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
Create a directory to use for this practical, move into it using ```cd``` and copy the tar archive there using this command:  
```
cp /media/penelopeprime/chiara/EBI_NGSBioinfo_Feb2021.tar .
```

**Please DO NOT work in the shared directory. If unsure, use ```pwd``` to check in which directory you are.**  
You can then open this file using the command:  
```
tar -xvf EBI_NGSBioinfo_Feb2021.tar
```

You should now find a folder called **VariantCalling** containing read data 
(in subfolders lane1, lane2), a reference genome (Saccharomyces_cerevisiae.EF4.68.dna.toplevel.fa) 
and a file with the sequences of some of Illumina primers and adapters (primers_adapters.fa).  

****
Question: Can you recognize the read data? Which is read 1 and which is read 2?
****


