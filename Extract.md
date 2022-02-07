---
title: Extract Risk Alleles and Local Ancestry Information
filename: Extract.md
---

Compare to standard GWAS model, Tractor takes local ancestry into account and therefore may help improve statistical power and provide a less biased effect size estimate. In the [previous step](Rfmix.md), we demonstrated how to perform local ancestry inference using Rfmix. In this post, we will use its output files, as well as phased vcf files, to generate a more interpretable intermediate file which will be used in statistical modeling.


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


## Run `ExtractTracts.py`

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

The msp file contains local ancestry for each strand

```
AFR=0       EUR=1
CHROM POS SAMPLE1.0 SAMPLE1.1   ...
22    10  1         0       ...
22    93  1         1       ...
22    106 0         1       ...
22    223 0         1      ...

...
```


By checking both files simultaneously, we would immediately know the local ancestry background of risk allele, as the diagram shows. 






