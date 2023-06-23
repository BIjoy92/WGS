## Strelka2

```
#!/bin/bash
	
source hg38_pipeline.cfg
	
BED_path=WGS/
mkdir ${strelka2_path}/$3_$1_$2_output
	
call_start=`date +%Y.%m.%d.%H:%M`
	
configureStrelkaGermlineWorkflow.py \
--bam ${View_path}/$1_$2_sort.bam \
--callRegions ${BED_path}/without_decoy.hg38.bed.gz \
--referenceFasta ${ref_fasta} \
--runDir ${strelka2_path}/$3_$1_$2_output/strelkaGermlineWorkflow

${strelka2_path}/$3_$1_$2_output/strelkaGermlineWorkflow/runWorkflow.py -m local -j 32
```
