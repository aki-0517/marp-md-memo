この論文では以下の内容が**明確に行われています**：

---

### 1. **トランスポゾン（TE）の特定**

✅ **行っている**

* **DIRS1 retrotransposon**（Dictyostelium Intermediate Repeat Sequence 1）というトランスポゾンの同定が行われており、以下のような解析が含まれています：

  > "Transposable elements were identified with tblastx v2.14.0 with e-values below 10⁻¹⁵ and reference transposable element sequences from Repbase (Genetic Information Research Institute) filtered to D. discoideum."

* DIRS1 は染色体の端に多く存在しており、**図3aや3bでも可視化**されています。

* 小分子RNA（siRNAなど）との関連や、発現制御にも触れられています。

---

### 2. **テロメアの特定**

✅ **行っている**

* D. firmibasis では、D. discoideum とは異なり**テロメア反復配列が同定された**と明記されています：

  > "In the new D. firmibasis assembly however, potential telomeric repeats with sequences 5′-GAGGAGAGAGTCCCTTTTTTT-3′ and 5′-GGGGAGAGACA-3′ could be identified."

* これらの繰り返し配列は、bowtieを用いて検出され、染色体の末端でのみ検出されたことも報告されています。

* 染色体1および6の両端、3および4の片端に**テロメアが存在**していることが示されています。

---

### 結論：

この論文では、

* **DIRS1 を含むトランスポゾンの同定・分布解析**
* **明確なテロメア配列の同定と局在解析**

の両方が行われており、非常に包括的なゲノムアノテーション研究となっています。


この論文（Edelbroek et al. 2024）の**アセンブリ以降の手順**を以下にまとめます。対象は主に「アセンブリ」→「ポリッシング」→「アノテーション」→「バリデーション・比較解析」部分です。

---

## 🧬 アセンブリ以降の手順まとめ

### 1. **アセンブリとポリッシング**

- **アセンブリ初期処理:**
    
    - Nanoporeリードを**Guppy v6.3.2**でアダプタ除去＆スプリット
        
    - **Filtlong**で1,000 bp未満のリードを除去し、長さ重視で最良の5 Gbpを選定
        
- **初期アセンブリ:**
    
    - **Flye v2.9.1**（nano-hqモード）でデノボアセンブリ
        
- **ロングリードで2回のポリッシング:**
    
    - **Medaka v1.7.2**を用いてポリッシング
        
- **ショートリード＋ロングリードで再ポリッシング:**
    
    - **NextPolish**で「task = best」モードにて：
        
        - ロングリードで2回
            
        - ショートリードで4回（2種のアルゴリズム）
            

---

### 2. **アセンブリの評価と比較**

- **カバレッジ評価:**
    
    - **minimap2** + **samtools**でロング/ショートリードそれぞれのカバレッジを算出
        
    - 平均カバレッジ：ロング199x、ショート540x、合計739x
        
- **旧アセンブリとの比較:**
    
    - **satsuma2**と自作スクリプトでゲノム比較
        
    - ギャップなし、完全長染色体構成（旧アセンブリには未定義塩基4.1 Mbあり）
        

---

### 3. **ゲノムアノテーション**

- **mRNA-seq解析：**
    
    - **STAR**でゲノムマッピング（3段階の発生ステージごと）
        
    - **Trinity**（genome-guided）でトランスクリプト構築（intron上限5,000 bp）
        
- **アノテーション統合:**
    
    - **MAKER v3.01.04**で以下の情報を統合し、遺伝子予測：
        
        - 組み立てられたトランスクリプト
            
        - _D. discoideum_ UniProtプロテオーム（ホモロジー）
            
    - **tRNAscan-SE v2.0.12**：379個のtRNA遺伝子を検出
        
    - **ShortStack v4.0.1**：miRNAの同定
        
    - **Infernal v1.1.4 + Rfam**：rRNA, snRNA, snoRNAなど
        
    - **tblastx + Repbase**：転移因子の同定
        
- **アノテーション合計：**
    
    - タンパク質コーディング遺伝子：10,564
        
    - ncRNA含む全体で**11,044遺伝子**
        

