## BWA

```
#!/bin/bash

source hg38_pipeline.cfg
	
bwa index ${ref_fasta}
	
bwa mem ${ref_fasta} $1_R1.fastq.gz $1_R2.fastq.gz -R '@RG\tID:'$2'\tSM:'$2'\tPL:ILLUMINA' -t 64 > ${BWA_output}/$2_BWA.sam
	
samtools view -@ 64 -Sb ${BWA_output}/$2_BWA.sam > ${View_path}/$2_BWA.bam
samtools sort -@ 64 ${View_path}/$2_BWA.bam -o ${View_path}/$2_BWA_sort.bam
samtools index -@ 64 ${View_path}/$2_BWA_sort.bam

```
