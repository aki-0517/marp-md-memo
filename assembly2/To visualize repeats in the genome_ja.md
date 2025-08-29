RepeatModeler 自体は BAM や BED ファイルを提供しません。生成するのはリピートライブラリ（FASTA のコンセンサス配列）のみです。可視化は RepeatMasker と TRF の出力から行い、それらを可視化しやすい形式へ変換する必要があります。

## 可視化用ファイルの取得

### RepeatMasker 出力の変換

**RepeatMasker の出力:**

- `.out` ファイル（詳細なアノテーション）
- `.gff` ファイル（オプション指定時）
- 直接の BED/BAM 出力はなし

**RepeatMasker の .out を BED に変換:**

```bash
# 基本的な変換
awk 'NR>3 {print $5"\t"$6-1"\t"$7"\t"$10"_"$11"\t"$1"\t"$9}' \
    RM_masked_output/ragtag_polished_round1.fasta.out > RM_masked_output/all_repeats.bed

# リピートクラスごとに色分け
awk 'NR>3 {
    if($11 ~ /LINE/) color="255,0,0"        # LINE を赤
    else if($11 ~ /SINE/) color="0,255,0"   # SINE を緑
    else if($11 ~ /LTR/) color="0,0,255"    # LTR を青
    else if($11 ~ /Satellite/) color="255,0,255" # Satellite をマゼンタ
    else color="128,128,128"                 # その他は灰色
    print $5"\t"$6-1"\t"$7"\t"$10"_"$11"\t"$1"\t"$9"\t"$6-1"\t"$7"\t"color
}' RM_masked_output/ragtag_polished_round1.fasta.out > RM_masked_output/repeats_colored.bed
```

**RepeatMasker から GFF を生成:**

```bash
# RepeatMasker の組み込み GFF 変換を利用
RepeatMasker -lib mygenome-families.fa -gff -pa 8 genome.fasta

# genome.fasta.out.gff が作成される
```

### TRF 出力の変換
```bash
# 0) TRF の用意（未導入なら）
# conda install -c bioconda trf

# 1) 変数に入力FASTAを設定（未マスクがあれば差し替え）
GENOME=RM_masked_output/ragtag_polished_round1.fasta.masked

# 2) 出力置き場
mkdir -p trf_results

# 3) TRF 実行（-d: .dat出力, -h: ヒットのヘッダ省略）
trf "$GENOME" 2 5 5 80 10 40 2000 -d -h

# 4) 生成された .dat の場所とファイル名を確認
# 例: RM_masked_output/ragtag_polished_round1.fasta.masked.2.5.5.80.10.40.2000.dat
ls -lh RM_masked_output/*.2.5.5.80.10.40.2000.dat | cat

# 5) trf_results/ に移動（以降の手順とパスを揃える）
mv RM_masked_output/$(basename "$GENOME").2.5.5.80.10.40.2000.dat trf_results/
```

**TRF を BED に変換:**

```bash
# タンデムリピートの基本 BED（安全な書き方）
awk 'BEGIN{OFS="\t"} {print $1,$2-1,$3,$14"("$4"x)",$8,"."}' \
    trf_results/ragtag_polished_round1.fasta.masked.2.5.5.80.10.40.2000.dat > tandem_repeats.bed

# 高スコアのタンデムリピートのみ
awk 'BEGIN{OFS="\t"} $8>500 {print $1,$2-1,$3,$14"_score"$8,$8,"."}' \
    trf_results/ragtag_polished_round1.fasta.masked.2.5.5.80.10.40.2000.dat > high_score_tandems.bed
```

## IGV 向けファイルの作成

### マルチトラック BED ファイル

