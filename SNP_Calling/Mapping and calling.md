# Read mapping and variant calling
## 1. Mapping
Pipeline for read trimming, mapping, sorting and duplicate marking.
```
#!/bin/sh
java -Xmx25g -jar trimmomatic-0.39.jar PE 111_1B_1.fq.gz 111_1B_2.fq.gz 111_1B_1_trimmed.fq.gz 111_1B_2_trimmed.fq.gz 
bwa mem -M $ref 111_1B_1_trimmed.fq.gz 111_1B_2_trimmed.fq.gz | samtools view -bS - > 111_1B.bam
java -Xmx20g -jar picard.jar  SortSam I=111_1B.bam O=111_1B.sort.bam SORT_ORDER=coordinate
java -Xmx20g -jar picard.jar  MarkDuplicates I=111_1B.sort.bam O=111_1B.sort.dedup.bam M=111_1B.marked_dup_metrics.txt REMOVE_DUPLICATES=true CREATE_INDEX=true VALIDATION_STRINGENCY=LENIENT
```
## 2. GVCF
Generating individual GVCFs across autosomal chromosomes using GATK HaplotypeCaller
```
java -Xmx20g -jar GenomeAnalysisTK.jar \\
		-T HaplotypeCaller \\
		-R $ref \\
		-ERC GVCF \\
		-variant_index_type LINEAR \\
		-variant_index_parameter 128000 \\
		-nct 20 \\
		-L {chrom} \\
		-I $BAM/{ID}.sort.dedup.bam \\
		-o $GVCF/{chrom}_{ID}.g.vcf.gz 
```
## 3.Genotype
Per sample VCF generation from GVCFs using GATK GenotypeGVCFs
```
java -Xmx100g -jar $GATK \\
	-R $ref \\
	-T GenotypeGVCFs \\
        --variant ./chr.list \\
	-o $Genotype/Chr.vcf.gz \\
	--disable_auto_index_creation_and_locking_when_reading_rods \\
	--includeNonVariantSites \\
        -nt 10 \\
```
## 4.Filter
Per sample SNP selection and filtering using GATK4
```
gatk4 SelectVariants \\
        -R $ref \\
        --select-type-to-include SNP \\
        -restrict-alleles-to BIALLELIC \\
        --remove-unused-alternates \\
        --exclude-non-variants \\
        -exclude-filtered \\
        -V chr.filter.SNP.vcf.gz \\
        -O chr.clean.SNP.vcf.gz
```
