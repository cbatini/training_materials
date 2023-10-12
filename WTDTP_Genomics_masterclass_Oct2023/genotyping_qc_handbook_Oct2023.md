# Genotyping QC   
### Chiara Batini  
### WT DTP Genomic data masterclass  
### 18/10-13/11/2023  

## Summary  

In this practical we will take raw genotyping data and 
perform the following QC steps:  




## Dataset

The data used here is a subset of re-sequencing data from *Saccharomyces cerevisciae*.
Characteristics of the experiment:  

* Yeast genome: 12.5 Mbp; 16 chromosomes
* Whole genome sequencing
* Paired-end reads, 108bp, one library, 2 lanes

## Getting the data  

A tar archive containing all the files needed for this practical (EBI_NGSBioinfo_Nov2022.tar) 
is available in the shared directory (/media/penelopeprime/GenomeBioinformaticsNov2022/Day1/).  
**Create a directory to use for this practical**, move into it using `cd` and copy the tar archive there using this command:  
```
cp /media/penelopeprime/GenomeBioinformaticsNov2022/Day1/EBI_NGSBioinfo_Nov2022.tar .
```

**Please DO NOT work in the shared directory. If unsure, use `pwd` to check in which directory you are.**  
You can then open this file using the command:  
```
tar -xvf EBI_NGSBioinfo_Nov2022.tar
```

You should now find a folder called **VariantCalling** containing read data 
(in subfolders lane1, lane2), a reference genome (Saccharomyces_cerevisiae.EF4.68.dna.toplevel.fa), 
a file with the coordinates of the yeast mtDNA (mito.intervals), and a file 
with the sequences of some of Illumina primers and adapters (primers_adapters.fa).  

-----
:question: :question: :question: :question: **Questions:**  

* Can you recognize the read data? 
* Which is read 1 and which is read 2?   
-----

