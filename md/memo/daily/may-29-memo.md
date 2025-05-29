## マッピング～Pilon 実行
- ここでは `bash` を想定しています。

---

### 1. アセンブリのインデックス化（未実行なら）

```bash
bwa index flye_assembly/40-polishing/filtered_contigs.fasta
```

---

### 2. 短リードのマッピングと BAM 化

#### 方法 A: BWA が gzip を直接扱える場合

```bash
bwa mem -t 8 flye_assembly/40-polishing/filtered_contigs.fasta \
    ./Dicty-genome/Stationary_S1_R1.fastq.gz \
    ./Dicty-genome/Stationary_S1_R2.fastq.gz \
  | samtools view -bS - \
  | samtools sort -@ 8 -o pilon_round1.sorted.bam

samtools index pilon_round1.sorted.bam
```

#### 方法 B: プロセス置換で明示的に解凍する場合

```bash
bwa mem -t 8 flye_assembly/40-polishing/filtered_contigs.fasta \
    <(zcat ./Dicty-genome/Stationary_S1_R1.fastq.gz) \
    <(zcat ./Dicty-genome/Stationary_S1_R2.fastq.gz) \
  | samtools view -bS - \
  | samtools sort -@ 8 -o pilon_round1.sorted.bam

samtools index pilon_round1.sorted.bam
```

---

### 3. Pilon ポリッシング

```bash
java -Xmx16G -jar /usr/local/bin/pilon.jar \
  --genome flye_assembly/40-polishing/filtered_contigs.fasta \
  --frags pilon_round1.sorted.bam \
  --output pilon_round1_polished \
  --threads 8 \
  --changes \
  --vcf
```

---

#### 4. ２回目のラウンド（任意）

１回目で得られた `pilon_round1_polished.fasta` を再インデックスし、同様にマッピング→Pilon 実行を繰り返します。

```bash
bwa index pilon_round1_polished.fasta

bwa mem -t 8 pilon_round1_polished.fasta \
    <(zcat ./Dicty-genome/Stationary_S1_R1.fastq.gz) \
    <(zcat ./Dicty-genome/Stationary_S1_R2.fastq.gz) \
  | samtools view -bS - \
  | samtools sort -@ 8 -o pilon_round2.sorted.bam

samtools index pilon_round2.sorted.bam

java -Xmx16G -jar /usr/local/bin/pilon.jar \
  --genome pilon_round1_polished.fasta \
  --frags pilon_round2.sorted.bam \
  --output pilon_round2_polished \
  --threads 8 \
  --changes \
  --vcf
```

これで、Flye の最終フィルタ済みコンティグに対して Pilon ポリッシングが行えます。
