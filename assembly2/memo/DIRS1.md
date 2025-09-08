

https://www.ncbi.nlm.nih.gov/nuccore/M11339.1


OK！`dd_dirs1.fasta` をカスタムライブラリとして使い、`fir-chromesome1.fna` から**特異的にDIRS-1配列だけを検索する**コマンドに修正しますね。


# **_D. firmibasis_**
### 1️⃣ RepeatMasker 実行

`RepeatMasker`コマンドに **`-nolow`** オプションを追加します。また、結果が混ざらないように出力ディレクトリ名も変更します。

Bash

```
RepeatMasker -pa 4 -lib dd_dirs1.fasta -gff -dir DIRS1_nolow_output -engine ncbi -nolow /home/aki/GCA_036169595.1/ncbi_dataset/data/GCA_036169595.1/fir-chromesome1.fna
```

- **`-nolow`**: これが修正点です。このオプションを追加することで、シンプルリピートや低複雑性領域の検索が無効になり、純粋に`-lib`で指定した`dd_dirs1.fasta`との相同性のみを検索します。
    
- **`-dir DIRS1_nolow_output`**: 前回の結果と区別するため、出力ディレクトリ名を変更しました。
    

---

### 2️⃣ GFF から BED への変換

新しい出力ディレクトリ名に合わせて、GFFからBEDへの変換コマンドのパスを修正します。

Bash

```
gff2bed < DIRS1_nolow_output/fir-chromesome1.fna.out.gff > DIRS1_nolow_output/fir-chromesome1.DIRS1.bed
```

これで、シンプルリピートを除外し、`dd_dirs1.fasta`配列に特化したヒット領域のみを抽出したBEDファイルが作成されます。👍




# AX4 chromesome1
---

### ## 1️⃣ RepeatMasker 実行

`RepeatMasker`のライブラリを`Dfam`からあなたの`dd_dirs1.fasta`に変更し、シンプルリピートを無視する`-nolow`オプションを追加します。

Bash

```
RepeatMasker -pa 4 -lib dd_dirs1.fasta -gff -dir DIRS_masked_output_chr1 -engine ncbi -nolow chromosome1.fna
```

- **`-lib dd_dirs1.fasta`**: 修正点。Dfamライブラリの代わりに、あなたの特定のDIRS-1配列ファイルを指定します。
    
- **`-nolow`**: 修正点。シンプルリピートの検索を無効にし、`dd_dirs1.fasta`との相同性のみを検索させます。
    

---

### ## 2️⃣ DIRS のみ抽出（不要になります）

dd_dirs1.fastaのみをライブラリとして使ったので、RepeatMaskerの出力にはDIRS-1のヒットしか含まれません。

したがって、grepで"DIRS"を抽出するこのステップは不要になります。

---

### ## 3️⃣ GFF から BED への変換

`grep`が不要になったため、RepeatMaskerが直接出力したGFFファイルを`gff2bed`の入力として使います。

Bash

```
gff2bed < DIRS_masked_output_chr1/chromosome1.fna.out.gff > DIRS_masked_output_chr1/chromosome1.DIRS.bed
```

これで、`chromosome1.fna`の中から、あなたが指定したDIRS-1配列と相同性のある領域だけを抽出したBEDファイルが作成されます。👍