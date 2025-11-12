---
title: Recover Tracts (optional) (unfinished)
filename: Recover.md
---

![](images/TractorIcon.png)

> **NOTE:** This step is optional and currently only implemented for **two-way admixed populations**. If your analyses do not require long-range haplotypes, you can skip this step to save compute time.

This page describes scripts to detect and correct **switch errors** in phased data using local ancestry, helping to recover long-range tracts. For context, see Figure 1 in our manuscript. A **switch error** occurs when ancestries swap strands within a ~1 cM window, conditioned on heterozygous ancestry dosage. This tract recovery is currently limited to 2-way admixed populations, but all downstream analyses are compatible with multi-way admixed cohorts.

**Note:** Tract recovery is not required for analyses that do not use haplotypes, such as standard GWAS.
  
## Step 1: Detecting and correcting switch errors in local ancestry

The first step identifies **strand flips** in local ancestry calls.  

- **Input:** RFMix `.msp.tsv` file  
- **Script:** `unkink_2way_mspfile.py`  
- **Output:**  
  1. A text file documenting switch locations (`*_switches`)  
  2. A corrected local ancestry file (`*_unkinked`)  

Think of this as “unkinking a garden hose”: the script recovers long-range tracts disrupted by phasing and records the locations of detected strand switches.

**Example usage:**
```bash
./unkink_2way_mspfile.py --msp FILENAME_STEM
```

## Step 2: Correcting switch errors in phased genotypes

Next, switch errors are corrected in the **phased genotype (VCF) file** to improve haplotype continuity and long-range tract distribution. This switches file generated from the previous step is used to determine the positions that need to be flipped in the VCF file.

* **Input:**

  * Phased VCF file used in RFMix
  * `*_switches` file from Step 1
* **Script:** `unkink_2way_genofile.py`
* **Output:** Corrected phased VCF

**Example usage:**

```bash
./unkink_2way_genofile.py --switches SWITCHES_FILE --genofile INPUT_VCF
```

## Notes

* **Step 1** expects `.msp.tsv` files from RFMix.
* **Step 2** expects phased VCFs (`|` instead of `/`) and recommends stripping `INFO` and `FORMAT` annotations for parsing:

```bash
bgzip file.vcf
tabix -p vcf file.vcf.gz
bcftools annotate -x INFO,FORMAT file.vcf.gz > stripped_file.vcf
```

---

## Navigation

* [Main Page](README.md) 

* [Next Page (Extract Tracts)](Extract.md)      



