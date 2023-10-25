# Genotyping QC   
### Chiara Batini  
### WT DTP Genomic data masterclass  
### 25/10/2023  

## Summary  

This practical session follows (mostly) the tutorial from 
[Anderson et al 2010](https://drive.google.com/file/d/1tKREcp8m8qc5o8S0JfyyNu9R--LbAd0F/view?usp=sharing) and 
takes raw genotyping data to perform the following QC steps:  

* per-sample QC  
	+ discordant sex information  
	+ outliers for missing rate and heterozygosity  
	+ outliers for genetic similarity  
	
* per-variant QC  
	+ outliers for missing rate  
	+ minor allele frquency  
	+ deviation from Hardy Weinberg equilibrium  


## Getting the data  

A tar archive containing all the files needed for this practical is avalable 
[here](https://drive.google.com/file/d/1D4YgwAQVkYAxIY_8cpZaO4I4IHxWnlSg/view?usp=sharing).  
Open your browser in ALICE and download it there. 

**Create a directory to use for this practical**, 
move into it using `cd` and move the tar archive there using this command:  
```
mv ~/Downloads/array_tutorial_files.tar.gz .
```

You can then open this file using the command:  
```
tar xfvz array_tutorial_files.tar.gz
```

You should now find a number of files in your folder, including the raw data plink files.
```
raw-GWA-data.map
raw-GWA-data.ped
```  

-----
:question: :question: :question: :question: **Questions:**  

* How many individuals are included in the raw data?   
* How many variants are included in the raw data?   
-----


## Create PLINK binary files  

We want to convert PLINK map/ped files into bed/bim/fam files.  
These represent a compressed version of the map/ped with no loss of information.  

```
module load plink

plink \
	--file raw-GWA-data \
	--make-bed \
	--out raw-GWA-data

``` 

-----
:question: :question: :question: :question: **Questions:**  

* Which version of PLINK are you using?  
	+ Do you know much about the difference between v1.9 and v2?     
* Was there any loss of information during the conversion?  
* Have you saved any storage space?  
   
-----

## Per-sample QC  
The per-sample QC steps are aimed mostly at excluding possible 
sample mix-ups that happened in the lab, and samples that performed poorly in 
genotyping.  


### Discordant sex information  
Comparing the sex information we already have about the samples with the 
one that we can infer using genetic data is one step to check if there was any sample mix-up.  


```

plink \
	--bfile raw-GWA-data \
	--check-sex \
	--out raw-GWA-data-sex

grep PROBLEM raw-GWA-data-sex.sexcheck | awk '{ print $1" "$2}' > fail-sexcheck-qc.txt
```


-----
:question: :question: :question: :question: **Questions:**  

* How many individuals show discordant sex information?
	+ What would you do next?
   
-----

 
### Missing data and heterozygosity outliers    
We will exclude individuals with high missing data rates.  
We will use heterozygosity as a proxy for sample mix-up and exclude outliers.  


```

plink \
	--bfile raw-GWA-data \
	--missing \
	--out raw-GWA-data-miss

plink \
	--bfile raw-GWA-data \
	--het \
	--out raw-GWA-data-het

```


-----
:question: :question: :question: :question: **Questions:**  

* What is the difference between .imiss and .lmiss files?

-----

Open R:


```

library(tidyverse)

imiss <- read_table("raw-GWA-data-miss.imiss")

het <- read_table("raw-GWA-data-het.het") %>%
	rename( NNM = `N(NM)`,
		OHOM = `O(HOM)`) %>%
	mutate( het = (NNM - OHOM)/NNM ) %>%
	left_join( imiss, by = c("FID","IID") )
	
het %>%
	ggplot(aes(x = F_MISS, y = het ) ) +
	geom_point() +
	geom_hline( yintercept = mean(het$het) + 3 * sd(het$het), col = "red") +
	geom_hline( yintercept = mean(het$het) - 3 * sd(het$het), col = "red") +
	geom_vline( xintercept = 0.05, col = "blue")
	
het %>%
	filter( het >= mean(het) + 3 * sd(het) |
		het <= mean(het) - 3 * sd(het) |
		F_MISS >= 0.05 ) %>% 
	select( FID, IID ) %>%
	write.table("fail-imisshet-qc.txt", col.names = F, row.names = F , quote = F)


```


-----
:question: :question: :question: :question: **Questions:**  

* How many individuals have you identified as heterozygosity or missingness outliers?
   
-----



### Duplicates    
We will check for duplicated samples. To do so, we will first prune our set of variants and keep only 
independent variants (r2 < 0.2 ), and then exclude samples with PI_HAT > 0.98. 



```
plink \
	--bfile raw-GWA-data \
	--exclude high-LD-regions.txt \
	--autosome \
	--indep-pairwise 50 5 0.2 \
	--out raw-GWA-data-ld 

plink \
	--bfile raw-GWA-data \
	--extract raw-GWA-data-ld.prune.in \
	--genome \
	--out raw-GWA-data-ibd


```

Open R:

```
library(tidyverse)

ibd <- read.table("raw-GWA-data-ibd.genome", header = T)

ibd %>%
    filter(PI_HAT > 0.98) %>%
	select(FID1, IID1) %>%
	write.table("fail-duplicates-QC.txt", col.names = F, row.names = F , quote = F)


```


### Genetic similarity QC    
Genetic distance can be a confounding in GWASs so we will compare our samples to a reference 
panel and exclude individuals too distant from the main group.  

```

## extract variants in reference panel from your data
plink \
	--bfile raw-GWA-data \
	--extract hapmap3r2_CEU.CHB.JPT.YRI.no-at-cg-snps.txt \
	--make-bed \
	--out raw-GWA-data.hapmap-snps 

## attempt merging your data with reference panel	
plink \
	--bfile raw-GWA-data.hapmap-snps \
	--bmerge hapmap3r2_CEU.CHB.JPT.YRI.founders.no-at-cg-snps.bed \
	hapmap3r2_CEU.CHB.JPT.YRI.founders.no-at-cg-snps.bim \
	hapmap3r2_CEU.CHB.JPT.YRI.founders.no-at-cg-snps.fam \
	--extract raw-GWA-data.prune.in \
	--make-bed \
	--out raw-GWA-data.hapmap3r2.pruned
```

-----
:question: :question: :question: :question: **Questions:**  

* What's the problem?
   
-----


```

## repeat extraction excluding the problematic variants	
plink \
	--bfile raw-GWA-data \
	--extract hapmap3r2_CEU.CHB.JPT.YRI.no-at-cg-snps.txt \
	--exclude raw-GWA-data.hapmap3r2.pruned-merge.missnp \
	--make-bed \
	--out raw-GWA-data.hapmap-snps 

## attempt merging the two datasets again	
plink \
	--bfile raw-GWA-data.hapmap-snps \
	--bmerge hapmap3r2_CEU.CHB.JPT.YRI.founders.no-at-cg-snps.bed \
	hapmap3r2_CEU.CHB.JPT.YRI.founders.no-at-cg-snps.bim \
	hapmap3r2_CEU.CHB.JPT.YRI.founders.no-at-cg-snps.fam \
	--extract raw-GWA-data.prune.in \
	--make-bed \
	--out raw-GWA-data.hapmap3r2.pruned
	
## run PCA on merged dataset using only reference panel populations to calculate the loadings
plink \
	--bfile raw-GWA-data.hapmap3r2.pruned \
	--within population_def.txt \
	--pca \
	--pca-cluster-names CEU CHB JPT YRI \
	--out pca
 ```
 
 Open R:
 
 ```
library(tidyverse)
 
pca <- read.table("pca.eigenvec", header = F) %>%
	rename( PC1 = V3,
		PC2 = V4)
pops <- read.table("population_def.txt", header = F) %>%
	rename( POP = V3)

pca <- pca %>% 
	left_join(pops, by = c("V1", "V2") ) 
	
	
PC1_low <- pca %>%
		filter( POP == "CEU" & !is.na(POP)) %>%
		select( PC1 ) %>%
		mean()


pca %>% 
	ggplot(aes(x=PC1, y=PC2, colour = POP)) + 
	geom_point() +
	geom_vline( xintercept = mean(pca$PC1[pca$POP == "CEU" & !is.na(pca$POP)]) + 
			3* sd(pca$PC1[pca$POP == "CEU" & !is.na(pca$POP)]), linetype=3) +
	geom_vline( xintercept = mean(pca$PC1[pca$POP == "CEU" & !is.na(pca$POP)]) - 
			3* sd(pca$PC1[pca$POP == "CEU" & !is.na(pca$POP)]), linetype=3) +
	geom_hline( yintercept = mean(pca$PC2[pca$POP == "CEU" & !is.na(pca$POP)]) + 
			3* sd(pca$PC2[pca$POP == "CEU" & !is.na(pca$POP)]), linetype=3) +
	geom_hline( yintercept = mean(pca$PC2[pca$POP == "CEU" & !is.na(pca$POP)]) - 
			3* sd(pca$PC2[pca$POP == "CEU" & !is.na(pca$POP)]), linetype=3) 
			
pca %>%
	filter( POP == "study" ) %>%
	filter( PC1 > mean(pca$PC1[pca$POP == "CEU" & !is.na(pca$POP)]) + 
				3* sd(pca$PC1[pca$POP == "CEU" & !is.na(pca$POP)]) |
			PC1 < mean(pca$PC1[pca$POP == "CEU" & !is.na(pca$POP)]) - 
				3* sd(pca$PC1[pca$POP == "CEU" & !is.na(pca$POP)]) |
			PC2 >  mean(pca$PC2[pca$POP == "CEU" & !is.na(pca$POP)]) + 
				3* sd(pca$PC2[pca$POP == "CEU" & !is.na(pca$POP)]) |
			PC2 < mean(pca$PC2[pca$POP == "CEU" & !is.na(pca$POP)]) - 
				3* sd(pca$PC2[pca$POP == "CEU" & !is.na(pca$POP)])) %>%
	select(V1, V2) %>%
	write.table("fail-pops-QC.txt", col.names = F, row.names = F, quote = F)
	
```

	

-----
:question: :question: :question: :question: **Questions:**  

* How many individuals have you identified as outliers?
* Did you think the thresholds chosen were reasonable here? Would you change the number of sd to consider?
   
-----

### Clean dataset after per-sample QC	
Create a list of individuals not passing QC and exclude them from the dataset.  	
	
```
cat fail-* | sort | uniq > fail-qc-inds.txt

plink \
	--bfile raw-GWA-data \
	--remove fail-qc-inds.txt \
	--make-bed \
	--out clean-inds-GWA-data
```
	
	
	
## Per-variant QC  
The per-variant QC steps are aimed at excluding variants that performed poorly during genotyping.  
We usually exclude variants that:  

* have a missingness rate higher than 0.05  
* have a MAF lower than 1%  
* have a HWE p-value below 10e-4

```
plink \
	--bfile clean-inds-GWA-data \
	--autosome \
	--geno 0.05 \
	--maf 0.01 \
	--hwe 0.0001 \
	--make-bed \
	--out clean-GWA-data
```
	
	
-----
:question: :question: :question: :question: **Questions:**  

* How many individuals have you identified for each criterion?  
* Do you think the threshold used for HWE is reasonable? And would you exclude these variants? Why?    
   
-----


## Final clean dataset  

 	
-----
:question: :question: :question: :question: **Questions:**  

* How many individuals and how many variants have remained after all the filtering?  
* Is there anything you would have done differently? Why?  
* Can you plot the allele frequencies for the final dataset? Do they look as expected?
* When you are preparing the data for GWAS, and you know the case/control status, would you incorporate this information while filtering? How?        
-----

