染色体レベル（chromosome-level）アセンブリとは、ゲノム配列の大多数のコンティグ（contig）やスキャフォールド（scaffold）が「染色体一本分」または染色体セグメントに対応しており、ほぼ完全な染色体構造を反映している状態を指します。以下、詳しく説明します。

## 染色体レベルアセンブリとは

* **定義**
  染色体一本を構成する塩基配列が、ほぼ途切れなく一つのコンティグまたはスキャフォールドとして得られているアセンブリの段階です。ギャップ（Nでマスクされた領域）は存在しても、物理的・遺伝的マップや参照ゲノムとの比較で「この配列はこの染色体に対応する」と明確に割り当てられるレベルを指します。
* **特徴**

  * 各主要染色体に対応する大きな連続配列（大コンティグ／スキャフォールド）が存在する
  * ギャップ数やコンティグ数が染色体あたり非常に少ない
  * 染色体構造（テロメア、セントロメア領域など）の把握が可能
* **利点**

  * ゲノム構造変異（逆位、転座、重複など）の精密解析
  * 完全な遺伝子アノテーション
  * 比較ゲノム解析での高い信頼性

## 参照ゲノム dicty\_2.7 は染色体レベルか？

NCBIのAssemblyページでも「Level: Chromosome」と明記されており、*Dictyostelium discoideum* AX4 の参照ゲノム dicty\_2.7 は**染色体レベルのアセンブリ**として公開されています【([ncbi.nlm.nih.gov][1])】。

* Assembly Name：dicty\_2.7
* Level：Chromosome Genome ([ncbi.nlm.nih.gov][1])
* 6本の主要染色体（haploid 34 Mbp）を含む構造が、個別のコンティグ／スキャフォールドとして提供されている

したがって、このdicty\_2.7参照ゲノムをガイドとして用いれば、同じ染色体レベルのアセンブリを構築・評価する際に非常に有用です。

---

**まとめ**

* **染色体レベル**：染色体一本に対応する大規模連続配列を持つアセンブリレベル
* **dicty\_2.7参照**：NCBI上で染色体レベルと明示されており、染色体構造を十分に反映した高品質アセンブリ

これを基盤に、Nanopore＋Illuminaハイブリッドで得られた新規ドラフトを参照スキャフォールディングすれば、同等の染色体レベル品質を実現できます。

[1]: https://www.ncbi.nlm.nih.gov/assembly/255488?utm_source=chatgpt.com "dicty_2.7 - Genome - Assembly - NCBI"


## 要約

染色体レベルのアセンブリとは、ゲノムの各染色体が一連のギャップのない（または最小限のギャップのみの）コンティグ／スキャフォールドとして得られ、期待される全染色体が揃っている状態を指します。本研究では、新規 _D. firmibasis_ アセンブリの大きな6本のコンティグが、それぞれ _D. discoideum_ の6染色体にほぼ完全に対応するシンテニーを示し（図 4a）、テロメア反復配列や円形化したミトコンドリアゲノム、rDNAパリンドロームの完全アラインメントも確認しました（図 4b–d）。さらに、BUSCOによる95％以上の完全性、N50≫1 Mbの高い連続性、ギャップゼロまたは極めて少数のNsの状態など、複数の指標から「染色体レベル」であると結論付けています。

---

## なぜ“染色体レベル”と判断できるのか

### 1. コンティグ数と染色体数の一致