---

### 4. **バリデーションと比較ゲノミクス**

- **発現評価：**
    
    - 3ステージ×3リプリケート（9ライブラリ）でmRNA-seqを実施
        
    - 94%の遺伝子で発現（>100リード）を確認
        
- **ゲノム構造比較：**
    
    - _D. discoideum_との**シンテニー解析**（satsuma2）
        
    - 多くの染色体で整合性あり、特定領域に挿入や反転あり（DIRS1など）
        
    - **ミトコンドリア**と**rDNA含む線状エクストラ染色体**の保存も確認
        
- **telomeric repeatの同定：**
    
    - 新しいアセンブリでは明確なテロメア配列（例：5′-GAGGAGAGAGTCCCTTTTTTT-3′）を同定
        

---

## 🔗 データ公開先

- アセンブリ：GenBank (JAVFKY000000000)
    
- RNA-seq, DNA-seq：NCBI BioProject (PRJNA1008051)
    

---


以下に、「3. ゲノムアノテーション」と「4. バリデーションと比較ゲノミクス」の詳細を丁寧にまとめます。各解析ステップの目的・使用ツール・結果を分かりやすく整理しました。

---

## 3. 🧬 ゲノムアノテーション（詳細）

### 🔹 mRNA-seq解析

**目的：** 発現トランスクリプトからエビデンスを得て遺伝子予測を高精度に行うため。

- **試料：**  
    _D. firmibasis_ と _D. discoideum_ の3段階発生ステージ（計9サンプル/種）
    
    - Vegetative stage（栄養状態）
        
    - Slug stage（移動性多細胞体）
        
    - Fruiting body（子実体）
        
- **手順：**
    
    - RNAはTRIzolで抽出し、DNase処理とPhenol-Chloroform精製を実施
        
    - ライブラリ作成：TruSeq Stranded mRNA kit（polyA選択）
        
    - シーケンス：NovaSeq 6000（150 bp PE）
        
    - アラインメント：**STAR v2.7.10b** でゲノムマッピング
        
    - トランスクリプト構築：**Trinity v2.14.0**（genome-guided、intron最大5 kb）
        

---

### 🔹 MAKERによる遺伝子アノテーション統合

**目的：** トランスクリプト情報とホモロジー情報の統合により、信頼性の高い遺伝子予測を行う。

- **アノテーションソース：**
    
    1. **mRNA-seqから得られたトランスクリプト**
        
    2. _D. discoideum_ UniProtプロテオーム（UP000002195）
        
- **パイプライン：**
    
    - **MAKER v3.01.04**：両者の証拠を統合し、プロテインコーディング遺伝子を予測
        
    - **blastp v2.14.0**：_D. discoideum_ タンパク質とのホモロジー解析
        

---

### 🔹 ncRNA & 転移因子のアノテーション

|種類|ツール|補足|
|---|---|---|
|tRNA|tRNAscan-SE v2.0.12|379個検出（_D. discoideum_は418個）|
|miRNA|ShortStack v4.0.1|既知miRNAとsRNA-seq（PRJNA972620）を利用|
|Class I RNA|構造・モチーフベースで独自解析|10個同定（_D. discoideum_は38個）|
|その他ncRNA|Infernal v1.1.4 + Rfam（Amoebozoa）|rRNA, snRNA, snoRNA|
|転移因子|tblastx v2.14.0 + Repbase|DIRS1などのレトロトランスポゾン|

---

### ✅ 最終アノテーション数（手動チェック済み）:

- タンパク質コーディング遺伝子：**10,564**
    
- ncRNA等含む総遺伝子数：**11,044**
    
- 平均遺伝子密度：**70.6%**（主染色体上では比較的均一に分布）
    

---

## 4. 🔍 バリデーションと比較ゲノミクス（詳細）

### 🔹 発現バリデーション（mRNA-seq）

- mRNA-seqリードのマッピング率：**95%**
    
- 遺伝子への割り当て率（featureCounts）：**94%**
    
