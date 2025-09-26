## Structural composition and evolution of jujube centromere reveal a dominant role for LTR Retrotransposons
https://academic.oup.com/hr/advance-article/doi/10.1093/hr/uhaf244/8254148

## ltr retriever
https://github.com/oushujun/LTR_retriever

### TE sorter
https://github.com/zhangrengang/TEsorter









# 968876.pts-54.tardis

`LTR_retriever` を用いて、指定されたゲノムアセンブリファイル (`assembly2/assembly-results/ragtag_flye_scaffold/ragtag_polished_round1.fasta`) からLTRレトロトランスポゾンを同定する手順を以下にまとめます。

この手順は、`LTR_retriever` のドキュメントに記載されているベストプラクティスに基づいています。

---

## 概要

`LTR_retriever` を実行するには、大きく分けて以下の3つのステップが必要です。

1. **環境構築**: `conda` を用いて `LTR_retriever` とその依存ツールをインストールします。
    
2. **LTR候補の探索**: `LTRharvest` と `LTR_FINDER` をゲノムに実行し、LTRレトロトランスポゾンの候補領域をリストアップします。
    
3. **LTR_retrieverの実行**: 上記の候補リストを精査し、高精度なLTRレトロトランスポゾンのライブラリ作成とゲノムアノテーションを行います。
    

---

## 手順

### ステップ1: 環境構築 (Condaを使用)

まず、`LTR_retriever` と必要な依存ツール（RepeatMasker, BLAST+など）をインストールするための `conda` 環境を構築します。まだ `LTR_retriever` をダウンロードしていない場合は、`git` でクローンします。

Bash

```
# LTR_retrieverのリポジトリをクローン
git clone https://github.com/oushujun/LTR_retriever.git

conda config --set channel_priority flexible

# ymlファイルを使ってconda環境を構築（推奨、最も早い方法）
conda env create -f LTR_retriever/LTR_retriever.yml

# 作成した環境を有効化
conda activate LTR_retriever
```

これで、`LTR_retriever` を実行する準備が整いました。

### ステップ2: LTR候補の探索

`LTR_retriever` に入力するためのLTR候補ファイルを、`LTRharvest` と `LTR_FINDER` を使って作成します。

1. 変数の設定

作業を簡単にするため、ファイルパスを変数に設定しておきます。

Bash

```
# ゲノムファイルのパスを設定
GENOME="assembly2/assembly-results/ragtag_flye_scaffold/ragtag_polished_round1.fasta"

# 出力ファイルの接頭辞を設定
PREFIX="ragtag_polished_round1"

# 使用するCPUスレッド数を設定（環境に合わせて調整してください）
THREADS=16
```

2. LTRharvest の実行

LTRharvest は GenomeTools スイートに含まれるツールです。まずゲノムのインデックスを作成し、次に探索を実行します。

Bash

```
# インデックス作成
gt suffixerator -db ${GENOME} -indexname ${PREFIX} -tis -suf -lcp -des -ssp -sds -dna

# LTRharvestの実行
gt ltrharvest -index ${PREFIX} -minlenltr 100 -maxlenltr 7000 -mintsd 4 -maxtsd 6 -motif TGCA -motifmis 1 -similar 85 -vic 10 -seed 20 -seqids yes > ${PREFIX}.harvest.scn
```

3. LTR_FINDER_parallel の実行

次に、LTR_FINDER を並列実行して候補を探索します。

Bash

```
conda install -c bioconda ltr_finder_parallel

LTR_FINDER_parallel -seq ${GENOME} -threads ${THREADS} -harvest_out -size 1000000 -time 300
```

- **注意**: このコマンドにより、`*.finder.combine.scn` という名前のファイルが生成されます。実際のファイル名はゲノムファイル名に基づきますので、`ls` コマンドで確認してください。ここでは `${PREFIX}.finder.combine.scn` という名前で生成されたと仮定します。
    

4. 候補ファイルの結合

LTRharvest と LTR_FINDER の出力を一つのファイルにまとめます。

