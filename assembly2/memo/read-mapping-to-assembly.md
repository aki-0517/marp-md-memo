はい、承知いたしました。

ONTリード（Dicty_gDNA_NEB-2.fastq）が、アセンブルされたゲノム（ragtag_polished_round1.fasta）にどれだけ含まれているかを確認し、アセンブリの正確性（網羅性）を評価する手順をまとめます。

この評価には、リードをリファレンスゲノムにマッピングし、そのマッピング率を計算します。ONTのようなロングリードには**Minimap2**が非常に適しており、一般的に推奨されます。**Bowtie2**は主にショートリード用に設計されていますが、設定を調整することで使用可能です。両方の手順を解説します。

---

### 準備

作業を始める前に、以下のツールがインストールされていることを確認してください。

- `minimap2`
    
- `bowtie2`
    
- `samtools`
    

---

### 推奨手順: Minimap2 を使用する方法

Minimap2はONTリードのアライメントに最適化されており、高速かつ正確です。

#### **ステップ1: リードのマッピング**

まず、`minimap2`を使ってFASTQファイルのリードをFASTAファイル（アセンブリ結果）にマッピングします。

- `-ax map-ont`: Oxford Nanoporeリードのマッピング用に最適化されたプリセットです。
    
- 出力はSAM形式になります。
    

Bash

```
minimap2 -ax map-ont \
  assembly2/assembly-results/ragtag_flye_scaffold/ragtag_polished_round1.fasta \
  Dicty_gDNA_NEB-2.fastq > mapping_minimap2.sam
```

#### **ステップ2: SAMをBAMに変換・ソート**

次に、`samtools`を使って、人間が読めるSAM形式を、コンピュータが処理しやすい圧縮されたバイナリ形式（BAM）に変換し、ゲノム上の位置でソートします。

Bash

```
samtools view -bS mapping_minimap2.sam | samtools sort -o mapping_minimap2_sorted.bam
```

- `samtools view -bS`: SAM (`-S`) をBAM (`-b`) に変換します。
    
- `samtools sort -o`: 出力ファイル名を指定してソートします。
    

#### **ステップ3: マッピング率の確認**

最後に、`samtools flagstat`を使ってマッピングの統計情報を表示します。これにより、何パーセントのリードがアセンブリ上にマッピングされたかがわかります。

Bash

```
samtools flagstat mapping_minimap2_sorted.bam
```

**出力例と見方:**

```
1234567 + 0 in total (QC-passed reads + QC-failed reads)
...
1198765 + 0 mapped (97.10% : N/A)
...
```

この出力の「**mapped**」と書かれた行のパーセンテージ（この例では **97.10%**）が、アセンブリに含まれるリードの割合です。この数値が高いほど、アセンブリが元のリード情報をよく保持していることを示します。




```bash
[aki@tardis ~]$ samtools flagstat mapping_minimap2_sorted.bam
1829791 + 0 in total (QC-passed reads + QC-failed reads)
459364 + 0 secondary
435541 + 0 supplementary
0 + 0 duplicates
1829607 + 0 mapped (99.99% : N/A)
0 + 0 paired in sequencing
0 + 0 read1
0 + 0 read2
0 + 0 properly paired (N/A : N/A)
0 + 0 with itself and mate mapped
0 + 0 singletons (N/A : N/A)
0 + 0 with mate mapped to a different chr
0 + 0 with mate mapped to a different chr (mapQ>=5)
```

- illuminaでやってみる

- bowtie2

- repeatmodulerの結果は使える


ONTリードの代わりにIlluminaのペアエンドリード（Stationary_S1_R1.fastq.gz と Stationary_S1_R2.fastq.gz）を用いて、アセンブルされたゲノム（ragtag_polished_round1.fasta）の網羅性を評価する手順を以下にまとめます。

Illuminaのようなショートリードのマッピングには、**Bowtie2**が非常に効果的で、標準的なツールとして広く利用されています。

---

### **準備**

作業を始める前に、ONTリードの評価と同様に、以下のツールがインストールされていることを確認してください。

- `bowtie2`
    
- `samtools`
    

