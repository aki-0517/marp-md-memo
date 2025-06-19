以下の手順で、Flye で組み立てたアセンブリ（`assembly2/assembly-results/flye/assembly.fasta`）に対して、Illumina リードを使ったポリッシングを行うことができます。代表的には **Pilon** や **NextPolish** がよく使われますが、ここでは Pilon を例に示します。

---

## 1. 必要なツール・リソース

* **BWA**（リードマッピング用）
* **SAMtools**（SAM/BAM 操作用）
* **Picard**（マークドアップリケート用；任意）
* **Pilon**（ポリッシング）
* Java（Pilon 実行環境）
* Illumina PE リード

  * `./Dicty-genome/Stationary_S1_R1.fastq.gz`
  * `./Dicty-genome/Stationary_S1_R2.fastq.gz`
* Flye アセンブリ fasta

  * `assembly2/assembly-results/flye/assembly.fasta`

---

## 2. 前準備：インデックス作成

```bash
# 1) BWA インデックス
bwa index assembly2/assembly-results/flye/assembly.fasta

# 2) samtools faidx
samtools faidx assembly2/assembly-results/flye/assembly.fasta
```

---

## 3. Illumina リードのマッピング

```bash
# 1) マッピング
bwa mem -t 16 \
  assembly2/assembly-results/flye/assembly.fasta \
  ./Dicty-genome/Stationary_S1_R1.fastq.gz \
  ./Dicty-genome/Stationary_S1_R2.fastq.gz \
  > flye_illumina.sam

# 2) SAM→BAM，ソート
samtools view -bS flye_illumina.sam | \
  samtools sort -@ 8 -o flye_illumina.sorted.bam

# 3) インデックス作成
samtools index flye_illumina.sorted.bam
```

> **Tip:** 重複リードを除きたい場合は、Picard の `MarkDuplicates` を挟んでください。
>
> ```bash
> picard MarkDuplicates \
>   I=flye_illumina.sorted.bam \
>   O=flye_illumina.dedup.bam \
>   M=dup_metrics.txt \
>   REMOVE_DUPLICATES=true
> samtools index flye_illumina.dedup.bam
> ```

---

## 4. Pilon によるポリッシング

```bash
java -Xmx60G -jar pilon.jar \
  --genome assembly2/assembly-results/flye/assembly.fasta \
  --frags flye_illumina.sorted.bam \
  --output flye_polished_round1 \
  --threads 16 \
  --changes \
  --vcf
```

* `--changes`：変更ログを出力
* `--vcf`：バリアント情報を VCF で出力

---

## 5. 繰り返しポリッシング

ポリッシングは複数ラウンド行うと精度向上が期待できます。

1. `--genome` に前ラウンド（`flye_polished_round1.fasta`）を指定
2. BAM は再マッピングし直すか、そのまま使っても可
3. 2～3 ラウンド程度実施して、ミスマッチ／インデルが収束するまで繰り返します。

```bash
# 例：第2ラウンド
java -Xmx60G -jar pilon.jar \
  --genome flye_polished_round1.fasta \
  --frags flye_illumina.sorted.bam \
  --output flye_polished_round2 \
  --threads 16
```

---

## 6. ポリッシング後の評価

* **QUAST** や **BUSCO** で塩基精度（ミスマッチ率、ギャップ率）、完全性（BUSCO スコア）の改善を確認。
* VCF を見て、過補正していないかチェック。

---

## 代替ツールのご紹介

* **NextPolish**：`nextpolish.cfg` を用いて、短リード＋長リードの両方を使ったポリッシングが可能
* **Polca（MaSuRCA）**：高速かつ自動化レベルが高い短リードポリッシング
* **Racon + Pilon**：まず Racon（長リード対応）で軽く補正してから Pilon を回す二段階ポリッシング

---

### まとめ

1. Flye アセンブリ fasta を BWA＋samtools で整形
2. Illumina PE リードをマッピング → ソート・インデックス
3. Pilon でラウンドポリッシング
4. 評価ツールで改善度合いを確認
5. 必要に応じて 2–3 ラウンド繰り返し

この流れで、Illumina 短リードならではの高い塩基精度を Flye アセンブリに付与できます。お試しください！


### Flye 
4062777.pts-46.tardis  
```bash
bwa mem -t 16 \
  assembly2/assembly-results/flye/assembly.fasta \
  ./Dicty-genome/Stationary_S1_R1.fastq.gz \
  ./Dicty-genome/Stationary_S1_R2.fastq.gz \
  > flye_illumina.sam && \
samtools view -bS flye_illumina.sam | \
  samtools sort -@ 8 -o flye_illumina.sorted.bam && \
samtools index flye_illumina.sorted.bam && \
java -Xmx60G -jar pilon.jar \
  --genome assembly2/assembly-results/flye/assembly.fasta \
  --frags flye_illumina.sorted.bam \
  --output flye_polished_round1 \
  --threads 16 \
  --changes \
  --vcf

```


### Canu
4065337.pts-1.tardis  
```bash
bwa index assembly2/assembly-results/canu/dicty.contigs.fasta && \
samtools faidx assembly2/assembly-results/canu/dicty.contigs.fasta && \
bwa mem -t 16 \
  assembly2/assembly-results/canu/dicty.contigs.fasta \
  ./Dicty-genome/Stationary_S1_R1.fastq.gz \
  ./Dicty-genome/Stationary_S1_R2.fastq.gz \
  > canu_illumina.sam && \
samtools view -bS canu_illumina.sam | \
  samtools sort -@ 8 -o canu_illumina.sorted.bam && \
samtools index canu_illumina.sorted.bam && \
java -Xmx60G -jar pilon.jar \
  --genome assembly2/assembly-results/canu/dicty.contigs.fasta \
  --frags canu_illumina.sorted.bam \
  --output canu_polished_round1 \
  --threads 16 \
  --changes \
  --vcf

```