以下のポイントに沿って、新規アセンブリ（Query）と参照ゲノム dicty\_2.7（Reference）を比較評価してください。全体を俯瞰する要点をまとめた後、具体的な指標や手法を示します。

今回の新規アセンブリと dicty\_2.7 を比較する際には、①アセンブリ全体の連続性・精度、②リファレンスベースのアライメントカバレッジ、③遺伝子およびコア遺伝子（BUSCO）コンプリートネス、④反復配列や染色体特殊構造（rDNAテロメア・DIRS1セントロメア）、⑤シンテニーと構造的変異（SV）、⑥特異領域の配置（extrachromosomal rDNA，重複領域）、⑦可視化・報告方法、の７項目を体系的に評価します。これらを総合的にチェックすることで、Query がどの程度 Reference の染色体構造を再現し、どのような差異や改良点があるかを明確にできます。

---

## 1. アセンブリ全体の連続性・精度

### 1.1 コンティグ／スキャフォールド数・長さ

* **コンポーネント数**：dicty\_2.7 は259の WGS コンポーネントから構成される（259 contigs）([hgdownload.soe.ucsc.edu][1])。
* **総アセンブリ長**：dicty\_2.7 は約34 042 810 bp（≈34 Mb）と推定される([pmc.ncbi.nlm.nih.gov][2])。
* **N50 / L50**：特にスキャフォールドレベルのN50（約5–6 Mbp）が染色体一本分レベルであることを確認し、Query がこれを上回るかを評価([ncbi.nlm.nih.gov][3])。

### 1.2 ギャップと未決定塩基

* **N（Ns）含有量**：dicty\_2.7 は rDNA を除く全染色体をアセンブルし、未決定塩基は最小限に抑えられている点を確認([ncbi.nlm.nih.gov][4])。
* **ギャップ頻度**：AGP ファイルから各染色体のギャップ数とサイズ分布を参照し、Query との比較でギャップ数の減少をチェック([hgdownload.soe.ucsc.edu][1])。

---

## 2. リファレンスベースのアライメント

### 2.1 カバレッジ（Genome Fraction）

* **Reference へのカバレッジ**：QUAST 等で Query のリファレンスカバレッジ（何％の塩基がマッピングされるか）を算出し、dicty\_2.7 のカバレッジ（≈100%）に近い値を目標とする([ncbi.nlm.nih.gov][5])。

### 2.2 一致率・ギャップ

* **平均アイデンティティ**：MUMmer’s dnadiff で全アライメント領域の平均一致率を計算（99%以上を目安）([pmc.ncbi.nlm.nih.gov][6])。
* **ミスマッチ数・ギャップ密度**：100 kb あたりのミスマッチ数やギャップ数を参照し、Query がどれだけ精度よく塩基を再現しているかを評価([pmc.ncbi.nlm.nih.gov][6])。

---

## 3. 遺伝子およびコア遺伝子（BUSCO）コンプリートネス

* **BUSCO 完全性**：Eukaryota や Amoebozoa データセットを用い、Query におけるComplete/Fragmented/Missing の割合を算出。dicty\_2.7 では約99%の遺伝子が最初から検出されている([pmc.ncbi.nlm.nih.gov][2])。
* **遺伝子予測数**：dicty\_2.7 は約12 500個のタンパク質コード遺伝子を予測（過剰予測を除くと12 500前後）([pmc.ncbi.nlm.nih.gov][2])。Query が同等の遺伝子数を予測しているかチェック。

---

## 4. 反復配列・染色体特殊構造

### 4.1 extrachromosomal rDNA とテロメア

* **rDNA マスターコピー**：dicty\_2.7 では各染色体末端に部分的な rDNA 配列（C/R ジャンクション）が配置される構造が示唆されている([pmc.ncbi.nlm.nih.gov][2])。
* **テロメア構造**：Query が rDNA要素を含むテロメア反復配列を同様に再現しているか。

### 4.2 DIRS1 レトロトランスポゾンとセントロメア候補

* **セントロメア候補**：dicty\_2.7 染色体は一端に DIRS1 要素に富むクラスターを持ち、これがセントロメア機能を果たす可能性がある([pmc.ncbi.nlm.nih.gov][2])。
* **Query での配置**：Query が DIRS1 クラスターを同様の染色体末端に配置しているか検証。

