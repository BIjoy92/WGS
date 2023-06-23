## Deepvariant

```
#!/bin/bash
	
source hg38_pipeline.cfg
	
BIN_VERSION="0.10.0"
N_SHARDS="32"
	
mkdir $BASE/output
OUTPUT_DIR="$BASE/output"
	
docker run \
-v "${INPUT_DIR}":"/input" \
-v "${OUTPUT_DIR}:/output" \
google/deepvariant:"${BIN_VERSION}" \
/opt/deepvariant/bin/run_deepvariant \
--model_type=WGS \
--ref=/input/hg38_v0_Homo_sapiens_assembly38.fasta \
--reads=/input/$1_$2_sort.bam \
--output_vcf=/output/$3_$1_$2_Deepvariant.output.vcf.gz \
--output_gvcf=/output/$3_$1_$2_Deepvariant.output.g.vcf.gz \
--num_shards=$N_SHARDS

docker system prune -f
```
