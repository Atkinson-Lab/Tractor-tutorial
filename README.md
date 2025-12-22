![](images/TractorIcon.png)

## Tractor - A GWAS tool for admixed population 

Welcome to Tractor!

For a comprehensive understanding of Tractor's methodology and application, we invite you to read our following **2021 Nature Genetics publication** [here](https://www.nature.com/articles/s41588-020-00766-y). 

> Atkinson, E.G., Maihofer, A.X., Kanai, M. et al. Tractor uses local ancestry to enable the inclusion of admixed individuals in GWAS and to boost power. Nat Genet 53, 195–204 (2021).

We also recently developed Tractor-Mix, which enables modeling admixture with relatedness. Please refer to our preprint [here](https://www.medrxiv.org/content/10.1101/2025.05.27.25328444v1). 

> Tan T, Vergara-Lope A, Martínez-Magaña JJ, Shah NN, Yuan K, Berumen J, Alegre-Díaz J, Kuri-Morales P, Tapia-Conyer R, Gelenter J, Montalvo-Ortiz JL, Zhou W, Torres JM, Atkinson EG. Extending Genome-Wide Association Studies to admixed cohorts with high degrees of relatedness. medRxiv [Preprint]. 2025 Jun 9:2025.05.27.25328444. doi: 10.1101/2025.05.27.25328444. PMID: 40585113; PMCID: PMC12204420.


To facilitate users in implementing the method, we've curated a series of tutorials available here. These tutorials cover each step of the Tractor pipeline using a toy dataset. Please [download](https://github.com/Atkinson-Lab/Tractor-tutorial/blob/main/test_data.zip) and unzip the provided file with the toy data to follow along.

Please note that while this tutorial offers a high-level guide to running Tractor/Tractor-Mix, users are advised to ensure the reliability and accuracy of phasing and local ancestry inference, as these factors influence the Tractor GWAS results.


## Tractor pipeline step-by-step tutorials:

- **Step 0**: [Phasing and Local Ancestry Inference](Rfmix.md)
- **Step 1**: [Recovering Tracts](Recover.md)
- **Step 2**: [Extracting tracts and ancestry dosages](Extract.md)
- **Step 3a**: [Tractor GWAS with local implementation (suitable for smaller dataset)](Local.md)
- **Step 3b**: [Tractor GWAS with Hail (suitable for large scale analysis)](Hail.md)
- **Step 4**: [Visualization of Tractor summary statistics](Visualization.md)



## Tractor-Mix pipeline step-by-step tutorials:

- **Step 0**: [Phasing and Local Ancestry Inference](Rfmix.md)
- **Step 1**: [Recovering Tracts (often skipped)](Recover.md)
- **Step 2**: [Extracting tracts and ancestry dosages](Extract.md)
- **Step 3**: [Estimate PC/GRM](PC-GRM.md)
- **Step 4**: [Mix effect model with Tractor-Mix](GLMM.md)