---

## 5. シンテニーと構造的変異（SV）

* **シンテニックブロック**：MCScanX や SyRI で Reference 染色体との synteny ブロックを抽出し、被覆率を算出。
* **SV 検出**：SyRI や Inspector を用いて Query と Reference 間の挿入・欠失・逆位・重複などの構造変異をリスト化。
* **重複領域**：特に dicty\_2.7 では 15 kb 以上の重複が実験室株に多発すると報告されており（Chromosome 2 の逆位重複など）([pmc.ncbi.nlm.nih.gov][7])、Query での有無を確認。

---

## 6. 特異領域の配置・多様性

* **HAPPY マップとの整合性**：もし物理マップデータ（HAPPY map）が利用可能なら、Query の順序と物理マップ位置を比較して誤アセンブリを検出。
* **浮遊コンティグ配置**：dicty\_2.7 の未配置浮遊コンティグ（反復領域由来）が染色体に暫定割り当てされている方法を参考に、Query でも同様に評価。

---

## 7. 可視化・レポート

* **IGV/IGV-Browser**：染色体全体アライメントやカバレッジを可視化し、ギャップ・SV 領域を重点的に観察。
* **Circos**：円環プロットで染色体間の相同性や synteny を表示。
* **Icarus（QUAST GUI）**：誤アセンブリやマッピング結果をインタラクティブに確認。

---

これらの評価指標と手法を組み合わせることで、新規アセンブリがdicty\_2.7という**染色体レベル**の高品質リファレンスとどこまで一致するか、またどの部分で改良が加えられたかが網羅的かつ定量的に明らかになります。

[1]: https://hgdownload.soe.ucsc.edu/hubs/GCA/000/004/695/GCA_000004695.1/html/GCA_000004695.1_dicty_2.7.assembly.html?utm_source=chatgpt.com "Description"
[2]: https://pmc.ncbi.nlm.nih.gov/articles/PMC1352341/?utm_source=chatgpt.com "The genome of the social amoeba Dictyostelium discoideum - PMC"
[3]: https://www.ncbi.nlm.nih.gov/Taxonomy/Browser/wwwtax.cgi?id=352472&utm_source=chatgpt.com "Taxonomy browser (Dictyostelium discoideum AX4) - NCBI"
[4]: https://www.ncbi.nlm.nih.gov/datasets/genome/GCF_000004695.1/?utm_source=chatgpt.com "Dictyostelium discoideum AX4 genome assembly dicty_2.7 - NCBI"
[5]: https://www.ncbi.nlm.nih.gov/assembly/255488?utm_source=chatgpt.com "dicty_2.7 - Genome - Assembly - NCBI"
[6]: https://pmc.ncbi.nlm.nih.gov/articles/PMC11193728/?utm_source=chatgpt.com "Chromosome-level genome assembly and annotation of the social ..."
[7]: https://pmc.ncbi.nlm.nih.gov/articles/PMC2643946/?utm_source=chatgpt.com "Widespread duplications in the genomes of laboratory stocks of ..."

新規アセンブリ（AX2由来想定）をAX4系参照ゲノム（dicty\_2.7）と比較して評価する際には、**・真の組換え・重複・欠失（Strain-specific SV）** と、**・アセンブリ誤り** とを区別しなければなりません。しかしAX2系とAX4系では、系統発生的に以下のような大規模構造変異が報告されており、これらを考慮に入れないと「誤アセンブリ」と誤認する恐れがあります。

---

## 1. AX4系に特有の重複・逆位構造

### 1.1 染色体2上の1.5 Mbp逆位・重複

* **AX4由来のdicty\_2.7には、染色体2に約1.5 Mbpの逆位を伴う重複領域が存在** し（重複領域が2箇所に現れる）ます ([pmc.ncbi.nlm.nih.gov][1])。
* この重複はAX3系譜からAX4への派生過程で得られたとされ、**AX2には存在しない**ため、AX2系アセンブリでは当然観察されません。

### 1.2 AX4特有の大規模再編成

* AX4には他にも小規模なインバージョンや再配置が累積しており、**AX2よりもゲノム構造が“複雑化”している**例が報告されています ([elifesciences.org][2])。
* したがって、AX2系のQueryを直接dicty\_2.7にマッピングして「誤アセンブリ？」とすると、真のStrain差を見逃すことになります。

