
# input VCF files
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

# Reproducibility test
### 1. concatenating VCF files (chr 1~22 & X, Y chromosome 제외)
```bash
~/dir$ bcftools concat *.recal.vcf -Oz -o ./bcftools_concat/combined.recal.vcf.gz
```

`bcftools`는 1.19 버전 사용
```bash
(base) user@vbmi:~$ bcftools --version
bcftools 1.19
Using htslib 1.19
Copyright (C) 2023 Genome Research Ltd.
License Expat: The MIT/Expat license
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
```
### 2. .bed/.bim/.fam 파일로 변환
```bash
PLINK v1.90b7.1 64-bit (18 Oct 2023)
Options in effect:
  --biallelic-only strict
  --make-bed
  --out combined_recal_biallelic_only_strict
  --vcf combined.recal.vcf.gz

Hostname: vbmi
Working directory: /data/02_ByProject/Ulsan/data/Ulsan_4K_Final/concat_only_biallele
Start time: Sun Apr 14 17:07:17 2024

Random number seed: 1713082037
1031697 MB RAM detected; reserving 515848 MB for main workspace.
--vcf: combined_recal_biallelic_only_strict-temporary.bed +
combined_recal_biallelic_only_strict-temporary.bim +
combined_recal_biallelic_only_strict-temporary.fam written.
(4563408 variants skipped.)
46364045 variants loaded from .bim file.
3617 people (0 males, 0 females, 3617 ambiguous) loaded from .fam.
Ambiguous sex IDs written to combined_recal_biallelic_only_strict.nosex .
Using 1 thread (no multithreaded calculations invoked).
Before main variant filters, 3617 founders and 0 nonfounders present.
Calculating allele frequencies... done.
Total genotyping rate is 0.997459.
46364045 variants and 3617 people pass filters and QC.
Note: No phenotypes present.
--make-bed to combined_recal_biallelic_only_strict.bed +
combined_recal_biallelic_only_strict.bim +
combined_recal_biallelic_only_strict.fam ... done.

End time: Sun Apr 14 20:54:32 2024

```
### 3. QC
```bash
PLINK v1.90b7.1 64-bit (18 Oct 2023)
Options in effect:
  --bfile combined_recal_biallelic_only_strict
  --geno 0.01
  --hwe 1e-6
  --maf 0.05
  --make-bed
  --out combined_recal_biallelic_only_strict_QC2

Hostname: vbmi
Working directory: /data/02_ByProject/Ulsan/data/Ulsan_4K_Final/concat_only_biallele
Start time: Wed Apr 17 15:55:37 2024

Random number seed: 1713336937
1031697 MB RAM detected; reserving 515848 MB for main workspace.
46364045 variants loaded from .bim file.
3617 people (0 males, 0 females, 3617 ambiguous) loaded from .fam.
Ambiguous sex IDs written to combined_recal_biallelic_only_strict_QC2.nosex .
Using 1 thread (no multithreaded calculations invoked).
Before main variant filters, 3617 founders and 0 nonfounders present.
Calculating allele frequencies... done.
Total genotyping rate is 0.997459.
690906 variants removed due to missing genotype data (--geno).
--hwe: 632179 variants removed due to Hardy-Weinberg exact test.
39570525 variants removed due to minor allele threshold(s)
(--maf/--max-maf/--mac/--max-mac).
5470435 variants and 3617 people pass filters and QC.
Note: No phenotypes present.
--make-bed to combined_recal_biallelic_only_strict_QC2.bed +
combined_recal_biallelic_only_strict_QC2.bim +
combined_recal_biallelic_only_strict_QC2.fam ... done.

End time: Wed Apr 17 15:58:27 2024

```
이후, phenotype(건강검진)과의 consistency를 위해`.fam` 파일의 FID를 모두 0으로 전환함.
```R
fam <- fread("0.genotype/combined_recal_biallelic_only_strict_QC2.fam")
fam$V1 <- 0
write.table(fam, file = "0.genotype/combined_recal_biallelic_only_strict_QC2.fam", row.names = F, col.names = F, sep = '\t', quote = F)
```