```bash
# リピートタイプ毎に別トラックを作成
mkdir igv_tracks

# サテライトリピート
awk 'NR>3 && $11 ~ /Satellite/ {print $5"\t"$6-1"\t"$7"\t"$10"\t"$1"\t"$9}' \
    RM_masked_output/ragtag_polished_round1.fasta.out > igv_tracks/satellites.bed

# 転移因子（TE）
awk 'NR>3 && ($11 ~ /LINE|SINE|LTR|DNA/) {print $5"\t"$6-1"\t"$7"\t"$10"\t"$1"\t"$9}' \
    RM_masked_output/ragtag_polished_round1.fasta.out > igv_tracks/transposons.bed

# 不明/未知のリピート（セントロメア候補）
awk 'NR>3 && $11 ~ /Unknown|Unclassified/ {print $5"\t"$6-1"\t"$7"\t"$10"\t"$1"\t"$9}' \
    RM_masked_output/ragtag_polished_round1.fasta.out > igv_tracks/novel_repeats.bed

# 高コピーリピート（セントロメア候補）
awk 'NR>3 && $1 > 1000 {print $5"\t"$6-1"\t"$7"\t"$10"_copy"$1"\t"$1"\t"$9}' \
    RM_masked_output/ragtag_polished_round1.fasta.out > igv_tracks/high_copy_repeats.bed
```

### リピート密度の BigWig を作成

```bash
# 必要ならツールをインストール
# conda install -c bioconda bedtools ucsc-bedgraphtobigwig

# genome サイズファイルを作成（事前に .fai を作成）
# samtools faidx RM_masked_output/ragtag_polished_round1.fasta.masked
awk '{print $1"\t"$2}' RM_masked_output/ragtag_polished_round1.fasta.masked.fai > genome.sizes

# ウィンドウごとにリピート密度を計算
bedtools makewindows -g genome.sizes -w 10000 > windows.bed
bedtools coverage -a windows.bed -b RM_masked_output/all_repeats.bed > repeat_density.bedgraph

# なめらかな可視化のため BigWig に変換
bedGraphToBigWig repeat_density.bedgraph genome.sizes repeat_density.bw
```

## IGV 可視化のセットアップ

### IGV へのファイル読み込み

1. **ゲノムを読み込む:**

    ```bash
    # .fai インデックスがなければ作成
    samtools faidx RM_masked_output/ragtag_polished_round1.fasta.masked
    ```

    - 参照として `RM_masked_output/ragtag_polished_round1.fasta.masked` をロード
2. **リピートトラックを読み込む:**

    - `satellites.bed` - サテライトリピート
    - `tandem_repeats.bed` - TRF のタンデムリピート
    - `high_copy_repeats.bed` - 高コピー要素
    - `repeat_density.bw` - 全体のリピート密度
3. **トラック表示を調整:**

    - トラックを右クリック → 色を設定
    - 高さや squish/expand を調整
    - 密な領域は「Collapsed」表示が便利

### 高度な可視化

**リピートのヒートマップを作成:**

```bash
# 強度情報つきで全リピートを結合
awk 'NR>3 {print $5"\t"$6-1"\t"$7"\t"$10"\t"$1}' \
    RM_masked_output/ragtag_polished_round1.fasta.out | \
bedtools map -a windows.bed -b stdin -c 5 -o sum > repeat_heatmap.bedgraph

bedGraphToBigWig repeat_heatmap.bedgraph genome.sizes repeat_heatmap.bw
```

**セントロメア候補領域:**

```bash
# 高いタンデムリピート密度 かつ 高コピーの散在リピートが重なる領域を抽出
bedtools intersect -a high_score_tandems.bed -b high_copy_repeats.bed -wa -wb | \
bedtools merge -d 5000 > centromere_candidates.bed
```

## IGV セッション例

**トラックの上からの並び:**

1. 遺伝子/アノテーション（あれば）
2. セントロメア候補（太い赤トラック）
3. サテライトリピート（青）
4. タンデムリピート（緑）
5. 新規/未知リピート（橙）
6. すべての転移因子（灰色、collapsed）
7. リピート密度（BigWig ヒートマップ）

**IGV でのリピート可視化のコツ:**

- 「Squished」表示をリピート密集領域で活用
- リピートのファミリー/タイプごとに色分け
- リピートカテゴリごとにトラックをグループ化
- 全ゲノム俯瞰には BigWig を使用
- ズームレベルの目安:
    - 染色体ビュー: 全体の分布を把握
    - 100 kb ビュー: リピートクラスターの把握
    - 10 kb ビュー: 個々の要素を確認
    - 1 kb ビュー: 構造の詳細を確認

