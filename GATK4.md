## GATK4

```
#!/bin/bash

source hg38_pipeline.cfg

## BaseRecalibrator
	
baserecal_start=`date +%Y.%m.%d.%H:%M`
	
list="1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 X Y M"
	
for i in $list
do

 outfile=$GATK4_path/1.BQSR/$3_$1_$2_recal_data_$i.table
 gatk --java-options "-Xmx8G -XX:+UseParallelGC -XX:ParallelGCThreads=8 -XX:-UsePerfData" BaseRecalibrator \
 -R $ref_fasta \
 -I ${View_path}/$1_$2_sort.bam \
 -O $outfile \
 -L chr$i \
 --known-sites ${dbsnp_vcf} \
 --known-sites ${known_indels} \
 --known-sites ${Mills_indels} \
 --tmp-dir ${tmp_dir} &
	
done
	
wait
	
list="1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 X Y M"
	
for i in $list
do
        gatk --java-options "-Xmx8G -XX:-UsePerfData" GatherBQSRReports \
        -I $GATK4_path/1.BQSR/$3_$1_$2_recal_data_$i.table \
        -O $GATK4_path/1.BQSR/$3_$1_$2_GatherBQSR_output.table
done

mkdir $GATK4_path/2.ApplyBQSR

rm -rf $GATK4_path/1.BQSR/$3_$1_$2_recal_data_*.table

for i in $list
do

## Apply Base Quality Score Recalibration model

 gatk --java-options "-Xmx8G -XX:-UsePerfData" ApplyBQSR \
 -R $ref_fasta \
 -I ${View_path}/$1_$2_sort.bam \
 -L chr$i \
 -bqsr-recal-file $GATK4_path/1.BQSR/$3_$1_$2_GatherBQSR_output.table  \
 -O $GATK4_path/2.ApplyBQSR/$3_$1_$2_BQSR_chr$i.bam &

done

wait
	
mkdir $GATK4_path/3.GatherBam

find -name "$3_$1_$2_BQSR_chr*.bam" > $3_$1_$2_bqsr.list
	
java -Xmx8g -Djava.io.tmpdir=$tmp_dir -jar $picard/picard.jar GatherBamFiles I=$3_$1_$2_bqsr.list O=$GATK4_path/3.GatherBam/$3_$1_$2_gather_BAM.bam
	
samtools index -@ 64 $GATK4_path/3.GatherBam/$3_$1_$2_gather_BAM.bam

## Haplotype Caller
	
mkdir $GATK4_path/4.GATK4_HaploCall
	
list="1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 X Y M"
	
for i in $list
	
do
	
 gatk --java-options "-Xmx8g -XX:-UsePerfData" HaplotypeCaller -R $ref_fasta --dbsnp ${dbsnp_vcf} -I $GATK4_path/3.GatherBam/$3_$1_$2_gather_BAM.bam -O $GATK4_path/4.GATK4_HaploCall/$3_$1_$2_chr$i.g.vcf.gz -L chr$i -ERC GVCF -ip 100 --pcr-indel-model NONE -G StandardAnnotation -G AS_StandardAnnotation --RF OverclippedReadFilter --filter-too-short 25 --max-alternate-alleles 3 --pairHMM AVX_LOGLESS_CACHING_OMP --native-pair-hmm-threads 8 --tmp-dir $tmp_dir &
	
done
	
wait
	
## filter VCF
	
mkdir $GATK4_path/5.FilterVCF
	
list="1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 X Y M"
	
for i in $list
do
	
	       java -Xmx8g -Xms8g -Djava.io.tmpdir=$tmp_dir -jar $picard/picard.jar FilterVcf I=$GATK4_path/4.GATK4_HaploCall/$3_$1_$2_chr$i.g.vcf.gz O=$GATK4_path/5.FilterVCF/$3_$1_$2_filter_chr$i.g.vcf.gz &
	
done
	
wait
	
## Merge VCF
	
java -Xmx8g -Xms8g -jar $picard/picard.jar MergeVcfs \
 I=$GATK4_path/6.MergeVCF/$3_$1_$2_vcf.list \
 O=$GATK4_path/6.MergeVCF/$3_$1_$2_mergeVCF.g.vcf.gz

## Genotype gvcf

mkdir $GATK4_path/7.Genotype

gvcf_start=`date +%Y.%m.%d.%H:%M`
	
gatk --java-options "-Xmx8g -XX:-UsePerfData" GenotypeGVCFs \
	 -R $ref_fasta \
	 -V $GATK4_path/6.MergeVCF/$3_$1_$2_mergeVCF.g.vcf.gz \
	 -O $GATK4_path/7.Genotype/$3_$1_$2_gvcf.vcf.gz \
	 --tmp-dir $tmp_dir
	
tabix -p vcf $GATK4_path/7.Genotype/$3_$1_$2_gvcf.vcf.gz
```