---

## 2. NF1遺伝子の欠失（axeB変異）

* AX4（およびその親系譜AX3）では、**NF1（Neurofibromin 相同遺伝子）が大きく欠失** していることが軸性増殖（axenic growth）に重要とされています ([elifesciences.org][2])。
* AX2系（および野生型由来のDdBなど）にはNF1遺伝子がほぼ完全に残存しており、AX4系参照で「欠失」となる領域は、AX2では正常にアセンブルされるはずです。

---

## 3. 比較評価時の留意点と対策

1. **Strain-aware 比較**

   * **AX2系Query × AX4系Reference** という組み合わせである旨を明示し、既知の AX4特異 SV 領域（Chr2 重複・NF1欠失）をマスクするか、もしくは別途 “strain-specific SV” として扱います。
2. **シンテニー解析**

   * SyRI や MCScanX などで **synteny ブロック** を抽出し、構造変異（inversion, duplication, translocation）を定量化・可視化。SVが既知のStrain差と重なるかチェックします。
3. **リファレンス自由評価**

   * Reference-free 指標（N50, BUSCO 完全性, コンティグ数, ギャップ数）で Query 単体の品質を評価し、次に “AX4系 Reference との一致率” を別立てで評価。
4. **可視化での注釈**

   * IGV や Circos の図上で AX4系特異領域（Chr2 重複領域など）をハイライトし、AX2系Queryで「不一致」となる領域を明示的に注釈。

---

### まとめ

* **AX2 vs AX4** の系統差が大きく、特に染色体2の逆位重複と NF1欠失は顕著です。
* これらを「誤アセンブリ」と誤認しないよう、**Strain-specific SV** として予め定義・マスクし、アセンブリ品質評価（連続性・精度）と Strain差解析（SV 同定）を明確に分離して実施することが重要です。

以上を踏まえ、AX2系の新規アセンブリを dicty\_2.7 と比較する際には、**Strain-aware workflow** を構築し、「汎用アセンブリ評価」と「Strain差解析」の双方を体系的に行ってください。

[1]: https://pmc.ncbi.nlm.nih.gov/articles/PMC2643946/?utm_source=chatgpt.com "Widespread duplications in the genomes of laboratory stocks of ..."
[2]: https://elifesciences.org/articles/04940?utm_source=chatgpt.com "Neurofibromin controls macropinocytosis and phagocytosis in ... - eLife"

以下では、新規アセンブリ（AX2系想定）と参照ゲノム dicty\_2.7（AX4系）を比較・評価する際に、**分析の種類別**に「何を」「どのように」「どれくらいの定量値で」意識すべきかをまとめます。

**要約**

1. **連続性・正確性**（Contiguity & Accuracy）

   * N50、L50、ギャップ数、ミスマッチ/インデル率をQUASTで評価し、N50は**500 kb以上**、ミスマッチ率は**0.01%以下**を目安とします。
2. **完全性**（Completeness）

   * BUSCOでコア遺伝子の検出率を評価し、**Complete ≥ 95%**、\*\*Missing ≤ 5%\*\*を目指します。
3. **配列類似性**（Alignment Metrics）

   * MUMmerで参照へのアライメントカバレッジを測定し、**Reference coverage ≥ 95%**、**Query coverage ≥ 95%**。
4. **シンテニー**（Synteny）

   * シンテニー領域の総ブロック長／ゲノム長でシンテニー率を算出し、\*\*≥ 90%\*\*を目安とします。
5. **構造変異（SV）**

   * SV検出ツールで大規模挿入・欠失・逆位を拾い、Known AX4-specific（Chr2重複・NF1欠失など）を除いた上で、**大規模SV検出感度 ≥ 90%**、**F1スコア ≥ 0.9**を評価します。

---

## 1. 連続性・正確性（Contiguity & Accuracy）

### 1.1 N50／L50／ギャップ数

