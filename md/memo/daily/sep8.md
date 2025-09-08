

出てきたのは

```bash
(base) [aki@tardis ~]$ grep -i "DIRS" /home/aki/GCA_036169595.1/ncbi_dataset/data/GCA_036169595.1/DIRS_masked_output_chr1/fir-chromesome1.fna.out.gff \
>   | awk -F'\t' '{print $9}' \
>   | sed -E 's/.*Target \"Motif:([^"]+)\".*/\1/' \
>   | sort -u
DIRS-12_DR
DIRS-1e_Amnio
DIRS-4_DR
DIRS-6_DR
DIRS-8B_DR
DIRS-8_DR
```

---

提供されたリストの中では、論文で言及されている**DIRS1に直接相当するものはありません**。

リストにある`DIRS-12_DR`や`DIRS-1e_Amnio`などは、公共のリピート配列データベース（おそらくRepbase）に登録されている、他の生物種（`DR` = Danio rerio/ゼブラフィッシュ, `Amnio` = 羊膜類）で見つかったDIRSスーパーファミリーに属する別のトランスポゾンです。

---

## 解説

### 1. DIRS1とは何か

論文で言及されている**DIRS1**は、社会性アメーバである_Dictyostelium discoideum_（ディクチオステリウム・ディスコイデウム）で最初に見つかった特定のレトロトランスポゾン「_Dictyostelium_ Intermediate Repeat Sequence 1」の固有名です。

### 2. なぜリストにDIRS1がないのか

ユーザーが実行したコマンドは、おそらくRepeatMaskerのようなツールでゲノム全体からリピート配列を網羅的に検索した結果だと思われます。その際、使用したリピート配列のライブラリが、特定の生物種に限定しない**汎用的なデータベース**だったため、_D. firmibasis_の配列が、データベースに登録されている様々な生物のDIRS関連配列と相同性（類似性）があると判定されたのでしょう。

つまり、`grep`の結果は「_D. firmibasis_のゲノム上には、ゼブラフィッシュのDIRS-12や羊膜類のDIRS-1eと似た配列が存在する」ということを示しており、「_D. discoideum_のDIRS1そのもの」を指しているわけではありません。

### 3. 論文で言及されているDIRS1を見つけるには

論文の著者らはおそらく、既知の_D. discoideum_のDIRS1配列を参照配列として用い、_D. firmibasis_ゲノムとの相同性検索（BLASTなど）を行って、そのゲノム上のDIRS1領域を特定したと考えられます。

もしご自身で_D. firmibasis_ゲノム上のDIRS1領域を特定したい場合は、以下の手順が考えられます。

1. **参照配列の取得**: RepbaseやNCBIなどのデータベースから、_Dictyostelium discoideum_のDIRS1（Repbase ID: **DIRS1_DD** など）のコンセンサス配列または実際の配列データを取得します。
    
2. **相同性検索の実行**: 取得したDIRS1配列をクエリ（問い合わせ配列）として、`blastn`などのツールを使い、_D. firmibasis_のゲノム配列（`fir-chromesome1.fna`など）をデータベース（検索対象）として検索を実行します。
    

これにより、論文で議論されている_D. firmibasis_のゲノム上のDIRS1に相当する領域を直接見つけることができます。



https://www.ncbi.nlm.nih.gov/nuccore/M11339.1


OK！`dd_dirs1.fasta` をカスタムライブラリとして使い、`fir-chromesome1.fna` から**特異的にDIRS-1配列だけを検索する**コマンドに修正しますね。

元のワークフローを活かす**`RepeatMasker`を使う方法**と、より一般的で高速な**`BLAST`を使う方法**の2通りを解説します。

---

### ## 方法1：RepeatMaskerのカスタムライブラリ機能を使う（推奨）

これが一番簡単な修正方法です。Dfamライブラリの代わりに、あなたの`dd_dirs1.fasta`ファイルを直接指定します。

#### 1️⃣ RepeatMasker 実行

Bash

```
RepeatMasker -pa 4 -lib dd_dirs1.fasta -gff -dir DIRS1_specific_output -engine ncbi /home/aki/GCA_036169595.1/ncbi_dataset/data/GCA_036169595.1/fir-chromesome1.fna
```

- **`-lib dd_dirs1.fasta`**: これが変更点です。公共のライブラリではなく、手元のFASTAファイルをカスタムライブラリとして指定します。
    
- **`-dir DIRS1_specific_output`**: 出力ディレクトリ名を分かりやすく変更しました。
    

この方法の利点:

ライブラリにDIRS-1しか含まれていないため、出力されるGFFファイル (DIRS1_specific_output/fir-chromesome1.fna.out.gff) にはDIRS-1のヒット結果しか含まれません。したがって、次のgrepコマンドは不要になり、ワークフローがシンプルになります。

---

#### 2️⃣ GFF から BED への変換

