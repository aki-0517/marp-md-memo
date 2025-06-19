反復配列（リピート領域）は、ゲノムアセンブリの"落とし穴"になることが多い重要な要素です。上で示した連続性（N50 など）や精度（ミスマッチ・インデル率）、BUSCO、ゲノムカバレッジは、あくまで"全体"の指標なので、リピート領域の扱いまでは直接評価していません。以下のような手順で、反復配列を別に考慮してみることをおすすめします。

---

## 1. 反復配列って何？

* **種類**：衛星DNA、トランスポゾン（LINE/SINE/LTR）、マイクロサテライトなど
* **問題点**：似た配列がゲノム中に多数存在するため、アセンブリの際に「どこにつなげるべきか」があいまいになりやすい

---

## 2. 反復配列を評価する手順例

1. **リピーターマスキング（RepeatMasker）**

   * まずアセンブリ配列に対して RepeatMasker をかけ、どの領域が既知のリピートかラベル付けします。
   * 結果として、ゲノム中のリピートの種類・長さ・総割合（％）がわかります。

2. **リピートカバレッジの算出**

   * RepeatMasker の出力から「リピート領域全体の長さ」を集計し、アセンブリ全長に対する割合を計算。
   * 参照ゲノムでも同様に計算すれば、"過度にマスクされていないか"や "逆にリピートが欠落していないか"を比較できます。

3. **コンティグとリピートの重なり分析**

   * 各コンティグがどの程度リピート領域をまたいでいるかを調べます。
   * 例えば、長いコンティグにリピートが散在していれば「リピートをまたぐ良好なつながり」が得られていると判断できますが、リピート領域で断片化している場合は "アセンブリの難所" と言えます。

4. **LAI（LTR Assembly Index）**

   * 真核生物では LTRレトロトランスポゾンが大きなリピート成分を占めるため、LAI を計算するとアセンブリのリピート解像度が定量的に評価できます。
   * LAI≥10 なら "高品質なリピートアセンブリ" とされることが多いです。

5. **Misassembly のチェック**

   * QUAST や MUMmer を使い、参照ゲノムとのアライメントで "リピート領域での不整合"（逆向き配置、過大/過小コピーなど）を可視化・集計します。
   * SyRI などで構造バリアントを検出すると、リピートによる誤配置や欠落が詳しくわかります。

---

## 3. ツール・コマンド例

```bash
# 1) RepeatMasker でマスキング
RepeatMasker -species "dictyostelium discoideum" assembly.fasta

# 2) マスクレポートから割合を集計（簡易例）
grep "^Total interspersed" *.out

# 3) QUAST で misassemblies 確認
quast.py assembly.fasta -R reference.fna --eukaryote

# 4) LAI 計算（GenomeTools の LTR_retriever を利用）
LTR_retriever -genome assembly.fasta -inharvest harvest.out
```

---

### ポイントまとめ

* **既存指標だけでは不十分**：N50 や BUSCO はリピート特有の問題を隠すことがある
* **マスクと可視化が第一歩**：どれだけリピートを拾えているか、逆に欠落しているかを確認
* **LAI や misassembly カウント**：リピートをまたいだ連続性・正確性をより詳細に評価

まずは RepeatMasker でマスク→レポート確認、QUAST/MUMmer で misassembly を可視化、必要に応じて LAI を算出、という流れで解析してみてください。これで「リピート配列をどれだけ正確につなげられているか」がクリアになります。



### exec
```bash
RepeatMasker \
  -species "dictyostelium discoideum" \
  -pa 8 \
  -xsmall \
  /home/aki/assembly2/assembly-results/flye/assembly.fasta
```



以下では、Flyeで組み立てたアセンブリ（例: `assembly2/assembly-results/flye/assembly.fasta`）に対して、反復配列（リピート領域）を評価・可視化する一連の手順を示します。要点をまとめると：

- **Step 1.** RepeatMaskerで既知リピートをマスクし、リピート領域の種類・割合を把握
    
- **Step 2.** マスク結果からゲノム全体におけるリピートのカバレッジを算出
    
- **Step 3.** Bedtoolsでコンティグとリピートの重なりを解析
    
- **Step 4.** LTR_rretrieverでLAI（LTR Assembly Index）を計算し、レトロトランスポゾンの解像度を定量評価
    
- **Step 5.** QUAST／MUMmer＋SyRIで、リピート領域におけるMisassemblyや構造変異を可視化
    

---

## Step 1. RepeatMaskerによるマスキング

まず、既知の反復配列をソフトマスク※して、各リピートの種類と分布を取得します。

```bash
# Flyeアセンブリに対してRepeatMasker実行
RepeatMasker \
  -species "dictyostelium discoideum" \
  -pa 8 \
  -xsmall \
  assembly2/assembly-results/flye/assembly.fasta
```

- `-species`：対象生物種を指定します。
    
- `-pa`：並列コア数。
    