### 4. Covariate (PCA, age, age_sq, BMI)
- PCA
```bash
PLINK v2.00a6LM AVX2 Intel (23 Nov 2023)
Options in effect:
  --bfile 0.genotype/combined_recal_biallelic_only_strict_QC2
  --out 2.statistics/4.PCA2
  --pca 10

Hostname: vbmi
Working directory: /data/02_ByProject/Ulsan/GWAS_3617_re_onlybiallelic
Start time: Wed Apr 17 15:59:55 2024

Random number seed: 1713337195
1031697 MiB RAM detected, ~1006298 available; reserving 515848 MiB for main
workspace.
Using up to 128 threads (change this with --threads).
3617 samples (0 females, 0 males, 3617 ambiguous; 3617 founders) loaded from
0.genotype/combined_recal_biallelic_only_strict_QC2.fam.
5470435 variants loaded from
0.genotype/combined_recal_biallelic_only_strict_QC2.bim.
Note: No phenotype data present.
Calculating allele frequencies... done.
Excluding 4 variants on non-autosomes from GRM construction.
Constructing GRM: done.
Correcting for missingness... done.
Excluding 4 variants on non-autosomes from PCA.
Extracting eigenvalues and eigenvectors... done.
--pca: Eigenvectors written to 2.statistics/4.PCA2.eigenvec , and eigenvalues
written to 2.statistics/4.PCA2.eigenval .

End time: Wed Apr 17 16:02:36 2024

```
- Covariate 파일 생성
```R
pacman::p_load(data.table)

pca <- fread("2.statistics/4.PCA2.eigenvec")[,]
colnames(pca) <- c("FID", "IID", paste0("PC", 1:10))
pca$FID <- 0
view(pca)

pheno <- fread("1.phenotype/KOREA4K_healthcheck_latest_240308_forplink2.txt")
view(pheno)

covar_ <- pheno[, c("IID", "Age", "Body_Mass_Index")]
covar_$Age_sq <- covar_$Age ** 2

covar <- dplyr::inner_join(pca, covar_, by='IID')
view(covar)
covar$FID <- as.numeric(covar$FID)
covar$IID <- as.numeric(covar$IID)
write.table(covar, file = "3.GWAS/3.pheno_PCA2_covar.txt", row.names = F, sep = '\t', quote = F)
```
### 5. GWAS
```bash
PLINK v2.00a6LM AVX2 Intel (23 Nov 2023)
Options in effect:
  --adjust
  --bfile 0.genotype/combined_recal_biallelic_only_strict_QC2
  --ci 0.95
  --covar 3.GWAS/3.pheno_PCA2_covar.txt
  --covar-name Age,Age_sq,Body_Mass_Index,PC1,PC2,PC3,PC4,PC5,PC6,PC7,PC8,PC9,PC10
  --covar-variance-standardize
  --glm hide-covar sex
  --out 3.GWAS/GAWS_weight_onlybiallelic_240417
  --pheno 1.phenotype/KOREA4K_healthcheck_latest_240308_forplink2.txt
  --pheno-name Weight
  --require-pheno

Hostname: vbmi
Working directory: /data/02_ByProject/Ulsan/GWAS_3617_re_onlybiallelic
Start time: Wed Apr 17 16:18:37 2024

Random number seed: 1713338317
1031697 MiB RAM detected, ~995502 available; reserving 515848 MiB for main
workspace.
Using up to 128 threads (change this with --threads).
3617 samples (0 females, 0 males, 3617 ambiguous; 3617 founders) loaded from
0.genotype/combined_recal_biallelic_only_strict_QC2.fam.
5470435 variants loaded from
0.genotype/combined_recal_biallelic_only_strict_QC2.bim.
1 quantitative phenotype loaded (2985 values).
--require-pheno: 632 samples removed.
13 covariates loaded from 3.GWAS/3.pheno_PCA2_covar.txt.
2985 samples (0 females, 0 males, 2985 ambiguous; 2985 founders) remaining
after main filters.
2985 quantitative phenotype values remaining after main filters.
--covar-variance-standardize: 13 covariates transformed.
Calculating allele frequencies... done.
--glm linear regression on phenotype 'Weight': done.
Results written to 3.GWAS/GAWS_weight_onlybiallelic_240417.Weight.glm.linear .
--adjust: Genomic inflation est. lambda (based on median chisq) = 0.994853.
(Treating lambda as 1 in GC-corrected p-value calculation.)
--adjust values (5470435 tests) written to
3.GWAS/GAWS_weight_onlybiallelic_240417.Weight.glm.linear.adjusted .

End time: Wed Apr 17 16:18:49 2024

```

