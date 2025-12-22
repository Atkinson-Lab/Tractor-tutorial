---
title: GLMM
filename: GLMM.md
---

![](images/TractorIcon.png) 

# Step 4: Mixed effect model with Tractor-Mix

Now that all components of Tractor-Mix are assembled, we are ready to fit the null model using GMMAT. Before doing so, we first need to load the phenotype data, PCs, and the GRM into R.

For generalized linear mixed models (GLMMs), it is common practice to use a sparse GRM. Specifically, we recommend masking the GRM to retain only pairwise relatedness values greater than 0.05. This sparsification can lead to substantial improvements in computational efficiency, often reducing runtime by a factor of 10, depending on cohort size.

We strongly recommend using a sparse GRM, especially when working with cohorts that include thousands of individuals.


In R script:
```
library(dplyr)
library(Matrix)

## merge the PC and the phenotype file
PC = read.table("./admixed_cohort/ASW.pruned.eigenvec", sep = "\t", col.names=c("IID", "PC1", "PC2"))
PHE = read.table("./phenotype/Phe_logistic.txt", sep = "\t", header = TRUE)
df = inner_join(PHE, PC)


## read the GRM
GRM = as.matrix(read.table("./admixed_cohort/ASW.pruned.rel", header = FALSE))
GRM_id = read.table("./admixed_cohort/ASW.pruned.rel.id", header = FALSE, col.names=c("IID"))
row.names(GRM) = GRM_id$IID
colnames(GRM) = GRM_id$IID

# mask values < 0.05
GRM[GRM < 0.05] = 0
GRM = as(GRM, "sparseMatrix") 
```

With these components ready, we can now fit a null model with GMMAT:

```{r}
library(GMMAT)
Model_Null =   glmmkin(fixed = y ~  PC1 + PC2, 
                       data = df, id = "IID", kins = GRM, 
                       family = binomial())
```



We can now proceed to run Tractor-Mix across all variants. By default, the standard Tractor-Mix model does not include local ancestry as a covariate in the association model. This means the effect sizes are estimated based on ancestry-specific genotypes, but local ancestry itself is not adjusted for in the regression model. For the toy data, we use `AC_threshold = 1`, but you may need to adjust this parameter for a larger cohort. This is because Tractor-Mix may not be calibrated when allele counts are low. In the manuscript, we used `AC_threshold = 50`. 

```{r}
source("TractorMix.score.R")

TractorMix.score(obj = Model_Null, 
                 infiles = c("./admixed_cohort/ASW.phased.anc0.dosage.txt", 
                             "./admixed_cohort/ASW.phased.anc1.dosage.txt"),
                 outfiles = "result_uncond.tsv", 
                 AC_threshold = 1)
                 
                
```



Alternatively, you may consider performing a conditional analysis, which more closely mirrors the original Tractor framework. In this approach, the model includes local ancestry explicitly, following the form: $y \sim LA + G1 + G2$. This implementation is available in our pipeline; however, it has not yet been thoroughly tested. We recommend using it with caution.


```
source("TractorMix.score_cond.R")
TractorMix.score_cond(obj = Model_Null, 
                 infiles_geno = c("./admixed_cohort/ASW.phased.anc0.dosage.txt", 
                             "./admixed_cohort/ASW.phased.anc1.dosage.txt"),
                 infiles_la = c("./admixed_cohort/ASW.phased.anc0.hapcount.txt"), 
                 outfiles = "result_cond.tsv", 
                 AC_threshold = 1)
```


## [Main Page](README.md)