- 発現確認（>100リード）：**10,564中 9,934遺伝子（94%）**
    

⇒ これにより、アノテーションの網羅性と正確性が裏付けられた。

---

### 🔹 ゲノム構造比較（シンテニー解析）

- **比較対象：**
    
    - 旧 _D. firmibasis_ アセンブリ（ASM27748v1）
        
    - _D. discoideum_（AX4系統 dicty_2.7アセンブリ）
        
- **手法：**
    
    - **satsuma2** を用いたシンテニー検出
        
    - 図示には **circlize**, **karyoploteR**（Rパッケージ）を使用
        
- **主な発見：**
    
    - 染色体2, 3, 5はほぼ完全な整合性（相同な _D. discoideum_ 染色体あり）
        
    - 染色体1は反転や挿入（DIRS1）あり
        
    - 染色体4・6は _D. discoideum_ 染色体2との整合性ありつつ分割されている
        

---

### 🔹 ミトコンドリア・エクストラ染色体の保存

- **ミトコンドリア：**
    
    - 完全に環状にアセンブルされ、_D. discoideum_と高い保存性を持つ
        
- **rDNA含有のエクストラ染色体：**
    
    - _D. discoideum_ではパリンドローム構造を持つ線状DNA
        
    - _D. firmibasis_では2つの小コンティグとして同定され、rRNA（5S, 5.8S, 18S, 28S）を含む
        

---

### 🔹 テロメア配列の同定

- 新アセンブリでは、明確なテロメア配列が以下のように検出された：
    
    - 配列例1: `5′-GAGGAGAGAGTCCCTTTTTTT-3′`
        
    - 配列例2: `5′-GGGGAGAGACA-3′`（2つで1ユニット）
        
- **検出条件：**
    
    - Bowtie v1.3.1 により1–2ミスマッチ許容で検索
        
    - 染色体1と6の両端、染色体3と4の片端でテロメアリピートを確認
        
    - 他の部位では見つからず、テロメアとしての正当性が高い
        

---


---

### 🔹 理由（要点）

#### 1. **トランスポゾン（TE）の特定**

- **推奨タイミング：**  
    → ゲノムアノテーション（タンパク質コーディング遺伝子予測）が終わった**後**が望ましい。
    
- **理由：**
    
    - TE は **誤って遺伝子として予測されることがある**（false positives）
        
    - 先にMAKERなどで遺伝子予測した後に、**そのうちTE由来のものをマスクまたは再注釈する**のが一般的
        
    - 特に `RepeatMasker` や `tblastx + Repbase` の結果を元に、TE 領域を分類し直すフローが多い
        

---

#### 2. **テロメア配列の特定**

- **タイミングに縛りはない**が、**最終アセンブリの確認的に後で行う方が自然**
    
- **理由：**
    
    - テロメア解析は主に「アセンブリの完全性確認」「染色体末端の構造確認」のため
        
    - 遺伝子アノテーションとは独立しており、**アノテーション前でも後でもOK**
        
    - ただし、遺伝子密度やDIRS1の局在などとあわせて可視化するには、アノテーション後の方が便利
        

---

## 🔁 推奨フロー（簡略版）

```text
1. ゲノムアセンブリ（FASTA完成）
2. mRNA-seq 取得・クリーンアップ
3. 遺伝子予測（MAKER + Trinity + ホモロジー）
4. TEアノテーション（RepeatMasker / tblastx）
5. ncRNAアノテーション（miRNA, tRNA, etc.）
6. テロメア配列の特定（末端repeat探索）
7. 可視化・統合解析（Circos等）
```

---

## ✅ 結論まとめ

|処理|アノテーション前にやってもよい？|推奨タイミング|備考|
|---|---|---|---|
|トランスポゾン特定|❌（非推奨）|アノテーション後|誤認識防止のため後が良い|
|テロメア特定|⭕（OK）|いつでも可|表現・可視化の都合で後でも良い|

---

必要であれば、RepeatMasker + Repbase や telomere repeat finder の実行例も提供できます！