## 代替の可視化ツール

**大規模データで IGV が重い場合:**

```bash
# Web ベースの JBrowse2 を利用
# JBrowse 互換形式へ変換

# あるいは UCSC Genome Browser 形式
# genome.ucsc.edu にカスタムトラックをアップロード
```

**コマンドラインによる可視化:**

```bash
# リピート密度を素早くテキスト要約
bedtools makewindows -g genome.sizes -w 100000 | \
bedtools coverage -a stdin -b RM_masked_output/all_repeats.bed | \
awk '{print $1":"$2"-"$3"\t"$7}' > repeat_density_summary.txt
```

重要な点は、RepeatModeler がライブラリを作成し、RepeatMasker がそれをゲノムにマッピングしてアノテーションを作成し、そのアノテーションを可視化形式へ変換していく、という流れです。このワークフローにより、セントロメア領域の特定に役立つ包括的なリピート可視化が可能になります。

**Circos プロット**（円形ゲノムビュー）:

```bash
# circos をインストールしてリピート密度プロットを作成
# 全ゲノムのリピート分布を俯瞰するのに有用
conda install -c bioconda circos

# リピート可視化用の circos 設定を作成
# 染色体周囲にリピート密度ヒートマップを表示
```

**Plotly/Bokeh**（インタラクティブプロット）:

```bash
# リピート分布を探索するための対話的プロットを作成
# 論文品質の図に適する
```

## 実行手順まとめ（最短ルート）

以下は、IGV で基本的な可視化を行うための最短手順です。プロジェクトのルート（このドキュメントがあるディレクトリ）で実行してください。

1) 参照 FASTA のインデックス作成（未作成なら）

```bash
samtools faidx RM_masked_output/ragtag_polished_round1.fasta.masked
```

2) RepeatMasker の .out から BED を作成（全リピート）

```bash
awk 'NR>3 {print $5"\t"$6-1"\t"$7"\t"$10"_"$11"\t"$1"\t"$9}' \
    RM_masked_output/ragtag_polished_round1.fasta.out > RM_masked_output/all_repeats.bed
```

3) TRF 出力からの BED 生成（タンデムリピート）
```bash
# 0) TRF の用意（未導入なら）
# conda install -c bioconda trf

# 1) 変数に入力FASTAを設定（未マスクがあれば差し替え）
GENOME=RM_masked_output/ragtag_polished_round1.fasta.masked

# 2) 出力置き場
mkdir -p trf_results

# 3) TRF 実行（-d: .dat出力, -h: ヒットのヘッダ省略）
trf "$GENOME" 2 5 5 80 10 40 2000 -d -h

# 4) 生成された .dat の場所とファイル名を確認
# 例: RM_masked_output/ragtag_polished_round1.fasta.masked.2.5.5.80.10.40.2000.dat
ls -lh RM_masked_output/*.2.5.5.80.10.40.2000.dat | cat

# 5) trf_results/ に移動（以降の手順とパスを揃える）
mv RM_masked_output/$(basename "$GENOME").2.5.5.80.10.40.2000.dat trf_results/
```

```bash
# タンデムリピートの基本 BED
awk 'BEGIN{OFS="\t"} /^@/ {chr=substr($0, 2); next} /^[0-9]+/ {print chr, $1-1, $2, $14"("$4"x)", $8, "."}' trf_results/ragtag_polished_round1.fasta.masked.2.5.5.80.10.40.2000.dat > tandem_repeats.bed

# 高スコアのタンデムリピートのみ
awk 'BEGIN{OFS="\t"} /^@/ {chr=substr($0, 2); next} /^[0-9]+/ && $8 > 500 {print chr, $1-1, $2, $14"_score"$8, $8, "."}' trf_results/ragtag_polished_round1.fasta.masked.2.5.5.80.10.40.2000.dat > high_score_tandems.bed
```


