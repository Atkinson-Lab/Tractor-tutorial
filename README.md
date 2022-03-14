![](images/TractorIcon.png)

## Tractor - A GWAS tool for admixed population 

Thanks for your interest in Tractor!

The methodology and utility of Tractor is more fully described in our 2021 Nature Genetics publication [here](https://www.nature.com/articles/s41588-020-00766-y), "Tractor uses local ancestry to enable the inclusion of admixed individuals in GWAS and to boost power". We ask that you cite this manuscript for work utilizing the Tractor software.

&nbsp;  


To assist users with implementing the method, we have prepared a series of tutorials here that go over each step of Tractor pipeline with a toy dataset. Please download and unzip the [file](https://github.com/Atkinson-Lab/Tractor-tutorial/blob/main/tutorial-data.zip) with this toy data if you hope to follow along. 

(As a note, this tutorial serves as a high-level guide to running Tractor, and it’s the users' responsibility to make sure phasing and local ancestry inference are reliable before running the method, as these will affect results.)


&nbsp;  
&nbsp;  

## Tractor pipeline step-by-step tutorials:


Step0. [Phasing and Local Ancestry Inference](Rfmix.md)

Step1. [Recovering Tracts](Recover.md)

Step2. [Extracting tracts and ancestry dosages](Extract.md)

Step3a. [Tractor GWAS with Hail (suitable for large scale analysis)](Hail.md)

Step3b. [Tractor GWAS with local implementation (suitable for smaller dataset)](Local.md)

