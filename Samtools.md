## Samtools

```
#!/bin/bash

source pipeline.cfg
	
## RealignerTargetCreator
	
mkdir ${samtools_path}/1.realign
	
java -Xmx4g -Xms4g -jar ${GATK3}/GenomeAnalysisTK.jar -T RealignerTargetCreator \
	 -R ${ref_fasta} \
	 -I ${Sort_path}/$2_Markdup_sort.bam \
	 -nt 4 \
	 -known ${Mills_indels} \
	 -known ${dbsnp_vcf} \
	 -known ${known_indels} \
	 -o ${samtools_path}/1.realign/$1_$2_realignment_targets.intervals
	
# Realigner
	
java -Xmx4g -Xms4g -jar ${GATK3}/GenomeAnalysisTK.jar -T IndelRealigner \
	 -R ${ref_fasta} \
	 -I ${Sort_path}/$2_Markdup_sort.bam \
	 -targetIntervals ${samtools_path}/1.realign/$1_$2_realignment_targets.intervals \
	 -known ${Mills_indels} \
	 -known ${dbsnp_vcf} \
	 -known ${known_indels} \
	 -o ${samtools_path}/1.realign/$1_$2_realign.bam
	
## BaseRecalibrator
	
java -Xmx4g -Xms4g -jar ${GATK3}/GenomeAnalysisTK.jar -T BaseRecalibrator \
	 -R ${ref_fasta} \
	 -nct 4 \
	 -knownSites ${dbsnp_vcf} \
	 -knownSites ${known_indels} \
	 -knownSites ${Mills_indels} \
	 -I ${samtools_path}/1.realign/$1_$2_realign.bam \
	 -o ${samtools_path}/2.BaseRecal/$1_$2_recal.table
	
## PrintReads
	
java -Xmx4g -Xms4g -jar ${GATK3}/GenomeAnalysisTK.jar -T PrintReads \
	 -R ${ref_fasta} \
	 -I ${samtools_path}/1.realign/$1_$2_realign.bam \
	 --BQSR ${samtools_path}/2.BaseRecal/$1_$2_recal.table \
	 -o ${samtools_path}/2.BaseRecal/$1_$2_recal.bam \
	 -nct 4
	
## samtoolls mpileup
	
bcftools mpileup --threads 4 -Ou -f ${ref_fasta} ${samtools_path}/2.BaseRecal/$1_$2_recal.bam | bcftools call --threads 4 -vmO b -o ${samtools_path}/3.Samtools/$1_$2_mpileup.bcf

bcftools view --threads 4 ${samtools_path}/3.Samtools/$1_$2_mpileup.bcf > ${samtools_path}/3.Samtools/$1_$2_samtools_raw.vcf

bgzip -c ${samtools_path}/3.Samtools/$1_$2_samtools_raw.vcf > ${samtools_path}/3.Samtools/$1_$2_samtools_raw.vcf.gz

tabix -p vcf ${samtools_path}/3.Samtools/$1_$2_samtools_raw.vcf.gz
	
bcftools filter --threads 4 -O z -o ${samtools_path}/3.Samtools/$1_$2_mpileup_filter.vcf.gz -s LOWQUAL -i '%QUAL>=20' ${samtools_path}/3.Samtools/$1_$2_samtools_raw.vcf.gz
```
