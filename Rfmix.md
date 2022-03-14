---
title: Local Ancestry Deconvolution
filename: Rfmix.md
---

![](images/TractorIcon.png)

&nbsp;  
&nbsp;  

# Step0. Phasing and Local Ancestry inference
Before running the Tractor GWAS method, data will need to be phased and have their local ancstry inferred. To assist users with these precursor steps, we provide example code for running the full pipeline including these preliminary steps, assuming QC'ed but unphased cohort data.

&nbsp;  
&nbsp; 

### Data
We have provided an [example dataset](https://github.com/Atkinson-Lab/Tractor-tutorial/blob/main/tutorial-data.zip) that you may analyze to follow along with this tutorial. Please download and upzip the data from here if so.

The example cohort dataset we are going to use here consists of chromosome 22 for 61 African American individuals from the [Thousand Genome Project](https://www.internationalgenome.org/). These individuals are two-way admixed with components from continental Europe (EUR) and continental Africa (AFR). We simulated phenotypes for these individuals for use in the GWAS.

We also provide a haplotype reference panel that can be downloaded from  [Shapeit](https://mathgen.stats.ox.ac.uk/impute/data_download_1000G_phase1_integrated_SHAPEIT2_16-06-14.html), which will be used for phasing, along with phased reference VCF files for local ancestry inference. Here is a complete list of the files:

```
data
│
└───ADMIX_COHORT
│       |ASW.unphased.vcf.gz
│   
└───AFR_EUR_REF
|       |YRI_TSI.phased.vcf
|       |YRI_TSI.tsv
|       |chr22.genetic.map.modified.txt
|
└───HAP_REF    
|       |chr22.hap.gz
|       |chr22.legend.gz
|       |ALL.sample
|       |chr22.genetic.map.txt
└───PHENO   
        |Phe.txt
        |Phe2.txt
```



&nbsp;  
&nbsp;  


## Software 
[Shapeit2: v2.r837](https://mathgen.stats.ox.ac.uk/genetics_software/shapeit/shapeit.html)  

[Rfmix version 2](https://github.com/slowkoni/rfmix/blob/master/MANUAL.md)  

[Tractor (requires the following packages: numpy, statsmodel, pandas)](https://github.com/Atkinson-Lab/Tractor)


&nbsp;  
&nbsp;  


## Statistical Phasing 

&nbsp;  
&nbsp;  
 
Cohort genotype data is often released in [VCF format](https://www.internationalgenome.org/wiki/Analysis/Variant%20Call%20Format/vcf-variant-call-format-version-40/), which is the form we start from here. Though the human genome is diploid, sequencing/genotyping technology can only capture information regarding the genotypes that are present but not their orientation; e.g. the haplotype. We therefore don't know which allele is on which strand of the chromosome. The purpose of statistical phasing is to recover the configuration of alleles across a chromosome, as the diagram shows:

![](images/SHAPEIT.png)


Notice that each entry is separated with slash (e.g. `0/1`), and that means the VCF file is unphased. By performing phasing, we will figure out the most likely configuration of allele positions. After performing the following steps, we should get a phased VCF file, with the slash substituted by a vertical bar (e.g. `1|0`) which indicates that the data is phased.


&nbsp;  
&nbsp;  
&nbsp;  
 

Although many software has been developed for statistical phasing, here we will use [shapeit](https://mathgen.stats.ox.ac.uk/genetics_software/shapeit/shapeit.html#output) to perform phasing based on a reference haplotype panel. According to the manual [here](https://mathgen.stats.ox.ac.uk/genetics_software/shapeit/shapeit.html#reference), the phasing procedure consists of the following 4 steps:

&nbsp;  

#### A. Downloading 1000 genome phase 1 reference panel (already attached in HAP_REF folder)
 
&nbsp;  
&nbsp;  

#### B. Aligning common variants between query vcf and reference panel

The goal of this step is to find the common variants of the haplotype reference panel and our admixed population. Running the following script, Shapeit will typically produce two files `alignments.strand`, `alignments.strand.exclude`, which tell us the variants we should exclude in the downstream analysis. However, the dataset I am using here doesn’t have any variant conflicts, and therefore we can ignore this potential issue for now.

```       
shapeit -check \
        --input-vcf ADMIX_COHORT/ASW.unphased.vcf.gz \
        --input-map HAP_REF/chr22.genetic.map.txt \
        --input-ref HAP_REF/chr22.hap.gz HAP_REF/chr22.legend.gz HAP_REF/ALL.sample \
        --output-log alignments
```
    

The program will print some information and also throw some error message. You may pay attention to some of these messages:

```       
* 61 samples included
* 217153 SNPs included


# if reference panel sites are different from query vcf file, shapeit2 might print:
* #Missing sites in reference panel = 123456
* #Misaligned sites between panels = 123456
* #Multiple alignments between panels = 123456
```

&nbsp;  
&nbsp;  

#### C. Phasing (this is the most computational expensive step). 

We will perform the actual phasing in this step. Notice we should pass argument `--exclude-snp alignments.snp.strand.exclude` to the program if you encounter errors with your own dataset, so that conflict variants are excluded. In this command, we direct Shapeit to our cohort data (--input-vcf) which is in a directory with the input data, the reference panel and recombination map files, and tell it where to place the output (-O).

After running this command, you should find that two file (`ASW.phased.haps` & `ASW.phased.sample`) have been created in the `ADMIX_COHORT` output directory.


```       
shapeit  --input-vcf ADMIX_COHORT/ASW.unphased.vcf.gz \
      --input-map HAP_REF/chr22.genetic.map.txt \
      --input-ref HAP_REF/chr22.hap.gz HAP_REF/chr22.legend.gz HAP_REF/ALL.sample \
      -O ADMIX_COHORT/ASW.phased 
      
      
      # add this if shapeit throw error message:   --exclude-snp alignments.snp.strand.exclude
```    

&nbsp;  
&nbsp;       
      
 
#### D. Converting  shapeit output to vcf format.

Shapeit provides a convenient function to convert from its `haps`/`sample` file format to `vcf` format. Notice in the new vcf file we just created, slashes have changed to the pipe or vertical bar to seperate genotype. Additionally, we can zip the vcf file to save disc space.

```       
shapeit -convert \
        --input-haps ADMIX_COHORT/ASW.phased\
        --output-vcf ADMIX_COHORT/ASW.phased.vcf
        

bgzip -c ADMIX_COHORT/ASW.phased.vcf > ADMIX_COHORT/ASW.phased.vcf.gz
```   

--- 

(Note: your reference panel needs to be phased to be used in this manner. If you are hoping to use an unphased reference panel, you should consider running the Shapeit pipeline on the reference panel first, or to joint-phase your cohort and reference populations in a file together).

&nbsp;  
&nbsp;  
&nbsp;  
&nbsp;  


## Local Ancestry Inference

&nbsp;  
&nbsp;  

To run locan ancestry inference, we need a homogeneous phased reference panel composed of the relevant ancestries, and a phased admixed cohort vcf file. Admixed populations have chromosomes with a mosaic of ancestral tracts, as different chromosomal pieces will have been inherited from multiple ancestries. The length of these tracts is related to the demographic history of the population, with the average tract length being inversely proportional to the number of generations ago that the pulse of admixture occurred. This is because recombination will break down tracts over time.

For example, the 1st generation of the two-way admixed AFR-EUR population illustrated here inherited one full copy of chromosomes from EUR ancestry, and one from AFR ancestry. However, for more recent generations of this admixed population, the chromosomes will have been broken down into smaller ancestral pieces due to crossover events in meiosis.

![](images/localancestry.png)

&nbsp;  
&nbsp;  

The goal of running local ancestry inference is to "paint" the chromosomes based on the ancestry origin of each piece of the genome in each individual. In the downstream analysis of this Tractor pipeline, we will use local ancestry information to help us better understand the genotype-phenotype association.

![](images/inference.png)

&nbsp;  
&nbsp;  

Here we use [Rfmix](https://github.com/slowkoni/rfmix/blob/master/MANUAL.md) to perform local ancestry inference, given its high accuracy in multi-way admixed populations. RFmix takes the following argument and will produce 4 files `ASW.deconvoluted.fb.tsv`, `ASW.deconvoluted.msp.tsv`, `ASW.deconvoluted.rfmix.Q`, `ASW.deconvoluted.sis.tsv`, most of which will be used in the downstream analysis. This argument points to our cohort file, the reference panel, a map indicating which population each reference individual pertains to, a recombination map for the relevant chromosome, and the location where output should be deposited. To ensure that local ancestry is performing well in your target cohort, it's worth double checking the global ancestry fractions provided in `ASW.deconvoluted.rfmix.Q` to make sure the global ancestry proportions are as expected. 


```
rfmix -f ADMIX_COHORT/ASW.phased.vcf.gz \
        -r AFR_EUR_REF/YRI_GBR.phased.vcf.gz \
        -m AFR_EUR_REF/YRI_GBR.tsv \
        -g AFR_EUR_REF/chr22.genetic.map.modified.txt \
        -o ADMIX_COHORT/ASW.deconvoluted \
        --chromosome=22
```






Now you have your local ancestry calls and are ready for Tractor! We will demonstrate how you can optionally improve phasing by iterating between phasing and local ancestry inference in the [next post](Recover.md), or if your goal is just variant-level tests (e.g. GWAS), you may skip to Step 2.


&nbsp;  
&nbsp; 

## [Main Page](README.md)

## [Next Page (Recover phasing)](Recover.md)



