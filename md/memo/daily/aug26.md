## Getting Visualization Files
に沿ってやる

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



表示されないファイルはすべて、ファイルサイズが**0バイト**、つまり**空っぽ**だからです。中身が何もないため、IGVに読み込んでも何も表示されません。

---

### ## 原因

ファイルが空になっているのは、それらを生成したコマンド（`awk`や`bedtools intersect`）が、指定した条件に一致するデータを**1行も見つけられなかった**ことを意味します。

- satellites.bed (0バイト)
    
    RepeatMaskerの出力ファイル（.out）の中に、11列目が "Satellite" に一致する行が存在しなかったということです。
    
- transposons.bed (0バイト)
    
    同様に、11列目に "LINE", "SINE", "LTR", "DNA" のいずれかを含む行が存在しなかったということです。
    
- novel_repeats.bed (0バイト)
    
    11列目に "Unknown" または "Unclassified" を含む行が存在しなかったということです。
    
- centromere_candidates.bed (0バイト)
    
    bedtools intersectが、high_score_tandems.bedとhigh_copy_repeats.bedの間で重複（intersect）する領域を1つも見つけられなかったということです。
    

つまり、コマンドのエラーではなく、**解析対象のゲノムとRepeatMaskerの解析結果そのものに、該当するデータが含まれていなかった**、という生物学的な結果と言えます。

---

### ## 確認・対応方法

本当に元のファイルにデータがないのか、念のため確認してみましょう。`grep`コマンドを使って、RepeatMaskerの出力ファイル（`.out`）に指定したパターンのリピートが存在するかどうかを直接検索できます。

Bash

```
# .outファイルの中に "Satellite" という単語が含まれる行があるか検索
grep "Satellite" RM_masked_output/ragtag_polished_round1.fasta.out

# .outファイルの中に "LINE" という単語が含まれる行があるか検索
grep "LINE" RM_masked_output/ragtag_polished_round1.fasta.out
```

これらのコマンドを実行して何も表示されなければ、やはり元のデータに存在しないことが確定します。

**今後の対応**としては、

- `awk`の検索条件を見直す（例: 大文字/小文字を区別しないようにする、別のキーワードを探す）。
    
- RepeatMaskerで利用したリピートライブラリが、解析したい生物種に対して適切だったか再検討する。
    
    などが考えられます。

はい、承知いたしました。

ファイルが0バイトで生成されるのを防ぐ、あるいは生成された場合にすぐに気づけるように、各コマンドの直後にファイルサイズをチェックする処理を追加するのが最も確実な方法です。

---

### ## 基本的な考え方

シェルスクリプトでは、コマンドの実行直後に`if`文を使って特定の条件をチェックできます。ファイルが空かどうかは `[ ! -s "ファイル名" ]` という条件で判定します。

- `-s "ファイル名"`: ファイルが存在し、サイズが0より大きい場合に真（true）になります。
    
- `!`: 「ではない（NOT）」という意味の否定演算子です。
    
- **`[ ! -s "ファイル名" ]`**: ファイルが存在しない、またはサイズが0の場合に真（true）になります。
    

これを利用して、「もしファイルが空だったら、警告メッセージを表示する」という処理を組み込みます。

---

### ## 具体的な修正方法

例えば、`satellites.bed` を作成するコマンドは以下のようになります。

#### **修正前**

Bash

```
awk 'NR>3 && $11 ~ /Satellite/ {print ...}' ... > igv_tracks/satellites.bed
```

#### **修正後（警告メッセージ付き）**

Bash

```
# 1. 出力ファイルを変数に格納しておく
SATELLITE_BED="igv_tracks/satellites.bed"

# 2. awkコマンドを実行
awk 'NR>3 && $11 ~ /Satellite/ {print ...}' ... > "$SATELLITE_BED"

# 3. 直後にファイルが空かどうかをチェック
if [ ! -s "$SATELLITE_BED" ]; then
    # ファイルが空の場合、警告メッセージを表示する
    echo "【警告】$SATELLITE_BED は空ファイルです。元のデータに 'Satellite' が含まれていなかった可能性があります。"
fi
```

