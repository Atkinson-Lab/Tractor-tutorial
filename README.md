## Tractor - A GWAS tool for admixed population 

Thanks for your interest in Tractor!

The methodology and utility of Tractor is more fully described in our publication [here](https://www.nature.com/articles/s41588-020-00766-y), "Tractor uses local ancestry to enable the inclusion of admixed individuals in GWAS and to boost power", which is was published in Nature Genetics here. We ask that you cite this manuscript for work utilizing the Tractor software.

&nbsp;  

In this repo, we provide scripts comprising the full Tractor pipeline for (1) recovering long-range haplotypes as informed by local ancestry deconvolution, (2) extracting ancestral segments and calculating ancestral minor allele dosages, and (3) running our local ancestry aware GWAS model.

&nbsp;  

The pipeline is currently designed assuming RFmix_v2 and phased VCF format genotype input and consists of separate scripts for maximum flexibility of use. These scripts can be run in sequence to do all the steps or, if long-range tracts will not impact the goals of the analysis, the user can skip right to ancestry dosage calculation and running our GWAS model.

&nbsp;  

Important: The success of Tractor relies on good local ancestry calls. Please ensure your LAI performance is highly accurate before running a Tractor GWAS. We recommend admixture simulations to test accuracy for your demographic use case, which can be done with a program such as https://github.com/williamslab/admix-simu.

&nbsp;  
&nbsp;  

## Tractor pipeline step-by-step tutorials:


Step0. [Phasing and Local Ancestry Inference](Rfmix.md)

Step1. [Recovering Tracts](Recover.md)

Step2. [Extracting tracts and ancestry dosages](Extract.md)

Step3a. [Tractor GWAS with Hail (suitable for large scale analysis)](Hail.md)

Step3b. [Tractor GWAS with local implementation (suitable for smaller dataset)](Local.md)