_Dictyostelium discoideum_ AX2（またはAX4）株のようなATリッチで特殊なゲノムにおいて、**テロメアの特定**は一般的な“TTAGGG”パターンとは異なるアプローチが必要です。以下に、AX2（AX4）ゲノムで行われた実際の方法や、再現可能な手法を説明します。

---

## 🔍 テロメア特定の方法（_D. discoideum_ AX2 例）

### ✅ **背景**

- _D. discoideum_ のゲノムでは、**一般的なテロメアリピートは見つかっていません**（Eichinger et al., _Nature_ 2005）。
    
- 代わりに、染色体末端には**DIRS1レトロトランスポゾン**や**A/T-richリピート**があることが知られている。
    
- _D. firmibasis_ の例では、「**GAGGAGAGAGTCCCTTTTTTT**」や「**GGGGAGAGACA**」といった非典型的なリピートがテロメアとして機能していると考えられる。
    

---

## 🧪 方法ステップ（一般的かつ_D. discoideum_向け）

### 1. **ゲノムアセンブリの染色体末端抽出**

```bash
bedtools flank -i chrom_sizes.bed -g chrom_sizes.bed -l 10000 -r 0 > chrom_ends.bed
bedtools getfasta -fi genome.fasta -bed chrom_ends.bed > chrom_ends.fasta
```

- 染色体の両端10 kbpなどを抽出する。
    

---

### 2. **繰り返し配列の同定**

#### A. **自己BLAST**

```bash
makeblastdb -in chrom_ends.fasta -dbtype nucl
blastn -query chrom_ends.fasta -db chrom_ends.fasta -outfmt 6 > self_blast.tsv
```

→ 高頻度に繰り返される20～30 bp程度のモチーフを検出

#### B. **Tandem Repeat Finder (TRF)**

```bash
trf chrom_ends.fasta 2 7 7 80 10 50 500 -d -h
```

- テロメアリピートに似た短いタンデムリピートを検出するのに適している
    

---

### 3. **既知パターンとの照合（リピートスクリーニング）**

以下のような既知配列を用いて検索（Bowtie など）

#### 例：D. firmibasisで見つかった配列（参考）

- `GAGGAGAGAGTCCCTTTTTTT`
    
- `GGGGAGAGACA`
    

```bash
bowtie-build genome.fasta genome_index
bowtie -v 2 -a --best genome_index -c GAGGAGAGAGTCCCTTTTTTT
```

---

### 4. **位置情報の可視化**

- Rの `circlize` や `karyoploteR` を使ってテロメア候補の位置をプロットし、末端に存在するか確認。
    
- 小RNAや遺伝子密度と重ねて、機能的意味を探る。
    

---

## 🧾 文献でのテロメア状構造の証拠（AX2/AX4）

- Eichinger et al., _Nature_, 2005  
    → テロメア配列が見つからず、「DIRS1」や「A+Tリッチな終端」が構造的な末端とされている
    
- Glöckner et al., _Genome Res._, 2001  
    → DIRS1 は染色体末端近くにクラスター状に存在
    

---

## ✅ まとめ：_D. discoideum_ テロメア特定手法

|ステップ|ツール|目的|
|---|---|---|
|染色体末端抽出|`bedtools`|末端の配列だけ取り出す|
|繰り返し検出|`TRF`, `blastn`|タンデム/散在リピート探索|
|既知配列検索|`bowtie`, `grep`|G-richまたはA/T-rich配列のスクリーニング|
|可視化|`R (circlize, karyoploteR)`|テロメア候補の染色体上位置を確認|

---

了解しました！  
`ragtag_polished_round1.fasta` がアセンブリ後の最終配列であれば、それに対して **テロメア様リピートの特定**を行うことができます。

以下に、**Linuxコマンドラインで実行できる再現性の高いパイプライン**（簡単な方法から本格的な解析まで）を提供します。

---

## ✅ ステップ 1：染色体末端（5kbまたは10kb）の抽出

### 必要ツール：