このように、ファイルを生成する各コマンドの直後にチェック処理を挟むことで、実行結果をその場で把握できます。

---

### ## 修正版パイプラインの例

以下に、前回のパイプライン全体にこのチェック処理を組み込んだ例を示します。

Bash

```
#!/bin/bash
# スクリプトの途中でエラーが起きたら停止する設定
set -e

# --- 1. 変数の設定とディレクトリ作成 ---
GENOME="RM_masked_output/ragtag_polished_round1.fasta.masked"
RM_OUT="RM_masked_output/ragtag_polished_round1.fasta.out"
mkdir -p trf_results igv_tracks

# --- ファイルチェック用の関数を定義 ---
# この関数を各コマンドの後で呼び出す
check_file() {
    FILE_PATH=$1
    # ファイルが存在しない、またはサイズが0の場合
    if [ ! -s "$FILE_PATH" ]; then
        echo "【警告】$FILE_PATH は空ファイルとして生成されました。"
    else
        echo "✅ $FILE_PATH が正常に生成されました。"
    fi
}

# --- 2. RepeatMasker (.out) からBEDを作成 ---
echo "--- RepeatMaskerの出力を処理中 ---"
ALL_REPEATS_BED="RM_masked_output/all_repeats.bed"
awk 'NR>3 {print $5"\t"$6-1"\t"$7"\t"$10"_"$11"\t"$1"\t"$9}' "$RM_OUT" > "$ALL_REPEATS_BED"
check_file "$ALL_REPEATS_BED"

SATELLITE_BED="igv_tracks/satellites.bed"
awk 'NR>3 && $11 ~ /Satellite/ {print $5"\t"$6-1"\t"$7"\t"$10"\t"$1"\t"$9}' "$RM_OUT" > "$SATELLITE_BED"
check_file "$SATELLITE_BED"

TRANSPOZON_BED="igv_tracks/transposons.bed"
awk 'NR>3 && ($11 ~ /LINE|SINE|LTR|DNA/) {print $5"\t"$6-1"\t"$7"\t"$10"\t"$1"\t"$9}' "$RM_OUT" > "$TRANSPOZON_BED"
check_file "$TRANSPOZON_BED"

HIGH_COPY_BED="igv_tracks/high_copy_repeats.bed"
awk 'NR>3 && $1 > 1000 {print $5"\t"$6-1"\t"$7"\t"$10"_copy"$1"\t"$1"\t"$9}' "$RM_OUT" > "$HIGH_COPY_BED"
check_file "$HIGH_COPY_BED"

# --- 3. TRF出力からBEDを生成 ---
# (TRF実行とファイル移動は省略)
DAT_FILE_PATH="trf_results/ragtag_polished_round1.fasta.masked.2.5.5.80.10.40.2000.dat"
HIGH_SCORE_TANDEM_BED="high_score_tandems.bed"
echo "--- TRFの出力を処理中 ---"
awk 'BEGIN{OFS="\t"} /^@/ {chr=substr($0, 2); next} /^[0-9]+/ && $8 > 500 {print chr, $1-1, $2, $14"_score"$8, $8, "."}' "$DAT_FILE_PATH" > "$HIGH_SCORE_TANDEM_BED"
check_file "$HIGH_SCORE_TANDEM_BED"

# --- 5. セントロメア候補を抽出 ---
echo "--- セントロメア候補を抽出中 ---"
CENTROMERE_BED="centromere_candidates.bed"
bedtools intersect -a "$HIGH_SCORE_TANDEM_BED" -b "$HIGH_COPY_BED" -wa -wb | \
bedtools merge -d 5000 > "$CENTROMERE_BED"
check_file "$CENTROMERE_BED"

echo "--- 全ての処理が完了しました ---"

```

このスクリプトを実行すると、各ファイルの生成後に「✅ 正常に生成されました」または「【警告】空ファイルです」というメッセージが表示されるため、どのステップでデータが生成されなかったのかが一目瞭然になります。

