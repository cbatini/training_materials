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

:question: :question: :question: :question: **Questions:**  

* Can you recognize the read data? 
* Which is read 1 and which is read 2?   


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


:question: :question: :question: :question: **Questions**  

* What is the total number of sequences in each of the paired end fastq files? Do both lanes provide the same sequencing output (same number of reads)?
* What type of encoding is used in the fastq files?
* What is the length of the sequences in the fastq files? Is the length the same for both lanes?
* How is the quality of the reads across the pair? Is one read better than the other?
* Is there any issue with the data?

## Use Trimmomatic to remove primer and adapter sequences   

**Trimmomatic** is a java tool for performing a range of trimming tasks 
on Illumina paired end and single end read data. The manual can be found [here](https://github.com/usadellab/Trimmomatic/blob/main/README.md). 

Go back to the overrepresented sequences module of FastQC. 
This is where FastQC would tell you if a significant proportion (>1%) 
of your reads are contaminated with adapter sequences. As you can see from 
the *Possible Source* column, FastQC has found a number of reads contaminated 
with different Illumina primer and adapter sequences, so we will run Trimmomatic to remove them.  

Trimmomatic needs a fasta file containing the sequences you want to trim from your data; 
this can be created by using fasta files provided that contain the standard Illumina adapter 
and primer sequences or it can be customised. In this case we will use the prepared 
`primers_adapters.fa` file that contains the sequences indicated in FastQC and their 
reverse complement sequences.  

for lane1
```
trimmomatic PE -phred33 -threads 1 -trimlog \
lane1/trimm_logfile lane1/s-7-1.fastq lane1/s-7-2.fastq \
lane1/s-7-1.paired.fastq lane1/s-7-1.unpaired.fastq \
lane1/s-7-2.paired.fastq lane1/s-7-2.unpaired.fastq \
ILLUMINACLIP:primers_adapters.fa:2:30:10 MINLEN:36
```

for lane2
```
trimmomatic PE -phred33 -threads 1 -trimlog \
lane2/trimm_logfile lane2/s-7-1.fastq lane2/s-7-2.fastq \
lane2/s-7-1.paired.fastq lane2/s-7-1.unpaired.fastq \
lane2/s-7-2.paired.fastq lane2/s-7-2.unpaired.fastq \
ILLUMINACLIP:primers_adapters.fa:2:30:10 MINLEN:36
```


The parameters used in this command are defined as follows:  
| option | meaning |
|-----------------------------------|---------------------------------------------------------------|
| PE | data is paired end |
| -phred33 | quality scores are 33 offset |
| -threads 1 | number of threads to use |
| -trimlog lane1/trimm_logfile | name of logfile for summary information |
| lane1/s-7-1.fastq | name of input fastq file for left reads |
| lane1/s-7-2.fastq | name of input fastq file for right reads |
| lane1/s-7-1.paired.fastq | paired trimmed output fastq file for left reads |
| lane1/s-7-1.unpaired.fastq | unpaired trimmed output fastq file for left reads |
| lane1/s-7-2.paired.fastq | paired trimmed output fastq file for right reads |
| lane1/s-7-2.unpaired.fastq | unpaired trimmed output fastq file for right reads |
| ILLUMINACLIP | parameters for the adapter clipping |
|   primers_adapters.fa | text file of adapter sequences to search for |
|   :2:30:10 | adapter-read alignment settings - see manual for explanation |
|   MINLEN:36 | delete reads trimmed below length MINLEN |

:question: :question: :question: :question: **Questions**  
* According to the Trimmomatic screen output, 
what is the number and percentage of read pairs that ‘both survived’ adapter trimming?  
* How many pairs of reads have been trimmed and then deleted by Trimmomatic in this step?      


## Use Trimmomatic to trim low quality bases
The FastQC *Per Base Sequence Quality* module has already told us that there could be 
some issues with the quality scores of the last few bases of the reads, especially for 
read2 in both lanes. We will use Trimmomatic to trim poor quality bases from the 3’ 
end of the reads. Trimmomatic also checks the 5’ end for poor quality bases. 
The command below will carry out the trimming on the adapter trimmed fastq files we created above.

for lane1
```
trimmomatic PE -phred33 -threads 1 -trimlog \
lane1/trimm_logfile2 \
lane1/s-7-1.paired.fastq lane1/s-7-2.paired.fastq \
lane1/s-7-1.trim.paired.fastq lane1/s-7-1.trim.unpaired.fastq \
lane1/s-7-2.trim.paired.fastq lane1/s-7-2.trim.unpaired.fastq \
LEADING:3 TRAILING:3 SLIDINGWINDOW:4:15  MINLEN:36
```

for lane2
```
trimmomatic PE -phred33 -threads 1 -trimlog \
lane2/trimm_logfile2 \
lane2/s-7-1.paired.fastq lane2/s-7-2.paired.fastq \
lane2/s-7-1.trim.paired.fastq lane2/s-7-1.trim.unpaired.fastq \
lane2/s-7-2.trim.paired.fastq lane2/s-7-2.trim.unpaired.fastq \
LEADING:3 TRAILING:3 SLIDINGWINDOW:4:15  MINLEN:36
```




The parameters used in this command are defined as follows:  
| option | meaning |
|-----------------------------------|---------------------------------------------------------------|
| PE | data is paired end |
| -phred33 | quality scores are 33 offset |
| -threads 1 | number of threads to use |
| -trimlog lane1/trimm_logfile2 | name of logfile for summary information |
| lane1/s-7-1.paired.fastq | name of input adapter trimmed left fastq file |
| lane1/s-7-2.paired.fastq | name of input adapter trimmed right fastq file |
| lane1/s-7-1.trim.paired.fastq | paired trimmed output fastq file for left reads |
| lane1/s-7-1.trim.unpaired.fastq | unpaired trimmed output fastq file for left reads |
| lane1/s-7-1.trim.paired.fastq | paired trimmed output fastq file for right reads |
| lane1/s-7-1.trim.unpaired.fastq | unpaired trimmed output fastq file for right reads |
| LEADING:3 | Trim 5’ bases with quality score < 3 |
| TRAILING:3 | Trim 3’ bases with quality score < 3 |
| SLIDINGWINDOW:4:15 | sliding window trimming - see manual for explanation |
| MINLEN:36 | delete reads trimmed below length MINLEN |



:question: :question: :question: :question: **Questions** 
Check the final fastq files (lane*/s-7-1.trim.paired.fastq and lane*/s-7-2.trim.paired.fastq) 
with FastQC before proceeding to alignment.























