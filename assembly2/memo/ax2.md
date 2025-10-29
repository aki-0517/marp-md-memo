# Dictyostelium discoideum AX2株に関する主要な学術論文リスト

お客様が実施された *Dictyostelium discoideum* AX2株のゲノムアセンブリの正確性、特に**反復領域**の検証に役立つと考えられる主要な学術論文をリストアップしました。

*Dictyostelium discoideum*のゲノム研究は、主にAX4株の配列決定（リファレンスゲノム）を中心に進められましたが、AX2株は実験室株として広く使用されており、そのゲノム構造、特に反復配列やコピー数多型（CNV）に関する研究が重要です。

## 1. ゲノムアセンブリとリファレンスゲノムに関する基礎論文

これらの論文は、*D. discoideum*のゲノム構造、反復配列の特性、およびAX4株とAX2株の間の重要な違いを理解するための基礎となります。

| タイトル | 著者 | 発表年 | 関連性 |
| :--- | :--- | :--- | :--- |
| **The genome of the social amoeba *Dictyostelium discoideum*** | Eichinger et al. | 2005 | *D. discoideum*の初期リファレンスゲノム（AX4株）の発表論文。反復配列（DIRS-1など）のクラスター化や、AX4株特有の大きな重複（AX2にはない）について言及されています。|
| **Widespread duplications in the genomes of laboratory stocks of *Dictyostelium discoideum*** | Bloomfield et al. | 2008 | AX2株を含む複数の実験室株のゲノムを比較し、AX2株にも大きな重複（duplication）が存在すること、およびその重複が**DIRS-1**などの反復配列に隣接していることを報告しています。お客様のAX2アセンブリの反復領域の検証に特に重要です。|
| **The Complex Repeats of *Dictyostelium discoideum*** | Glöckner et al. | 2001 | *D. discoideum*の**反復配列**（DIRS-1, TRE3-A,B, TRE5-A, skipperなど）の性質と量を詳細に解析した初期の論文。反復領域の特性を理解する上で不可欠です。|
| **Crawling into a new era—the *Dictyostelium* genome project** | Eichinger et al. | 2003 | ゲノムプロジェクトの概要と、*D. discoideum*のゲノムが持つ高いA/T含量や反復配列の多さといったアセンブリ上の課題について論じています。|

## 2. 反復配列（トランスポゾン）に特化した論文

反復領域の正確なアセンブリには、これらのトランスポゾンファミリーの構造と挙動に関する知識が不可欠です。

| タイトル | 著者 | 発表年 | 関連性 |
| :--- | :--- | :--- | :--- |
| **Retrotransposon Domestication and Control in *Dictyostelium discoideum*** | Malicki et al. | 2017 | TRE5-A、**DIRS-1**、Skipper-1といった主要なトランスポゾン要素（TE）の構造と、宿主による制御機構についてレビューしています。|
| ***Dictyostelium discoideum* RNA-dependent RNA polymerase RrpC silences the centromeric retrotransposon DIRS-1 post-transcriptionally and is required for the maintenance of centromere integrity** | Wiegand et al. | 2014 | **DIRS-1**がセントロメアの構成要素として機能していること、およびその転写後サイレンシング機構について詳細に論じています。DIRS-1の配列とゲノム上の位置の検証に役立ちます。|
| **Skipper, an LTR retrotransposon of *Dictyostelium*** | Leng et al. | 1998 | Skipperという別の主要なトランスポゾンファミリーの構造と特徴を記述しています。|

## 3. AX2株を主要な実験株として使用している論文

これらの論文は、AX2株がリファレンスゲノム（AX4）と異なるゲノム背景を持つにもかかわらず、広く使用されていることを示しています。

| タイトル | 著者 | 発表年 | 関連性 |
| :--- | :--- | :--- | :--- |
| **The necessity of mitochondrial genome DNA for normal development of *Dictyostelium* cells** | Chida et al. | 2004 | AX2株を用いてミトコンドリアDNAの役割を研究した論文。AX2株の基本的な細胞生物学的特性に関する情報源となります。|
| **Cell-type specific RNA-Seq reveals novel roles and regulatory programs for terminally differentiated *Dictyostelium* cells** | Kin et al. | 2018 | AX2株を用いてRNA-Seq解析を実施しており、AX2株の遺伝子発現やアノテーションの検証に役立ちます。|
| **Protein kinases of *Dictyostelium discoideum*, strain AX-2** | T. Winckler et al. | 1979 | AX2株の初期の特性評価に関する論文の一つです。|

## 重要な注意点：AX2とAX4のゲノムの違い

