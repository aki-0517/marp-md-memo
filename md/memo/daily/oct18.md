承知いたしました。

```bash
--- Step 1: Extracting FASTA sequences ---
--- Step 2: Creating genomes file for plotsr ---
--- Step 3: Running nucmer, syri, and plotsr pipeline ---
Reading Coords - WARNING - Chromosomes IDs do not match.
Reading Coords - WARNING - Matching them automatically. For each reference genome, most similar query genome will be selected. Check mapids.txt for mapping used.
Reading Coords - WARNING - Reference chromosome NC_007089.4_RagTag_pilon has high fraction of inverted alignments with its homologous chromosome in the query genome (NC_007090.3_RagTag_pilon). Ensure that same chromosome-strands are being compared in the two genomes, as different strand can result in unexpected errors.
/home/aki/miniconda3/envs/syri_env/lib/python3.11/multiprocessing/pool.py:48: FutureWarning: Series.__getitem__ treating keys as positions is deprecated. In a future version, integer keys will always be treated as labels (consistent with DataFrame behavior). To access a value by position, use `ser.iloc[pos]`
  return list(map(*args))
syri.NC_007089.4_RagTag_pilon - ERROR - No syntenic region found for chromosome: NC_007089.4_RagTag_pilon. This is potentially caused by the two assemblies having different strands for this chromosomes. Reverse complement the chromosome to ensure that the same strands are analysed. Exiting.
usage: Plotting structural rearrangements between genomes [-h] [--sr SR] [--bp BP] --genomes GENOMES [--markers MARKERS] [--tracks TRACKS] [--chrord CHRORD] [--chrname CHRNAME] [-o O] [--itx] [--chr CHR]
                                                          [--reg REG] [--rtr] [--nosyn] [--noinv] [--notr] [--nodup] [-s S] [--cfg CFG] [-R] [-f F] [-H H] [-W W] [-S S] [-d D]
                                                          [-b {agg,cairo,pdf,pgf,ps,svg,template}] [-v] [--lf LOGFIN] [--log {DEBUG,INFO,WARN}] [--version]
Plotting structural rearrangements between genomes: error: argument --sr: can't open 'syri_flye89_vs_canu90syri.out': [Errno 2] No such file or directory: 'syri_flye89_vs_canu90syri.out'
```
前回のsyriエラーを解決するため、問題となっていたクエリ側のコンティグ (canu_NC_007090.3.fasta) を逆相補鎖に変換するステップを組み込んだ修正版のコマンドです。

---

### 修正版コマンド

元のスクリプトに数行の修正・追加を行っています。ハイライトされた行が主な変更点です。

Bash

```
#!/bin/bash

# --- 変数の設定 ---
REF_ORIG="assembly2/assembly-results/ragtag_flye_scaffold/ragtag_polished_round1.fasta"
QRY_ORIG="assembly2/assembly-results/ragtag_canu_scaffold/canu_polished_round1.fasta"
REF_CONTIG="NC_007089.4_RagTag_pilon"
QRY_CONTIG="NC_007090.3_RagTag_pilon"
REF_TARGET="flye_NC_007089.4.fasta"
QRY_TARGET="canu_NC_007090.3.fasta"
# 変更点①: 逆相補鎖にしたクエリFASTA用の新しいファイル名を定義
QRY_TARGET_RC="canu_NC_007090.3.rc.fasta"
PREFIX_TARGET="flye89_vs_canu90"
OUT_PREFIX_TARGET="syri_flye89_vs_canu90"
GENOMES_FILE="genomes_target.txt"
OUTPUT_PDF="plot_flye89_vs_canu90.pdf"

# --- 以下、コマンド本体 ---
echo "--- Step 1: Extracting FASTA sequences and applying reverse complement ---" && \
samtools faidx ${REF_ORIG} ${REF_CONTIG} > ${REF_TARGET} && \
samtools faidx ${QRY_ORIG} ${QRY_CONTIG} > ${QRY_TARGET} && \
\
# 変更点②: seqtkを使ってクエリコンティグを逆相補鎖に変換し、新しいファイルに保存
echo "--- Applying reverse complement to query contig ---" && \
seqtk seq -r ${QRY_TARGET} > ${QRY_TARGET_RC} && \
\
echo "--- Step 2: Creating genomes file for plotsr ---" && \
# 変更点③: plotsr用のファイルリストを、逆相補鎖にしたファイル名に修正
echo -e "${QRY_TARGET_RC}\n${REF_TARGET}" > ${GENOMES_FILE} && \
\
echo "--- Step 3: Running nucmer, syri, and plotsr pipeline ---" && \
# 変更点④: nucmerの入力ファイルを逆相補鎖のFASTAファイルに修正
nucmer --maxmatch --prefix=${PREFIX_TARGET} ${REF_TARGET} ${QRY_TARGET_RC} && \
\
delta-filter -1 ${PREFIX_TARGET}.delta > ${PREFIX_TARGET}.filter.delta && \
show-coords -THrd ${PREFIX_TARGET}.filter.delta > ${PREFIX_TARGET}.coords && \
\
# 変更点⑤: syriの入力(-q)を逆相補鎖のFASTAファイルに修正
syri -c ${PREFIX_TARGET}.coords \
     -d ${PREFIX_TARGET}.filter.delta \
     -r ${REF_TARGET} \
     -q ${QRY_TARGET_RC} \
     --prefix ${OUT_PREFIX_TARGET} && \
\
plotsr --sr ${OUT_PREFIX_TARGET}syri.out \
       --genomes ${GENOMES_FILE} \
       -H 10 -W 15 \
       -o ${OUTPUT_PDF} && \
\
echo "--- Pipeline finished successfully! Check ${OUTPUT_PDF} ---"
```