```bash
(base) [aki@tardis ~]$ awk 'NR>3 {print $11}' RM_masked_output/ragtag_polished_round1.fasta.out | sort | uniq -c | sort -nr
 121814 Simple_repeat
  26694 Unspecified
```

あなたのゲノムからRepeatMaskerによって検出されたリピートは、ほぼすべてが**`Simple_repeat`**（単純反復配列）と**`Unspecified`**（分類不明なリピート）の2種類に分類されています。

---

### ## この結果が意味すること

この結果は、以前に`satellites.bed`や`transposons.bed`のファイルが空になった理由を明確に説明しています。元のデータに "Satellite" や "LINE", "SINE" といった分類が**そもそも存在しなかった**のです。

- **Simple_repeat**: (ATATAT...)n のようなマイクロサテライトやミニサテライトなど、単純な塩基配列の繰り返しです。
    
- **Unspecified**: RepeatMaskerが使用したリピートライブラリ内の、どの既知のファミリーにも一致しなかったリピート配列です。
    

これはエラーではなく、あなたのゲノムの繰り返し配列が持つ特徴を示した**重要な解析結果**です。

---

### ## 修正案：実際に存在する分類でBEDファイルを作成する

この発見に基づき、`awk`コマンドを修正して、意味のある（空ではない）BEDファイルを作成しましょう。

以前の`satellites.bed`や`transposons.bed`を作成する代わりに、`Simple_repeat`と`Unspecified`をそれぞれ抽出するコマンドを実行します。

Bash

```
# 1. 'Simple_repeat' を抽出して simple_repeats.bed を作成
awk 'NR>3 && $11 ~ /Simple_repeat/ {print $5"\t"$6-1"\t"$7"\t"$10"\t"$1"\t"$9}' RM_masked_output/ragtag_polished_round1.fasta.out > igv_tracks/simple_repeats.bed

# 2. 'Unspecified' を抽出して unspecified_repeats.bed を作成
awk 'NR>3 && $11 ~ /Unspecified/ {print $5"\t"$6-1"\t"$7"\t"$10"\t"$1"\t"$9}' RM_masked_output/ragtag_polished_round1.fasta.out > igv_tracks/unspecified_repeats.bed
```

これで、`simple_repeats.bed`と`unspecified_repeats.bed`という、**中身のあるトラックファイル**が作成されるはずです。これらのファイルをIGVで読み込むことで、ゲノム上の単純反復配列や未知のリピートがどこに分布しているかを視覚的に確認できます。

デバッグを通じて、ご自身のデータの特性が明らかになりましたね。素晴らしい進展です！✅







Unspecified（分類不明）なリピートを減らすためには、**RepeatModelerで作成したカスタムライブラリと、既存の公的データベースを組み合わせてRepeatMaskerを実行する**のが最も効果的で標準的な改善方法です。

`Unspecified`が多くなるのは、RepeatModelerが「繰り返し配列だ」と認識はしたものの、それが既知のどのファミリー（LINE, SINE, LTRなど）にも似ていなかったためです。これは、真に新規のリピートである可能性もあれば、公的データベースには存在するものの、RepeatModelerの分類アルゴリズムではうまく分類できなかったリピートである可能性もあります。

---

### ## 改善手順：公的データベースとのハイブリッド解析

この改善策は、前回の手順に**「公的データベースの準備」**と**「ライブラリのマージ」**というステップを追加する形になります。

#### ### ステップ1：公的リピートデータベースを準備する

まず、生物種に合わせた信頼性の高いリピートデータベースをダウンロードします。**Dfam**が現在推奨されることが多いです。

1. **Dfamデータベースのダウンロード**
    
    - Dfamのウェブサイトから、対象とする生物群のデータベースをダウンロードします。例えば、近縁な生物がいればその生物群（例: ほ乳類、魚類、昆虫類など）のFASTAファイルを取得します。
        
    - もし近縁種がなければ、`Dfam.h5`という全生物種を含むデータベースを取得し、そこから必要な部分を抽出することもできます。今回は例として、手動でダウンロードした `vertebrates_Dfam.fa` があると仮定します。
        