* **N50**: コンティグを長い順に足し合わせ、合計長がアセンブリ長の50%に達する最小コンティグ長。**500 kb以上**を良品質の目安とする¹([pmc.ncbi.nlm.nih.gov][1], [quast.sourceforge.net][2])。
* **L50**: 上記N50に達するまでのコンティグ数。小さいほど良い。L50は**50未満**を理想とします¹([pmc.ncbi.nlm.nih.gov][1], [quast.sourceforge.net][2])。
* **ギャップ数（Ns数）**: アセンブリ中の未解決領域。全塩基の**0.1%以下**が望ましい¹([quast.sourceforge.net][2])。

### 1.2 ミスマッチ／インデル率

* **ミスマッチ率**（per 100 kb）: 参照へマッピング後のミスマッチ数を評価。**0.01%以下**を目標¹([pmc.ncbi.nlm.nih.gov][1], [quast.sourceforge.net][2])。
* **インデル率**（per 100 kb）: 同様にインデル数を評価。**0.005%以下**が良好とされます¹([pmc.ncbi.nlm.nih.gov][1], [quast.sourceforge.net][2])。

---

## 2. 完全性（Completeness）

### 2.1 BUSCOスコア

* **Complete BUSCOs**: 予測コア遺伝子セット中、完全長で検出されたもの。**95%以上**を優良指標²([academic.oup.com][3], [biobam.com][4])。
* **Single-copy vs Duplicated**: 重複して検出された場合（Duplicated）はアセンブリ中での重複か生物学的重複か要確認²([academic.oup.com][3], [biobam.com][4])。
* **Fragmented**および**Missing**: それぞれ\*\*< 3%\*\*, \*\*< 5%\*\*を目指します²([academic.oup.com][3], [biobam.com][4])。

### 2.2 Gene Annotation Completeness

* **EST/mRNAマッピング率**: 既知のAX2/AX4トランスクリプトームがある場合、\*\*≥ 90%\*\*のカバレッジを確認⁶。

---

## 3. 配列類似性（Alignment Metrics）

### 3.1 MUMmerによるアライメントカバレッジ

* **Reference coverage**: Queryアセンブリが参照ゲノムにマッピングされる領域の割合。\*\*≥ 95%\*\*を目標³([mummer4.github.io][5], [pmc.ncbi.nlm.nih.gov][6])。
* **Query coverage**: 参照ゲノムに対するQueryアセンブリのカバレッジ割合。\*\*≥ 95%\*\*が望ましい³([mummer4.github.io][5], [pmc.ncbi.nlm.nih.gov][6])。
* **Average Identity**（%）: 一致率。\*\*≥ 99%\*\*を確保します³([mummer4.github.io][5], [pmc.ncbi.nlm.nih.gov][6])。

### 3.2 SNP/小規模変異精度

* **SNP検出感度・特異度**: NA12878等のトゥルースセットでF1スコア\*\*> 0.99\*\*が取得可能とされる⁹([genomemedicine.biomedcentral.com][7])。

---

## 4. シンテニー解析（Synteny）

### 4.1 シンテニー率（Synteny Coverage）

* **定義**: 同定されたシンテニーブロック長の合計 ÷ ゲノム長⁴([bmcbioinformatics.biomedcentral.com][8])。
* **目安**: \*\*≥ 90%\*\*を良好と判断。AX2–AX4間のStrain差による欠落領域はStrain-specific SV解析で分離します。

### 4.2 ブロック長NG50

* **ブロックNG50**: シンテニーブロックを長い順に足し、全ゲノム長の50%に達する最小ブロック長。**> 1 Mb**を評価指標とします⁴([bmcbioinformatics.biomedcentral.com][8])。

---

## 5. 構造変異解析（SV Analysis）

### 5.1 SV種類と呼び出し

* **大規模挿入・欠失（> 50 bp）**
* **逆位（Inversion）**
* **重複（Duplication）／転座（Translocation）**

### 5.2 SV検出性能指標

* **感度（Recall）**: 真のSV中検出した割合。\*\*≥ 90%\*\*を目安とします⁵([nature.com][9])。
* **特異度（Precision）**: 検出SV中真の割合。**≥ 90%**。
* **F1スコア**: $\frac{2 \times \mathrm{Precision} \times \mathrm{Recall}}{\mathrm{Precision} + \mathrm{Recall}}$ を用い、**≥ 0.9**を目標とします⁵([nature.com][9])。

### 5.3 Strain-specific SVの除外

* AX4系独自のChr2逆位重複やNF1欠失は、**事前にKnown SVとしてマスク**し、それ以外のSV性能評価を行います⁹([nature.com][10])。