---

### **推奨手順: Bowtie2 を使用する方法**(1608801.pts-1.tardis)

Bowtie2は、Illuminaから出力されるような短いリードをリファレンスゲノムに高速かつ正確にマッピングするために設計されています。

#### **ステップ1: Bowtie2用インデックスの作成**

まず、マッピングを行う前に、リファレンスゲノム（アセンブリ結果）からBowtie2が使用するインデックスファイルを作成する必要があります。この処理は最初の一度だけで結構です。

- `bowtie2-build`: FASTAファイルからインデックスを構築するコマンドです。
    
- 最初の引数にリファレンスゲノムのFASTAファイルを、2番目の引数にインデックスの基盤となる名前（任意の名前）を指定します。
    

Bash

```
# ragtag_polished_round1.fasta があるディレクトリで実行するか、
# 正しいパスを指定してください。
# ここではインデックス名を "ragtag_polished_genome" とします。
bowtie2-build assembly2/assembly-results/ragtag_flye_scaffold/ragtag_polished_round1.fasta ragtag_polished_genome
```

このコマンドを実行すると、同じディレクトリに`.bt2`という拡張子を持つ複数のファイルが生成されます。これらがインデックスファイルです。

#### **ステップ2: リードのマッピング**

次に、作成したインデックスを使用して、Illuminaのペアエンドリード（R1とR2）をマッピングします。

- `-x`: ステップ1で作成したインデックスの基盤となる名前を指定します。
    
- `-1`: ペアエンドリードのRead1ファイル（R1）を指定します。
    
- `-2`: ペアエンドリードのRead2ファイル（R2）を指定します。
    
- `-S`: 出力ファイルをSAM形式で指定します。
    
- （補足）入力ファイルが `.gz` 形式でも、Bowtie2は通常、自動で解凍して処理します。
    

Bash

```
bowtie2 -x ragtag_polished_genome \
  -1 ./Dicty-genome/Stationary_S1_R1.fastq.gz \
  -2 ./Dicty-genome/Stationary_S1_R2.fastq.gz \
  -S mapping_bowtie2.sam
```

#### **ステップ3: SAMをBAMに変換・ソート**

Minimap2の手順と同様に、`samtools`を使ってSAMファイルを圧縮・ソートし、効率的に扱えるBAMファイルに変換します。

Bash

```
samtools view -bS mapping_bowtie2.sam | samtools sort -o mapping_bowtie2_sorted.bam
```

#### **ステップ4: マッピング率の確認**

最後に、`samtools flagstat`を使ってマッピングの統計情報を取得します。

Bash

```
samtools flagstat mapping_bowtie2_sorted.bam
```

**出力例と見方:**

```
2500000 + 0 in total (QC-passed reads + QC-failed reads)
...
2450000 + 0 mapped (98.00% : N/A)
2500000 + 0 paired in sequencing
...
2400000 + 0 properly paired (96.00% : N/A)
...
```

この出力で注目すべきは以下の2点です。

- **`mapped`**: 全体のリードのうち、ゲノム上のどこかにマッピングできたリードの割合（この例では **98.00%**）。この値が高いほど、アセンブリがリード情報を網羅していることを示します。
    
- **`properly paired`**: マッピングされたリードのうち、ペアの向きや距離が期待通りに正しくマッピングされたものの割合（この例では **96.00%**）。この値もアセンブリの正確性を評価する上で重要な指標となります。
    

これらの手順により、Illuminaリードを用いてアセンブリの網羅性と正確性を評価することができます。
```bash
  164576795 (100.00%) were paired; of these:
    62654905 (38.07%) aligned concordantly 0 times
    70565108 (42.88%) aligned concordantly exactly 1 time
    31356782 (19.05%) aligned concordantly >1 times
    ----
    62654905 pairs aligned concordantly 0 times; of these:
      29866310 (47.67%) aligned discordantly 1 time
    ----
    32788595 pairs aligned 0 times concordantly or discordantly; of these:
      65577190 mates make up the pairs; of these:
        20741240 (31.63%) aligned 0 times
        11819615 (18.02%) aligned exactly 1 time
        33016335 (50.35%) aligned >1 times
93.70% overall alignment rate
```