# SNU pipeline (가장 처음 구축한 파이프라인)
- Y chromosome 포함
- multiallelic variants 포함
- QC 조건은 1K 논문과 동일

### 1. bcftools norm을 통해 multiallelic variant를 split함
```bash
~/dir$ time parallel 'bcftools norm -m -any {} -Oz -o {.}.split.vcf.gz' ::: ../chr*.recal.vcf.gz chrY.raw.vcf.gz
~/dir$ time parallel 'bcftools index {}' ::: ../chr*.recal.vcf.split.vcf.gz chrY.raw.vcf.gz
```

### 2. concatenating: Y chromosome 포함된 VCF 파일 생성
```bash
# Concatenating every VCFs except ChrY
~/dir$ bcftools concat *.recal.vcf -Oz -o ./bcftools_concat/concat_nochrY.vcf.gz

# Create an empty VCF for female samples on ChrY
~/dir$ bcftools query -l chrX.recal.vcf > ./bcftools_concat/all.samples.txt 
~/dir$ bcftools query -l chrY.raw.vcf > ./bcftools_concat/male.samples.txt 
~/dir$ awk 'NR==FNR{a[$1];next} !($1 in a)' ./bcftools_concat/male.samples.txt ./bcftools_concat/all.samples.txt > ./bcftools_concat/female.samples.txt
~/dir$ cd bcftools_concat
~/dir/bcftools_concat$ grep '^##' ../chrY.raw.vcf >> chrY.empty.vcf 
~/dir/bcftools_concat$ echo -ne "#CHROM\tPOS\tID\tREF\tALT\tQUAL\tFILTER\tINFO\tFORMAT" > header.tmp 
~/dir/bcftools_concat$ awk '{printf("\t%s", $0)}' female.samples.txt >> header.tmp 
~/dir/bcftools_concat$ echo >> header.tmp 
~/dir/bcftools_concat$ cat chrY.empty.vcf header.tmp > chrY.empty.with.samples.vcf 
~/dir/bcftools_concat$ rm header.tmp
~/dir/bcftools_concat$ bgzip -c chrY.empty.with.samples.vcf > chrY.empty.with.samples.vcf.gz
~/dir/bcftools_concat$ tabix -p vcf chrY.empty.with.samples.vcf.gz

# Create ChrY VCF File containing all samples
~/dir/bcftools_concat$ bgzip -c ../chrY.raw.vcf > chrY.raw.vcf.gz 
~/dir/bcftools_concat$ tabix -p vcf chrY.raw.vcf.gz 
~/dir/bcftools_concat$ bcftools merge chrY.raw.vcf.gz chrY.empty.with.samples.vcf.gz -Oz -o chrY.all.samples.vcf.gz

# Final concatenation
~/dir/bcftools_concat$ bcftools query -l concat_nochrY.vcf.gz > all.samples.txt 
~/dir/bcftools_concat$ bcftools view -S all.samples.txt chrY.all.samples.vcf.gz -Oz -o chrY.all.samples.sorted.vcf.gz
~/dir/bcftools_concat$ bcftools index concat_nochrY.vcf.gz
~/dir/bcftools_concat$ bcftools index chrY.all.samples.sorted.vcf.gz
~/dir/bcftools_concat$ bcftools concat concat_nochrY.vcf.gz chrY.all.samples.sorted.vcf.gz -Oz -o concat.vcf.gz 
```

