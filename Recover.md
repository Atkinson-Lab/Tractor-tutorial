---
title: Recover Tracts (optional)
filename: Recover.md
---


**This page is devoted to describing our scripts to detect and correct switch errors in phased data using local ancestry to recover long-range tracts. See Figure 1 in our manuscript for additional context. A switch error is defined as a swap in ancestries within a 1 cM window to opposite strands conditioned on heterozygous ancestral dosage. Currently, the tract recovery step is only implemented for a 2-way admixed setting. All subsequent steps are compatible with multi-way admixed cohorts. Tract recovery is not required for analyses that do not consider haplotypes, including GWAS. **

&nbsp;  
&nbsp;  

## Identifying and correcting switch errors in local ancestry calls

The first (optional) step is to detect strand flips in the data by utilizing local ancestry information. This step uses RFmix ancestry calls as input and is implemented with the script UnkinkMSPfile.py, which tracks switch locations and corrects them. Output consists of 2 files: a text file documenting switch locations for each individual (input msp file name suffixed with "switches") and a corrected local ancestry file (input msp file name suffixed with "unkinked"), as in unkinking a garden hose. This first step recovers long range tracts which are disrupted by statistical phasing using the local ancestry calls and tracks the location of the identified strand switches.

Example usage: python UnkinkMSPfile.py --msp FILENAME_STEM

&nbsp;  
&nbsp;  

## Correcting switch errors in genotype data


Next we also need to correct switch errors in the phased genotype file (VCF format). This recovers fuller haplotypes and improves the long-range tract distribution. This step is implemented with the script UnkinkGenofile.py and expects as input the phased VCF file that was fed into RFmix and the 'switches' file generated with the previous step. This switches file is used to determine the positions that need to be flipped in the VCF file.

Example usage: python UnkinkGenofile.py --switches SWITCHES_FILE --genofile INPUT_VCF

Notes: Stem filenames will be appended in each step. The first step expects the standard .msp.tsv extension from RFMix ancestry calls as input. The second step expects a VCF file with the .vcf extension.

Tractor expects all VCF files to be phased (i.e. genotypes contain pipes rather than slashes), and it is recommended to strip their INFO and FORMAT annotations prior to running to ensure good parsing. This can be accomplished with bcftools on bgzipped and tabix indexed vcfs:


```
bgzip file.vcf
tabix -p vcf file.vcf.gz
bcftools annotate -x INFO,FORMAT file.vcf.gz > stripped_file.vcf
```



## To Do List

This page is directly copied from Tractor wiki page. Further editing is required.
