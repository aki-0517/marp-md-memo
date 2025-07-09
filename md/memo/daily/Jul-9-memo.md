```bash
quast.py \
  -o quast_ragtag_comparison \
  -R /home/aki/ncbi_dataset/ncbi_dataset/data/GCF_000004695.1/GCF_000004695.1_dicty_2.7_genomic.fna \
  --threads 16 \
  /home/aki/assembly2/assembly-results/flye/assembly.fasta \
  /home/aki/assembly2/assembly-results/medaka_flye_round2/consensus.fasta \
  /home/aki/assembly2/assembly-results/ragtag_flye_scaffold/ragtag.scaffold.fasta \
  /home/aki/assembly2/assembly-results/ragtag_flye_scaffold/ragtag_polished_round1.fasta \
  /home/aki/assembly2/assembly-results/ragtag_flye_scaffold/ragtag_medaka_polished.fasta \
  -l raw_flye,medaka_round2,scaffold_only,pilon_polished,medaka_polished
  ```

  ```bash
  cd /home/aki/assembly2/assembly-results/ragtag_canu_scaffold && \

# 1) Pilon ポリッシュ（BWA→SAM→BAM→Pilon）  
bwa index ragtag.scaffold.fasta && \
samtools faidx ragtag.scaffold.fasta && \
bwa mem -t 16 ragtag.scaffold.fasta \
    /home/aki/Dicty-genome/Stationary_S1_R1.fastq.gz \
    /home/aki/Dicty-genome/Stationary_S1_R2.fastq.gz \
  | samtools view -bS - \
  | samtools sort -@8 -o canu_illumina.sorted.bam && \
samtools index canu_illumina.sorted.bam && \
java -Xmx60G -jar /home/aki/miniconda3/envs/pilon-env/share/pilon-1.24-0/pilon.jar \
    --genome ragtag.scaffold.fasta \
    --frags canu_illumina.sorted.bam \
    --output canu_polished_round1 \
    --threads 16 \
    --changes \
    --vcf && \

# 2) Medaka ポリッシュ  
conda run -n medaka-env medaka_consensus \
  -i /home/aki/Dicty_filtered.fastq \
  -d canu_polished_round1.fasta \
  -o medaka_canu_polished_round1 \
  -t 16 \
  -m r104_e81_sup_g610 && \

# 3) 最終コンセンサスを所望の名前でコピー  
cp medaka_canu_polished_round1/consensus.fasta canu_medaka_polished.fasta

```