#### ### ステップ2：RepeatModelerでカスタムライブラリを作成する

これは**前回の手順と同じ**です。`BuildDatabase`と`RepeatModeler`を実行し、`consensi.fa.classified`ファイルを入手します。

- **重要な出力**: `RM_3794378.TueAug191046142025/consensi.fa.classified`
    

---

### ## ステップ3：2つのライブラリを結合する【改善の核】

ダウンロードしたDfamライブラリと、RepeatModelerが作成したカスタムライブラリを1つのファイルに結合（マージ）します。

- **目的**: 公的データベースの網羅性と、ゲノム特異的なカスタムライブラリの発見能力を両立させる。
    
- **コマンド**:
    
    Bash
    
    ```
    cat vertebrates_Dfam.fa RM_3794378.TueAug191046142025/consensi.fa.classified > combined_repeats.fa
    ```
    
- **説明**:
    
    - `cat`コマンドは、複数のファイルを連結して1つのファイルにするコマンドです。
        
    - `combined_repeats.fa`という名前で、Dfamとカスタムライブラリの両方が含まれた、より強力なハイブリッドライブラリが完成します。
        

---

### ## ステップ4：結合ライブラリでRepeatMaskerを実行する

最後に、ステップ3で作成した結合ライブラリを使ってRepeatMaskerを実行します。

- **目的**: 改善されたハイブリッドライブラリを用いて、ゲノムのマスキング精度を向上させる。
    
- **コマンド**:
    
    Bash
    
    ```
    RepeatMasker -lib combined_repeats.fa -gff -pa 8 ragtag_polished_round1.fasta
    ```
    
- **説明**:
    
    - `-lib`オプションに、前回指定した`consensi.fa.classified`の代わりに、結合済みの**`combined_repeats.fa`**を指定します。
        

### ## なぜこの方法で改善されるのか？

- RepeatModelerが見つけたものの`Unspecified`としか分類できなかったリピートが、もしDfamデータベースに登録されていれば、RepeatMaskerはそのリピートを**Dfamの情報に基づいて正しく分類**（例: `LTR/Gypsy`など）してくれます。
    
- 一方で、Dfamにも載っていない真に新規のリピートは、カスタムライブラリの情報に基づいてマスクされます。
    

このハイブリッドアプローチにより、それぞれのライブラリの弱点を補い合い、`Unspecified`を大幅に減らし、より正確で情報量の多いアノテーションを得ることが可能になります。



細胞性粘菌である**ディクチオステリウム（Dictyostelium discoideum）**の場合に特化した、Unspecifiedを減らすための具体的な改善手順を解説します。

基本的な戦略は前回と同じく**「カスタムライブラリ」と「公的データベース」のハイブリッド**ですが、使用する公的データベースをディクチオステリウムに最適化する点が重要です。

---

### ## なぜディクチオステリウムに特化したライブラリが必要か？

ディクチオステリウムには、**DIRS**や**Skipper**といった、この生物（またはその近縁種）に特有の非常にユニークな転移因子（レトロトランスポゾン）ファミリーが存在します。

RepeatModelerはこれらの配列を「繰り返し配列である」と検出はできますが、そのユニークさゆえに既知の分類（LINE, SINEなど）に当てはめることができず、結果として`Unspecified`と分類してしまいます。

そこで、これらのユニークなリピートがあらかじめ登録されている公的データベースと組み合わせることが極めて効果的になります。

---

### ## 改善手順：ディクチオステリウム版

#### ### ステップ1：公的リピートデータベースを準備する

ディクチオステリウムの繰り返し配列情報が含まれている**RepBase**から、専用のライブラリをダウンロードするのが最も確実です。

1. **RepBaseからライブラリを入手する**
    
    - GIRI (Genetic Information Research Institute) の **RepBase** にアクセスします。
        
    - データベースを検索し、**"Dictyostelium discoideum"** のライブラリを入手します。（利用にはユーザー登録が必要な場合があります）
        
    - ここでは、入手したライブラリのファイル名を仮に `discoideum_repbase.fa` とします。
        

