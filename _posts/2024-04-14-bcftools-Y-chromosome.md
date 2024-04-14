---
layout: single
title: "[Population Genetics] bcftools concat: unequal number of samples - dealing with Y chromosome"
categories: Population_Genetics
tags: [population_genetics, bioinformatics, GWAS, bcftools]
toc: true
author_profile: false
sidebar:
    nav: "docs"
---

# 0. Introduction

When performing population genetics analysis, it is very common to deal with variant calling format (VCF) or binary variant call format (BCF, binary version of VCF), and `bcftools` is used a lot.

`bcftools` can be installed using [this link](https://www.htslib.org/download/) which includes installation guide.

---
Now I'm using `bcftools` version 1.19
```bash
(base) user@vbmi:~$ bcftools --version
bcftools 1.19
Using htslib 1.19
Copyright (C) 2023 Genome Research Ltd.
License Expat: The MIT/Expat license
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
```
---

While combining multiple VCF(BCF) files into one, for downstream analysis, there are two strategies:
- `bcftools merge` to combine sample-specific VCF files into one
- `bcftools concat` to combine chromosome-specific VCF files into one.

One problem while dealing with chromosome-specific VCF files, is that the **VCF file corresponding to the Y chromosome only has male samples.** (Y chromosome trolling again today;;)

If you utilize `bcftools concat` without considering this issue, you'll get an "***unequal number of samples***" error, and cannot concatenate the VCF file corresponding to the Y chromosome.

In case if you are excluding Y chromosome variants in downstream analysis such as genome wide association studies (GWAS), you don't need to solve it, but if not, the Y chromosome should be integrated and analyzed.

here are the steps for troubleshooting, and integrate every chromosome-VCF files including Y chromosome:
1. Create an empty VCF file containing only female sample columns and ChrY rows
2. Using `bcftools merge`, merge the empty VCF with the ChrY VCF file (with male samples)
3. Perform `bcftools concat` with the remaining autosome and ChrX VCF files

Let's take it step by step

---

# Input files

here is the structure of VCF files.
```bash
~/dir$ tree 
. 
├── chr10.recal.vcf 
├── chr10.recal.vcf.idx 
├── chr11.recal.vcf 
├── chr11.recal.vcf.idx 
├── chr12.recal.vcf 
├── chr12.recal.vcf.idx 
├── chr13.recal.vcf 
├── chr13.recal.vcf.idx 
├── chr14.recal.vcf 
├── chr14.recal.vcf.idx 
├── chr15.recal.vcf 
├── chr15.recal.vcf.idx 
├── chr16.recal.vcf 
├── chr16.recal.vcf.idx 
├── chr17.recal.vcf 
├── chr17.recal.vcf.idx 
├── chr18.recal.vcf 
├── chr18.recal.vcf.idx 
├── chr19.recal.vcf 
├── chr19.recal.vcf.idx 
├── chr1.recal.vcf 
├── chr1.recal.vcf.idx 
├── chr20.recal.vcf 
├── chr20.recal.vcf.idx 
├── chr21.recal.vcf 
├── chr21.recal.vcf.idx 
├── chr22.recal.vcf 
├── chr22.recal.vcf.idx 
├── chr2.recal.vcf 
├── chr2.recal.vcf.idx 
├── chr3.recal.vcf 
├── chr3.recal.vcf.idx 
├── chr4.recal.vcf 
├── chr4.recal.vcf.idx 
├── chr5.recal.vcf 
├── chr5.recal.vcf.idx 
├── chr6.recal.vcf 
├── chr6.recal.vcf.idx 
├── chr7.recal.vcf 
├── chr7.recal.vcf.idx 
├── chr8.recal.vcf 
├── chr8.recal.vcf.idx 
├── chr9.recal.vcf 
├── chr9.recal.vcf.idx 
├── chrX.recal.vcf 
├── chrX.recal.vcf.idx 
├── chrY.raw.vcf 
└── chrY.raw.vcf.idx 

0 directories, 48 files
```
There are VCF files and index files. (It doesn't matter if you have compressed VCF files (.vcf.gz))

# Concatenating every VCFs except ChrY

Let's concat the remaining files except ChrY first, as you may need other downstream analyses later, such as GWAS excluding the Y chromosome. It would also be a good idea to do this for backup purposes.

This step can be omitted if there are storage issues. Instead, concatenate the files for all chromosomes at the end.

```bash
~/dir$ bcftools concat *.recal.vcf -Oz -o ./bcftools_concat/concat_nochrY.vcf.gz
```
`*.recal.vcf` will get every files ending with `.recal.vcf`, and it will skip `chrY.raw.vcf`.

# Create an empty VCF for female samples on ChrY

First, let's get all the sample IDs of female.

All female sample names can be pulled by subtracting the sample name of `ChrY.recal.vcf` (male samples) from the sample name of `ChrX.recal.vcf` (all samples).

*(If there are 'other' genders, it might be annoying, just pull it from the codebook lol)*

Just in case, after this work, check whether the number of samples for male and female matches in the codebook or etc.

```bash
~/dir$ bcftools query -l chrX.recal.vcf > ./bcftools_concat/all.samples.txt 
~/dir$ bcftools query -l chrY.raw.vcf > ./bcftools_concat/male.samples.txt 
~/dir$ awk 'NR==FNR{a[$1];next} !($1 in a)' ./bcftools_concat/male.samples.txt ./bcftools_concat/all.samples.txt > ./bcftools_concat/female.samples.txt
```

So we've planeted the files `all.samples.txt`, `male.samples.txt`, and `female.samples.txt`, each with a sample name on a single line.

Let's get started to create an empty VCF file:

First, create a header of VCF. In VCF files, the header starts with '##', so we'll pull the header from another VCF file. (Or you can just Ctrl+C and Ctrl+V, if your header is short)

```bash
~/dir$ cd bcftools_concat
~/dir/bcftools_concat$ grep '^##' ../chrY.raw.vcf >> chrY.empty.vcf 
~/dir/bcftools_concat$ echo -ne "#CHROM\tPOS\tID\tREF\tALT\tQUAL\tFILTER\tINFO\tFORMAT" > header.tmp 
~/dir/bcftools_concat$ awk '{printf("\t%s", $0)}' female.samples.txt >> header.tmp 
~/dir/bcftools_concat$ echo >> header.tmp 
~/dir/bcftools_concat$ cat chrY.empty.vcf header.tmp > chrY.empty.with.samples.vcf 
~/dir/bcftools_concat$ rm header.tmp
```

So we've achieved our goal in this step: creating `chrY.empty.with.samples.vcf` file.

Let's compress and index it.

```bash
~/dir/bcftools_concat$ bgzip -c chrY.empty.with.samples.vcf > chrY.empty.with.samples.vcf.gz
~/dir/bcftools_concat$ tabix -p vcf chrY.empty.with.samples.vcf.gz
```

# Create ChrY VCF file containing all samples

Likewize, `bgzip` the `ChrY.raw.vcf` file and index it with `tabix`.
Afterwards, `bcftools merge` completes the creation of the ChrY VCF, including all samples.

```bash
~/dir/bcftools_concat$ bgzip -c ../chrY.raw.vcf > chrY.raw.vcf.gz 
~/dir/bcftools_concat$ tabix -p vcf chrY.raw.vcf.gz 
~/dir/bcftools_concat$ bcftools merge chrY.raw.vcf.gz chrY.empty.with.samples.vcf.gz -Oz -o chrY.all.samples.vcf.gz
```

# Final concatenation

Let's concat the `concat_noChrY.vcf.gz` created in the first step, and the corresponding ChrY file. Then it's done!

Before that, just in case, let's start with sorting!

```bash
~/dir/bcftools_concat$ bcftools query -l concat_nochrY.vcf.gz > all.samples.txt 
~/dir/bcftools_concat$ bcftools view -S all.samples.txt chrY.all.samples.vcf.gz -Oz -o chrY.all.samples.sorted.vcf.gz
```

then indexing

```bash
~/dir/bcftools_concat$ bcftools index concat_nochrY.vcf.gz
~/dir/bcftools_concat$ bcftools index chrY.all.samples.sorted.vcf.gz
```

Now it's final concatenating

```bash
~/dir/bcftools_concat$ bcftools concat concat_nochrY.vcf.gz chrY.all.samples.sorted.vcf.gz -Oz -o concat.vcf.gz 
```

The final `concat.vcf.gz` created now contains all variant information for all samples and all chromosomes.

THE END!