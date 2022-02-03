---
title: Local Ancestry Deconvolution
filename: Rfmix.md
---

![](images/TractorIcon.png)

&nbsp;  
&nbsp;  

# Step0. Phasing and Local Ancestry inference

&nbsp;  
&nbsp; 

### Data
We have provided an example dataset that you may follow along with this tutorial. Please download from here, and unzip them.

The dataset we are going to use here is 61 African Americans (only chromosome 22) from the Thousand Genome project – two-way admixtures of EUR and AFR population. Besides, we also provided the haplotype reference panel that is downloaded from  [Shapeit](https://mathgen.stats.ox.ac.uk/impute/data_download_1000G_phase1_integrated_SHAPEIT2_16-06-14.html), which will be used for phasing. We also provide phased reference vcf files (EUR and AFR ancestry, respectively) for local ancestry inference. Here is a complete list of the files:

```
data
│
└───ADMIX_COHORT
│       |ASW.unphased.vcf.gz
│   
└───AFR_EUR_REF
|       |YRI_TSI.phased.vcf
|       |YRI_TSI.tsv
|
└───HAP_REF    
        |chr22.haplotypes.gz
        |chr22.legend.gz
        |ALL.sample
        |chr22.genetic.map.txt
```



&nbsp;  
&nbsp;  


### 1. Statistical Phasing 

&nbsp;  
&nbsp;  
 
 Genotype and sequencing data is often written in vcf format. Due to the diploid nature of the human genome, the sequencing/genotyping technology can only capture the genotype information but not the haplotype. We therefore don't know which allele is on which strand of the chromosome. The purpose of statistical phasing is to recover the configuration of alleles across a chromosome, as the diagram shows:

![](images/SHAPEIT.png)


Notice that each entry is separated with slash (e.g. `0/1`), and that means the vcf file is unphased. By performing phasing, we will figure out the most likely configuration of allele positions. After performing the following steps, we should get a phased vcf file, with slash substituted by a vertical bar (e.g. `1|0`).


&nbsp;  
&nbsp;  
&nbsp;  
 

Although many software has been developed, here we will use [shapeit](https://mathgen.stats.ox.ac.uk/genetics_software/shapeit/shapeit.html#output) to perform phasing based on reference haploid panel. According to the manual [here](https://mathgen.stats.ox.ac.uk/genetics_software/shapeit/shapeit.html#reference), the phasing procedure consists of the following 4 steps:

&nbsp;  

#### A. Downloading 1000 genome phase 1 reference panel (already attached in HAP_REF folder)
 
&nbsp;  
&nbsp;  

#### B. Aligning common variants between query vcf and reference panel

The goal of this step is to find the common variants of the haplotype reference panel and our admixed population. Running the following script, Shapeit will produce two files `alignments.strand`, `alignments.strand.exclude`, which tell us the variants we should exclude in the downstream analysis. 

```       
shapeit -check \
        --input-vcf ADMIX_COHORT/ASW.unphased.vcf.gz \
        --input-map HAP_REF/chr22.genetic.map.txt \
        --input-ref HAP_REF/chr22.haplotypes.gz HAP_REF/chr22.legend.gz HAP_REF/ALL.sample \
        --output-log alignments
```
    

The program will print some information and also throw some error message. You may pay attention to some of these messages:

```       
* 61 samples included
* 217153 SNPs included


* #Missing sites in reference panel = 18360
* #Misaligned sites between panels = 171
* #Multiple alignments between panels = 0
```

&nbsp;  
&nbsp;  

#### C. Phasing (this is the most computational expensive step). 

We will perform the actual phasing in this step. Notice we should pass argument `--exclude-snp alignments.snp.strand.exclude` to the program, so that conflict variants can be excluded before phasing. After running this command, you should find two file (`ASW.phased.haps` & `ASW.phased.sample`) have been created in the `ADMIX_COHORT` directory.


```       
shapeit  --input-vcf ADMIX_COHORT/ASW.unphased.vcf.gz \
      --input-map HAP_REF/chr22.genetic.map.txt \
      --input-ref HAP_REF/chr22.haplotypes.gz HAP_REF/chr22.legend.gz HAP_REF/ALL.sample \
      -O ADMIX_COHORT/ASW.phased \
      --exclude-snp alignments.snp.strand.exclude
```    

&nbsp;  
&nbsp;       
      
 
#### D. Converting the shapeit outcome to vcf format.

Shapeit provides convenient function to convert from `haps` file format to `vcf` format. Notice in the new vcf file we just created, slashes are substituted by vertical bar to seperate genotype. Additionally, we can zip the vcf file to save disc space.

```       
shapeit -convert \
        --input-haps ADMIX_COHORT/ASW.phased\
        --output-vcf ADMIX_COHORT/ASW.phased.vcf
        

bgzip -c ADMIX_COHORT/ASW.phased.vcf > ADMIX_COHORT/ASW.phased.vcf.gz
```    