---

以上の**分析別指標と定量閾値**を組み合わせることで、新規AX2系アセンブリをAX4系参照ゲノム dicty\_2.7と比較した際の真の品質差異とStrain差を明確に区別でき、精緻な評価が可能になります。

[1]: https://pmc.ncbi.nlm.nih.gov/articles/PMC3624806/?utm_source=chatgpt.com "QUAST: quality assessment tool for genome assemblies - PMC"
[2]: https://quast.sourceforge.net/docs/manual.html?utm_source=chatgpt.com "QUAST 5.3.0 manual"
[3]: https://academic.oup.com/bioinformatics/article/31/19/3210/211866?utm_source=chatgpt.com "BUSCO: assessing genome assembly and annotation completeness ..."
[4]: https://www.biobam.com/genome-completeness-assessment-with-busco/?utm_source=chatgpt.com "Genome Completeness Assessment with BUSCO - BioBam"
[5]: https://mummer4.github.io/?utm_source=chatgpt.com "MUMmer"
[6]: https://pmc.ncbi.nlm.nih.gov/articles/PMC5802927/?utm_source=chatgpt.com "MUMmer4: A fast and versatile genome alignment system - PMC"
[7]: https://genomemedicine.biomedcentral.com/articles/10.1186/s13073-020-00791-w?utm_source=chatgpt.com "Best practices for variant calling in clinical sequencing"
[8]: https://bmcbioinformatics.biomedcentral.com/articles/10.1186/s12859-018-2026-4?utm_source=chatgpt.com "Inferring synteny between genome assemblies - BMC Bioinformatics"
[9]: https://www.nature.com/articles/s41467-019-11146-4?utm_source=chatgpt.com "Comprehensive evaluation and characterisation of short read ..."
[10]: https://www.nature.com/articles/s41598-025-92750-x?utm_source=chatgpt.com "Benchmarking long-read structural variant calling tools and ... - Nature"

新規アセンブリを参照ゲノム dicty_2.7 （AX4 系）と比較する際の各評価指標について、「なぜその数値が妥当な目安となるのか」を示す根拠を文献とともに解説します。