- `-xsmall`：ソフトマスク（小文字に置換）します([training.galaxyproject.org](https://training.galaxyproject.org/training-material/topics/genome-annotation/tutorials/repeatmasker/tutorial.html?utm_source=chatgpt.com "Genome Annotation / Masking repeats with RepeatMasker / Hands-on")).
    

これにより、`.out`・`.masked`・`.tbl`などのファイルが生成され、リピートの種類・コピー数・総長がレポートされます([repeatmasker.org](https://www.repeatmasker.org/webrepeatmaskerhelp.html?utm_source=chatgpt.com "Documentation - RepeatMasker")).

---

## Step 2. リピートカバレッジの算出

RepeatMaskerの出力から「リピート領域全長」を集計し、アセンブリ全長に対する割合を求めます。

```bash
# 総リピート長をgrepで抽出（例：interspersed repeats総長）
grep "^Total interspersed" /home/aki/assembly2/assembly-results/flye/assembly.fasta.tbl

# アセンブリ全長を取得
grep -v "^>" /home/aki/assembly2/assembly-results/flye/assembly.fasta | wc -c
```

- `assembly.fasta.tbl`にはリピートクラスごとの総長が記載されています([training.galaxyproject.org](https://training.galaxyproject.org/training-material/topics/genome-annotation/tutorials/repeatmasker/tutorial.html?utm_source=chatgpt.com "Genome Annotation / Masking repeats with RepeatMasker / Hands-on")).
    
- アセンブリ全長と比較し、リピート率（％）を計算してください。
    

---

## Step 3. コンティグ–リピート重なり解析

コンティグ単位で「リピートをまたいでいるか」「断片化されているか」を見るため、Bedtoolsを使ってBED形式で重なりを集計します。

```bash
# 1) マスク済み配列からリピート領域BEDを作成
grep -v "^#" assembly.fasta.out | awk '{print $5"\t"$6"\t"$7}' > repeats.bed

# 2) FlyeコンティグをBEDに変換
# samtools faidxを使って
samtools faidx assembly2/assembly-results/flye/assembly.fasta
cut -f1,2 assembly2/assembly-results/flye/assembly.fasta.fai | \
  awk '{print $1"\t0\t"$2}' > contigs.bed

# 3) Bedtoolsで重なり集計
bedtools intersect \
  -a contigs.bed \
  -b repeats.bed \
  -wo \
  | awk '{sum[$1]+=$NF} END{for(c in sum) print c, sum[c]}' > contig_repeat_overlap.txt
```

- `-wo`オプションで、重なった長さ（bp）を出力します([bedtools.readthedocs.io](https://bedtools.readthedocs.io/en/latest/content/tools/intersect.html?utm_source=chatgpt.com "intersect — bedtools 2.31.0 documentation - Read the Docs")).
    
- 出力から各コンティグのリピート被覆率を算出し、長いコンティグがどれだけリピートをまたいでいるかを評価します。
    

---

## Step 4. LAI（LTR Assembly Index）の計算

真核ゲノムで大きな割合を占めるLTRレトロトランスポゾンについて、アセンブリ品質を定量評価します。

```bash
# 1) harvest（LTRharvest）でLTR候補を取得
gt ltrharvest -index assembly.fasta -seqids yes > harvest.out

# 2) LTR_retriever実行
LTR_retriever \
  -genome assembly2/assembly-results/flye/assembly.fasta \
  -inharvest harvest.out \
  -threads 8

# 出力にRaw_LAI, Corrected_LAIが含まれる
```

- LAI ≥ 10 なら高品質リピートアセンブリとされます([github.com](https://github.com/oushujun/LTR_retriever/blob/master/LAI?utm_source=chatgpt.com "LTR_retriever/LAI at master - GitHub"), [github.com](https://github.com/oushujun/LTR_retriever?utm_source=chatgpt.com "LTR_retriever is a highly accurate and sensitive program ... - GitHub")).
    

---

## Step 5. Misassembly・構造変異の可視化

### 5.1 QUASTによるミスアセンブリ検出

```bash
quast.py \
  assembly2/assembly-results/flye/assembly.fasta \
  -R /home/aki/ncbi_dataset/.../reference.fna \
  --eukaryote \
  -o quast_repeat_report
```

- `Misassemblies by region` に、リピート領域での断片化・逆向き配置・過大コピーなどがレポートされます([quast.sourceforge.net](https://quast.sourceforge.net/docs/manual.html?utm_source=chatgpt.com "QUAST 5.3.0 manual"), [pmc.ncbi.nlm.nih.gov](https://pmc.ncbi.nlm.nih.gov/articles/PMC3624806/?utm_source=chatgpt.com "QUAST: quality assessment tool for genome assemblies - PMC")).
    

### 5.2 MUMmer＋SyRIによる詳細解析

```bash
# 1) 細かいアライメント（nucmer）
nucmer --maxmatch -l 100 -c 500 \
  reference.fna \
  assembly.fasta \
  -p flye_ref

# 2) SyRIで構造変異検出
syri \
  -c flye_ref.delta \
  -r reference.fna \
  -q assembly.fasta \
  -F B
```

- SyRIは繰り返し配列とオリジナル配列を区別してSVを分類します([genomebiology.biomedcentral.com](https://genomebiology.biomedcentral.com/articles/10.1186/s13059-019-1911-0?utm_source=chatgpt.com "SyRI: finding genomic rearrangements and local sequence ..."), [pmc.ncbi.nlm.nih.gov](https://pmc.ncbi.nlm.nih.gov/articles/PMC6913012/?utm_source=chatgpt.com "SyRI: finding genomic rearrangements and local sequence ...")).
    
- 得られた `.syri.out` から、リピート領域での「転座」「逆位」「重複」などを可視化してください。
    

---

### **まとめ**

1. **RepeatMasker** で既知リピートをソフトマスク → 種類・総長を取得
    
2. 出力レポートから **リピート率** を算出
    
3. **Bedtools intersect** でコンティグ単位の被覆解析
    
4. **LTR_rretriever** で **LAI** 計算 → LTR品質評価
    
5. **QUAST**／**MUMmer+SyRI** でリピート領域の Misassembly を可視化
    

これらを順に実施することで、Flyeアセンブリにおける反復配列の「存在」「分布」「組み立て精度」「構造的異常」を網羅的に評価できます。ぜひ手順に従い、得られた結果をもとにリピート領域のポリッシングやパラメータ調整を試みてください。


### 4071372.pts-51.tardis

以下の結果からわかることと、その背景・対策を整理します。

---

## 1. 結果の評価

- **Total interspersed repeats: 43 bp (0.00 %)**  
    35.1 Mb のゲノム中にわずか 43 bp しか「既知の散在性リピート」が検出されていません。これは事実上ゼロと言ってよいレベルです。
    
- **期待される Dictyostelium discoideum のリピート量**  
    文献によると、D. discoideum ゲノムの約 **10–15 %** はトランスポゾンや単純反復配列から成ると報告されています。
    
    - 移動性エレメントなど散在性リピートがゲノムの約 10 % を占める ([pmc.ncbi.nlm.nih.gov](https://pmc.ncbi.nlm.nih.gov/articles/PMC311061/?utm_source=chatgpt.com "The Complex Repeats of Dictyostelium discoideum - PMC"))
        
    - 単純反復配列（SSR）が 11–14 % を占める ([pmc.ncbi.nlm.nih.gov](https://pmc.ncbi.nlm.nih.gov/articles/PMC1352341/?utm_source=chatgpt.com "The genome of the social amoeba Dictyostelium discoideum - PMC"), [frontiersin.org](https://www.frontiersin.org/articles/10.3389/fcell.2021.752175/full?utm_source=chatgpt.com "Dictyostelium discoideum as a Model to Assess Genome Stability ..."))
        

→ 現状の 0.00 % という検出率は、あまりに少なすぎて「ライブラリに登録されていない」と考えられます。

---

## 2. 原因

1. **種特異的リピートがデータベースに未登録**
    
    - RepeatMasker のデフォルトライブラリ（RepBase など）には、Dictyostelium 特有のリピート要素が十分に含まれていない可能性が高いです。
        
2. **リピート配列が短く複雑すぎて検出困難**
    
    - D. discoideum の SSR は非常に多様かつ断片化されやすいため、既知ライブラリへのマッチが困難です。
        

---

## 3. 対策

1. **De novo リピートライブラリの構築**
    
    - **RepeatModeler** を使い、Flye アセンブリから species-specific なリピートライブラリを自動生成します。
        
        ```bash
        BuildDatabase -name flyeDB assembly.fasta
        RepeatModeler -database flyeDB -pa 8
        ```
        
    - 生成されたライブラリ（`consensi.fa.classified`）を RepeatMasker に読み込ませる。
        
        ```bash
        RepeatMasker \
          -lib consensi.fa.classified \
          -pa 8 \
          -xsmall \
          assembly.fasta
        ```
        
2. **マスク設定の調整**
    
    - `-s`（sensitive）モードを追加し、より厳密にスコアリングする。
        
        ```bash
        RepeatMasker -species "dictyostelium discoideum" -pa 8 -xsmall -s assembly.fasta
        ```
        
3. **複数ツールの併用**
    
    - **Tandem Repeats Finder** や **LTR_retriever** で単純反復／LTR を別々に検出し、結果を統合する。
        
    - SSR については **MISA** や **SSRIT** も有効です。
        

---

## 4. まとめ

- 現状の 0.00 % 検出率は “実質ゼロ” であり、Dicty のゲノムではありえない値です。
    
- 最も有効なのは、まず **RepeatModeler** による de novo ライブラリ構築 → それを基に **RepeatMasker** を再実行する方法です。
    
- 必要に応じて感度設定や専用ツールを組み合わせ、リピートの総量が約 10–15 % 程度になるか確認してください。