はい、承知いたしました。

Illuminaのようなショートリードのマッピングにおいても、**Minimap2**は特定のプリセットを使用することで非常に高速に動作します。アセンブリの網羅性を素早く確認したい場合には優れた選択肢です。

ただし、一般的にSNP（一塩基多型）の検出など、より高いマッピング精度が求められる解析では**Bowtie2**や**BWA-MEM**が標準的に使われる点にご留意ください。網羅性の評価が目的であれば、Minimap2で全く問題ありません。

以下にMinimap2を使った手順をまとめます。

-----

### \#\# 準備

以前の手順と同様に、以下のツールがインストールされていることを確認してください。

  * `minimap2`
  * `samtools`

-----

### \#\# 推奨手順: Minimap2 を使用する方法 (ショートリード版)(326193.pts-1.tardis)

この手順では、マッピングからBAMファイルへの変換・ソートまでをパイプ `|` を使って一気に行い、効率化します。

#### **ステップ1: リードのマッピング、変換、ソート**

`minimap2`を実行し、その出力を直接 `samtools` に渡して処理を進めます。これにより、中間ファイルである巨大なSAMファイルを保存する必要がなくなり、ディスク容量と時間を節約できます。

  * `minimap2 -ax sr`: **ショートリード（Short Read）のマッピング用に最適化されたプリセット**です。この指定が重要です。
  * `-t 8`: 使用するCPUのスレッド（コア）数を指定します。数値が大きいほど処理が速くなりますが、PCのスペックに合わせて調整してください（例: 4, 8, 16）。
  * `samtools view -bS -`: 標準入力（`-`）からSAM形式を受け取り、BAM形式（`-b`）に変換します。
  * `samtools sort -o ...`: 入力されたBAMをソートし、指定したファイル名で出力します。

<!-- end list -->

```bash
# CPUコアを8個使用する例
minimap2 -ax sr -t 8 \
  assembly2/assembly-results/ragtag_flye_scaffold/ragtag_polished_round1.fasta \
  ./Dicty-genome/Stationary_S1_R1.fastq.gz \
  ./Dicty-genome/Stationary_S1_R2.fastq.gz | \
  samtools view -bS - | \
  samtools sort -o mapping_minimap2_sr_sorted.bam
```

この一行（見た目上は複数行）のコマンドを実行するだけで、ソート済みのBAMファイルが生成されます。

-----

#### **ステップ2: マッピング率の確認**

最後に、生成されたBAMファイルから `samtools flagstat` を使ってマッピングの統計情報を取得します。この手順はBowtie2の場合と全く同じです。

```bash
samtools flagstat mapping_minimap2_sr_sorted.bam
```

**出力例と見方:**

```
2500000 + 0 in total (QC-passed reads + QC-failed reads)
...
2445000 + 0 mapped (97.80% : N/A)
2500000 + 0 paired in sequencing
...
2395000 + 0 properly paired (95.80% : N/A)
...
```

Bowtie2の時と同様に、**`mapped`**（マッピングされたリードの割合）と **`properly paired`**（ペアとして正しくマッピングされたリードの割合）のパーセンテージを確認することで、アセンブリの網羅性を評価できます。

この方法を使えば、Bowtie2よりも高速に評価を完了できる可能性が高いです 🚀。

# 結果
```bash
(base) [aki@tardis ~]$ samtools flagstat mapping_minimap2_sr_sorted.bam
330259133 + 0 in total (QC-passed reads + QC-failed reads)
329153590 + 0 primary
0 + 0 secondary
1105543 + 0 supplementary
0 + 0 duplicates
0 + 0 primary duplicates
319240206 + 0 mapped (96.66% : N/A)
318134663 + 0 primary mapped (96.65% : N/A)
329153590 + 0 paired in sequencing
164576795 + 0 read1
164576795 + 0 read2
294941372 + 0 properly paired (89.61% : N/A)
307807028 + 0 with itself and mate mapped
10327635 + 0 singletons (3.14% : N/A)
3588200 + 0 with mate mapped to a different chr
1164893 + 0 with mate mapped to a different chr (mapQ>=5)
```






