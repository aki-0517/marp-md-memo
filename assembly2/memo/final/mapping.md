```bash
minimap2 -ax map-ont \
  flye_repaired_assembly_v4.5_final/flye_assembly_FIXED_v4.5.fasta \
  Dicty_gDNA_NEB-2.fastq | \
  samtools view -bS - | \
  samtools sort -o mapping_flye_fixed.sorted.bam && \
samtools index mapping_flye_fixed.sorted.bam
```