## Novoalign
#### need license
```
#!/bin/bash

source hg38_pipeline.cfg
	
novo_path='Novoalign/novocraft'

${novo_path}/novoindex ${ref_fasta_path}/ hg38_v0_Homo_sapiens_assembly38.nix ${ref_fasta}
	
${novo_path}/novoalign -d ${ref_fasta_path}/hg38_v0_Homo_sapiens_assembly38.nix -f $1_S1_L001_R1_001.fastq.gz $1_S1_L001_R2_001.fastq.gz -o SAM $'@RG\tID:Sub1\tPL:illumina\tSM:Sub1' > $Novo_output/$1_Novo.sam
	
samtools view -@ 64 -Sb ${Novo_output}/$1_Novo.sam > ${View_path}/$3_$1_Novo.bam
samtools sort -@ 64 ${View_path}/$3_$1_Novo.bam -o ${View_path}/$3_$1_Novo_sort.bam
samtools index -@ 64 ${View_path}/$3_$1_Novo_sort.bam
```
