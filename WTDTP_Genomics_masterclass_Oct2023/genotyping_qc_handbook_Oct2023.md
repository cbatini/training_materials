# Genotyping QC   
### Chiara Batini  
### WT DTP Genomic data masterclass  
### 25/10/2023  

## Summary  

In this practical we will follow the tutorial from 
[Anderson et al 2010](https://drive.google.com/file/d/1tKREcp8m8qc5o8S0JfyyNu9R--LbAd0F/view?usp=sharing) and 
take raw genotyping data to perform the following QC steps:  

* per-sample QC  
	+ discordant sex information  
	+ outliers for missing rate and heterozygosity  
	
* per-variant QC  
	+ 



## Dataset


## Getting the data  

A tar archive containing all the files needed for this practical is avalable 
[here](https://drive.google.com/file/d/10Dhal1aB1VAAPbIR6F88AWlPmwxsWTtC/view?usp=share_link).  
Open your browser in ALICE and download it there. 

**Create a directory to use for this practical**, 
move into it using `cd` and move the tar archive there using this command:  
```
mv ~/Downloads/raw-GWA-data.tgz .
```

You can then open this file using the command:  
```
tar xfvz raw-GWA-data.tgz
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
Comparing the sex information we alrady have about the samples with the 
one that we can infer using genetic data is one step to check if there was any sample mix-up.  


```

plink \
	--bfile raw-GWA-data \
	--check-sex \
	--out raw-GWA-data-sex

grep PROBLEM raw-GWA-data-sex.sexcheck | awk '{ print $1" "$2}' > fail-sexcheck-qc.txt
```
 


