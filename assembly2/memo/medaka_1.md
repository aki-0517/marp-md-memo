フィルタ済みリードを使うことで、ノイズの多い短い・低品質リードを除外できるため、Medaka のポリッシング精度向上が期待できます。以下のように先ほどの手順を修正してください。

---

## 2′. フィルタ済みリードの利用

* フィルタ後のFASTQ
  `Dicty_filtered.fastq`

（※このファイルは `filtlong` コマンドで生成済みと仮定）

---

## 3′. Flye Assembly への Medaka ポリッシュ（修正版）

```bash
# 1) Minimap2 でマッピング（フィルタ済みリードを使用）
minimap2 -t 16 -ax map-ont \
  assembly2/assembly-results/flye/flye_polished_round1.fasta \
  Dicty_filtered.fastq \
  > flye_round1_ont.sam

# 2) SAM → BAM，ソート
samtools view -@8 -bS flye_round1_ont.sam \
  | samtools sort -@8 -o flye_round1_ont.sorted.bam

# 3) BAM のインデックス作成
samtools index flye_round1_ont.sorted.bam

# 4) Medaka コンセンサス生成
medaka_consensus \
  -i flye_round1_ont.sorted.bam \
  -d assembly2/assembly-results/flye/flye_polished_round1.fasta \
  -o medaka_flye_round2 \
  -t 16 \
  -m r941_min_high_g360
```

---

## 4′. Canu Assembly への Medaka ポリッシュ（修正版）

```bash
# 1) Minimap2 でマッピング（フィルタ済みリードを使用）
minimap2 -t 16 -ax map-ont \
  assembly2/assembly-results/canu/canu_polished_round1.fasta \
  Dicty_filtered.fastq \
  > canu_round1_ont.sam

# 2) SAM → BAM，ソート
samtools view -@8 -bS canu_round1_ont.sam \
  | samtools sort -@8 -o canu_round1_ont.sorted.bam

# 3) BAM のインデックス作成
samtools index canu_round1_ont.sorted.bam

# 4) Medaka コンセンサス生成
medaka_consensus \
  -i canu_round1_ont.sorted.bam \
  -d assembly2/assembly-results/canu/canu_polished_round1.fasta \
  -o medaka_canu_round2 \
  -t 16 \
  -m r941_min_high_g360
```

---

### ポイント

* **Dicty\_filtered.fastq** を指定することで、短い／低品質リードを除去したデータからマッピングを行います。
* Medaka モデル（`-m`）は使用しているフローセルに合わせて変更してください。
* 出力ディレクトリ内の `consensus.fasta` が最終ポリッシュ結果です。

これで、よりクリーンなリードを使った Medaka ポリッシングが実行できます。