```bash
# 1. 'Simple_repeat' を抽出して simple_repeats.bed を作成
awk 'NR>3 && $11 ~ /Simple_repeat/ {print $5"\t"$6-1"\t"$7"\t"$10"\t"$1"\t"$9}' RM_masked_output/ragtag_polished_round1.fasta.out > igv_tracks/simple_repeats.bed

# 2. 'Unspecified' を抽出して unspecified_repeats.bed を作成
awk 'NR>3 && $11 ~ /Unspecified/ {print $5"\t"$6-1"\t"$7"\t"$10"\t"$1"\t"$9}' RM_masked_output/ragtag_polished_round1.fasta.out > igv_tracks/unspecified_repeats.bed
```

4) IGV 用トラック（必要に応じて選択）

```bash
mkdir -p igv_tracks

# サテライトリピート
awk 'NR>3 && $11 ~ /Satellite/ {print $5"\t"$6-1"\t"$7"\t"$10"\t"$1"\t"$9}' \
    RM_masked_output/ragtag_polished_round1.fasta.out > igv_tracks/satellites.bed

# 転移因子（TE）
awk 'NR>3 && ($11 ~ /LINE|SINE|LTR|DNA/) {print $5"\t"$6-1"\t"$7"\t"$10"\t"$1"\t"$9}' \
    RM_masked_output/ragtag_polished_round1.fasta.out > igv_tracks/transposons.bed

# 不明/未知のリピート
awk 'NR>3 && $11 ~ /Unknown|Unclassified/ {print $5"\t"$6-1"\t"$7"\t"$10"\t"$1"\t"$9}' \
    RM_masked_output/ragtag_polished_round1.fasta.out > igv_tracks/novel_repeats.bed

# 高コピーリピート
awk 'NR>3 && $1 > 1000 {print $5"\t"$6-1"\t"$7"\t"$10"_copy"$1"\t"$1"\t"$9}' \
    RM_masked_output/ragtag_polished_round1.fasta.out > igv_tracks/high_copy_repeats.bed
```

5) リピート密度（BigWig）

```bash
awk '{print $1"\t"$2}' RM_masked_output/ragtag_polished_round1.fasta.masked.fai > genome.sizes
bedtools makewindows -g genome.sizes -w 10000 > windows.bed
bedtools coverage -a windows.bed -b RM_masked_output/all_repeats.bed > repeat_density.bedgraph
bedGraphToBigWig repeat_density.bedgraph genome.sizes repeat_density.bw
```

6) セントロメア候補の抽出

```bash
bedtools intersect -a high_score_tandems.bed -b igv_tracks/high_copy_repeats.bed -wa -wb | \
bedtools merge -d 5000 > centromere_candidates.bed
```

7) IGV で読み込む

- 参照: `RM_masked_output/ragtag_polished_round1.fasta.masked`
- トラック: `igv_tracks/*.bed`, `repeat_density.bw`, `centromere_candidates.bed`

### 任意（必要な場合のみ）

```bash
# リピート強度ヒートマップ（任意）
awk 'NR>3 {print $5"\t"$6-1"\t"$7"\t"$10"\t"$1}' \
    RM_masked_output/ragtag_polished_round1.fasta.out | \
bedtools map -a windows.bed -b stdin -c 5 -o sum > repeat_heatmap.bedgraph
bedGraphToBigWig repeat_heatmap.bedgraph genome.sizes repeat_heatmap.bw
```

### 前提ツール

- `samtools`, `bedtools`, `bedGraphToBigWig`（UCSC ツール）が利用可能であること
- それぞれ `conda` などで導入可能

