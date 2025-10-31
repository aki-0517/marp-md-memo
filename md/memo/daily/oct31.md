
# chromesome1
## canu 
![[Pasted image 20251029175328.png]]


## flye
![[Pasted image 20251031140744.png]]



# chromesome2
## canu
![[Pasted image 20251031150431.png]]

## flye
![[Pasted image 20251031150909.png]]


# chromesome6


## canu




## flye
![[Pasted image 20251031143159.png]]
![[Pasted image 20251031143548.png]]









```bash
# For flye assembly
minimap2 -ax map-ont \
  assembly2/assembly-results/ragtag_flye_scaffold/ragtag_polished_round1.fasta \
  Dicty_gDNA_NEB-2.fastq | \
  samtools view -F 2308 -q 30 -bS - | \
  samtools sort -l 9 -o mapping_final_q30.sorted.bam && \
samtools index mapping_final_q30.sorted.bam

# For canu assembly
minimap2 -ax map-ont \
  assembly2/assembly-results/ragtag_canu_scaffold/canu_polished_round1.fasta \
  Dicty_gDNA_NEB-2.fastq | \
  samtools view -F 2308 -q 30 -bS - | \
  samtools sort -l 9 -o mapping_final_canu_q30.sorted.bam && \
samtools index mapping_final_canu_q30.sorted.bam
```