#### ### ステップ2：RepeatModelerでカスタムライブラリを作成する

これは前回の手順と**全く同じ**です。ゲノムから _de novo_ で繰り返し配列を同定し、`consensi.fa.classified` ファイルを作成します。

- **コマンド (再掲)**:
    
    Bash
    
    ```
    # データベース構築
    BuildDatabase -name discoideum_db -engine ncbi ragtag_polished_round1.fasta
    
    # RepeatModeler実行
    RepeatModeler -database discoideum_db -engine ncbi -pa 8
    ```
    
- **重要な出力**: `[生成されたRMディレクトリ]/consensi.fa.classified`
    

---

### ## ステップ3：2つのライブラリを結合する【改善の核】

ダウンロードしたRepBaseライブラリと、RepeatModelerが作成したカスタムライブラリを1つのファイルに結合します。

- **目的**: DIRSやSkipperなど既知のユニークなリピートを正しく分類し、かつゲノムに潜む未知の新規リピートも捉える。
    
- **コマンド**:
    
    Bash
    
    ```
    cat discoideum_repbase.fa [生成されたRMディレクトリ]/consensi.fa.classified > discoideum_combined_repeats.fa
    ```
    
- **説明**:
    
    - `discoideum_combined_repeats.fa` という名前で、ディクチオステリウムの解析に最適化されたハイブリッドライブラリが完成します。
        

---

### ## ステップ4：結合ライブラリでRepeatMaskerを実行する

最後に、完成したハイブリッドライブラリを使ってRepeatMaskerを実行します。

- **コマンド**:
    
    Bash
    
    ```
    RepeatMasker -lib discoideum_combined_repeats.fa -gff -pa 8 ragtag_polished_round1.fasta
    ```
    
- **説明**:
    
    - `-lib`オプションに、結合済みの **`discoideum_combined_repeats.fa`** を指定します。
        

この手順により、RepeatModelerだけでは`Unspecified`としか分類できなかったリピートの多くが、RepBaseの情報に基づいて`DIRS`や`Skipper`など、生物学的に意味のある正しい分類名でアノテーションされるようになり、解析の質が格段に向上します。


他のpaper読んでどうやってteなど特定してるか調べる


## _Dictyostelium discoideum_ ゲノム研究における反復配列アノテーション手法の体系的調査結果

---

## 論文1