```bash
(base) [aki@tardis ~]$ ls -l RM_masked_output/
合計 96960
-rw-rw-r-- 1 aki aki  9878455  8月 26 11:24 all_repeats.bed
-rw-rw-r-- 1 aki aki 19180723  8月 25 12:22 ragtag_polished_round1.fasta.cat.gz
-rw-rw-r-- 1 aki aki 35558579  8月 25 12:22 ragtag_polished_round1.fasta.masked
-rw-rw-r-- 1 aki aki      490  8月 26 12:13 ragtag_polished_round1.fasta.masked.fai
-rw-rw-r-- 1 aki aki     8786  8月 25 12:22 ragtag_polished_round1.fasta.ori.out
-rw-rw-r-- 1 aki aki 20623783  8月 25 12:22 ragtag_polished_round1.fasta.out
-rw-rw-r-- 1 aki aki     2454  8月 25 12:22 ragtag_polished_round1.fasta.tbl
-rw-rw-r-- 1 aki aki 14014839  8月 26 11:25 repeats_colored.bed
(base) [aki@tardis ~]$ ls -lh igv_tracks/*.bed centromere_candidates.bed
-rw-rw-r-- 1 aki aki    0  8月 26 12:10 centromere_candidates.bed
-rw-rw-r-- 1 aki aki 187K  8月 26 11:31 igv_tracks/high_copy_repeats.bed
-rw-rw-r-- 1 aki aki    0  8月 26 11:31 igv_tracks/novel_repeats.bed
-rw-rw-r-- 1 aki aki    0  8月 26 11:31 igv_tracks/satellites.bed
-rw-rw-r-- 1 aki aki    0  8月 26 11:31 igv_tracks/transposons.bed
```



---

はい、承知いたしました。

ゲノム中のトランスポゾン（TE）を網羅的に同定・分類するパイプライン EDTA (Extensive de novo TE Annotator) を実行し、結果をIGVで可視化するための一連の手順をまとめます。

---

## 実行手順まとめ（EDTA版）

以下は、EDTAを用いてゲノムアノテーションを行い、IGVで可視化するための手順です。

### 1) EDTAの導入と設定（未導入なら） ⚙️

EDTAは多くの依存ツールを持つため、Condaを使って専用の環境を構築するのが最も簡単で確実です。

Bash

```
# 0) Conda環境の構築（初回のみ）
conda create -n EDTA2 -c conda-forge -c bioconda edta=2.0.0

# 1) 作成した環境をアクティベート
#    EDTAを実行する際は、必ずこのコマンドで環境に入る必要があります
conda activate EDTA2

# 2) (初回のみ) RepeatMaskerのライブラリ設定
#    もしRepeatMaskerのライブラリ（Dfamなど）を初めて使う場合、
#    対話形式での設定が必要になることがあります。その場合はEDTA実行中に指示が表示されます。
```

<hr/>

### 2) 入力ファイルの準備 🧬

TRFの時と同様に、解析対象のゲノムFASTAファイルを変数に設定します。

Bash

```
# 1) 変数に入力FASTAを設定
GENOME="RM_masked_output/ragtag_polished_round1.fasta.masked"

# 2) 出力ディレクトリを作成
mkdir -p edta_results
cd edta_results # EDTAは出力ファイルが多いため、専用ディレクトリでの実行を推奨
```

<hr/>

### 3) EDTAの実行 🖥️

EDTAパイプライン全体を実行します。**この処理はゲノムサイズによりますが、数時間〜数日かかる**ことがあります。`&` を付けてバックグラウンドで実行するのがお勧めです。

Bash

```
# EDTAのメインコマンドを実行
# --anno 1 で、最終的なアノテーションまで実行します
# --threads は利用可能なCPUコア数に合わせて調整してください
EDTA.pl \
  --genome ../"$GENOME" \
  --species Others \
  --step all \
  --anno 1 \
  --threads 8 \
  --force 1

# 上記コマンドの `&` はバックグラウンド実行を意味します。
# 実行状況は `tail -f EDTA.log` でリアルタイムに確認できます。
```

<hr/>


### 4) IGV用トラックの作成（BED形式への変換）

EDTAの実行が完了すると、`GFF3`形式のアノテーションファイル (`(ゲノム名).EDTA.gff3`) が生成されます。これをTEのクラスごとにBEDファイルへ変換し、IGVで見やすくします。

Bash