Bash

```
cat ${PREFIX}.harvest.scn ${GENOME}.finder.combine.scn > ${PREFIX}.rawLTR.scn
```

### ステップ3: LTR_retriever の実行

最後に、ステップ2で作成した候補ファイル (`${PREFIX}.rawLTR.scn`) を使って `LTR_retriever` を実行します。

Bash

```
LTR_retriever -genome ${GENOME} -inharvest ${PREFIX}.rawLTR.scn -threads ${THREADS}
```

このコマンドが完了すると、カレントディレクトリに以下のような主要な出力ファイルが生成されます。

- **`${GENOME}.LTRlib.fa`**: 非冗長なLTRレトロトランスポゾンのライブラリ（FASTA形式）。ゲノムアノテーションに利用できます。
    
- **`${GENOME}.pass.list.gff3`**: 信頼性の高いLTRレトロトランスポゾンの構造情報（GFF3形式）。
    
- **`${GENOME}.out.gff`**: 生成されたライブラリを使ってゲノム全体をアノテーションした結果（GFF形式）。
    
- **`${GENOME}.out.LAI`**: アセンブリの連続性を評価するLAI (LTR Assembly Index) スコア。
    

---

## 注意点

- **実行パス**: 上記のコマンドは、`LTR_retriever` や `gt` などのツールにパスが通っていることを前提としています。`conda activate` していれば通常は問題ありませんが、もし `command not found` エラーが出る場合は、`./LTR_retriever/LTR_retriever` のように実行ファイルのフルパスを指定してください。
    
- **計算時間**: ゲノムサイズによっては、各ステップで数時間から数日かかる場合があります。
    
- **配列名**: ドキュメントでは、FASTAファイルの配列名（`>` の後の文字列）を短くシンプルなものにすることが推奨されています。長い、あるいは特殊文字を含む配列名だと、途中でエラーが発生する可能性があります。

```bash
##############################
####### Result files #########
##############################

Table output for intact LTR-RTs (detailed info)
        ragtag_polished_round1.fasta.mod.pass.list (All LTR-RTs)
        ragtag_polished_round1.fasta.mod.nmtf.pass.list (Non-TGCA LTR-RTs)
        ragtag_polished_round1.fasta.mod.pass.list.gff3 (GFF3 format for intact LTR-RTs)

LTR-RT library
        ragtag_polished_round1.fasta.mod.LTRlib.redundant.fa (All LTR-RTs with redundancy)
        ragtag_polished_round1.fasta.mod.LTRlib.fa (All non-redundant LTR-RTs)
        ragtag_polished_round1.fasta.mod.nmtf.LTRlib.fa (Non-TGCA LTR-RTs)

Whole-genome LTR-RT annotation by the non-redundant library
        ragtag_polished_round1.fasta.mod.LTR.gff3 (GFF3 format)
        ragtag_polished_round1.fasta.mod.out.fam.size.list (LTR family summary)
        ragtag_polished_round1.fasta.mod.out.superfam.size.list (LTR superfamily summary)
```

```bash
gff2bed < ragtag_polished_round1.fasta.mod.pass.list.gff3 > ragtag_polished_round1.fasta.mod.pass.list.bed

sed 's/_R\t/_RagTag_pilon\t/g' ragtag_polished_round1.fasta.mod.pass.list.bed > ragtag_polished_round1.fasta.mod.pass.list.fixed.bed
```




# 1471231.pts-54.tardis

提供されたドキュメントを基に、ゲノムアセンブリファイル assembly2/assembly-results/ragtag_flye_scaffold/ragtag_polished_round1.fasta に対してTEsorterを実行する手順を以下にまとめます。

ゲノム配列全体を解析するため、`TEsorter`の **GENOMEモード** を使用します。これは、あらかじめ抽出されたTE配列を分類するデフォルトのELEMENTモードとは異なり、ゲノム全体からTEのタンパク質ドメインを直接探索する機能です。

---

