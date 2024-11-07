# NGS data: from initial QC to a ready-to-analyse variant set.  
### Chiara Batini  
### Genome bioinformatics: from short- to long-read sequencing  
### 18-22 November 2024  

## Summary  

Over the next three days, during the practical sessions, we will look at the 3q29 region of the human 
genome. A deletion in this region is responsible for the Chromosome 3q29 microdeletion syndrome which is characterised by 
neurodevelopmental and/or psychiatric manifestations including mild-to-moderate intellectual disability, 
autism spectrum disorder, anxiety disorders, attention-deficit/hyperactivity disorder, 
executive function deficits, graphomotor weakness, and psychosis/schizophrenia. 
The region is characterised by two segmental duplication (SD) blocks that comprise different 
SD segments with >98% sequence identity and are therefore [prone to non-allelic homologous 
recombination](https://genomemedicine.biomedcentral.com/articles/10.1186/s13073-023-01184-5]).

You will use this dataset throughout the course, but this handbook covers:  

* initial data QC  
* removal of adapter and primer sequences and low quality bases/reads  
* alignment to a reference sequence  
* BAM refinement   
* BAM QC and visualisation   



## Datasets   

**Only for the QC session**, we will first look at a small yeast dataset:  

* Yeast genome: 12.5 Mbp; 16 chromosomes  
* Whole genome sequencing  
* 108bp paired-end reads  


**After performing quality control** on the yeast data we will look at 
the human data. We have extracted 5 samples from the second 30X 1000 Genomes Project 
sequencing effort. You can find information about the specifics of the sequencing 
[here](https://www.ebi.ac.uk/ena/browser/view/PRJEB36890) and about the overall effort 
[here](https://www.internationalgenome.org/data-portal/data-collection/30x-grch38).   

In a nutshell, this is:  
 
* Human genome, subset of chromosome 3: 1.8Mb
* Whole genome sequencing  
* 150bp paired-end reads, 2 lanes  
* expected average coverage: 30X  

Samples included here:  

HG01493	ERR3988888
HG03519	ERR3989175
HG03522	ERR2304566
HG03579	ERR3989180
HG03617	ERR3989186



## Getting the data  
### Getting the yeast data   

A tar archive containing all the files needed for this practical (EBI_NGSBioinfo_Nov2023.tar) 
is available in the home directory (/home/training/chiara/).  
**Create a directory to use for this practical**, move into it using `cd` and copy the tar archive there using this command:  
```
cp -r ~/Documents/SRS_fastq/ .
```

**Please DO NOT work in the shared directory. If unsure, use `pwd` to check in which directory you are.**  
You can then open this file using the command:  
```
tar -xvf EBI_NGSBioinfo_Nov2023.tar
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


### Getting the human data  



## Assess the quality of the data using FastQC
[**FastQC**](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/) 
is a quality control tool for high throughput sequence data.  

You can launch FastQC with a graphical interface as follows:  
```
fastqc &
```
and upload your fastq files by using the File tab.  

> NOTE: The ‘&’ at the end of the command puts FastQC in the background, 
this means that the FastQC process will run independently of the shell, 
leaving the terminal free and allowing you to continue to use it.  

Otherwise you can use the command line interface and make an html report of the results as follows:  
```
fastqc <fastq_input_file>
```
You will then be able to open the html report using `firefox <html_file> &`.  

**Upload, or process on the command line, the yeast fastq files** and 
work your way through the analysis modules on the left hand side of the FastQC window/html, 
using the [FastQC documentation](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/Help/3%20Analysis%20Modules/), 
familiarize yourself with what each module is showing. 
Pay particular attention to the modules in the list below as they 
will help direct the downstream trimming and adapter removal steps:  

* Basic Statistics
* Per Base Sequence Quality
* Per Sequence Quality Scores
* Overrepresented Sequences

----
:question: :question: :question: :question: **Questions**  

* What is the total number of sequences in each of the paired end fastq files? Do both lanes provide the same sequencing output (same number of reads)?
* What is the length of the sequences in the fastq files? Is the length the same for both lanes?
* How is the quality of the reads across the pair? Is one read better than the other?
* Is there any issue with the data?
* Can you write a loop in bash to process all fastq files?  
----

Now **explore one sample** in FastQC and compare to what you have observed so far:  
----
:question: :question: :question: :question: **Questions**  

* How does the data look like? Any similar pattern to the yeast data?  
* What is the length of the sequences in the fastq files?  
* Is there any issue with the data?
* Can you write a loop in bash to process all fastq files?  
----


## Use Trimmomatic to remove primer and adapter sequences   

**Trimmomatic** is a java tool for performing a range of trimming tasks 
on Illumina paired end and single end read data. The manual can be found [here](https://github.com/usadellab/Trimmomatic/blob/main/README.md). 

**Using the yeast dataset** go back to the overrepresented sequences module of FastQC. 
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
trimmomatic.sh PE -phred33 -threads 1 -trimlog \
lane1/trimm_logfile lane1/s-7-1.fastq lane1/s-7-2.fastq \
lane1/s-7-1.paired.fastq lane1/s-7-1.unpaired.fastq \
lane1/s-7-2.paired.fastq lane1/s-7-2.unpaired.fastq \
ILLUMINACLIP:primers_adapters.fa:2:30:10 MINLEN:36
```

for lane2
```
trimmomatic.sh PE -phred33 -threads 1 -trimlog \
lane2/trimm_logfile lane2/s-7-1.fastq lane2/s-7-2.fastq \
lane2/s-7-1.paired.fastq lane2/s-7-1.unpaired.fastq \
lane2/s-7-2.paired.fastq lane2/s-7-2.unpaired.fastq \
ILLUMINACLIP:primers_adapters.fa:2:30:10 MINLEN:36
```


The parameters used in this command are defined as follows:  
| option/argument | meaning |
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
----
:question: :question: :question: :question: **Questions**  

* According to the Trimmomatic output on the terminal, 
what is the number and percentage of read pairs that ‘both survived’ adapter trimming?  
* How many pairs of reads have been trimmed and then deleted by Trimmomatic in this step?      
----

## Use Trimmomatic to trim low quality bases
The FastQC *Per Base Sequence Quality* module has already told us that there could be 
some issues with the quality scores of the last few bases of the reads, especially for 
read2 in both lanes. We will use Trimmomatic to trim poor quality bases from the 3’ 
end of the reads. Trimmomatic also checks the 5’ end for poor quality bases. 
The command below will carry out the trimming on the adapter trimmed fastq files we created above.

for lane1
```
trimmomatic.sh PE -phred33 -threads 1 -trimlog \
lane1/trimm_logfile2 \
lane1/s-7-1.paired.fastq lane1/s-7-2.paired.fastq \
lane1/s-7-1.trim.paired.fastq lane1/s-7-1.trim.unpaired.fastq \
lane1/s-7-2.trim.paired.fastq lane1/s-7-2.trim.unpaired.fastq \
LEADING:3 TRAILING:3 SLIDINGWINDOW:4:15  MINLEN:36
```

for lane2
```
trimmomatic.sh PE -phred33 -threads 1 -trimlog \
lane2/trimm_logfile2 \
lane2/s-7-1.paired.fastq lane2/s-7-2.paired.fastq \
lane2/s-7-1.trim.paired.fastq lane2/s-7-1.trim.unpaired.fastq \
lane2/s-7-2.trim.paired.fastq lane2/s-7-2.trim.unpaired.fastq \
LEADING:3 TRAILING:3 SLIDINGWINDOW:4:15  MINLEN:36
```




The parameters used in this command are defined as follows:  
| option/argument | meaning |
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


-----
:question: :question: :question: :question: **Questions**  

* Check the final fastq files (lane*/s-7-1.trim.paired.fastq and lane*/s-7-2.trim.paired.fastq) 
with FastQC before proceeding to alignment.
* Is there any trimming that you think should be performed on the human dataset?  
-----

## Alignment to a reference genome  
From now on we will focus on the human dataset and for convenience we are using only chromosome 3 
as the reference we are mapping too, rather than the whole human genome.  

### 1. Create index and dictionary files of the reference genome using samtools, bwa and picard   

Indices are necessary for quick access to specific information in very large files. 
Here we will create indices for the Saccharomyces reference genome for tools we will 
use downstream in the pipeline. 
For example the samtools index file, `ref_name.fai`, stores records of sequence identifier, 
length, the offset of the first sequence character in the file, the number of characters per 
line and the number of bytes per line.  
 
-----
:question: :question: :question: :question: **Questions**  

* As you generate each index look at the files created using the `ls` command and options (e.g. `-lrth`)
-----
**samtools index**
```
samtools faidx chr3.fa
```

**bwa index**   
```
bwa index -a is chr3.fa
```
`-a is`	Sets the algorithm to be used to construct a suffix array to the IS 
linear-time algorithm. It requires 5.37N memory where N is the size of the database. 
IS is moderately fast, but does not work with database larger than 2GB. 
IS is the default algorithm due to its simplicity. 
For the whole human genome you would need to use the algorithm implemented in BWT-SW. 

**picard dictionary**
```
picard CreateSequenceDictionary \
R=chr3.fa \
O=chr3.dict
```
----
:question: :question: :question: :question: **Questions**  

* Can you name the extensions of the files (e.g. ‘.txt’, ‘.sam’) 
that have been created for indices by each tool?
----

### 2. Align reads to the reference genome using BWA  
**BWA** uses the burrows wheeler algorithm to compress data and 
efficiently parse the reference for sequence matches. 
`bwa mem` is the latest bwa algorithm and is recommended for 
high-quality data as it is faster and more accurate than previous ones.  

**Align reads using bwa mem**   

For this step we will use a small subset dataset from sample ERR2304566.  
```

bwa mem -R '@RG\tID:'ERR2304566'\tLB:library\tPL:Illumina\tPU:lane\tSM:'ERR2304566'' \
chr3.fa \
~/Documents/SRS_fastq/subset/ERR2304566_3q29_1M_final_subset_R1.fastq.gz \
~/Documents/SRS_fastq/subset/ERR2304566_3q29_1M_final_subset_R2.fastq.gz > ERR2304566.sam

```



Options used (see bwa manual for more options):  

* `-R`: Add read group. A read group is a set of reads that were 
generated from a single run/lane/chip of a sequencing instrument. 
By adding read groups to a bam file we are adding a line in the bam header specifying a number of tags.  

The read group tags used here are:  

| tag | meaning |  
| ------- | ----------------------------------- |  
| ID | read group ID |  
| LB | read group library |  
| PL | read group platform (e.g Illumina, Ion Torrent, etc) |  
| PU | read group platform unit (e.g. flowcell lane, chip barcode, etc) |  
| SM | read group sample name |  

You can find more information about read groups [here](https://gatk.broadinstitute.org/hc/en-us/articles/360035890671-Read-groups).     



**Convert the new sam file to bam format (bam is the binary version of the sam format)**
```
samtools view -b ERR2304566.sam -o ERR2304566.bam
```

Options used:  

* `-b`: output a bam file  
* `-o`: output file  


**Sort the bam file**
```
samtools sort ERR2304566.bam -o ERR2304566_3q29_1M_sorted.bam 
```

Samtools sorts alignments by their leftmost chromosomal coordinates.    

**Index the sorted bam for fast access**
```
samtools index ERR2304566_3q29_1M_sorted.bam
```
-----
:question: :question: :question: :question: **Questions**  

* Can you guess the extension of the index file?  
* Have a look at the header of your new bam file (`samtools view -H ERR2304566_3q29_1M_sorted.bam`)  
	+ How many chromosomes are present and which version of the SAM is it?  
	+ Can you see the read group line? Don't worry if not, or if this is confusing, we will look at it in detail soon.    
* Use unix command more on your SAM file and check what is after the header…  
-----


Now we will pipe the commands we have used above to avoid saving all the itnerim files. We are using:  

* the symbol `|` (pipe): it indicates that the output of the command before 
the symbol should be used as the input of the command after the symbol  
* the symbol `-` (dash): it indicates the output of the previous command should be read as standard input   
* the symbols `&&` (ampersand): it indicates to run a command only if the previous one exited successfully 

```
bwa mem -R '@RG\tID:'ERR2304566'\tLB:library\tPL:Illumina\tPU:lane\tSM:'ERR2304566'' \
chr3.fa \
~/Documents/SRS_fastq/subset/ERR2304566_3q29_1M_final_subset_R1.fastq.gz \
~/Documents/SRS_fastq/subset/ERR2304566_3q29_1M_final_subset_R2.fastq.gz | \
samtools view -b - | samtools sort - -o ERR2304566_3q29_1M_sorted.bam &&

samtools index ERR2304566_3q29_1M_sorted.bam
```  

# END OF DAY1

## BAM refinement  

BAM refinement has three steps:  

1. local realignment  
2. base quality recalibration  
3. duplicate removal  

We will not run the first two steps in this practical, because:  

* **local realignment is not included anymore in the best practice (by GATK)** 
(and is not available as a tool in GATK 4) because the haplotype-based variant 
callers will take care of this issue during variant calling;  
* base quality recalibration is a slow process that takes time to run 
and requires a good list of variant sites for the species in consideration 
(e.g. for humans the latest version of dbSNP)  
You can find more information and command line examples [here](https://gatk.broadinstitute.org/hc/en-us/articles/360035890531-Base-Quality-Score-Recalibration-BQSR-).   
In the same page there is a paragraph ("No excuses") with suggestions on how to proceed when 
you don’t have a good catalog of variant sites for your species.  


**Duplicate removal with picard**  
PCR duplicates may confound coverage estimates 
and amplify the effects of mis-calls. We will remove duplicates 
using picard MarkDuplicates. As you may have guessed, 
picard doesn’t actually remove the duplicates, 
but it marks the reads in the bam by using the bitwise flag.  
```

picard MarkDuplicates INPUT=ERR2304566_3q29_1M_sorted.bam \
OUTPUT=ERR2304566_3q29_1M_final.bam METRICS_FILE=ERR2304566_dupl_metrics.txt &&

samtools index ${sample}_3q29_1M_final.bam 


for sample in $(cat ../sample_list)
do 

picard MarkDuplicates INPUT=${sample}_3q29_1M_sorted.bam \
OUTPUT=${sample}_3q29_1M_final.bam METRICS_FILE=${sample}_dupl_metrics.txt &&

samtools index ${sample}_3q29_1M_final.bam 

done

```
----
:question: :question: :question: :question: **Questions**  

* Index library_final.bam file  
----

## BAM QC with qualimap  
**Qualimap** is a tool for quick BAM QC and it works with DNA-seq, 
RNA-seq and ChiP-seq data. It has a graphical interface which you can run by typing:  
```
qualimap &
```  

You can then proceed to upload your bam file by selecting a New Analysis in the File panel,    
or you can produce an html report for your final bam.

Here blow the command line for one sample:  
```
qualimap bamqc -bam ~/Documents/SRS_bwa_bam_files/ERR2304566_3q29_1M_final.bam -outdir ERR2304566_qualimap_report
```  

If you are interested about other options: http://qualimap.conesalab.org/doc_html/command_line.html   

-----
:question: :question: :question: :question: **Questions**  

* Can you write a loop to process all five samples?  

Explore the different analyses that Qualimap has run and try and answer the 
following questions for sample ERR2304566 to understand the quality of your alignment.  

* What's the percentage of mapped reads?
* What's the percentage of duplicated reads? 
* What’s the average coverage? Is this equally distributed across the genome?
* What’s the average mapping quality? Is this equally distributed across the genome?
* Is there anything strange about this sample?  
-----

You have probably noticed that some of the metrics look odd, and many of the positions have coverage zero.  
This is because our data comes from a subregion of chr3, but we haven't specified this when running qualimap.  
Let's first create a file with the b38 region coordinates.  
```
echo "chr3	195428934	197230596" > region3
```

And then let's re-run qualimap specifying the region.  
```
qualimap bamqc -bam ~/Documents/SRS_bwa_bam_files/ERR2304566_3q29_1M_final.bam -gff region -outdir ERR2304566_region
```


-----
:question: :question: :question: :question: **Questions**  

* Has anything changed?  
* And does it look better?  

Put particular attention to duplicate rate, coverage distribution, insert size.   
Please note that not all the plots focus on the region only, but might still use the whole chromosome 3.  

-----


**So, what do you think it is going on here? Let's explore this bam file with other tools to find out.**



## Calculating coverage with samtools  
We are now focusing on our region of interest and calculate the avergae coverage 
for each position.  
```
samtools depth -a -b region ~/Documents/SRS_bwa_bam_files/ERR2304566_3q29_1M_final.bam > ERR2304566_coverage_region
```
----
:question: :question: :question: :question: **Questions**  

* Add options `-q` and `-Q` to calculate the coverage per position using a 
minimum BQ 20 and a minimum MQ 50 (do you know how and where to specify this in the command line?).   
* How is quality affecting coverage distribution? Compare the outputs in R:   
	+ check average coverage  
	+ plot coverage per position  
	+ check how many positions have coverage 0, or equal to and greater than 4 
	(or any other threshold you are curious about)  
---- 
Options used:    
                                                                                                                                                                                                                                                             
* `-a`: Output all positions (including those with zero depth)  
* `-r`: Only report depth in specified region  
* `-q`: Only count reads with base quality greater than INT  
* `-Q`: Only count reads with mapping quality greater than INT


Use the code below in R.   

Load these two libraries in R at the beginning of the session.  
If ggpubr is unavailable just install it quickly before starting.    

```
library(tidyverse)
library(ggpubr)
```

Now let's look at the coverage distribution with and without qualit filters.  
```
cov <- read.table("ERR2304566_coverage_region", header=F) %>% 
	rename( position = V2,
		coverage = V3 ) 
		
hqcov <- read.table("ERR2304566_coverage_region_hq", header=F) %>% 
	rename( position = V2,
		coverage = V3 )

p1 <- cov %>%
	ggplot( aes( x = position, y = coverage)) + 
	geom_point() + 
	ylim(0,160000) + 
	ggtitle("Raw coverage")
	
p2 <- hqcov %>%
	ggplot( aes( x = position, y = coverage)) + 
	geom_point() + 
	ylim(0,160000) + 
	ggtitle("Coverage after filtering BQ20 and MQ50")
	
ggarrange(plotlist = list(p1,p2), ncol = 1, nrow = 2)
```
----
:question: :question: :question: :question: **Questions**  

* Looking at the plots, have the filters impacted the coverage much?  
* Use summary() in R to check the distribution in each table.  
---- 

Right, so we have some pretty extreme coverage in one region.  
Discuss with your neighbour what you would do at this point.  

  


<details>
  <summary>Or you can click below to find out more!</summary>
  
###test  
  
</details>

```
p1 <- cov %>% 
	filter( V3 < 300 ) %>%
	ggplot( aes( x=V2, y=V3)) + 
	geom_point() + 
	xlim(195428935,196890000)
	
p2 <- hqcov %>% 
	filter( V3 < 300 ) %>%
	ggplot( aes( x=V2, y=V3)) + 
	geom_point() + 
	xlim(195428935,196890000)
	
ggarrange(plotlist = list(p1,p2), ncol = 1, nrow = 2)

mapqual <- read.table("ERR2304566_mapping_quality", header=F)

mapqual %>%
	ggplot(aes(x=V2,y=V3) ) +
	geom_point()


mapqual %>%
	ggplot(aes(x=V3) ) +
	geom_histogram()


```




https://sarahpenir.github.io/bioinformatics/awk/calculating-mapping-stats-from-a-bam-file-using-samtools-and-awk/

samtools view ERR2304566_3q29_1M_final.bam | cut -f 3,4,5 | grep -v '*' | sort > ERR2304566_mapping_quality

## BAM QC with individual software tools 
What follows is one of the possible ways of doing some of the things 
you have seen in the plots produced by qualimap - some of the commands 
below might be useful when you want to check something specific about your bam.  

**Look at duplication metrics file from bam refinement**
```
gedit dupl_metrics.txt &
```
----
:question: :question: :question: :question: **Questions**  

* What's the percentage of duplicated reads?   
----

**Get samtools flagstat metrics**
```
samtools flagstat ~/Documents/SRS_bwa_bam_files/ERR2304566_3q29_1M_final.bam		
```

```
samtools flagstat library_final.bam	
```

----
:question: :question: :question: :question: **Questions**  

* Can you see any difference between samtools flagstat output before and after refinement?
----


## BAM visualisation  
We will extract mtDNA from final the BAM to have a smaller file to visualise.
```
samtools view -bh -o mito.bam library_final.bam Mito
```  
----
:question: :question: :question: :question: **Questions**  

* Index the mito bam file  
* Do you know why we have used Mito to extract the mtDNA? Check your dictionary or your bam header.  
----

We will use IGV to visualise our alignment and, as it is installed on this container, you can just type:
```
igv &
```  

However, it is good to know that you can use the java web start version of IGV.  
Follow this link: https://igv.org/app/   
 
**In either case, be patient when you use IGV!**  

You will see that the default reference genome loaded is Human hg19. 
Load your reference genome (check Genomes - Load Genome from File) 
and then your bam file (check File - Load from File).  

You can try to load the bam file for the whole alignment 
(library_final.bam) but it may take some time.