`grep`が不要になったので、RepeatMaskerの出力GFFを直接BEDに変換します。

Bash

```
gff2bed < DIRS1_specific_output/fir-chromesome1.fna.out.gff > DIRS1_specific_output/fir-chromesome1.DIRS1.bed
```

これで、`fir-chromesome1.fna`内にある、`dd_dirs1.fasta`の配列と相同性のある領域だけがBEDファイルとして抽出されます。

---



「対策1：シンプルリピートの検索を無効にする」を反映するように、提供されたコマンドを修正します。

---

### ## 1️⃣ RepeatMasker 実行

`RepeatMasker`コマンドに **`-nolow`** オプションを追加します。また、結果が混ざらないように出力ディレクトリ名も変更します。

Bash

```
RepeatMasker -pa 4 -lib dd_dirs1.fasta -gff -dir DIRS1_nolow_output -engine ncbi -nolow /home/aki/GCA_036169595.1/ncbi_dataset/data/GCA_036169595.1/fir-chromesome1.fna
```

- **`-nolow`**: これが修正点です。このオプションを追加することで、シンプルリピートや低複雑性領域の検索が無効になり、純粋に`-lib`で指定した`dd_dirs1.fasta`との相同性のみを検索します。
    
- **`-dir DIRS1_nolow_output`**: 前回の結果と区別するため、出力ディレクトリ名を変更しました。
    

---

### ## 2️⃣ GFF から BED への変換

新しい出力ディレクトリ名に合わせて、GFFからBEDへの変換コマンドのパスを修正します。

Bash

```
gff2bed < DIRS1_nolow_output/fir-chromesome1.fna.out.gff > DIRS1_nolow_output/fir-chromesome1.DIRS1.bed
```

これで、シンプルリピートを除外し、`dd_dirs1.fasta`配列に特化したヒット領域のみを抽出したBEDファイルが作成されます。👍




Dictyostelium discoideum（社会性アメーバ）のアセンブリ結果において、rDNAを含むコンティグ NC_000895.1 がテロメアとして機能していることをIGVで視覚的に確認するための手順を以下にまとめます。

この作業の要点は、「**メインのアセンブリゲノムをリファレンスとしてIGVに読み込み、そこへrDNAコンティグをマッピングした結果をトラックとして表示する**」ことです。これにより、各染色体（scaffold）のどの位置にrDNA配列が存在するかが一目瞭然となります。

---

### ## 手順の概要

1. **準備**: 必要なソフトウェア（`samtools`, `minimap2`）を準備します。
    
2. **rDNA配列の抽出**: `ragtag_polished_round1.fasta` から `NC_000895.1` の配列だけを抜き出し、別のFASTAファイルとして保存します。
    
3. **アライメント**: 抽出したrDNA配列を、元のゲノムアセンブリ全体（`ragtag_polished_round1.fasta`）にマッピングします。
    
4. **IGVでの可視化**: IGVにゲノムアセンブリを読み込み、アライメント結果を表示して各染色体の末端を確認します。
    

---

### ## ステップ1：準備

作業を始める前に、ターミナル（コマンドライン）で以下のツールが利用可能であることを確認してください。もしインストールされていなければ、Condaなどを使ってインストールします。

- **samtools**: FASTAファイルの操作や、アライメント結果（SAM/BAM形式）の変換に必要です。
    
- **minimap2**: 高速な配列アライナー。コンティグ同士のアライメントに適しています。
    
- **IGV (Integrative Genomics Viewer)**: 結果を可視化するためのデスクトップアプリケーション。
    

Bash

```
# Condaを使ったインストール例
conda create -n mapping_tools -c bioconda samtools minimap2
conda activate mapping_tools
```

---

### ## ステップ2：rDNA配列 (`NC_000895.1`) の抽出

まず、巨大なアセンブリファイルからrDNAの配列だけを抜き出します。`samtools faidx` を使うのが最も確実で簡単です。

1. FASTAファイルのインデックスを作成
    
    これにより、samtoolsが高速に特定の配列へアクセスできるようになります。
    
    Bash
    
    ```
    samtools faidx assembly2/assembly-results/ragtag_flye_scaffold/ragtag_polished_round1.fasta
    ```
    
    実行すると、同階層に `.fai` ファイルが生成されます。
    
2. 特定の配列を抽出
    
    インデックスを利用して NC_000895.1 の配列を抽出し、rdna.fasta という名前で保存します。
    
    Bash
    
    ```
    samtools faidx assembly2/assembly-results/ragtag_flye_scaffold/ragtag_polished_round1.fasta NC_000895.1_RagTag_pilon > rdna.fasta
    ```
    
    ⚠️ 注意点:
    
    FASTAファイル内のヘッダー名が NC_000895.1 と完全に一致している必要があります。もしうまくいかない場合は、grep ">" ragtag_polished_round1.fasta コマンドでヘッダー名の一覧を確認してください。
    

