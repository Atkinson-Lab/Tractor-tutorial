---
title: PC/PC-GRM
filename: PC-GRM.md
---

![](images/TractorIcon.png) 

# Step 3: estimate PC/GRM (Tractor-Mix)

In the mixed model, we include principal components (PCs) to account for population structure and use the genetic relatedness matrix (GRM) to control for family relatedness. For studies involving admixed individuals, you can use standard PCs in combination with a standard GRM. Alternatively, the PC-AiR/PC-Relate framework can be used, which is specifically designed for admixed populations. PC-AiR separates population structure from familial relatedness, while PC-Relate estimates the GRM accordingly. This approach provides a more theoretically robust treatment of relatedness in admixed samples. However, based on our simulation studies, both approaches show comparable performance in admixture GWAS.


### Standard PC/GRM 

You can use PLINK 2.0 to estimate both PCs and the GRM, which are essential for controlling population structure and relatedness in mixed model association studies.

Before computing PCs or the GRM, we typically perform LD pruning to reduce redundant information and ensure independent variant selection.

```
plink2 --vcf admixed_cohort/ASW.unphased.vcf.gz \
  --set-all-var-ids @:#:\$r:\$a \
  --indep-pairwise 500kb 0.2 \
  --out admixed_cohort/ASW
```
This command generates a list of pruned variants in the file `ASW.prune.in`.


Next, extract the pruned variants and export them as a new VCF:

```
plink2 --vcf admixed_cohort/ASW.unphased.vcf.gz \
  --set-all-var-ids @:#:\$r:\$a \
  --extract admixed_cohort/ASW.prune.in \
  --export vcf bgz\
  --out admixed_cohort/ASW.pruned
```
After this step, you should have a file named `ASW.pruned.vcf.gz`, which contains a subset of independent variants suitable for PCA and GRM calculation.


Now, use the pruned VCF to compute the principal components and the GRM:

```
plink2 --vcf admixed_cohort/ASW.pruned.vcf.gz \
  --pca 2 \
  --out admixed_cohort/ASW.pruned
  
  
plink2 --vcf admixed_cohort/ASW.pruned.vcf.gz \
  --make-rel square \
  --out admixed_cohort/ASW.pruned
```

After these steps, you’ll have:

`ASW.pruned.eigenvec`: Contains the top principal components, which can be used to control for population structure.

`ASW.pruned.rel`: The GRM in square format, used to account for relatedness between individuals.

These files can now be used as covariates and input matrices in Tractor-Mix. 



&nbsp;  
&nbsp;  



### Estimating PCs and GRM with PC-AiR and PC-Relate (via GENESIS)


Running PC-AiR and PC-Relate is more involved than using standard PCA/GRM approaches. For a full explanation and usage examples, we refer readers to [the official tutorial](https://bioconductor.org/packages/release/bioc/vignettes/GENESIS/inst/doc/pcair.html). 

We used the [this code](https://github.com/Atkinson-Lab/Tractor-Mix-manuscript/blob/main/UKBB/TRACTOR/make_PC_GRM_from_GENESIS.R) for the manuscript.

Below is a brief summary of the key steps involved in running PC-AiR/PC-Relate using the GENESIS R package:

1. Run KING-Robust on the pruned VCF file.  

This step estimates pairwise kinship coefficients that are robust to population structure. PLINK2 provides a built-in implementation via the --make-king-table option.

2. Convert the pruned VCF to PLINK binary format (.bed/.bim/.fam).  

These files are required for conversion into GDS format, which is the input format used by GENESIS.


3. Convert the PLINK files to GDS format.  

The SNPRelate package is used to generate a .gds file from the PLINK binary files.

4. Perform PC-AiR analysis.  

PC-AiR partitions individuals into related and unrelated subsets, and computes principal components while accounting for relatedness. This helps separate population structure from family structure.

5. Estimate the GRM using PC-Relate.  

PC-Relate uses the output from PC-AiR to estimate pairwise relatedness, capturing both family-level and population-level structure.




















