# Population genetic structure
Population genetic structure was assessed via PCA, genetic distances, and ADMIXTURE analysis.
```
Step 1: PCA using smartpca to capture genetic variation.
smartpca -p bos_eigensoft_smartpca.par
Step 2: Compute pairwise genetic distances with PLINK.
plink -bfile HK --chr-set 29 --distance-matrix --out HK
Step 3: Infer population structure using ADMIXTURE.
admixture HK.prune.bed $i
admixture --cv HK.prune.bed $i | tee log${i}.snp.out
```
# Genetic diversity
## 1.Heterozygosity
Heterozygosity calculation from VCF using vcftools
```
vcftools --gzvcf SNP.vcf.gz --het --out HKF.snp
```
## 2.Nucleotide diversity
Nucleotide diversity calculation from VCF using vcftools
```
vcftools --gzvcf SNP.vcf.gz --keep HKF.list --window-pi 50000 --window-pi-step 20000 --out HKF.pi
```
## 3.Runs of Homozygosity
Runs of Homozygosity (ROH) analysis using PLINK
```
# Step 1: Identify individual ROH using PLINK
vcftools --gzvcf SNP.vcf.gz --plink --chrom-map chrID --out SNP
plink --file SNP --make-bed --chr-set 29 --out SNP
plink --bfile SNP --chr-set 29 --homozyg --homozyg-gap 1000 --homozyg-window-snp 50 --homozyg-snp 50 --out ROH

# Step 2: Calculate shared ROH across individuals using BEDTools
bedtools multiinter -i sample1.roh.bed sample2.roh.bed sample3.roh.bed > shared_roh_multi.bed
```
# Selection
```
Step 1: FST calculation between HK and other population
vcftools --gzvcf SNP.vcf.gz --weir-fst-pop HK.list other.list --fst-window-size 50000 --fst-window-step 20000 --out Fst-win 
Step 2: Nucleotide diversity (pi) for HK population
vcftools --gzvcf SNP.vcf.gz --keep HK.list --window-pi 50000 --window-pi-step 20000 --out HK.pi
```
# Introgression 
```
Step 1: D-statistics using qpDstat
qpDstat -p Dstats_taurine.par > result
Step 2: Genome-wide introgression scan using qpBrute
qpBrute --par hongkong.par --prefix hongkong --pops Banteng Gaur SAI EAI HongKong --out Buffalo
```
# CNV analysese
```
cnvnator -root root -tree dedup.recal.bam -chrom 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29
cnvnator -rootroot -tree dedup.recal.bam -his 250 -d /fasta/ -chrom 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29
cnvnator -root root -stat 250
cnvnator -root root -partition 250
cnvnator -root root -call 250 > cnv.call.tsv
```
