# NGS short-read data: from quality control to variant calling for SNPs and indels.  
### Chiara Batini  
### Genome bioinformatics: from short- to long-read sequencing  
### 17-21 November 2025  

## Summary  

Over the next two days, during the practical sessions, we will take an Illumina paired end dataset 
and perform the following steps:  

* initial data QC  
* removal of adapter and primer sequences and low quality bases/reads  
* alignment to a reference sequence  
* BAM refinement   
* BAM QC and visualisation  
* SNPs and short indels variant calling and filtering   


## Dataset

The data used here is a subset of re-sequencing data from *Saccharomyces cerevisiae*.  
Characteristics of the experiment:  

* Yeast genome: 12.5 Mbp; 16 chromosomes
* Whole genome sequencing
* Paired-end reads, 108bp, one library, 2 lanes

## Getting the data  

A tar archive containing all the files needed for this practical (EBI_NGSBioinfo_yeast.tar) 
is available in the home directory (/home/training/chiara/archive/).  
**Create a directory to use for this practical**, move into it using `cd` and copy the tar archive there using this command:  
```
cp /home/training/chiara/archive/EBI_NGSBioinfo_yeast.tar .
```

**Please DO NOT work in the shared directory. If unsure, use `pwd` to check in which directory you are.**  
You can then open this file using the command:  
```
tar -xvf EBI_NGSBioinfo_yeast.tar
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
Work your way through the analysis modules on the left hand side of the FastQC window/html, 
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
* What type of encoding is used in the fastq files?
* What is the length of the sequences in the fastq files? Is the length the same for both lanes?
* How is the quality of the reads across the pair? Is one read better than the other?
* Is there any issue with the data?
----
## Use Trimmomatic to remove primer and adapter sequences   

**Trimmomatic** is a java tool for performing a range of trimming tasks 
on Illumina paired end and single end read data. The manual can be found [here](https://github.com/usadellab/Trimmomatic/blob/main/README.md). 

Go back to the *Overrepresented sequences* module of FastQC. 
This is where FastQC would tell you if a significant proportion (>1%) 
of your reads are contaminated with adapter sequences. As you can see from 
the *Possible Source* column, FastQC has found a number of reads contaminated 
with different Illumina primer and adapter sequences, so we will run Trimmomatic to remove them.  

Trimmomatic needs a fasta file containing the sequences you want to trim from your data; 
this can be created by using fasta files [provided](https://github.com/usadellab/Trimmomatic/tree/main/adapters) that contain the standard Illumina adapter and primer sequences or it can be customised. In this case we will use the file I prepared `primers_adapters.fa` that contains the sequences indicated in FastQC and their reverse complement sequences.    

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
|   :2:30:10 | adapter-read alignment settings - see [manual](https://github.com/usadellab/Trimmomatic/blob/main/README.md) for explanation |
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
trimmomatic PE -phred33 -threads 1 -trimlog \
lane1/trimm_logfile2 \
lane1/s-7-1.paired.fastq lane1/s-7-2.paired.fastq \
lane1/s-7-1.trim.paired.fastq lane1/s-7-1.trim.unpaired.fastq \
lane1/s-7-2.trim.paired.fastq lane1/s-7-2.trim.unpaired.fastq \
LEADING:3 TRAILING:3 SLIDINGWINDOW:4:15 MINLEN:36
```

for lane2
```
trimmomatic PE -phred33 -threads 1 -trimlog \
lane2/trimm_logfile2 \
lane2/s-7-1.paired.fastq lane2/s-7-2.paired.fastq \
lane2/s-7-1.trim.paired.fastq lane2/s-7-1.trim.unpaired.fastq \
lane2/s-7-2.trim.paired.fastq lane2/s-7-2.trim.unpaired.fastq \
LEADING:3 TRAILING:3 SLIDINGWINDOW:4:15 MINLEN:36
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
| SLIDINGWINDOW:4:15 | sliding window trimming - see [manual](https://github.com/usadellab/Trimmomatic/blob/main/README.md) for explanation |
| MINLEN:36 | delete reads trimmed below length MINLEN |


-----
:question: :question: :question: :question: **Questions**  

* Check the final fastq files (lane*/s-7-1.trim.paired.fastq and lane*/s-7-2.trim.paired.fastq) 
with FastQC before proceeding to alignment.
-----

## Alignment to a reference genome  
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
samtools faidx Saccharomyces_cerevisiae.EF4.68.dna.toplevel.fa
```