このコマンドは、マッピングからソートまでを中間ファイルなしで一気に実行し、成功した場合にのみ (`&&`)、作成されたBAMファイルに対して `flagstat` を実行します。(326193.pts-1.tardis)

Bash

```
minimap2 -ax map-ont /home/aki/ncbi_dataset/ncbi_dataset/data/GCF_000004695.1/GCF_000004695.1_dicty_2.7_genomic.fna Dicty_gDNA_NEB-2.fastq | samtools view -bS | samtools sort -o mapping_minimap2_sorted.bam && samtools flagstat mapping_minimap2_sorted.bam
```

---

### コマンドの解説

- **`minimap2 ... | samtools view -bS`**
    
    - `minimap2` のマッピング結果 (SAM形式) はファイルに保存されず、標準出力を通じて直接 `samtools view` に渡されます。
        
    - `samtools view` はそのSAM形式のデータをBAM形式に変換します。
        
- **`... | samtools sort -o mapping_minimap2_sorted.bam`**
    
    - `samtools view` が出力したBAMデータは、さらにパイプを通じて `samtools sort` に渡されます。
        
    - `samtools sort` はそのデータをゲノム位置で並べ替え、最終的な `mapping_minimap2_sorted.bam` ファイルとして保存します。
        
- **`&& samtools flagstat mapping_minimap2_sorted.bam`**
    
    - `&&` は、左側のコマンド（マッピングとソート）が**成功した場合にのみ**、右側のコマンドを実行します。
        
    - BAMファイルが無事に作成された後、`samtools flagstat` が実行され、マッピング率などの統計情報が表示されます。



```bash
minimap2 -ax map-ont /home/aki/ncbi_dataset/ncbi_dataset/data/GCF_000004695.1/GCF_000004695.1_dicty_2.7_genomic.fna Dicty_gDNA_NEB-2.fastq | samtools view -bS | samtools sort -o mapping_minimap2_sorted.bam && samtools flagstat mapping_minimap2_sorted.bam
```
```bash
1682093 + 0 in total (QC-passed reads + QC-failed reads)
934886 + 0 primary
273373 + 0 secondary
473834 + 0 supplementary
0 + 0 duplicates
0 + 0 primary duplicates
1680713 + 0 mapped (99.92% : N/A)
933506 + 0 primary mapped (99.85% : N/A)
0 + 0 paired in sequencing
0 + 0 read1
0 + 0 read2
0 + 0 properly paired (N/A : N/A)
0 + 0 with itself and mate mapped
0 + 0 singletons (N/A : N/A)
0 + 0 with mate mapped to a different chr
0 + 0 with mate mapped to a different chr (mapQ>=5)
```

```bash
minimap2 -ax sr -t 8 \ /home/aki/ncbi_dataset/ncbi_dataset/data/GCF_000004695.1/GCF_000004695.1_dicty_2.7_genomic.fna \ ./Dicty-genome/Stationary_S1_R1.fastq.gz \ ./Dicty-genome/Stationary_S1_R2.fastq.gz | \ samtools view -bS - | \ samtools sort -o mapping_minimap2_sr_sorted.bam && \ samtools flagstat mapping_minimap2_sr_sorted.bam
```
```bash
[bam_sort_core] merging from 35 files and 1 in-memory blocks...
331453393 + 0 in total (QC-passed reads + QC-failed reads)
329153590 + 0 primary
0 + 0 secondary
2299803 + 0 supplementary
0 + 0 duplicates
0 + 0 primary duplicates
305050986 + 0 mapped (92.03% : N/A)
302751183 + 0 primary mapped (91.98% : N/A)
329153590 + 0 paired in sequencing
164576795 + 0 read1
164576795 + 0 read2
277395756 + 0 properly paired (84.28% : N/A)
289577062 + 0 with itself and mate mapped
13174121 + 0 singletons (4.00% : N/A)
4152190 + 0 with mate mapped to a different chr
1835028 + 0 with mate mapped to a different chr (mapQ>=5)
```