### 3. .bed/.bim/.fam 파일 변환
```bash
PLINK v2.00a6LM AVX2 Intel (23 Nov 2023)
Options in effect:
  --make-bed
  --new-id-max-allele-len 1000
  --out bcftools_concat
  --psam reordered_sex_info.psam
  --set-missing-var-ids @:#[b37]$1,$2
  --split-par hg38
  --vcf bcftools_concat.vcf.gz

Hostname: vbmi
Working directory: /data/02_ByProject/Ulsan/data/Ulsan_4K_Final/concat/final_merge
Start time: Thu Feb 29 17:28:57 2024

Random number seed: 1709195337
1031697 MiB RAM detected, ~984780 available; reserving 515848 MiB for main
workspace.
Using up to 128 threads (change this with --threads).
--vcf: 59367485 variants scanned.
--vcf: bcftools_concat-temporary.pgen + bcftools_concat-temporary.pvar.zst
written.
3617 samples (1616 females, 2001 males; 3617 founders) loaded from
reordered_sex_info.psam.
--split-par: 99318 chromosome codes changed.
59367485 variants loaded from bcftools_concat-temporary.pvar.zst.
Note: No phenotype data present.
Writing bcftools_concat.fam ... done.
Writing bcftools_concat.bim ... done.
Writing bcftools_concat.bed ... done.

End time: Thu Feb 29 20:28:54 2024

```
### 4. QC
```bash
PLINK v1.90b7.1 64-bit (18 Oct 2023)
Options in effect:
  --allow-extra-chr
  --bfile bcftools_concat
  --geno 0.01
  --hwe 1e-6
  --maf 0.01
  --make-bed
  --out bcftools_concat.QC

Hostname: vbmi
Working directory: /data/02_ByProject/Ulsan/data/Ulsan_4K_Final/concat/final_merge
Start time: Thu Mar 21 17:44:04 2024

Random number seed: 1711010644
1031697 MB RAM detected; reserving 515848 MB for main workspace.
59367485 variants loaded from .bim file.
3617 people (2001 males, 1616 females) loaded from .fam.
Using 1 thread (no multithreaded calculations invoked).
Before main variant filters, 3617 founders and 0 nonfounders present.
Calculating allele frequencies... done.
Total genotyping rate is 0.99048.
4980884 variants removed due to missing genotype data (--geno).
Warning: --hwe observation counts vary by more than 10%, due to the X
chromosome.  You may want to use a less stringent --hwe p-value threshold for X
chromosome variants.
--hwe: 1103203 variants removed due to Hardy-Weinberg exact test.
43245210 variants removed due to minor allele threshold(s)
(--maf/--max-maf/--mac/--max-mac).
10038188 variants and 3617 people pass filters and QC.
Note: No phenotypes present.
--make-bed to bcftools_concat.QC.bed + bcftools_concat.QC.bim +
bcftools_concat.QC.fam ... done.

End time: Thu Mar 21 17:47:52 2024

```
### 5. PCA 및 covariate 파일 생성
```bash
PLINK v1.90b7.1 64-bit (18 Oct 2023)
Options in effect:
  --allow-extra-chr
  --bfile 0.genotype/bcftools_concat.QC
  --out 2.statistics/4.PCA.QC
  --pca 20

Hostname: vbmi
Working directory: /data/02_ByProject/Ulsan/GWAS_3617_re
Start time: Thu Mar 21 17:52:41 2024

Random number seed: 1711011161
1031697 MB RAM detected; reserving 515848 MB for main workspace.
10038188 variants loaded from .bim file.
3617 people (2001 males, 1616 females) loaded from .fam.
Using up to 127 threads (change this with --threads).
Before main variant filters, 3617 founders and 0 nonfounders present.
Calculating allele frequencies... done.
Total genotyping rate is 0.998954.
10038188 variants and 3617 people pass filters and QC.
Note: No phenotypes present.
Excluding 284016 variants on non-autosomes from relationship matrix calc.
Relationship matrix calculation complete.
--pca: Results saved to 2.statistics/4.PCA.QC.eigenval and
2.statistics/4.PCA.QC.eigenvec .

End time: Thu Mar 21 18:52:12 2024

```
```R
setwd("/home/user/data/02_ByProject/Ulsan/GWAS_3617_re")
p_load(data.table)

pca <- fread("2.statistics/4.PCA.eigenvec")[,]
colnames(pca) <- c("FID", "IID", paste0("PC", 1:10))
pca$FID <- 0
view(pca)

pheno <- fread("1.phenotype/KOREA4K_healthcheck_latest_240308_forplink2.txt")
view(pheno)

covar_ <- pheno[, c("IID", "Age", "Body_Mass_Index")]
covar_$Age_sq <- covar_$Age ** 2

covar <- dplyr::inner_join(pca, covar_, by='IID')
view(covar)
covar$FID <- as.numeric(covar$FID)
covar$IID <- as.numeric(covar$IID)
write.table(covar, file = "3.GWAS/3.pheno_PCA_covar.txt", row.names = F, sep = '\t', quote = F)
```
### 6. GWAS
```bash
PLINK v2.00a6LM AVX2 Intel (23 Nov 2023)
Options in effect:
  --adjust
  --ci 0.95
  --covar 3.GWAS/3.pheno_PCA_covar.QC.txt
  --covar-name Age,Age_sq,Body_Mass_Index,PC1,PC2,PC3,PC4,PC5,PC6,PC7,PC8,PC9,PC10
  --covar-variance-standardize
  --glm hide-covar sex
  --out 3.GWAS/GWAS_re.Weight_afterQC
  --pfile 0.genotype/bcftools_concat.QC
  --pheno 1.phenotype/KOREA4K_healthcheck_latest_240308_forplink2.txt
  --pheno-name Weight
  --require-pheno

Hostname: vbmi
Working directory: /data/02_ByProject/Ulsan/GWAS_3617_re
Start time: Fri Mar 22 15:56:12 2024

Random number seed: 1711090572
1031697 MiB RAM detected, ~1001462 available; reserving 515848 MiB for main
workspace.
Using up to 128 threads (change this with --threads).
3617 samples (1616 females, 2001 males; 3617 founders) loaded from
0.genotype/bcftools_concat.QC.psam.
10038188 variants loaded from 0.genotype/bcftools_concat.QC.pvar.
1 quantitative phenotype loaded (2985 values).
--require-pheno: 632 samples removed.
13 covariates loaded from 3.GWAS/3.pheno_PCA_covar.QC.txt.
2985 samples (1212 females, 1773 males; 2985 founders) remaining after main
filters.
2985 quantitative phenotype values remaining after main filters.
--covar-variance-standardize: 13 covariates transformed.
Calculating allele frequencies... done.
--glm linear regression on phenotype 'Weight': done.
Results written to 3.GWAS/GWAS_re.Weight_afterQC.Weight.glm.linear .
--adjust: Genomic inflation est. lambda (based on median chisq) = 1.02217.
--adjust values (10038188 tests) written to
3.GWAS/GWAS_re.Weight_afterQC.Weight.glm.linear.adjusted .

End time: Fri Mar 22 15:57:14 2024

```