**bwa index**
```
bwa index -a is Saccharomyces_cerevisiae.EF4.68.dna.toplevel.fa
```
`-a is`	Sets the algorithm to be used to construct a suffix array to the IS 
linear-time algorithm. It requires 5.37N memory where N is the size of the database. 
IS is moderately fast, but does not work with database larger than 2GB. 
IS is the default algorithm due to its simplicity. 
For the whole human genome you would need to use `-a bwtsw`. 

**picard dictionary**
```
picard CreateSequenceDictionary \
R=Saccharomyces_cerevisiae.EF4.68.dna.toplevel.fa \
O=Saccharomyces_cerevisiae.EF4.68.dna.toplevel.dict
```
----
:question: :question: :question: :question: **Questions**  

* Can you name the extensions of the files (e.g. ‘.txt’, ‘.sam’) 
that have been created for indices by each tool?
----

### 2. Align reads to the reference genome using BWA  
**BWA** uses the burrows wheeler algorithm to compress data and 
efficiently parse the reference for sequence matches. 
`bwa mem` is the most widely used bwa algorithm and is recommended for 
high-quality data as it is faster and more accurate than previous ones.  
There is also a newer and more efficient tool, bwa-mem2, that produces 
the same alignment as bwa-mem but it is up to 3X faster. 
We will not use it today, but you can find more information here:
[bwa-mem2 github page](https://github.com/bwa-mem2/bwa-mem2)    

**Align reads using bwa mem**   
```
bwa mem -R '@RG\tID:1\tLB:library\tPL:Illumina\tPU:lane1\tSM:yeast' \
Saccharomyces_cerevisiae.EF4.68.dna.toplevel.fa \
lane1/s-7-1.trim.paired.fastq lane1/s-7-2.trim.paired.fastq > lane1.sam
```

Options used (see [bwa manual](https://bio-bwa.sourceforge.net/bwa.shtml) for more options):  

* `-R`: Add read group. A read group is a set of reads that were 
generated from a single run/lane/chip of a sequencing instrument. 
By adding read groups to a bam file we are adding a line in the bam header specifying a number of tags.  

The read group tags used here are:  

| tag | meaning |  
| ------- | ----------------------------------- |  
| ID | read group ID/name |  
| LB | read group library |  
| PL | read group platform (e.g Illumina, Ion Torrent, etc) |  
| PU | read group platform unit (e.g. flowcell lane, chip barcode, etc) |  
| SM | read group sample name |  

You can find more information about read groups [here](https://gatk.broadinstitute.org/hc/en-us/articles/360035890671-Read-groups).     



**Convert the new sam file to bam format (bam is the binary version of the sam format)**
```
samtools view -b lane1.sam -o lane1.bam
```

Options used:  

* `-b`: output a bam file  
* `-o`: output file  

**Sort the bam file**
```
samtools sort lane1.bam -o lane1_sorted.bam
```

Samtools sorts alignments by their leftmost chromosomal coordinates.    

**Index the sorted bam for fast access**
```
samtools index lane1_sorted.bam
```
-----
:question: :question: :question: :question: **Questions**  

* Can you guess the extension of the index file?  
* Have a look at the header of your new bam file (`samtools view -H lane1_sorted.bam`)  
	+ How many chromosomes are present and which version of the SAM is it?  
	+ Can you see the read group line? Don't worry if not, or if this is confusing, we will look at it in detail soon.    
* Use unix command more on your SAM file and check what is after the header…  
-----


For lane 2 we will pipe the commands (but keep in mind that you don’t have to - 
you could just use the same commands as for lane 1, but changing the name of the files).

We are using:  

* the symbol `|` (pipe): it indicates that the output of the command before 
the symbol should be used as the input of the command after the symbol  
* the symbol `-` (dash): it indicates the output of the previous command should be read as standard input   
* the symbols `&&` (ampersand): it indicates to run a command only if the previous one exited successfully 

```
bwa mem -R \
'@RG\tID:2\tLB:library\tPL:Illumina\tPU:lane2\tSM:yeast' \
Saccharomyces_cerevisiae.EF4.68.dna.toplevel.fa \
lane2/s-7-1.trim.paired.fastq lane2/s-7-2.trim.paired.fastq | \
samtools view -b - | samtools sort - -o lane2_sorted.bam  && 
samtools index lane2_sorted.bam
```  

### 3. Merge BAM files per library  
As we move towards BAM refinement and variant calling, we will merge our two lanes of data 
into a single bam file for the whole library. For this we will use picard MergeSamFiles.  
```
picard MergeSamFiles INPUT=lane1_sorted.bam INPUT=lane2_sorted.bam OUTPUT=library.bam
```   
----
:question: :question: :question: :question: **Questions**  

* **Index the merged bam file** using samtools.  
* If you wanted/had to convert your library bam file into cram format, 
which samtools command would you use?  
----

# END OF DAY1

## BAM refinement  

Historically BAM refinement had three steps:  

1. local realignment  
2. base quality recalibration  
3. duplicate removal  

We will not run the first two steps in this practical, because:  

* **local realignment is not included anymore in the best practice by GATK** 
(and is not available as a tool in GATK 4) because the haplotype-based variant 
callers will take care of this issue during variant calling;  
* base quality recalibration is a slow process that takes time to run 
and requires a good list of variant sites for the species in consideration 
(e.g. for humans the latest version of dbSNP)  
You can find more information and command line examples [here](https://gatk.broadinstitute.org/hc/en-us/articles/360035890531-Base-Quality-Score-Recalibration-BQSR-).   
In the same page there is a paragraph (No excuses) with suggestions on how to proceed when 
you don’t have a good catalog of variant sites for your species.  


**Duplicate removal with picard**  
PCR duplicates may confound coverage estimates 
and amplify the effects of mis-calls. We will remove duplicates 
using picard MarkDuplicates. As you may have guessed, 
picard doesn’t actually remove the duplicates, 
but it marks the reads in the bam by using the bitwise flag.  
```
picard MarkDuplicates INPUT=library.bam OUTPUT=library_final.bam METRICS_FILE=dupl_metrics.txt
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
or you can produce an html report for your final bam using this command:  
```
qualimap bamqc -bam library_final.bam -outdir qualimap_report
```  

If you are interested about other options: http://qualimap.conesalab.org/doc_html/command_line.html   

-----
:question: :question: :question: :question: **Questions**  
Explore the different analyses that Qualimap has run and try and answer the 
following questions to understand the quality of your alignments.  

* What's the percentage of mapped reads?
* What's the percentage of duplicated reads? 
* What’s the average coverage? Is this equally distributed across the genome?
* What’s the fraction of the reference to have at least 2X coverage? and 4X?
* What’s the average mapping quality? Is this equally distributed across the genome?
-----

## BAM QC with individual software tools  
**If it is too late, do not worry about this part and proceed to BAM visualization.**   

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
samtools flagstat library.bam		
```

```
samtools flagstat library_final.bam	
```

----
:question: :question: :question: :question: **Questions**  

* Can you see any difference between samtools flagstat output before and after refinement?
----

**This part includes using R, do not proceed if you are not familiar with R**
 (otherwise pair up with someone who can use R).  
 
Look at the coverage per position in mitochondria.
```
samtools depth -a -r Mito:1-85779 library_final.bam > mito_coverage
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


## BAM visualisation  
You can extract the mitochondrial DNA from final the BAM to have a smaller file to visualise, or you cna try and use your final library bam file directly.
```
samtools view -bh -o mito.bam library_final.bam Mito
```  
----
:question: :question: :question: :question: **Questions**  

* Index the mito bam file  
* Do you know why we have used Mito to extract the mtDNA? Check your dictionary or your bam header.  
----

We will use IGV to visualise our alignment and, as it is installed on this virtual machine, you can just type:
```
igv.sh
```  

However, it is good to know that you can use the java web start version of IGV.  
Follow this link: https://igv.org/app/   
 
**In either case, be patient when you use IGV!**  

You will see that the default reference genome loaded is Human hg19. 
Load your reference genome (check Genomes - Load Genome from File) 
and then your bam file (check File - Load from File).  

You can try to load the bam file for the whole alignment 
(library_final.bam) but it may take some time.


## Variant calling  
Once the alignments have been refined, SNPs and INDELs differences 
between the data and the reference genome can be identified and qualified. 
GATK, samtools and Freebayes are popular softwares to carry out this analysis. 
Here we will use GATK 4.0 HaplotypeCaller.  

----
:question: :question: :question: :question: **Questions**  

* Are the reference index and dictionary in the same directory as the reference file?  
----
If you are starting your analyses directly from a bam file created by someone else, 
make sure you have the same reference genome they have used for the alignment. 
It is essential that the contigs in the reference are the same (in number, length and ID) 
to those used in the bam file.  


**GATK HaplotypeCaller**  
Call raw variants with no filters on chromosome 1.  

----
:question: :question: :question: :question: **Questions**  

* what is the ID for this chromosome?  
----
```
gatk HaplotypeCaller \
-R Saccharomyces_cerevisiae.EF4.68.dna.toplevel.fa \
-I library_final.bam \
-L I \
-O gatk_variants_raw_I.vcf 
```

Call raw variants on chromosome 1 using a filter of minimum base quality 20 and minimum mapping quality 50.  
```
gatk HaplotypeCaller \
-R Saccharomyces_cerevisiae.EF4.68.dna.toplevel.fa \
-I library_final.bam \
-L I \
-mbq 20 \
--minimum-mapping-quality 50 \
-O gatk_variants_raw_I_bq20_mq50.vcf 
```

Options used:  

* `-R`: path to the reference genome  
* `-I`:	path to input file  
* `-O`:	path (and name) to the output file  
* `-L`:	specify the genomic intervals on which to operate; 
you can use samtools-style intervals either explicitly on the command line 
(e.g. -L chr1 or -L chr1:100-200) or by loading in a file containing 
a list of intervals (e.g. -L myFile.intervals)  
* `-mbq`: minimum base quality required to consider a base for calling  
* `--minimum-mapping-quality`: minimum read mapping quality required to consider a read for calling  

----
:question: :question: :question: :question: **Questions**  

* what's the difference (if any) between the options for defining minimum BQ and MQ thresholds?  
* have a look at the vcf file and familiarise yourself with it  
* how many variants have been called in each case? 
Can you explain the difference? (hint: you can use `grep` and options to count the variants in the vcf files)  
* check the GATK HaplotypeCaller [user manual](https://gatk.broadinstitute.org/hc/en-us/articles/360037225632-HaplotypeCaller) to see other options.  
---

Now we are going to extract only SNPs from both vcf files.  
```
gatk SelectVariants \
-R Saccharomyces_cerevisiae.EF4.68.dna.toplevel.fa \
--variant gatk_variants_raw_I.vcf \
-O gatk_variants_raw_I_SNP.vcf \
--select-type SNP 
```
----
:question: :question: :question: :question: **Questions**  

* Repeat this command on the second vcf file.  
* Can you tell how many INDELs were called in each case?  
* And has the BQ20+MQ50 filter changed this picture?   
----

**FreeBayes**  
We have seen that the BQ20+MQ50 filter has an impact on the variants called, 
so we will use this filter for the variant calling with FreeBayes.  

Call raw variants using a filter of minimum base quality 20 and minimum mapping quality 50.  
```
freebayes \
-q 20 \
-m 50 \
-u \
-f Saccharomyces_cerevisiae.EF4.68.dna.toplevel.fa library_final.bam \
-r I > freebayes_variants_raw_I_bq20_mq50.vcf
```

Options used:  
                                                                                                                                                                                                                                                                 
* `-f`: path to the reference genome  
* `-q`: exclude alleles from analysis if their supporting 
base quality is less than the number specified (20 in our case)  
* `-m`: exclude alignments from analysis if they have 
a mapping quality less than the number specified

----
:question: :question: :question: :question: **Questions**  

* Check the FreeBayes [user manual](https://github.com/freebayes/freebayes) to see other options. 
	+ Can you tell why we have used the `-u` option?  
	+ **Keep in mind:** This was a pragmatic choice for this practical, however, 
as per FreeBayes manual page, **Users are strongly cautioned against using these, 
because removing this information is very likely to reduce detection power**.  
* Select SNPs from the FreeBayes vcf file using gatk SelectVariants.  
----

**Compare the GATK HaplotypeCaller and the FreeBayes vcf files**
```
vcftools \
--vcf gatk_variants_raw_I_bq20_mq50_SNP.vcf \
--diff freebayes_variants_raw_I_bq20_mq50_SNP.vcf \
--diff-site \
--out compare
```

Use the `more` command to view the compare file.  
The file column headers are as follows: CHROM POS1 POS2 IN_FILE REF1 REF2 ALT1 ALT2. 
The 4th column (IN_FILE) indicates if the SNP was found 
in both vcf files (B) or the first vcf file (1) or the second vcf file(2).  

-----
:question: :question: :question: :question: **Questions**  

* How many SNPs are present in both gatk and freebayes?  
* How many are present only in one of the two?  
* The compare.log file may be useful (do not worry about the warnings though). See if you can also get the answer from using a `grep` command (hint: option `-w` may be useful here; do you agree?).  
-----



## Variant filtering  

# Filtering using depth of coverage  
We will use VCFtools to filter variants from our vcf files. 
The aim of VCFtools is to provide easily accessible methods 
for working with complex genetic variation data in the form of VCF files. 
It allows to filter vcf files as well as to manipulate them in many useful ways.  

We will apply the following filters in two different ways:  

* d=3: minimum coverage 3  
* Q=20: minimum QUAL 20  

**This uses DP in the INFO field!**  
```
cat gatk_variants_raw_I_bq20_mq50_SNP.vcf | vcf-annotate -f d=3/Q=20 > gatk_variants_I_flt.vcf
```

Now use vcftools again with the same filters but with a different command.  

**This uses DP in the FORMAT field!**  
```
vcftools \
	--vcf gatk_variants_raw_I_bq20_mq50_SNP.vcf \
	--minDP 3 \
	--minQ 20 \
	--out gatk_variants_I_flt2 \
	--recode \
	--recode-INFO-all
```

As it is working on the data per sample, it substitutes the individual genotypes 
with missing data if they don't pass the filter [./. instead of 0/0 or 0/1 or 1/1].  

----
:question: :question: :question: :question: **Questions**  

* Have a look at the first few variants to understand the different outputs 
of these two commands. 
	+ Can you see any difference?  
	+ Can you appreciate how the two filtering commands behaved?  
* How many variants passed the filters in each case?  
	+ Can you explain the difference?   
	+ To answer this question it may be helpful to exclude 
	variants with missing data for the second output using this command:
	```
	vcftools \
	--vcf gatk_variants_I_flt2.recode.vcf \
	--max-missing 1 \
	--out gatk_variants_I_flt2_nomissing \
	--recode \
	--recode-INFO-all
	```
	+ Can you understand what this command does?  
----

# Filtering using more annotations  
We have seen that GATK reccommends using VQSR if you have enough samples.  
If not, there are some reccommendations using hard-filtering, 
including several annotations.  

Here an example of how to do this, includign a simple bash command to extract the information from the FILTER field.  

```
gatk VariantFiltration \
   -R Saccharomyces_cerevisiae.EF4.68.dna.toplevel.fa \
   -V gatk_variants_raw_I_bq20_mq50.vcf \
   -O gatk_variants_raw_I_bq20_mq50.filtered.vcf \
   --filter-name "QD" \
   --filter-expression "QD < 2.0" \
   --filter-name "MQ" \
   --filter-expression "MQ < 40.0" \
   --filter-name "FS" \
   --filter-expression "FS > 60.0" \
   --filter-name "SOR" \
   --filter-expression "SOR > 3.0" \
   --filter-name "MQRankSum" \
   --filter-expression "MQRankSum < -12.5" \
   --filter-name "ReadPosRankSum" \
   --filter-expression "ReadPosRankSum < -8.0"  
   
   
cat gatk_variants_raw_I_bq20_mq50.filtered.vcf | \
	grep -v '#' | \
	cut -f 7 | \
	sort | \
	uniq -c
```



**EXTRA**  
There are a couple of extra activities you may want to try at this point:  

* you could filter your vcf files in a different way; you could use additional metrcis, or a subset of metrics, or different thresholds for the same metrics, with vcftools or gatk:  
	+ What would you do differently? Discuss this with those around you.  
* you could run the extra exercise explained below  
	
	

## Extra exercise commands  


A tar archive containing all the files needed for this exercise (extra_exercise.zip) 
is available [here](https://drive.google.com/file/d/16b48OPq-uKcs1tLlPEjpn-qvVrkMu7dO/view?usp=share_link).  
**Create a directory to use for this practical**, move into it using `cd` and copy the zipfile there using this command:  
```
cp /home/training/Downloads/extra_exercise.zip .
```

**Please DO NOT work in the shared directory. If unsure, use `pwd` to check in which directory you are.**  
You can then unzip this file using the command:  
```
unzip extra_exercise.zip
```

You should now find a folder called **extra_exercise** containing read data 
(in subfolder fastq) and a reference genome (rCRS.fa).  

Here below a list of commands that can help you.  


-----
:question: :question: :question: :question: **Questions**  

* Can you follow what each command does and why it is needed? Use a text editor to comment the commands.    
* Have I missed anything out? Would you do anything differently?  
* Can you appreciate the difference between this approach and what we did when using yeast data?  
-----


```
cd extra_exercise 

samtools faidx rCRS.fa

bwa index -a is rCRS.fa 

picard CreateSequenceDictionary \
R=rCRS.fa \
O=rCRS.dict



ls fastq/ | cut -f 1 -d "_" | uniq > sample_names

mkdir bam_files

for sample in $(cat sample_names)
do
	bwa mem -R '@RG\tID:'"$sample"'\tLB:library\tPL:Illumina\tPU:lane\tSM:'"$sample"'' \
	rCRS.fa \
	fastq/${sample}_R1.fastq.gz fastq/${sample}_R2.fastq.gz | \
	samtools view -b - | samtools sort - -o bam_files/${sample}_sorted.bam  && 

	samtools index bam_files/${sample}_sorted.bam 
done



mkdir gvcf_files

for sample in $(cat sample_names)
do
	gatk HaplotypeCaller  \
		-R rCRS.fa \
		-I bam_files/${sample}_sorted.bam  \
		-O gvcf_files/${sample}.g.vcf.gz \
		-ERC GVCF
done

gatk GenomicsDBImport \
	-V gvcf_files/NA06994.g.vcf.gz \
	-V gvcf_files/NA07048.g.vcf.gz \
	-V gvcf_files/NA07357.g.vcf.gz \
	-V gvcf_files/NA10851.g.vcf.gz \
	-V gvcf_files/NA11829.g.vcf.gz \
	-V gvcf_files/NA11831.g.vcf.gz \
	-V gvcf_files/NA11992.g.vcf.gz \
	-V gvcf_files/NA11994.g.vcf.gz \
	-V gvcf_files/NA12003.g.vcf.gz \
	-V gvcf_files/NA12043.g.vcf.gz \
	-V gvcf_files/NA12144.g.vcf.gz \
	-V gvcf_files/NA12154.g.vcf.gz \
	-V gvcf_files/NA12155.g.vcf.gz \
	-V gvcf_files/NA12707.g.vcf.gz \
	-V gvcf_files/NA12716.g.vcf.gz \
	-V gvcf_files/NA12750.g.vcf.gz \
	-V gvcf_files/NA12812.g.vcf.gz \
	-V gvcf_files/NA12814.g.vcf.gz \
	-V gvcf_files/NA12872.g.vcf.gz \
	-V gvcf_files/NA12874.g.vcf.gz \
    --genomicsdb-workspace-path my_database \
	--intervals NC_012920.1_rCRS

gatk GenotypeGVCFs \
	-R rCRS.fa \
	-V gendb://my_database \
	-O all_samples.vcf.gz



gatk VariantFiltration \
   -R rCRS.fa \
   -V all_samples.vcf.gz \
   -O all_samples.filered.vcf.gz \
   --filter-name "QD" \
   --filter-expression "QD < 2.0" \
   --filter-name "MQ" \
   --filter-expression "MQ < 40.0" \
   --filter-name "FS" \
   --filter-expression "FS > 60.0" \
   --filter-name "SOR" \
   --filter-expression "SOR > 3.0" \
   --filter-name "MQRankSum" \
   --filter-expression "MQRankSum < -12.5" \
   --filter-name "ReadPosRankSum" \
   --filter-expression "ReadPosRankSum < -8.0"  
```

















