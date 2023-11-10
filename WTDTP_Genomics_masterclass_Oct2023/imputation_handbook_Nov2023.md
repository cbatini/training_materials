# Imputation   
### Chiara Batini  
### WT DTP Genomic data masterclass  
### 13/11/2023  

## Summary  
This tutorial will introduce you to:  

* pre-imputation QC
	+ genotyping QC  
	+ liftover from b37 to b38  
	+ pre-imputation QC tool from Will Rayner [here](https://www.well.ox.ac.uk/~wrayner/tools/)  
		- we will use it with the TOPMed reference panel  
* imputation to TOPMed    
* post-imputation QC  

## Dataset 

We are using the data from chr22 from the 1000 Genomes Project Phase1.  
These were stored in a vcf file which contained dosage values.  
I have converted the dosages to hard calls using plink2 with this command:
```
plink2 \
	--vcf ALL.chr22.phase1_release_v3.20101123.snps_indels_svs.genotypes.new_names.vcf.gz dosage=DS \
	--hard-call-threshold 0 \
	--make-bed \
	--out 1KGP_Ph1_chr22
```
You do not need to run this, it is just for your information.  
We are going to treat these data as if they were generated through genotyping.  
The data is in b37.  

-----
:question: :question: :question: :question: **Questions:**  

* what would you expect in the output here?   
-----

## Pre-imputation QC  

### Genotyping QC  
First thing first, let's perform the initial genotyping QC and remove variants that have:  

* more than 5% missing data  
* MAF < 1%
* a p value for HWE test < 10E-06

```
module load plink 

plink \
	--bfile 1KGP_Ph1_chr22 \
	--geno 0.05 \
	--maf 0.01 \
	--hwe 0.000001 \
	--make-bed \
	--out 1KGP_Ph1_chr22_qced 
```

-----
:question: :question: :question: :question: **Questions:**  

* how many variants were excluded for each criterion?  
* does this make sense to you knowing that we are using data imputed from sequencing data?  
* how many variants are left after the QC?  
* is there anything else you would add to this initial QC?       
-----


### Liftover of plink files from b37 to b38  
Usually liftover is performed on a list of variants, and for this you can use the UCSC liftOver tool 
either via the [command line](https://hgdownload.cse.ucsc.edu/admin/exe/linux.x86_64/) or 
via the web interface [here](https://genome.ucsc.edu/cgi-bin/hgLiftOver).  
The first option will allow you to input an unlimited number of variants, but 
to use it on the command line, you will need to download the appropriate chain file.  

Here we are going to use the [liftOverPlink tool](https://github.com/sritchie73/liftOverPlink) 
to liftover plink files from build 37 to 38. This tool makes use of the liftOver tool and it will need 
the appropriate chain file.  
The liftOverPlink tool works with python v2, so we need to create a conda environment to use it.  


```

module load plink
module load plink2

## if you haven't done it yet, create a conda environment with python2
conda create -n python2 python=2

## activate the conda environment with python2
conda activate python2

### Liftover from b37 to b38
## download liftOverPlink
git clone https://github.com/sritchie73/liftOverPlink.git

## download liftOver and make it executable
wget http://hgdownload.cse.ucsc.edu/admin/exe/linux.x86_64/liftOver
chmod u+x liftOver

## download the chain file b37 to b38
wget https://hgdownload.soe.ucsc.edu/gbdb/hg19/liftOver/hg19ToHg38.over.chain.gz


## liftover plink files from b37 to b38
# convert from bed to ped
plink2 \
--bfile 1KGP_Ph1_chr22_qced \
--export ped \
--out 1KGP_Ph1_chr22_qced

# liftover plink files
python liftOverPlink/liftOverPlink.py \
-m 1KGP_Ph1_chr22_qced.map \
-p 1KGP_Ph1_chr22_qced.ped \
-o 1KGP_Ph1_chr22_qced_b38 \
-c hg19ToHg38.over.chain.gz \
-e ./liftOver


###remove bad lifted variants
## be mindful, these are different from unlifted variants 
python liftOverPlink/rmBadLifts.py \
--map 1KGP_Ph1_chr22_qced_b38.map \
--out 1KGP_Ph1_chr22_qced_b38_lifted \
--log 1KGP_Ph1_chr22_qced_b38_lifted.dat

cut -f 2 1KGP_Ph1_chr22_qced_b38_lifted.dat > to_exclude.dat

plink \
--pedmap 1KGP_Ph1_chr22_qced_b38 \
--allow-extra-chr \
--make-bed \
--out 1KGP_Ph1_chr22_qced_b38_lifted \
--exclude to_exclude.dat

## deactivate the conda environment  
conda deactivate

```


-----
:question: :question: :question: :question: **Questions:**  

* how many variants were unlifted?  
* and how many were badly lifted?  
* did you manage to appreciate the difference?      
-----


### Pre-imputation QC using the tool from Will Rayner  
Will Rayner has written some very useful code that can be 
used to perform some QC steps before proceeding to imputation.  
From the webpage, the code:  

* Checks: Strand, alleles, position, Ref/Alt assignments and frequency differences  
In addition to the reference file, it requires the plink .bim and .frq files  
* Produces: A set of plink commands to update or remove SNPs based on the checks  
* Updates: Strand, position, ref/alt assignment  
* Removes: A/T & G/C SNPs if MAF > 0.4, 
SNPs with differing alleles, 
SNPs with > 0.2 allele frequency difference, 
SNPs not in reference panel  

You will need the site table that I prepared from chr22, you 
can download it [here]().

Here the details of how I prepared the chr22 site table, in case you 
want to know. 
**You don't need to do this!**
 
```
## download tool to create TOPMed site file
wget https://www.well.ox.ac.uk/~wrayner/tools/CreateTOPMed.zip

## download the TOPMed VCF file
This can be downloaded from here https://bravo.sph.umich.edu/freeze5/hg38/

####create TOPMed site file
vcftools 
	--gzvcf ALL.TOPMed_freeze5_hg38_dbSNP.vcf.gz \
	--chr chr22 \
	--recode \
	--recode-INFO-all \
	--out ALL.TOPMed_freeze5_hg38_dbSNP_chr22.vcf

perl CreateTOPMed.pl \
-i ALL.TOPMed_freeze5_hg38_dbSNP_chr22.vcf
```
Let's perform the pre-imputation QC with Will Rayner's tool now.  

```
###calculate frequencies for bf_b38 data
plink2 \
--bfile 1KGP_Ph1_chr22_qced_b38_lifted \
--freq \
--out 1KGP_Ph1_chr22_qced_b38_lifted_freq

### download WRayner tool  
wget https://www.well.ox.ac.uk/~wrayner/tools/HRC-1000G-check-bim-v4.3.0.zip

###perform WRayner pre imputation QC
perl \
HRC-1000G-check-bim.pl \
-b 1KGP_Ph1_chr22_qced_b38_lifted.bim \
-f 1KGP_Ph1_chr22_qced_b38_lifted_freq.afreq \
-r PASS.Variants.TOPMed_freeze5_hg38_dbSNP_chr22.tab.gz \
-c 22 \
-h

### run plink code - you can check it if you want to know what's going on
bash Run-plink.sh  
```

-----
:question: :question: :question: :question: **Questions:**  

* how many variants were excluded for each criterion?  
* how many vairants are left for imputation?        
-----




## Imputation to TOPMed  
I have run the imputation for chr22 on the [TOPMed Imputation Server](https://imputation.biodatacatalyst.nhlbi.nih.gov/#!).  
The TOPMed reference panel is not publicly available and this 
is the only way to use it for imputation. Similarly the Haplotype Reference Panel has its own webpage.  

First of all you need to prepare the vcf files, one per chromosome and bgzipped. 

```
####prepare vcf for imputation
sed -i -e "s/^22/chr22/" 1KGP_Ph1_chr22_qced_b38_lifted-updated-chr22.vcf
bgzip 1KGP_Ph1_chr22_qced_b38_lifted-updated-chr22.vcf
```

Then you can submit a job to the server through a very intuitive web interface.  


## Post-imputation QC  
Right so, the job run successfully on the imputation server and you can download 
the output files from [here]().

There are several files, but we will focus on:  

* chr22.info - containing information about the variants  
* chr22.dose.vcf - a vcf file with imputed data  


-----
:question: :question: :question: :question: **Questions:**  

* how many variants were imputed?            
-----

First we will do some checks in R to see how well the imputation worked.  

```
library(tidyverse)
library(ggpubr)
library(scales)

### read info file and topmed site file
### this takes ~2mins
info <- read_tsv("chr22.info.gz") %>%
	select(SNP, MAF, Rsq) %>%
	separate( SNP, into = c("CHR","POS","REF","ALT"), sep=":") %>%
	mutate( CHR = str_replace(CHR, "chr","")) %>%
	mutate_at( c("CHR","POS"), as.numeric)


topmed <- read_tsv("PASS.Variants.TOPMed_freeze5_hg38_dbSNP_chr22.tab.gz") %>%
	mutate(MAF.REF = case_when( AF <= 0.5 ~ AF,
			AF > 0.5 ~ 1 - AF)) %>%
	rename( CHR = `#CHROM` )

## plot histograms of Rsq values in different MAF groups
p1 <- info %>% 
	filter(MAF < 0.01) %>% 
	ggplot(aes(Rsq)) + 
	geom_histogram() + 
	ggtitle("MAF < 0.01") + 
	scale_y_continuous(labels = unit_format(unit = "M", scale = 1e-6)) 

p2 <- info %>% 
	filter(MAF >= 0.01 & MAF < 0.05) %>% 
	ggplot(aes(Rsq)) + 
	geom_histogram() + 
	ggtitle("0.01 <= MAF < 0.05") + 
	scale_y_continuous(labels = unit_format(unit = "M", scale = 1e-6))

p3 <- info %>% 
	filter(MAF >= 0.05) %>% 
	ggplot(aes(Rsq)) + 
	geom_histogram() + 
	ggtitle("MAF >= 0.05") + 
	scale_y_continuous(labels = unit_format(unit = "M", scale = 1e-6)) -> p3

ggarrange(p1, p2, p3, ncol = 2, nrow = 2) %>% 
	annotate_figure(top = text_grob("INFO score distribution in chr22", color = "blue", face = "bold", size = 18))


## plot MAF vs TopMed MAF - this takes ~1min

info <- info %>%
	left_join( topmed, by = c("CHR","POS","REF","ALT") ) 
	
info %>%
	ggplot( aes(x=MAF, y=MAF.REF) ) +
	geom_point()
```


-----
:question: :question: :question: :question: **Questions:**  

* did you think the plots looked good?  
* would you check anything else?  
* can you check how many variants pass a number of maf/info filters?  
    + for example, how many variants with MAF < 0.01 have info > 0.8?  
    + continue with other examples  
* you might have noticed some cryptic columns in the info file, 
more information on what they mean [here](https://genome.sph.umich.edu/wiki/Minimac3_Info_File)
-----