---
## 참고용 Venn Diagram 생성 코드
```R
p_load(VennDiagram)

ulsan_4k_weight <- fread("/home/user/data/02_ByProject/Ulsan/data/Ulsan_4K/2022_6_9_GWAS_Data_WHtR_WWtR_NOBMI/3.GWAS/Korea4K_GWAS.Weight.Weight.glm.linear")
snu_4k_weight <- fread("./3.GWAS/GAWS_weight_onlybiallelic_240417.Weight.glm.linear") # our GWAS

p_threshold = 1e-4 # 1e-2, 1e-4

ulsan_4k_weight_sig <- ulsan_4k_weight[ulsan_4k_weight$P < p_threshold, c("#CHROM", "POS")]
snu_4k_weight_sig <- snu_4k_weight[snu_4k_weight$P < p_threshold, c("#CHROM", "POS")]

ulsan_4k_weight_sig_set <- paste(ulsan_4k_weight_sig$`#CHROM`, ulsan_4k_weight_sig$POS)
snu_4k_weight_sig_set <- paste(snu_4k_weight_sig$`#CHROM`, snu_4k_weight_sig$POS)

intersection <- intersect(ulsan_4k_weight_sig_set, snu_4k_weight_sig_set)

jaccard <- length(intersection) / length(union(ulsan_4k_weight_sig_set, snu_4k_weight_sig_set))
overlap <- length(intersection) / min(length(ulsan_4k_weight_sig_set), length(snu_4k_weight_sig_set))
dice <- 2 * length(intersection) / (length(ulsan_4k_weight_sig_set) + length(snu_4k_weight_sig_set))



venn.diagram(list(ulsan_weight=ulsan_4k_weight_sig_set, snu_weight=snu_4k_weight_sig_set),
             filename = paste0("./venndiagram_weight_UlsanVsSnu_240417_", get("p_threshold"), ".png"),
             fill=c("blue", "red"),
             alpha=0.5,
             cat.fontface="bold",
             cat.pos=c(-20, 20),
             cat.dist=c(0.03, 0.03),
             cat.cex=0.5,
             sub.cex=0.5,
             cex=0.8,
             sub.pos=c(0.8, 1),
             sub=paste("Jaccard similarity = ", round(jaccard, 3),
                       "\nOverlap coefficient = ", round(overlap, 3),
                       "\nDice similarity = ", round(dice, 3), sep=""),
             main=paste0("GWAS result comparison: Ulsan vs SNU (p-val threshold: ", get("p_threshold"), ")"),
             main.cex=0.8,
             main.pos=c(0.5,1.05),
             main.fontface = "bold")

```