---

### ## ステップ3：rDNAをゲノムアセンブリへアライメント

次に、抽出した `rdna.fasta` を、元の `ragtag_polished_round1.fasta` 全体に対してマッピングし、その位置情報を記録したBAMファイルを作成します。

1. minimap2でアライメント
    
    asm5 プリセットは、アセンブリ結果同士を比較する際に適しています。
    
    Bash
    
    ```
    minimap2 -ax asm5 \
      assembly2/assembly-results/ragtag_flye_scaffold/ragtag_polished_round1.fasta \
      rdna.fasta > rdna_on_assembly.sam
    ```
    
    これにより、アライメント結果が `rdna_on_assembly.sam` というSAMファイルに出力されます。
    
2. SAMをソート済みBAMに変換
    
    IGVで効率よく表示するために、SAMファイルをバイナリ形式のBAMファイルに変換し、ゲノム上の位置でソートします。
    
    Bash
    
    ```
    samtools view -bS rdna_on_assembly.sam | samtools sort -o rdna_on_assembly.sorted.bam
    ```
    
3. BAMファイルのインデックスを作成
    
    IGVがBAMファイル内のデータに高速アクセスするために、インデックスファイル（.bai）を作成します。
    
    Bash
    
    ```
    samtools index rdna_on_assembly.sorted.bam
    ```
    
    実行すると、`rdna_on_assembly.sorted.bam.bai` というファイルが生成されます。
    

これで、IGVで表示するためのファイル（`ragtag_polished_round1.fasta`, `rdna_on_assembly.sorted.bam`, `rdna_on_assembly.sorted.bam.bai`）がすべて揃いました。

---

![[Pasted image 20250908144209.png]]


That result is exactly what we'd expect, and it confirms your alignment worked correctly!

It shows that the rDNA contig (`NC_000895.1_RagTag_pilon`, the "read" in column 4) aligned perfectly to the reference scaffold of the same name (`NC_000895.1_RagTag_pilon`, column 1) from start (`0`) to end (`55553`). This is the main, perfect **"self-alignment"**.

---

### ## Why isn't the telomere information there?

The original goal was to find where rDNA aligns to the **ends of other chromosomes**.

By default, aligners like `minimap2` are optimized to find the single **best location** for a given sequence. In this case, the best location for the rDNA sequence is, of course, itself. It finds this perfect match and doesn't actively report the other, lower-quality matches at the telomeres of different chromosomes.

---

### ## Solution: Re-run the alignment to find all locations 🗺️

We need to tell `minimap2` to report not just the best hit, but also all the **secondary alignments**. We can do this by adding a few options to the command.

#### ### Step 1: Re-run `minimap2` to find secondary alignments

Run this modified `minimap2` command. The new options, `-N 50` and `-p 0`, tell it to report up to 50 secondary (alternative) alignments without filtering them by score.

Bash

```
minimap2 -ax asm5 -N 50 -p 0 \
  assembly2/assembly-results/ragtag_flye_scaffold/ragtag_polished_round1.fasta \
  rdna.fasta > rdna_on_assembly_all.sam
```

#### ### Step 2: Convert the new SAM to a sorted BAM

Bash

```
samtools view -bS rdna_on_assembly_all.sam | samtools sort -o rdna_on_assembly_all.sorted.bam
```

#### ### Step 3: Convert the new BAM to a BED file

Bash

```
bedtools bamtobed -i rdna_on_assembly_all.sorted.bam > rdna_alignments_all.bed
```

---

### ## Expected Result

Now, when you look inside the new file, `rdna_alignments_all.bed`, you should see many more lines:

Bash

```
head rdna_alignments_all.bed
```

The output should look something like this:

- The **first line** will be the same perfect self-alignment as before.
    
- **Subsequent lines** should show alignments to the other chromosomes, likely with start positions near `0` or end positions that match the full length of that chromosome. These are the telomeric hits you were looking for!
    

**Example of expected output:**

```
NC_000895.1_RagTag_pilon   0       55553   NC_000895.1_RagTag_pilon   60   +
NC_007087.3_RagTag_pilon   0       8532    NC_000895.1_RagTag_pilon   40   +
NC_007087.3_RagTag_pilon   4951120 4959406 NC_000895.1_RagTag_pilon   40   -
NC_007088.5_RagTag_pilon   0       9104    NC_000895.1_RagTag_pilon   42   +
...
```






# 次にすること

1. canuでもdirs1特定までやる。-> 長さ比較をやって、どっちの方が精度いいか確かめる
2. ax4とax2のdirs1の長さと構造の比較をする。シンテニー比較する以下のように![[Pasted image 20250908154103.png]]
https://genomebiology.biomedcentral.com/articles/10.1186/s13059-025-03578-7/figures/6
3. de novoで、まだわかってないrepeatも特定する
4. rdnaをちゃんとうまく特定する