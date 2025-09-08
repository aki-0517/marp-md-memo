

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