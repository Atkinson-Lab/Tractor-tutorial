---
title: Tractor with Local python file
filename: Local.md
---

![](images/TractorIcon.png)

&nbsp;  
&nbsp;  

## Intuition

Tractor implicitly assumes that the risk alleles that reside in different local ancestry backgrounds may have different marginal effect sizes (e.g. due to LD, MAF, phenotyping differences, or true effect size differences), and therefore performing LAI may provide better biological insights, allows for less-biased effect size estimates, and potentially increased gene discovery power. In this section, we will first compare Tractor with standard GWAS analysis and admixture mapping, and we will then go over the `RunTractor.py` script to run the Tractor method locally or in a HPC setting.

&nbsp;  
&nbsp;  

## Compare GWAS, Admixture mapping and Tractor 

In standard GWAS, we use individuals' genotype information but don't take into account local ancestry information. In the following diagram, if we sum up `AFR-Risk` and `EUR-Risk`, and perform a linear/logistic regression, we will recover a standard GWAS input.

In admixture mapping, we assume local ancestries affect the phenotype but don't take into account the allelic information. In this setting, we look at the number of copies of a specific ancestry background (`AFR` in this example). Summing up `AFR-Risk` and `AFR-noRisk` and performing a linear/logistic regression, we will recover admixture mapping.

Tractor takes both local ancestry and risk alleles into account and can be viewed as a combined method of GWAS and admixture mapping. In the Tractor model, we have 3 explanatory variables: X1 represents the number of `AFR` local ancestry haplotypes at a variant (identical to the X term used in admixture mapping); X2 represents the number of risk alleles on an AFR background - `AFR-Risk`, X3 represents the number of risk alleles on an EUR background `EUR-Risk`. In this way, we can get ancestry-specific summary statistics as output. In other words, we will observe if risk alleles on an `AFR` background may affect the phenotype differently compared with risk alleles on `EUR`. Such a case of differing marginal effect sizes at GWAS SNPs occur due to different tagging of causal variants depending on the ancestry; LD blocks, polymorphic loci, and allele frequencies differ across ancestries due to different populations' demographic histories.


![Methods comparison](images/TractorModel.png)


&nbsp;  
&nbsp;  

## Run Tractor locally

We have prepared a python script `RunTractor.py` that allows users to run Tractor GWAS in a local machine/high performance computer. Please prepare the following software packages before we start running the model:
* Python3
* pandas
* numpy
* statsmodel

&nbsp;  

For `Phe.txt` file, we expect users to prepare a tsv file with the following format:


```
IID	y	COV	...
SAMPLE1	1	0.32933076689022	...
SAMPLE2	1	1.42015135263198	...
SAMPLE3	0	1.57285800531035	...
SAMPLE4	0	-0.303899298288934	...
SAMPLE5	1	0.491130537379353	...
...	...	...	...
```

The tsv file should have at least 2 columns, with the first column indicates sample ID, and the second column indicates phenotype. If you have more covariates to add (such as PCs), users should append them as new columns. 

* Please make sure all columns (expect the first column) of `Phe.txt` were encoded as numerical type. We currently do not automatically convert string to numbers.
* Please make sure the sample ID order of `Phe.txt` is consistent with `--hapdose` files. `RunTractor.py` do not sort sample order.
* Please include header in `Phe.txt`


&nbsp;  


To perform linear regression on a (simulated) continuous phenotype in our admixed cohort, simply type the following command in your terminal:

```
Rscript run_tractor.R --hapdose ADMIX_COHORT/ASW.phased --phe PHENO/Phe.txt --method linear --out SumStats.tsv
```

This generates our summary statistic output, which has the estimated ancestry-specific p value and effect sizes for each locus for each ancestry. More specifically, you shall see these columns in the summary statistics:
```
CHROM:              Chromosome 
POS:                Position 
ID:                 SNP ID
REF:                Reference allele 
ALT:                Alternate allele 
AF_anc0:            Allele frequency for anc0 (AFR); computed as sum(dosage)/sum(local ancestry)
AF_anc1:            Allele frequency for anc1 (EUR); computed as sum(dosage)/sum(local ancestry)
LAprop_anc0:        Local ancestry proportion for anc0 (AFR); computed as sum(local ancestry)/2 * sample size
LAprop_anc1:        Local ancestry proportion for anc1 (EUR); computed as sum(local ancestry)/2 * sample size
LAeff_anc0:         Effect size for the local ancestry term (X1 term in Tractor)
LApval_anc0:        p value for the local ancestry term (X1 term in Tractor)
Geff_anc0:          Effect size for alternate alleles that are interited from anc0 (AFR) (X2 term in Tractor)
Geff_anc1:          Effect size for alternate alleles that are interited from anc1 (EUR) (X3 term in Tractor)
Gpval_anc0:         p value for alternate alleles that are interited from anc0 (AFR) (X2 term in Tractor)
Gpval_anc1:         p value for alternate alleles that are interited from anc1 (EUR) (X3 term in Tractor)
```

To visualize Tractor GWAS results, we may use the R package [qqman](https://cran.r-project.org/web/packages/qqman/vignettes/qqman.html) to draw a manhattan plot of our sumstats, and indeed we see a top hit in the AFR with some LD friends. Note in particular that we have a hit in only the AFR ancestry background in this example, no signal is observed in the EUR plot. We recommend plotting QQ plots with the lambda GC values shown in addition to your Manhattan plots to ensure your GWAS was well controlled.

```
library(qqman)
sumstats = read.csv("SumStats.tsv", sep = "\t")

par(mfrow=c(1,2))
manhattan(sumstats[!is.na(sumstats$Gpval_anc0),], chr="CHROM", bp="POS", snp="ID", p="Gpval_anc0",
          xlim = c(min(sumstats$POS), max(sumstats$POS)), ylim = c(0,15), main = "AFR")
manhattan(sumstats[!is.na(sumstats$Gpval_anc1),], chr="CHROM", bp="POS", snp="ID", p="Gpval_anc1",
          xlim = c(min(sumstats$POS), max(sumstats$POS)), ylim = c(0,15), main = "EUR")
```

![](images/Manhattan.png)

&nbsp;  

## [Main Page](README.md)
