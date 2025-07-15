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

```bash
quast.py   -o quast_canu_comparison_medaka   -R /home/aki/ncbi_dataset/ncbi_dataset/data/GCF_000004695.1/GCF_000004695.1_dicty_2.7_genomic.fna   /home/aki/assembly2/assembly-results/canu/dicty.contigs.fasta   /home/aki/assembly2/assembly-results/ragtag_canu_scaffold/ragtag.scaffold.fasta   /home/aki/assembly2/assembly-results/ragtag_canu_scaffold/canu_medaka_polished.fasta   --threads 16   --labels canu_raw,canu_scaffold,canu_medaka_polished
```
```bash
RepeatMasker \
  -pa 8 \
  -species "dictyostelium discoideum" \
  -xsmall \
  /home/aki/assembly2/assembly-results/ragtag_flye_scaffold/ragtag_medaka_polished.fasta

```

```bash
quast.py \
  -r /home/aki/ncbi_dataset/ncbi_dataset/data/GCF_000004695.1/GCF_000004695.1_dicty_2.7_genomic.fna \
  -o /home/aki/quast_repeatmasker_comparison \
  -t 8 \
  -l \
    reference,flye_beforeRM,flye_afterRM,canu_beforeRM,canu_afterRM \
  /home/aki/ncbi_dataset/ncbi_dataset/data/GCF_000004695.1/GCF_000004695.1_dicty_2.7_genomic.fna \
  /home/aki/assembly2/assembly-results/ragtag_flye_scaffold/ragtag_medaka_polished.fasta \
  /home/aki/assembly2/assembly-results/ragtag_flye_scaffold/ragtag_medaka_polished.fasta.masked \
  /home/aki/assembly2/assembly-results/ragtag_canu_scaffold/canu_medaka_polished.fasta \
  /home/aki/assembly2/assembly-results/ragtag_canu_scaffold/canu_medaka_polished.fasta.masked
```

https://genomebiology.biomedcentral.com/articles/10.1186/s13059-025-03578-7#Sec11:~:text=Repetitive%20sequence%20annotation,of%20each%20chromosome.

やるべきこと

repetetive region探す(片方で)
その後 over探す


1x2の方が長い(repetetiveがいっぱいある)


```bash
bedtools getfasta \
  -fi assembly2/assembly-results/ragtag_flye_scaffold/ragtag_medaka_polished.fasta \
  -bed canu_unique.bed \
  -fo canu_unique.fa && \
prokka \
  --outdir prokka_canu_unique \
  --prefix canu_unique \
  --cpus 8 \
  --compliant \
  --force \
  canu_unique.fa && \
blastn \
  -query canu_unique.fa \
  -db dicty_nucl \
  -out prokka_canu_unique/canu_unique.blastn.tsv \
  -outfmt "6 qseqid sseqid pident length evalue bitscore stitle" \
  -num_threads 8


```


```bash
(prokka_env) [aki@tardis ~]$ ls -lh prokka_canu_unique/
合計 4.5M
-rw-rw-r-- 1 aki aki 1.2M  7月 15 12:04 canu_unique.blastn.tsv
-rw-rw-r-- 1 aki aki  33K  7月 15 12:04 canu_unique.err
-rw-rw-r-- 1 aki aki  44K  7月 15 12:04 canu_unique.faa
-rw-rw-r-- 1 aki aki 122K  7月 15 12:04 canu_unique.ffn
-rw-rw-r-- 1 aki aki 202K  7月 15 12:04 canu_unique.fna
-rw-rw-r-- 1 aki aki 203K  7月 15 12:04 canu_unique.fsa
-rw-rw-r-- 1 aki aki 397K  7月 15 12:04 canu_unique.gbk
-rw-rw-r-- 1 aki aki 266K  7月 15 12:04 canu_unique.gff
-rw-rw-r-- 1 aki aki 1.3M  7月 15 12:04 canu_unique.log
-rw-rw-r-- 1 aki aki 732K  7月 15 12:04 canu_unique.sqn
-rw-rw-r-- 1 aki aki  39K  7月 15 12:04 canu_unique.tbl
-rw-rw-r-- 1 aki aki  14K  7月 15 12:04 canu_unique.tsv
-rw-rw-r-- 1 aki aki   85  7月 15 12:04 canu_unique.txt
```