**引用**: Eichinger L, Pachebat JA, Glöckner G, et al. (2005). The genome of the social amoeba Dictyostelium discoideum. Nature. 435(7038):43-57. doi: 10.1038/nature03481[pmc.ncbi.nlm.nih](https://pmc.ncbi.nlm.nih.gov/articles/PMC1352341/)

**使用データベース**: 種特異的なカスタムリピートライブラリ

**使用手法**: **ハイブリッドアプローチ**（カスタムライブラリとポリモルフィズム解析、HAPPY mapによる組み立て支援）

**使用ツール**: カスタムソフトウェア、HAPPY mapping、YAC low-coverage sequencing

**引用文**:

> "For chromosomes 1, 2 and 3, inspection of polymorphisms, combined with HAPPY maps, allowed unambiguous assembly in many cases. For chromosomes 4, 5 and 6, low-coverage sequencing of AX4-derived YACs alleviated the problems by providing a local dataset within which the troublesome repeat element was present as a single copy."[pmc.ncbi.nlm.nih](https://pmc.ncbi.nlm.nih.gov/articles/PMC1352341/)

---

## 論文2

**引用**: Glöckner G, Szafranski K, Winckler T, et al. (2001). The complex repeats of Dictyostelium discoideum. Genome Research. 11(4):585-594. doi: 10.1101/gr.162201[pmc.ncbi.nlm.nih](https://pmc.ncbi.nlm.nih.gov/articles/PMC311061/)

**使用データベース**: **カスタムリピートライブラリ**（_Dictyostelium_ 特異的な複合反復要素データベース）

**使用手法**: **de novo**（類似性検索による新規要素同定とコンセンサス配列構築）

**使用ツール**: **WU-BLASTN 2.0**, **CLUSTALW V1.8**, **GAP4**, **AlnK**, **tRNAscan-SE**

**引用文**:

> "A given seed sequence or the consensus of a repeat sequence alignment was used as a probe to find matching reads in the available genomic shotgun sequences via WU-BLASTN 2.0... Multiple and profile alignments were performed using CLUSTALW V1.8."[pmc.ncbi.nlm.nih](https://pmc.ncbi.nlm.nih.gov/articles/PMC311061/)

---

## 論文3

**引用**: Sucgang R, Kuo A, Tian X, et al. (2011). Comparative genomics of the social amoebae Dictyostelium discoideum and Dictyostelium purpureum. Genome Biology. 12(2):R20. doi: 10.1186/gb-2011-12-2-r20[pmc.ncbi.nlm.nih](https://pmc.ncbi.nlm.nih.gov/articles/PMC3188802/)

**使用データベース**: **カスタムリピートライブラリ**

**使用手法**: **de novo**（RepeatMaskerとカスタムリピートライブラリを利用）

**使用ツール**: **RepeatMasker**

**引用文**:

> "First, the genome assembly is masked using RepeatMasker and a custom repeat library..."[pmc.ncbi.nlm.nih](https://pmc.ncbi.nlm.nih.gov/articles/PMC3188802/)

---

## 論文4

**引用**: Heidel AJ, Lawal HM, Felder M, et al. (2024). Chromosome-level genome assembly and annotation of the social amoeba Dictyostelium firmibasis. Scientific Data. 11:594. doi: 10.1038/s41597-024-03513-8[nature](https://www.nature.com/articles/s41597-024-03513-8)

**使用データベース**: **RepBase** (Genetic Information Research Institute) filtered to D. discoideum

**使用手法**: **相同性ベース**

**使用ツール**: **tblastx v2.14.0**

**引用文**:

> "Transposable elements were identified with tblastx v2.14.0 with e-values below 10^-15 and reference transposable element sequences from Repbase (Genetic Information Research Institute) filtered to D. discoideum."[nature](https://www.nature.com/articles/s41597-024-03513-8)

---

## 主要な知見

1. **データベース利用の変遷**: 初期の研究（2001-2011年）では**RepBaseやDfamなどの公的データベースは使用されておらず**、_Dictyostelium_ 特異的なカスタムリピートライブラリが主流でした。近年（2024年）になってようやくRepBaseの利用が確認されました。[nature+3](https://www.nature.com/articles/s41597-024-03513-8)
    
2. **手法の特徴**:
    
    - **de novo アプローチ**が主流で、RepeatModelerなどの標準的なde novoツールではなく、独自のパイプラインを使用[pmc.ncbi.nlm.nih+1](https://pmc.ncbi.nlm.nih.gov/articles/PMC311061/)
        
    - **ハイブリッドアプローチ**では、カスタムライブラリとゲノム特異的な解析手法を組み合わせ[pmc.ncbi.nlm.nih](https://pmc.ncbi.nlm.nih.gov/articles/PMC1352341/)
        
3. **技術的制約の影響**: _Dictyostelium_ ゲノムの高いA+T含量（77.6%）と複雑なリピート構造により、標準的なリピートアノテーション手法の適用が困難で、種特異的なアプローチが必要でした。[pmc.ncbi.nlm.nih+1](https://pmc.ncbi.nlm.nih.gov/articles/PMC311061/)
    
4. **ツールのバージョン情報**: 具体的なバージョン情報の記載は限定的で、多くの研究でカスタムソフトウェアやパイプラインが使用されていました。
    

5. [https://pmc.ncbi.nlm.nih.gov/articles/PMC1352341/](https://pmc.ncbi.nlm.nih.gov/articles/PMC1352341/)
6. [https://pmc.ncbi.nlm.nih.gov/articles/PMC311061/](https://pmc.ncbi.nlm.nih.gov/articles/PMC311061/)
7. [https://pmc.ncbi.nlm.nih.gov/articles/PMC3188802/](https://pmc.ncbi.nlm.nih.gov/articles/PMC3188802/)
8. [https://www.nature.com/articles/s41597-024-03513-8](https://www.nature.com/articles/s41597-024-03513-8)
9. [https://github.com/Dfam-consortium/RepeatModeler](https://github.com/Dfam-consortium/RepeatModeler)
10. [https://academic.oup.com/g3journal/article/12/8/jkac160/6619165](https://academic.oup.com/g3journal/article/12/8/jkac160/6619165)
11. [https://www.frontiersin.org/journals/microbiology/articles/10.3389/fmicb.2017.01869/full](https://www.frontiersin.org/journals/microbiology/articles/10.3389/fmicb.2017.01869/full)
12. [https://pmc.ncbi.nlm.nih.gov/articles/PMC99139/](https://pmc.ncbi.nlm.nih.gov/articles/PMC99139/)
13. [https://www.nature.com/articles/s42003-023-05322-y](https://www.nature.com/articles/s42003-023-05322-y)
14. [https://pmc.ncbi.nlm.nih.gov/articles/PMC156086/](https://pmc.ncbi.nlm.nih.gov/articles/PMC156086/)
15. [https://academic.oup.com/nar/article/34/suppl_1/D423/1133253](https://academic.oup.com/nar/article/34/suppl_1/D423/1133253)
16. [https://www.biorxiv.org/content/10.1101/582072v1.full.pdf](https://www.biorxiv.org/content/10.1101/582072v1.full.pdf)
17. [https://pmc.ncbi.nlm.nih.gov/articles/PMC4513396/](https://pmc.ncbi.nlm.nih.gov/articles/PMC4513396/)
18. [https://pubmed.ncbi.nlm.nih.gov/15875012/](https://pubmed.ncbi.nlm.nih.gov/15875012/)
19. [https://www.frontiersin.org/journals/neuroscience/articles/10.3389/fnins.2022.886837/full](https://www.frontiersin.org/journals/neuroscience/articles/10.3389/fnins.2022.886837/full)
20. [https://pmc.ncbi.nlm.nih.gov/articles/PMC12278014/](https://pmc.ncbi.nlm.nih.gov/articles/PMC12278014/)
21. [https://www.jstage.jst.go.jp/article/ggs/94/6/94_18-00024/_pdf](https://www.jstage.jst.go.jp/article/ggs/94/6/94_18-00024/_pdf)
22. [https://pmc.ncbi.nlm.nih.gov/articles/PMC4575855/](https://pmc.ncbi.nlm.nih.gov/articles/PMC4575855/)
23. [https://academic.oup.com/nar/article/47/21/10994/5580909](https://academic.oup.com/nar/article/47/21/10994/5580909)
24. [https://journals.plos.org/ploscompbiol/article?id=10.1371%2Fjournal.pcbi.0010071](https://journals.plos.org/ploscompbiol/article?id=10.1371%2Fjournal.pcbi.0010071)
25. [https://pmc.ncbi.nlm.nih.gov/articles/PMC3045581/](https://pmc.ncbi.nlm.nih.gov/articles/PMC3045581/)
26. [http://dictybase.org/DictyAnnualConference/dicty2001.pdf](http://dictybase.org/DictyAnnualConference/dicty2001.pdf)
27. [http://dictybase.org/DictyAnnualConference/dicty2002.pdf](http://dictybase.org/DictyAnnualConference/dicty2002.pdf)
28. [https://mhlw-grants.niph.go.jp/system/files/2006/061031/200604003A/200604003A0024.pdf](https://mhlw-grants.niph.go.jp/system/files/2006/061031/200604003A/200604003A0024.pdf)
29. [https://academic.oup.com/nar/article/37/6/1809/1029900](https://academic.oup.com/nar/article/37/6/1809/1029900)