---

### 主な変更点の解説

1. 新しいファイル名の定義 (QRY_TARGET_RC):
    
    逆相補鎖にしたFASTAファイルを保存するための新しい変数 QRY_TARGET_RC を追加しました。これにより、元のファイルと修正後のファイルが混ざるのを防ぎます。
    
2. seqtk seq -rコマンドの追加:
    
    samtools faidxでクエリコンティグを抽出した後、seqtk seq -r ${QRY_TARGET} > ${QRY_TARGET_RC} を実行して塩基配列を逆相補鎖に変換し、新しいファイル (canu_NC_007090.3.rc.fasta) に保存します。
    
3. 入力ファイルの差し替え:
    
    以降の nucmer, syri, そしてplotsr用のゲノムリストファイル (genomes_target.txt) でクエリゲノムを指定する箇所を、すべて新しい逆相補鎖ファイル (${QRY_TARGET_RC}) に差し替えました。
    

この修正版スクリプトを実行することで、`syri`がストランドの向きの不一致でエラーを起こすことなく、パイプラインが最後まで正常に完了するはずです。



17行目から27行目のテーブルから括弧と色のスタイルを削除した版を提案します：

| Parameter              | Flye_Polished | Canu_Polished |
| ---------------------- | ------------: | ------------: |
| contigs (>= 0 bp)      |            11 |            29 |
| Total length (>= 0 bp) |      34861082 |      35997303 |
| N50                    |       8060236 |       5606788 |
| Genome coverage (%)    |        97.172 |        97.059 |
| Duplication ratio      |         1.035 |         1.066 |

このように、HTMLの`<span>`タグと色指定、および括弧内の数値変化をすべて削除し、シンプルな数値のみのテーブルにしました。



はい、承知いたしました。Oxford Nanopore (ONT) のようなロングリードをマッピングする場合は、**Bowtie2** ではなく **Minimap2** を使用するのが一般的であり、精度と速度の面で強く推奨されます。

以下に、`Dicty_gDNA_NEB-2.fastq` をリファレンスゲノムにマッピングする手順を Minimap2 を使って示します。

---

### **ステップ1: Minimap2によるマッピングとBAMへの変換・ソート**

Minimap2は、ONTリードのマッピングに最適化されたツールです。Bowtie2のように事前にインデックスを作成する必要はなく、マッピングからBAMファイルの生成、ソートまでを一つのコマンドで効率的に実行できます。

- `minimap2`: ロングリード用のアライナーです。
    
- `-a`: 出力形式をSAM形式に指定します。
    
- `-x map-ont`: 入力データがONTのゲノムリードであることを示すプリセットです。この指定により、最適なパラメータが自動で選択されます。
    
- `samtools sort`: `minimap2`からの出力をパイプ(`|`)で受け取り、直接ソート済みBAMファイルとして出力します。
    

#### **Bashコマンド**

Bash

```
# minimap2でマッピングし、その結果を直接samtoolsでソート済みBAMファイルに変換します。
# 途中で巨大なSAMファイルが生成されないため、効率的です。

minimap2 -ax map-ont \
  assembly2/assembly-results/ragtag_flye_scaffold/ragtag_polished_round1.fasta \
  ./Dicty_gDNA_NEB-2.fastq | \
  samtools sort -o mapping_ont_sorted.bam
```

---

### **ステップ2: マッピング率の確認**

最後に、生成されたBAMファイルを使って `samtools flagstat` を実行し、マッピングの統計情報を確認します。この手順はIlluminaの時と全く同じです。

#### **Bashコマンド**

Bash

```
samtools flagstat mapping_ont_sorted.bam
```

このコマンドを実行すると、リードの総数、マッピングされたリードの数や割合などが表示されます。

### **まとめ**

|**目的**|**Illumina (ショートリード)**|**ONT (ロングリード)**|
|---|---|---|
|**推奨ツール**|`Bowtie2`|`Minimap2`|
|**インデックス作成**|`bowtie2-build` で事前に作成|不要|
|**マッピングコマンド**|`bowtie2 -x ... -1 ... -2 ...`|`minimap2 -ax map-ont ...`|

この手順で、ONTリードのマッピングをスムーズに進めることができます。

```bash
minimap2 -ax map-ont \
  assembly2/assembly-results/ragtag_flye_scaffold/ragtag_polished_round1.fasta \
  ./Dicty_gDNA_NEB-2.fastq | \
  samtools sort -o mapping_ont_sorted.bam && \
  samtools flagstat mapping_ont_sorted.bam
```