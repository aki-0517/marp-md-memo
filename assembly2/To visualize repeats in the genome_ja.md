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

