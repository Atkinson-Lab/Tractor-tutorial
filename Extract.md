---
title: Extract Risk Alleles and Local Ancestry Information
filename: Extract.md
---

Compare to standard GWAS model, Tractor takes local ancestry into account and therefore may help improve statistical power and provide a less biased effect size estimate. In the [previous step](Rfmix.md), we demonstrated how to perform local ancestry inference using Rfmix. In this post, we will use its output files, as well as phased vcf files, to generate more interpretable intermediate files which will be used in statistical modeling.


&nbsp;  
&nbsp;  

## Prerequisites

#### Download Tractor Scripts

Tractor scripts can be easily installed with:
```
git clone https://github.com/Atkinson-Lab/Tractor.git
```

&nbsp;  
&nbsp;  

#### Check files in `ADMIX_COHORT` directory

We have demonstrated how to perform phasing and local ancestry inference respectively. To recap, after running phasing with Shapeit2, we should have 
```
ASW.phased.haps
ASW.phased.sample
ASW.phased.vcf
ASW.phased.vcf.gz
```
The `ASW.phased.vcf.gz` is converted from standard shapeit2 output, and will be used as an argument for `ExtractTracts.py`.

&nbsp;  

We have also performed local ancestry inference with Rfmix, and the following files have been generated
```
ASW.deconvoluted.fb.tsv	
ASW.deconvoluted.msp.tsv
ASW.deconvoluted.rfmix.Q
ASW.deconvoluted.sis.tsv
```

The `ASW.deconvoluted.msp.tsv` file will be later used as an argument for `ExtractTracts.py`.



&nbsp;  
&nbsp;  
&nbsp;  


## Extract Tracts

We provide a script that can simultaneously extract risk allele information and local ancestry information. Simply type the following command in terminal:
```
python Tractor/ExtractTracts.py \
      --msp ADMIX_COHORT/ASW.deconvoluted \
      --vcf ADMIX_COHORT/ASW.phased \
      --zipped \
      --num-ancs 2
```

6 files will be generated:
```
ASW.phased.anc0.dosage.txt
ASW.phased.anc0.hapcount.txt
ASW.phased.anc0.vcf
ASW.phased.anc1.dosage.txt
ASW.phased.anc1.hapcount.txt
ASW.phased.anc1.vcf
```

&nbsp;  

#### What happened under the hood?

Let's take a peek at our input files `ASW.phased.vcf.gz` and `ASW.deconvoluted.msp.tsv`

The phased vcf file contains haploid for each individual
```
CHROM POS REF ALT SAMPLE1   ...
22    10  A   T   1|0       ...
22    93  G   A   0|0       ...
22    106 G   A   0|0       ...
22    223 G   A   0|1       ...

...
```

The msp file contains local ancestry for each strand. Notice its coordinates is different from vcf file -- msp file uses a window to specify LA information. For example, the first row of the following msp file is telling us this: for SAMPLE1, strand0 from position1-position52 is inherited from `EUR` ancestry and strand1 from position1-position52 is inherited from `AFR` background.

```
AFR=0       EUR=1
CHROM SPOS  EPOS SAMPLE1.0 SAMPLE1.1   ...
22    1     52  1         0       ...
22    52    101 1         1       ...
22    101   190 0         1       ...
22    190   283 0         1       ...

...
```


&nbsp;  

By checking both files simultaneously, we can figure out the local ancestry background of risk allele, as the upper diagram shows. To represent both LA and risk allele information, we need to recode each variant into 4 columns, with each column represents a unique combination of LA and risk allele (`AFR-nonRisk`, `AFR-Risk`, `EUR-nonRisk`, `EUR-Risk`). In the diagram below, at the first variant, SAMPLE1 has one copy of `AFR-nonRisk`, and one copy of `EUR-Risk`, and therefore will be encoded to **[1,0,0,1]**. 

&nbsp;  

To further compress the information, `ExtractTracts.py` did an additional step -- adding `AFR-nonRisk` and `AFR-Risk`, which could be interpreted as the copies of `AFR` local ancestry for that variant. 

&nbsp;  


![](images/ExtractTract.png)


&nbsp; 

We will generate 6 files in this step, and 3 of them will be used in the next step for statistical modeling.
```
ASW.phased.anc0.hapcount.txt        (used as X1)
ASW.phased.anc0.dosage.txt          (used as X2)
ASW.phased.anc1.dosage.txt          (used as X3)
```