```
# 0) GFFファイル名を変数に設定
GFF_FILE=ragtag_polished_round1.fasta.mod.EDTA.intact.gff3  

# 1) LTR（レトロトランスポゾン）を抽出
awk 'BEGIN{OFS="\t"} $3=="repeat_region" && /Classification=LTR/ {split($9, a, /[=;]/); print $1, $4-1, $5, a[2], ".", "."}' "$GFF_FILE" > ltr_repeats.bed

# 2) TIR（DNAトランスポゾン）を抽出
awk 'BEGIN{OFS="\t"} $3=="repeat_region" && /Classification=TIR/ {split($9, a, /[=;]/); print $1, $4-1, $5, a[2], ".", "."}' "$GFF_FILE" > tir_repeats.bed

# 3) Helitronを抽出
awk 'BEGIN{OFS="\t"} $3=="repeat_region" && /Classification=Helitron/ {split($9, a, /[=;]/); print $1, $4-1, $5, a[2], ".", "."}' "$GFF_FILE" > helitron_repeats.bed

# 4) 分類不明（Unclassified）なTEを抽出
awk 'BEGIN{OFS="\t"} $3=="repeat_region" && /Classification=Unknown/ {split($9, a, /[=;]/); print $1, $4-1, $5, a[2], ".", "."}' "$GFF_FILE" > unclassified_repeats.bed
```

<hr/>

### 5) IGV で読み込む

- **参照:** `RM_masked_output/ragtag_polished_round1.fasta.masked` (一つ上の階層にある元ファイル)
    
- **トラック:** `edta_results` ディレクトリ内に作成された以下のファイル
    
    - `ltr_repeats.bed`
        
    - `tir_repeats.bed`
        
    - `helitron_repeats.bed`
        
    - `unclassified_repeats.bed`

```bash
(EDTA2) [aki@tardis ~]$ ls -lh *.bed
-rw-rw-r-- 1 aki aki 643K  7月 14 12:18 canu_unique.bed
-rw-rw-r-- 1 aki aki 643K  7月 14 12:15 canu_unique_regions.bed
-rw-rw-r-- 1 aki aki    0  8月 26 12:10 centromere_candidates.bed
-rw-rw-r-- 1 aki aki    0  8月 28 12:48 helitron_repeats.bed
-rw-rw-r-- 1 aki aki  64K  8月 27 17:08 high_score_tandems.bed
-rw-rw-r-- 1 aki aki    0  8月 28 12:48 ltr_repeats.bed
-rw-rw-r-- 1 aki aki  725  8月  4 13:06 regions.bed
-rw-rw-r-- 1 aki aki 9.7M  8月 26 10:35 repeats.bed
-rw-rw-r-- 1 aki aki 2.9M  8月 27 16:48 tandem_repeats.bed
-rw-rw-r-- 1 aki aki 4.4M  8月 27 17:03 tandem_repeats_final.bed
-rw-rw-r-- 1 aki aki 2.9M  8月 27 17:01 tandem_repeats_fixed.bed
-rw-rw-r-- 1 aki aki    0  8月 28 12:48 tir_repeats.bed
-rw-rw-r-- 1 aki aki    0  8月 28 12:49 unclassified_repeats.bed
-rw-rw-r-- 1 aki aki 139K  8月 26 12:13 windows.bed
```




# 原因（要点）

- 属性フィールドのキーや大文字小文字が混在している。例：
    
    - `classification=LTR/Gypsy`（小文字の `classification=`）
        
    - `Classification=DNA/Helitron`（先頭大文字の `Classification=`）  
        こちらの差で、最初に使っていた `awk` の `/Classification=LTR/` のような **大文字固定のパターンがヒットしていません**。
        
- `classification` の値が `LTR` 単体ではなく `LTR/Gypsy` のようにスラッシュ付きで出ている → 単純な `=LTR` マッチでは取り逃がす。
    
- 「完全長要素」を示す行は普通 **`repeat_region`** だが、Helitron の場合は GFF の第3列が `helitron` や `TE_struc` のようになっているケースがある（サンプルに該当）。つまり **$3==repeat_region の条件だけでは Helitron を拾えない**。
    

以上がファイルがゼロになる主要因です。

---

# まず現状の分類一覧を確認（推奨）

GFF 内に実際どんな `classification` 値があるか確認します。1行で実行できます：