新規アセンブリでは、6本の大きなコンティグが得られ、_D. firmibasis_ の既知の染色体数（6本）と一致していることから、全染色体が一つのコンティグとして組み立てられていることを示しています ([researchgate.net](https://www.researchgate.net/post/What-are-the-different-grades-of-genome-completeness?utm_source=chatgpt.com "What are the different \"grades\" of genome completeness?"))。

### 2. 高い連続性（N50・ギャップ）

N50はコンティグ長の統計指標で、50％のゲノム長をカバーする最小のコンティグ長を示します。大規模ゲノムでN50が1 Mbを大きく超える場合、ほぼ染色体に匹敵する連続性を持つと判断できます ([en.wikipedia.org](https://en.wikipedia.org/wiki/N50%2C_L50%2C_and_related_statistics?utm_source=chatgpt.com "N50, L50, and related statistics"))。また、ギャップ（連続しない領域をNsで示す）がほぼゼロであることも「完成度」を裏付けます ([researchgate.net](https://www.researchgate.net/post/What-are-the-different-grades-of-genome-completeness?utm_source=chatgpt.com "What are the different \"grades\" of genome completeness?"))。

### 3. シンテニー（相同性ある領域の共保存）

他種リファレンス（_D. discoideum AX4_）とのシンテニー解析により、各コンティグが対応染色体に沿って大規模に整列していることを視覚的に確認しています。シンテニーは染色体レベルでのコリニアリティ（遺伝子順序の保存）を示し、染色体丸ごとの組み立てを示す強力な証拠です ([en.wikipedia.org](https://en.wikipedia.org/wiki/Synteny?utm_source=chatgpt.com "Synteny"))。

### 4. テロメア／ミトコンドリアゲノムの完全アセンブリ

テロメア反復配列（TTAGGGなど）がコンティグ両端に存在すること、ミトコンドリアゲノムが円形にギャップゼロでアセンブルされていることは、真の染色体末端から末端までの組立て（T2T, telomere-to-telomere）を示します ([nature.com](https://www.nature.com/articles/s41588-025-02137-x?utm_source=chatgpt.com "A telomere-to-telomere genome assembly coupled with multi-omic ..."))。

### 5. BUSCOによる遺伝子完全性評価

BUSCO解析は、ユニバーサルに保存されるシングルコピー遺伝子セットの回復率を測定し、95％以上の「Complete」スコアは高い遺伝子空間の完全性を示します ([pacb.com](https://www.pacb.com/blog/beyond-contiguity/?utm_source=chatgpt.com "assessing the quality of genome assemblies with the 3 Cs - PacBio"))。

---

## 自分の _Dicty_ アセンブリで同様に「染色体レベル」を示すには

1. **コンティグ数の確認**
    
    - 期待する染色体数（_D. discoideum_ では6本）とコンティグ数を比較し、一致または近いかを確認します ([researchgate.net](https://www.researchgate.net/post/What-are-the-different-grades-of-genome-completeness?utm_source=chatgpt.com "What are the different \"grades\" of genome completeness?"))。
        
2. **連続性指標の算出**
    
    - N50/L50を計算し、N50が1 Mbを大きく超えるか評価します ([en.wikipedia.org](https://en.wikipedia.org/wiki/N50%2C_L50%2C_and_related_statistics?utm_source=chatgpt.com "N50, L50, and related statistics"))。
        
    - ギャップ数（Nsの総数）を確認し、できるだけ少ないことを目指します。
        
3. **BUSCO解析**
    
    - BUSCOツールで95％以上のCompleteスコアを目標に評価します ([busco.ezlab.org](https://busco.ezlab.org/busco_userguide.html?utm_source=chatgpt.com "User guide BUSCO v5.8.2 - EZlab"))。
        
4. **参照ゲノムとのシンテニー解析**
    
    - MUMmer（`nucmer`）→ mummerplot、または SyRI で相同領域の整合状況を可視化し、染色体丸ごとのマッピングを確認します ([en.wikipedia.org](https://en.wikipedia.org/wiki/Synteny?utm_source=chatgpt.com "Synteny"))。
        
5. **テロメア配列の検出**
    
    - 各コンティグ両端にテロメア反復配列（例: TTAGGG）をスキャンし、存在を確認します。
        
6. **追加データの活用（任意）**
    
    - Hi-Cや光学マップを用いたスキャフォルディングで、染色体構造をさらに検証・向上させる方法もあります ([nature.com](https://www.nature.com/articles/s42003-022-04341-5?utm_source=chatgpt.com "A chromosome-level genome assembly reveals genomic ... - Nature"))。
        

以上の多角的な指標を組み合わせることで、自分のアセンブリが「染色体レベル」であると総合的に結論付けることができます。

新規アセンブリが「染色体レベル」であることを示すには、各コンティグが染色体の末端（テロメア）まで連続的に組み立てられているかどうかを確認することが重要です。コンティグ両端にテロメア反復配列（例: TTAGGG）が検出されると、そのコンティグが真に染色体端から端までカバーしていることを示し、ギャップや欠失がない「テロメア・ツー・テロメア」アセンブリ（T2T）の達成を裏付けます ([nature.com](https://www.nature.com/articles/s41586-020-2547-7?utm_source=chatgpt.com "Telomere-to-telomere assembly of a complete human X chromosome"))([vtechworks.lib.vt.edu](https://vtechworks.lib.vt.edu/items/12af85a5-78ac-4cc0-95cd-943c12d51345?utm_source=chatgpt.com "A Chromosomal Assembly of the New World Malaria Mosquito Ends ..."))。

## テロメアの役割と特徴

- **染色体末端の保護機能**  
    テロメアは染色体末端をDNA修復機構から保護し、非相同末端結合による染色体融合を防ぎます ([en.wikipedia.org](https://en.wikipedia.org/wiki/Telomere?utm_source=chatgpt.com "Telomere"))。
    
- **保存された反復配列**  
    多くの真核生物では「TTAGGG」のような6 bpモチーフが数千回繰り返され、G-四重らせん構造形成にも寄与します ([academic.oup.com](https://academic.oup.com/nar/article/52/20/12456/7798794?utm_source=chatgpt.com "Haplotype-resolved de novo assembly revealed unique ..."))。
    

## 各コンティグ端でテロメアを検出する理由

1. **染色体完全性の確認**  
    コンティグ両端にテロメア反復があると、その配列が染色体末端まで途切れず組み立てられた証拠となり、全長染色体アセンブリ（T2T）の達成度を示します ([nature.com](https://www.nature.com/articles/s41586-020-2547-7?utm_source=chatgpt.com "Telomere-to-telomere assembly of a complete human X chromosome"))。
    
2. **アセンブリ品質のQC指標**  
    最新のハイレゾリューションゲノムプロジェクトでは、コンティグ末端へのテロメア検出が品質管理のゴールドスタンダードとされています ([pmc.ncbi.nlm.nih.gov](https://pmc.ncbi.nlm.nih.gov/articles/PMC10462168/?utm_source=chatgpt.com "Genome assembly in the telomere-to-telomere era - PMC"))([vtechworks.lib.vt.edu](https://vtechworks.lib.vt.edu/items/12af85a5-78ac-4cc0-95cd-943c12d51345?utm_source=chatgpt.com "A Chromosomal Assembly of the New World Malaria Mosquito Ends ..."))。
    
3. **欠失や断片化の検出**  
    テロメアが検出されない末端は、ギャップや断片化を示唆し、追加データ（長尺リードやHi-C）による補間や再スキャフォルディングの必要性を教えてくれます ([academic.oup.com](https://academic.oup.com/dnaresearch/article/32/1/dsae036/7927918?utm_source=chatgpt.com "Near-complete telomere-to-telomere de novo genome assembly in ..."))([biostars.org](https://www.biostars.org/p/424539/?utm_source=chatgpt.com "How to blastn a very short query sequence? Identifying telomeres in ..."))。
    
4. **スキャフォルディング／オリエンテーションの検証**  
    テロメア位置を目印にすることで、複数コンティグを結合したスキャフォルドの向きや順序が正しいかどうかを確認できます ([pacb.com](https://www.pacb.com/blog/genomes-vs-gennnnes-difference-contigs-scaffolds-genome-assemblies/?utm_source=chatgpt.com "the difference between contigs and scaffolds in genome assemblies"))([en.wikipedia.org](https://en.wikipedia.org/wiki/Contig?utm_source=chatgpt.com "Contig"))。
    
5. **解析手法の標準化**  
    Telomere motif（例: TTAGGG）を用いたBLAST検索や専用ツールで定量的に反復数を測定し、アセンブリの「端点」を客観的に評価するプロトコルが確立されています ([biostars.org](https://www.biostars.org/p/424539/?utm_source=chatgpt.com "How to blastn a very short query sequence? Identifying telomeres in ..."))([tandfonline.com](https://www.tandfonline.com/doi/full/10.2144/btn-2018-0057?utm_source=chatgpt.com "Full article: A Bioinformatics Approach to Identify Telomere Sequences"))。