お客様のゲノムアセンブリの検証において最も重要な点は、**AX2株とAX4株のゲノムが同一ではない**という事実です。

*   **リファレンスゲノム (AX4)**: 2005年に発表されたリファレンスゲノムはAX4株を基にしています。この株は、非軸性株NC4から派生したAX3株のサブクローンであり、**染色体2に約1.5 Mbの大きな逆位重複**（inverted duplication）を持つことが知られています [Eichinger et al., 2005]。
*   **AX2株**: AX2株はAX4株とは異なる系統の株であり、この**1.5 Mbの重複は存在しない**ことが確認されています [Bloomfield et al., 2008]。しかし、Bloomfieldらの研究では、AX2株にも独自のコピー数多型（CNV）や重複が存在することが示されています。

したがって、お客様のAX2アセンブリの検証では、**AX4リファレンスゲノムとの単純な比較だけでなく**、特に**Bloomfield et al. (2008)** の論文を参照し、AX2株特有のゲノム構造の有無を確認することが推奨されます。反復領域の正確なアセンブリのためには、DIRS-1やTREsといったトランスポゾンファミリーの配列とゲノム上の配置に特に注意を払う必要があります。

---
## 引用文献

1.  Eichinger, L., Pachebat, J. A., Glöckner, G., Rajandream, M. A., Sucgang, R., Berriman, M., ... & Kay, R. R. (2005). **The genome of the social amoeba *Dictyostelium discoideum***. *Nature*, 435(7038), 43-57. [PMC1352341](https://pmc.ncbi.nlm.nih.gov/articles/PMC1352341/)
2.  Bloomfield, G., Tanaka, Y., Skelton, J., Ivens, A., & Kay, R. R. (2008). **Widespread duplications in the genomes of laboratory stocks of *Dictyostelium discoideum***. *Genome Biology*, 9(4), R75. [PMC2643946](https://pmc.ncbi.nlm.nih.gov/articles/PMC2643946/)
3.  Glöckner, G., Szafranski, K., Winckler, T., Dingermann, T., Quail, M. A., Cox, E., ... & Eichinger, L. (2001). **The Complex Repeats of *Dictyostelium discoideum***. *Genome Research*, 11(4), 585-594. [PMC311061](https://pmc.ncbi.nlm.nih.gov/articles/PMC311061/)
4.  Eichinger, L., & Rivero, F. (2003). **Crawling into a new era—the *Dictyostelium* genome project**. *The EMBO Journal*, 22(10), 2321-2326. [PMC156086](https://pmc.ncbi.nlm.nih.gov/articles/PMC156086/)
5.  Malicki, M., Iliopoulou, M., & Hammann, C. (2017). **Retrotransposon Domestication and Control in *Dictyostelium discoideum***. *Frontiers in Microbiology*, 8, 1869. [PMC5626815](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC5626815/)
6.  Wiegand, S., Meier, D., Seehafer, C., Malicki, M., & Hammann, C. (2014). ***Dictyostelium discoideum* RNA-dependent RNA polymerase RrpC silences the centromeric retrotransposon DIRS-1 post-transcriptionally and is required for the maintenance of centromere integrity**. *Nucleic Acids Research*, 42(5), 3330-3340. [PMC3950669](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC3950669/)
7.  Leng, P., Kessin, R. H., & Devine, S. E. (1998). **Skipper, an LTR retrotransposon of *Dictyostelium***. *Nucleic Acids Research*, 26(8), 2008-2014. [PMC147514](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC147514/)
8.  Chida, J., Yamaguchi, H., Amagai, A., & Tanaka, Y. (2004). **The necessity of mitochondrial genome DNA for normal development of *Dictyostelium* cells**. *Journal of Cell Science*, 117(15), 3141-3147. [JCS:117/15/3141](https://journals.biologists.com/jcs/article-abstract/117/15/3141/53673)
9.  Kin, K., Irie, M., & Tanaka, Y. (2018). **Cell-type specific RNA-Seq reveals novel roles and regulatory programs for terminally differentiated *Dictyostelium* cells**. *BMC Genomics*, 19(1), 803. [PMC6237072](https://bmcgenomics.biomedcentral.com/articles/10.1186/s12864-018-5146-3)
10. T. Winckler, M. B. C. (1979). **Protein kinases of *Dictyostelium discoideum*, strain AX-2**. *European Journal of Biochemistry*, 96(3), 557-564. [DOI:10.1111/j.1432-1033.1979.tb13066.x](https://onlinelibrary.wiley.com/doi/abs/10.1111/j.1432-1033.1979.tb13066.x)