```bash
awk -F'\t' '{
  split($9, a, ";");
  for(i=1;i<=length(a);i++){
    if(a[i] ~ /[cC]lassification=/){
      split(a[i], kv, "=");
      print kv[2];
    }
  }
}' ragtag_polished_round1.fasta.mod.EDTA.intact.gff3 | sort | uniq -c | sort -nr
```

（出力を貼ってくれれば、どの値を優先すべきか具体的に調整できます。）

---

# 安定して BED を作る汎用スクリプト（そのままコピペで実行可）

- 意味：`classification` を大文字小文字関係なく取り出し、**完全要素（repeat_region 等）** のみを対象にして LTR / Helitron / TIR / Unclassified に振り分けます。
    
- 実行前に既存の `.bed` を消すので安全です。
    

```bash
# 古いファイルを削除
rm -f ltr_repeats.bed tir_repeats.bed helitron_repeats.bed unclassified_repeats.bed

# 抽出
awk -F'\t' 'BEGIN{OFS="\t"}
{
  attr=$9;
  cls="";
  n=split(attr,parts,";");
  for(i=1;i<=n;i++){
    # 小文字化してキーをチェック（case-insensitive）
    p=parts[i];
    lp=tolower(p);
    if(lp ~ /^classification=/ || lp ~ /^class=/){
      split(p,kv,"=");
      cls=kv[2];
      break;
    }
  }
  if(cls=="") next;

  # 対象は完全要素行（repeat_region）か、helitron/TE_strucのような行
  if($3 != "repeat_region" && tolower($3) !~ /helitron/ && tolower($3) !~ /te_struc/ && tolower($3) !~ /transposable/){
    next;
  }

  lcls=tolower(cls);
  if(lcls ~ /ltr/){
    print $1, $4-1, $5, cls, ".", "." >> "ltr_repeats.bed"
  } else if(lcls ~ /helitron/){
    print $1, $4-1, $5, cls, ".", "." >> "helitron_repeats.bed"
  } else if(lcls ~ /tir/ || lcls ~ /dna\/tir/){
    print $1, $4-1, $5, cls, ".", "." >> "tir_repeats.bed"
  } else if(lcls ~ /unknown/ || lcls ~ /unclassified/){
    print $1, $4-1, $5, cls, ".", "." >> "unclassified_repeats.bed"
  }
}
END{
  close("ltr_repeats.bed"); close("helitron_repeats.bed");
  close("tir_repeats.bed"); close("unclassified_repeats.bed");
}' ragtag_polished_round1.fasta.mod.EDTA.intact.gff3
```

> ここで `$4-1` は BED の 0-based start に合わせています（IGV 用の標準形式）。

---

# 生成後の確認コマンド

```bash
# 行数確認
wc -l *.bed

# 中身を少し見る
head -n 10 ltr_repeats.bed
head -n 10 helitron_repeats.bed
```

---

# 追加チェック（まだゼロなら）

- EDTA の `intact.gff3` は「完全長のみ」を出すため **断片化した TE が入っていない** 可能性があります。`TEanno` や `*.EDTA.TEanno.gff3` の方に多数の断片／非完全要素があるので、そちらを同じスクリプトで処理してみてください：
    

```bash
ls -lh *.EDTA.*.gff3
# 例:
# ragtag_polished_round1.fasta.mod.EDTA.TEanno.gff3 があればそれを使う
```

- もし `TEanno` にデータが多ければ同じ awk をそのファイル名に対して実行してください。
    

---

# 補足（理由の要約）

あなたが示してくれた GFF の例には

- `classification=LTR/Gypsy`（小文字） がある一方で
    
- `Classification=DNA/Helitron`（大文字） の行もあり、  
    さらに
    
- Helitron は `$3` が `helitron` になっているなど **フォーマットのばらつき** があるため、最初の単純な正規表現ではヒットしなかった — というのが真因です。
    

---

やってみて、`wc -l *.bed` の結果（各 .bed の行数）か、先ほどの「分類一覧コマンド」の出力を貼ってくれたら、必要なら更にフィルタを微調整して一発で IGV 向けに整えます。