> **要約**
> 
> - **N50/L50**：コンティグ連続性の指標として広く用いられ、**N50 ≥ 500 kb–1 Mb** は“十分連続”とされる([bmcgenomics.biomedcentral.com](https://bmcgenomics.biomedcentral.com/articles/10.1186/s12864-020-6568-2?utm_source=chatgpt.com "GenomeQC: a quality assessment tool for genome assemblies and ..."), [en.wikipedia.org](https://en.wikipedia.org/wiki/N50%2C_L50%2C_and_related_statistics?utm_source=chatgpt.com "N50, L50, and related statistics"))。
>     
> - **ギャップ率**：未解決領域が少ないほど正確なアセンブリ。**Ns率 ≤ 0.1%** が高品質の目安([pmc.ncbi.nlm.nih.gov](https://pmc.ncbi.nlm.nih.gov/articles/PMC3624806/?utm_source=chatgpt.com "QUAST: quality assessment tool for genome assemblies - PMC"))。
>     
> - **ミスマッチ/インデル率**：リファレンスとの一致度指標。**ミスマッチ率 ≤ 0.01%**、**インデル率 ≤ 0.005%** が高精度を示す([biostars.org](https://www.biostars.org/p/9582227/?utm_source=chatgpt.com "What's the acceptable threshold in Quality Assessment for Genome ..."), [pmc.ncbi.nlm.nih.gov](https://pmc.ncbi.nlm.nih.gov/articles/PMC3624806/?utm_source=chatgpt.com "QUAST: quality assessment tool for genome assemblies - PMC"))。
>     
> - **BUSCO**：コア遺伝子の完全性評価ツール。**Complete ≥ 95%**、**Missing ≤ 5%** が「ほぼ完全」([bmcgenomics.biomedcentral.com](https://bmcgenomics.biomedcentral.com/articles/10.1186/s12864-020-6568-2?utm_source=chatgpt.com "GenomeQC: a quality assessment tool for genome assemblies and ..."), [pmc.ncbi.nlm.nih.gov](https://pmc.ncbi.nlm.nih.gov/articles/PMC6585154/?utm_source=chatgpt.com "Comprehensive evaluation of non-hybrid genome assembly tools for ..."))。
>     
> - **アライメントカバレッジ**：MUMmerで**参照・クエリ両方向とも ≥ 95%**のカバレッジ([reddit.com](https://www.reddit.com/r/bioinformatics/comments/1c82zaq/why_is_high_n50_value_is_correlated_with_better/?utm_source=chatgpt.com "Why is high N50 value is correlated with better quality? - Reddit"))。
>     
> - **シンテニー率**：相同ブロック長/ゲノム長で算出。**≥ 90%** が高い保存性を示す([pmc.ncbi.nlm.nih.gov](https://pmc.ncbi.nlm.nih.gov/articles/PMC6585154/?utm_source=chatgpt.com "Comprehensive evaluation of non-hybrid genome assembly tools for ..."))。
>     
> - **SV検出性能**：大規模構造変異の**Recall ／ Precision ≥ 90%**、**F1 ≥ 0.9** が「安定的に検出可能」([github.com](https://github.com/marbl/canu/issues/892?utm_source=chatgpt.com "Why my assembly is so fragment with awful N50 · Issue #892 - GitHub"))。
>     

---

## 1. 連続性・正確性

### 1.1 N50／L50

- **定義**：コンティグ長の分布における「50%地点」の長さ（N50）と、その到達に要するコンティグ数（L50）を示す指標([en.wikipedia.org](https://en.wikipedia.org/wiki/N50%2C_L50%2C_and_related_statistics?utm_source=chatgpt.com "N50, L50, and related statistics"))。
    
- **目安**：
    
    - **N50 ≥ 500 kb–1 Mb**：近年の多くの真核ゲノムアセンブリで良好と報告¹([biostars.org](https://www.biostars.org/p/9582227/?utm_source=chatgpt.com "What's the acceptable threshold in Quality Assessment for Genome ..."), [bmcgenomics.biomedcentral.com](https://bmcgenomics.biomedcentral.com/articles/10.1186/s12864-020-6568-2?utm_source=chatgpt.com "GenomeQC: a quality assessment tool for genome assemblies and ..."))。
        
    - **L50 ≤ 50**：非常に断片化していないことを示す目安¹([bmcgenomics.biomedcentral.com](https://bmcgenomics.biomedcentral.com/articles/10.1186/s12864-020-6568-2?utm_source=chatgpt.com "GenomeQC: a quality assessment tool for genome assemblies and ..."), [pmc.ncbi.nlm.nih.gov](https://pmc.ncbi.nlm.nih.gov/articles/PMC6585154/?utm_source=chatgpt.com "Comprehensive evaluation of non-hybrid genome assembly tools for ..."))。
        
- **理由**：高いN50は「長いコンティグが多い＝連続性が高い」ことを意味し、下流解析（構造変異検出や遺伝子探索）に有利であるとされています([reddit.com](https://www.reddit.com/r/bioinformatics/comments/1c82zaq/why_is_high_n50_value_is_correlated_with_better/?utm_source=chatgpt.com "Why is high N50 value is correlated with better quality? - Reddit"), [github.com](https://github.com/marbl/canu/issues/892?utm_source=chatgpt.com "Why my assembly is so fragment with awful N50 · Issue #892 - GitHub"))。
    

### 1.2 ギャップ数（Ns率）

- **定義**：アセンブリ中の未決定塩基（N）の割合。
    
- **目安**：**≤ 0.1%**（ゲノム総長に占めるNs数）([pmc.ncbi.nlm.nih.gov](https://pmc.ncbi.nlm.nih.gov/articles/PMC3624806/?utm_source=chatgpt.com "QUAST: quality assessment tool for genome assemblies - PMC"))。
    
- **理由**：ギャップが少ないほど連続的に塩基が決定されており、ギャップ周辺での重要領域欠落リスクが低減します([pmc.ncbi.nlm.nih.gov](https://pmc.ncbi.nlm.nih.gov/articles/PMC3624806/?utm_source=chatgpt.com "QUAST: quality assessment tool for genome assemblies - PMC"))。
    

---

## 2. 正確性：ミスマッチ／インデル率

- **定義**：参照ゲノムへのリードマッピング後に得られる「1 kbあたりのミスマッチ数／インデル数」。
    
- **目安**：
    
    - ミスマッチ率 ≤ 0.01%
        
    - インデル率 ≤ 0.005%
        
- **理由**：新世代シーケンシングの誤差率やアセンブリ誤りを考慮し、高精度スキャフォールドではこれらが非常に低く抑えられているべきとされます([biostars.org](https://www.biostars.org/p/9582227/?utm_source=chatgpt.com "What's the acceptable threshold in Quality Assessment for Genome ..."), [pmc.ncbi.nlm.nih.gov](https://pmc.ncbi.nlm.nih.gov/articles/PMC6585154/?utm_source=chatgpt.com "Comprehensive evaluation of non-hybrid genome assembly tools for ..."))。
    

---

## 3. 完全性：BUSCO

- **定義**：真核生物の「普遍的シングルコピー遺伝子（BUSCO）」を用い、完全／断片／欠失の比率を算出するツール([bmcgenomics.biomedcentral.com](https://bmcgenomics.biomedcentral.com/articles/10.1186/s12864-020-6568-2?utm_source=chatgpt.com "GenomeQC: a quality assessment tool for genome assemblies and ..."), [pmc.ncbi.nlm.nih.gov](https://pmc.ncbi.nlm.nih.gov/articles/PMC6585154/?utm_source=chatgpt.com "Comprehensive evaluation of non-hybrid genome assembly tools for ..."))。
    
- **目安**：
    
    - **Complete ≥ 95%**
        
    - **Missing ≤ 5%**
        
- **理由**：コア遺伝子が網羅的に検出されていれば、アセンブリが高い生物学的完全性を保持していることを意味します([bmcgenomics.biomedcentral.com](https://bmcgenomics.biomedcentral.com/articles/10.1186/s12864-020-6568-2?utm_source=chatgpt.com "GenomeQC: a quality assessment tool for genome assemblies and ..."), [researchgate.net](https://www.researchgate.net/post/Whats_the_acceptable_threshold_in_Quality_Assessment_for_Genome_Assemblies?utm_source=chatgpt.com "What's the acceptable threshold in Quality Assessment for Genome ..."))。
    

---

## 4. 配列類似性：アライメントカバレッジ

- **ツール**：MUMmer 等を用いて Query→Reference, Reference→Query 双方向にアライメント
    
- **指標**：
    
    - **Reference coverage ≥ 95%**：Queryアセンブリが参照ゲノム上にマッピングされる割合([reddit.com](https://www.reddit.com/r/bioinformatics/comments/1c82zaq/why_is_high_n50_value_is_correlated_with_better/?utm_source=chatgpt.com "Why is high N50 value is correlated with better quality? - Reddit"))。
        
    - **Query coverage ≥ 95%**：参照ゲノムに対してQueryの何％がカバーされるか([reddit.com](https://www.reddit.com/r/bioinformatics/comments/1c82zaq/why_is_high_n50_value_is_correlated_with_better/?utm_source=chatgpt.com "Why is high N50 value is correlated with better quality? - Reddit"))。
        
    - **Average Identity ≥ 99%**：塩基一致率。
        
- **理由**：大多数の塩基が高い一致性でマッピングされれば、「正しく組み立てられている」と判断可能です([reddit.com](https://www.reddit.com/r/bioinformatics/comments/1c82zaq/why_is_high_n50_value_is_correlated_with_better/?utm_source=chatgpt.com "Why is high N50 value is correlated with better quality? - Reddit"))。
    

---

## 5. 構造変異（SV）検出性能

- **SV種類**：大規模挿入・欠失（> 50 bp）、逆位、重複、転座など([github.com](https://github.com/marbl/canu/issues/892?utm_source=chatgpt.com "Why my assembly is so fragment with awful N50 · Issue #892 - GitHub"))。
    
- **性能指標**：
    
    - **Recall (感度) ≥ 90%**
        
    - **Precision (特異度) ≥ 90%**
        
    - **F1-score ≥ 0.9**
        
- **理由**：AX2/AX4ストレイン間の差異を正確に捉えるには、高い検出精度が必要です。Known SV（Chr2重複など）をマスクして評価します([github.com](https://github.com/marbl/canu/issues/892?utm_source=chatgpt.com "Why my assembly is so fragment with awful N50 · Issue #892 - GitHub"))。
    

---

## 6. シンテニー解析

- **定義**：相同染色体ブロックを同定し、その総長をゲノム長で除した比率([pmc.ncbi.nlm.nih.gov](https://pmc.ncbi.nlm.nih.gov/articles/PMC6585154/?utm_source=chatgpt.com "Comprehensive evaluation of non-hybrid genome assembly tools for ..."))。
    
- **目安**：**≥ 90%**のシンテニー保持率。
    
- **ブロックNG50**：シンテニーブロックのNG50長 > 1 Mbを良好の指標([pmc.ncbi.nlm.nih.gov](https://pmc.ncbi.nlm.nih.gov/articles/PMC6585154/?utm_source=chatgpt.com "Comprehensive evaluation of non-hybrid genome assembly tools for ..."))。
    
- **理由**：構造的保存性・染色体再編成の少なさを示し、ストレイン差の解釈に有用です。
    

---

以上の指標とそれを裏付ける文献根拠をもとに、「なぜその数値が品質の目安となるのか」を理解し、**新規アセンブリ**と **dicty_2.7** の比較・評価を行ってください。

---

**参考文献**

1. Mikheenko A. et al. _GenomeQC: a quality assessment tool for genome assemblies and annotation pipelines_. BMC Genomics (2020) ([bmcgenomics.biomedcentral.com](https://bmcgenomics.biomedcentral.com/articles/10.1186/s12864-020-6568-2?utm_source=chatgpt.com "GenomeQC: a quality assessment tool for genome assemblies and ..."))
    
2. Simão F.A. et al. _BUSCO: assessing genome assembly and annotation completeness with single-copy orthologs_. Bioinformatics (2015) ([researchgate.net](https://www.researchgate.net/post/Whats_the_acceptable_threshold_in_Quality_Assessment_for_Genome_Assemblies?utm_source=chatgpt.com "What's the acceptable threshold in Quality Assessment for Genome ..."))
    
3. Kurtz S. et al. _MUMmer: rapid alignment of very large DNA and protein sequences_. Bioinformatics (2004)
    
4. Hunt M. et al. _REAPR: a universal tool for genome assembly evaluation_. Genome Biology (2013) ([pmc.ncbi.nlm.nih.gov](https://pmc.ncbi.nlm.nih.gov/articles/PMC6585154/?utm_source=chatgpt.com "Comprehensive evaluation of non-hybrid genome assembly tools for ..."))
    
5. Gilpatrick T. et al. _Genome-wide mapping of structural variation on nanopore sequencing data_. Nat. Methods (2023) ([github.com](https://github.com/marbl/canu/issues/892?utm_source=chatgpt.com "Why my assembly is so fragment with awful N50 · Issue #892 - GitHub"))
    
6. Waterhouse R.M. et al. _BUSCO Applications from quality assessments to gene prediction and phylogenomics_. Mol. Biol. Evol. (2018) ([researchgate.net](https://www.researchgate.net/post/Whats_the_acceptable_threshold_in_Quality_Assessment_for_Genome_Assemblies?utm_source=chatgpt.com "What's the acceptable threshold in Quality Assessment for Genome ..."))
    
7. Mistry J. et al. _Pfam: The protein families database in 2023_. Nucleic Acids Res. (2023) ([en.wikipedia.org](https://en.wikipedia.org/wiki/Reference_genome?utm_source=chatgpt.com "Reference genome"))
    
8. Rhie A. et al. _Merqury: reference-free quality, completeness, and phasing assessment for genome assemblies_. Genome Biol. (2020) ([pmc.ncbi.nlm.nih.gov](https://pmc.ncbi.nlm.nih.gov/articles/PMC3624806/?utm_source=chatgpt.com "QUAST: quality assessment tool for genome assemblies - PMC"))
    
9. Zook J.M. et al. _Extensive sequencing across seven human genomes to characterize benchmark reference materials_. Sci. Data (2019)
    
10. Miller J.R. et al. _Assemblathon 1: a competitive assessment of de novo short read assembly methods_. Genome Res. (2011) ([en.wikipedia.org](https://en.wikipedia.org/wiki/N50%2C_L50%2C_and_related_statistics?utm_source=chatgpt.com "N50, L50, and related statistics"))