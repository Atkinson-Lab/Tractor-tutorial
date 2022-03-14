---
title: Tractor with Local python file
filename: Local.md
---

![](images/TractorIcon.png)

&nbsp;  
&nbsp;  

## Intuition

Tractor implicitly assumes that the risk alleles that reside in different local ancestry background present different effect size, and therefore performing LAI may provide better biological insights, allows a less-biased estimates, and potentially increase power. Here in this section, we will first compare Tractor with standard GWAS analysis, and admixture mapping, and we will then go over `RunTractor.py` to run Tractor method locally or in a HPC setting.

&nbsp;  
&nbsp;  

## Compare GWAS, Admixture mapping and Tractor 

In standard GWAS, we assume that local ancestry background doesn’t play a role in determining the phenotypes, and therefore is ignorable. In the following diagram, if we sum up `AFR-Risk` and `EUR-Risk`, and perform a linear/logistic regression, we will recover a standard GWAS

In Admixture mapping, we assume local ancestries affect phenotype while risk alleles don't. In this setting, we look at the number of copies of a specific ancestry background (`AFR` in this example). Summing up `AFR-Risk` and `AFR-noRisk` and performing a linear/logistic regression, we will recover admixture mapping.

Tractor takes both local ancestry and risk alleles into account and can be viewed as a combined method of GWAS and Admixture mapping. In tractor model, we have 3 explanatory variables: X1 represents the number of `AFR` local ancestry for a variants (identical to Admixture mapping); X2 represents the number of `AFR-Risk`, X3 represents `EUR-Risk`. We have seen a boosted statistical power when effect sizes are different across ancestries. In other words, risk alleles on `AFR` background affect the phenotype differently compared with risk alleles on `EUR`.


![Methods comparison](images/TractorModel.png)


&nbsp;  
&nbsp;  

## Run Tractor locally

We have prepared a python script `RunTractor.py` that allows users to run Tractor in a local machine/high performance computer. Please prepare the following softwares before we start running the model:
* Python3
* pandas
* numpy
* statsmodel


To perform a linear regression on a continuous phenotype, simply type the following command in your terminal:

```
python RunTractor.py --hapdose ADMIX_COHORT/ASW.phased --phe PHENO/Phe.txt --method linear --out SumStats.tsv
```

Here I use R package `qqman` to draw a manhattan plot of our sumstats, and indeed we found a top hit with some LD friends.

```
library(qqman)
sumstats = read.csv("SumStats.tsv", sep = "\t")

par(mfrow=c(1,2))
manhattan(sumstats[!is.na(sumstats$ANC0P),], chr="CHROM", bp="POS", snp="ID", p="ANC0P",
          xlim = c(min(sumstats$POS), max(sumstats$POS)), ylim = c(0,15), main = "AFR")
manhattan(sumstats[!is.na(sumstats$ANC1P),], chr="CHROM", bp="POS", snp="ID", p="ANC1P",
          xlim = c(min(sumstats$POS), max(sumstats$POS)), ylim = c(0,15), main = "EUR")
```

![](images/Manhattan.png)


That's the end of the tutorial, please refer to [Tractor Wiki](https://github.com/Atkinson-Lab/Tractor/wiki) page for a more detailed manual.



## [Main Page](README.md)