### ## 前提条件：TEsorterのインストール

まず、`TEsorter`とその依存ソフトウェアがインストールされていることを確認してください。ドキュメントで推奨されている`bioconda`を使用するのが最も簡単です。

Bash

```
conda install -c bioconda tesorter
```

これにより、`hmmscan`や`blast+`などの必要なツールも同時にインストールされます。

---

はい、承知いたしました。

Dictyostelium discoideum（和名：タマホコリカビ）のゲノムアセンブリにTEsorterを実行するための手順を、この生物種に合わせて最適化して説明します。

---

### ## 🔬 データベースの選択

`Dictyostelium discoideum`は原生生物（アメーボゾア）であり、**植物でも後生動物でもありません**。そのため、特定の生物群（`-db rexdb-plant`や`-db rexdb-metazoa`）に特化したデータベースではなく、植物と動物の両方を広くカバーする**デフォルトの`rexdb`データベースを使用するのが最も適切**です。これにより、進化的に保存された多様なトランスポゾン（TE）のドメインを検出できる可能性が最も高くなります。

コマンド実行時に`-db`オプションを省略すれば、自動的に`rexdb`が使われます。

---

### ## 💻 実行コマンド

基本的な手順は同じですが、データベースの選択を考慮してコマンドを以下のように設定します。

`TEsorter`の**GENOMEモード** (`-genome`オプション) を使って、ゲノム全体からTEのタンパク質ドメインを探索します。

#### ### 推奨コマンド例

Bash

```
TEsorter assembly2/assembly-results/ragtag_flye_scaffold/ragtag_polished_round1.fasta \
    -genome \
    -p 20 \
    -prob 0.9 \
    -cov 30 \
    -eval 1e-5 \
    -score 1
```

**コマンドの解説:**

- **`assembly2/assembly-results/...fasta`**: 解析対象のゲノムファイルです。
    
- **`-genome`**: ゲノム配列全体を解析するモードを有効にします。
    
- **`-p 20`**: 使用するCPUコア数です。お使いの計算機の環境に合わせて調整してください。
    
- **`-prob 0.9`, `-cov 30`, `-eval 1e-5`, `-score 1`**: 信頼性の高いTEドメインのみを検出するための推奨パラメータです。感度を上げたい場合は、これらの閾値を緩めることも可能です（例: `-cov 10 -eval 1e-2`）。
    

---

### ## 📂 出力ファイルについて

**GENOMEモード**で実行した場合、個々のTE配列の分類リスト (`*.cls.tsv`など) は**出力されません**。

代わりに、ゲノム上に存在するTE関連のタンパク質ドメインに関する情報が出力されます。

- **`*.rexdb.dom.gff3`**: ゲノム上のどこにドメインが存在するかを示すアノテーションファイルです。ゲノムブラウザで可視化する際に非常に役立ちます。
    
- **`*.rexdb.dom.faa`**: 検出されたドメインのタンパク質配列です。このファイルを使って、系統解析など次のステップに進むことができます。
    
- **`*.rexdb.dom.tsv`**: 検出されたドメインの詳細な情報（スコア、位置など）がまとめられたリストです。
    

#### ### (オプション) TEドメイン領域のマスキング

ゲノム配列中のTEドメイン領域をマスク（例えば、遺伝子予測の前に除外する）したい場合は、`-mask`オプションを追加できます。

- **ソフトマスキング** (塩基を小文字に変換):
    
    Bash
    
    ```
    TEsorter ... -genome -mask soft ...
    ```
    
- **ハードマスキング** (塩基を'N'に変換):
    
    Bash
    
    ```
    TEsorter ... -genome -mask hard ...
    ```
    

以上の手順で、`Dictyostelium discoideum`のゲノムアセンブリに対するTEドメインの探索を実行できます。

```bash
gff2bed < ragtag_polished_round1.fasta.rexdb.dom.gff3 > ragtag_polished_round1.fasta.rexdb.dom.bed


```







![[Pasted image 20250919163820.png]]