- [`seqkit`](https://bioinf.shenwei.me/seqkit/)（高速FASTA処理）
    
- または `bedtools`＋`.fai` でも可
    
### 例：両端10kbを抽出（`seqkit`版）

```bash
# インデックス作成（高速処理のため）
seqkit fx2tab -n -l assembly2/assembly-results/ragtag_flye_scaffold/ragtag_polished_round1.fasta > lengths.txt

# 左右両端10kbを抽出（片方ずつ）
mkdir telomere_ends
while read id len; do
  seqkit subseq -r 1:10000 -w 0 -i -n -t DNA ragtag_polished_round1.fasta -r "$id:1-10000" > telomere_ends/${id}_head.fasta
  start=$((len-9999))
  seqkit subseq -r "$start:$len" -w 0 -i -n -t DNA ragtag_polished_round1.fasta -r "$id:$start-$len" > telomere_ends/${id}_tail.fasta
done < lengths.txt
```

---

## ✅ ステップ 2：候補リピートの探索（簡易）

### 方法A：既知パターン（例：GAGGAGAGAGTCCCTTTTTTT）を grep で探索

```bash
grep -B1 -A1 "GAGGAGAGAGTCCCTTTTTTT" telomere_ends/*.fasta
```

### 方法B：Tandem Repeat Finder（TRF）で探索

```bash
# https://tandem.bu.edu/trf/trf.download.html から trf を取得しておく
trf telomere_ends/all_ends.fasta 2 7 7 80 10 50 500 -d -h
```

出力ファイル（`.dat`）に繰り返し構造が記載されます。

---

## ✅ ステップ 3：全染色体での分布可視化（任意）

→ 繰り返しが本当に末端にしか無いかを確認したい場合

### 方法：既知のパターンを genome 全体で検索

```bash
# index化
bowtie-build ragtag_polished_round1.fasta genome_index

# 例の配列で検索（2ミスマッチまで許容）
bowtie -v 2 -a --best genome_index -c GAGGAGAGAGTCCCTTTTTTT > telomere_hits.txt
```

---

## ✅ 結果の解釈

- **検出される繰り返しが、染色体の端に集中していれば**、それはテロメア候補と考えられます。
    
- 逆にゲノム全体に散在している場合は、テロメアではなく内部リピートである可能性が高いです。
    

---

## 🔁 参考：自動解析スクリプト化（必要なら提供可）

- bash スクリプト（自動で両端抽出→リピート検出）
    
- Snakemake フロー（FASTA → 末端抽出 → grep/TRF → レポート）
    
- 可視化（R + `circlize` や `karyoploteR`）
    

---

### 👉 次にどうしましょうか？

- 上記を **スクリプト化**して渡しましょうか？
    
- それとも実行環境で試した結果の**出力解釈のサポート**が欲しいですか？
    

必要に応じて進めます。

telomere_ends.fasta.2.7.7.80.10.50.500.dat
```
Tandem Repeats Finder Program written by:

Gary Benson
Program in Bioinformatics
Boston University
Version 4.09


Sequence: NC_000895.1_RagTag_pilon:0-10000



Parameters: 2 7 7 80 10 50 500


6654 6685 15 2.1 15 94 0 55 65 0 3 31 1.08 TAATAAATATAAGAA TAATAAATATAAGAATAATAAATATAATAATA
8619 8659 12 3.5 12 86 6 66 70 9 9 9 1.34 AAGTAAAAAAAC AAGTAAAAAAACAAGTAAAACACAAGTAAAAAAACAAGTAA
9816 9874 25 2.5 23 75 8 64 76 6 6 10 1.16 AATAAAGAAAAAACAAAAAATAA AATAAAGAAAAAACAAAAAATAAAACCAAGAAACATAAGAAAATATAAAATAAAGAAAA


Sequence: NC_000895.1_RagTag_pilon:45553-55553



Parameters: 2 7 7 80 10 50 500


3185 3220 18 2.0 18 88 0 54 50 8 2 38 1.47 AATTAAATAATACTTTAA AATTACATAGTACTTTAAAATTAAATAATACTTTAA


Sequence: NC_007087.3_RagTag_pilon:0-10000



Parameters: 2 7 7 80 10 50 500


```