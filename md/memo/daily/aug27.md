
- 未特定repetitive regionsをコマンドラインからrepbaseで検索
- ch-1~ch-6, rdnaと名前つけるべき
- 
### edta
https://github.com/oushujun/EDTA

### trf 
https://github.com/Benson-Genomics-Lab/TRF

### ncbi
https://www.ncbi.nlm.nih.gov/

https://www.ncbi.nlm.nih.gov/bioproject/?term=DIRS-1_Dictyostelium discoideum_のDIRS-1配列データ


# Dictyostelium discoideumおよび関連種のセントロメア領域アセンブリ手法の比較分析

## 1. はじめに

本報告書は、Dictyostelium discoideumおよび関連種のゲノムアセンブリにおいて、セントロメア領域がどのように扱われ、アセンブリされたかに関する研究論文を分析し、その手法を比較することを目的とする。セントロメアは染色体分離に不可欠な領域であるが、その構造は種間で多様であり、アセンブリが困難な場合が多い。本報告書では、複数の論文からセントロメア領域のアセンブリに関する情報を抽出し、そのアプローチを詳細に記述する。

## 2. 論文ごとのセントロメア領域アセンブリ手法

### 2.1. Centromere sequence and dynamics in Dictyostelium discoideum [1]

**アセンブリ手法の概要:**

この研究では、Dictyostelium discoideumのセントロメア領域の配列と動態を解析するために、以下の手法が用いられた。

1.  **全染色体アセンブリの取得:** Dictybaseから全染色体アセンブリが取得された。
2.  **BLASTデータベースの作成:** ゲノムショットガンシーケンシングプロジェクトからの全リードを用いてBLASTデータベースが作成された。
3.  **リードのBLAST処理:** 厳密なBLASTパラメータ（95%以上の同一性、100塩基以上）を用いて、生シーケンシングリードが既存の配列に対してBLAST処理された。
4.  **反復要素の確認:** 以前に定義された反復要素の存在を確認するために、再度BLASTが使用された。
5.  **コンティグの構築:** ゲノムプロジェクトで実装された命名規則により、染色体特異的なビンが構築された。暫定的に曖昧さのない配列を含むリードは、GAPアセンブラー（http://staden.sourceforge.net/）を用いて各染色体ごとに個別にアセンブリされた。
6.  **シードコンティグの拡張:** 現在のD. discoideumアセンブリには存在しないユニークな配列コンティグが「シード」コンティグとして使用された。以下のいくつかのアセンブリステップが、それ以上配列を追加できなくなるまで繰り返された。
    *   クローン（リードペア）のもう一方の端からの対応するリードを組み込んでコンティグを拡張する。
    *   リードペア情報によって隣接していると示されたコンティグを結合する。
    *   誤って組み込まれたリードを手動で確認し、削除する。
    *   インデル、切断領域、およびユニークな配列を特徴づける。
    *   BLASTを使用して、生シーケンシングリードデータベース内の類似リードを検索する。
7.  **ギャップの充填:** 接続クローン上のプライマーウォーキングによってシーケンシングギャップが充填された。反復領域のギャップに対してリードペアのサポートが存在しない場合、PCRアプローチが欠落配列の取得に使用された。ギャップは、2つのコンティグ末端と一致するトランスポゾンのコンセンサス配列を用いて埋めることができた。

**セントロメアの同定:**

この論文では、セントロメア領域が主にDIRS型トランスポゾンで構成されていることが示唆されている。セントロメア機能を持つこれらの領域は、細胞周期のインターフェーズおよびメタフェーズにおいて染色体末端がクラスター化するという観察から裏付けられている。また、これらの領域が小RNAを産生することも、セントロメア機能の調節に関与している可能性が指摘されている。

### 2.2. Chromosome-level genome assembly and annotation of the social amoeba Dictyostelium firmibasis [2]

**アセンブリ手法の概要:**

この研究では、Dictyostelium firmibasisの染色体レベルのゲノムアセンブリとアノテーションのために、以下の手法が用いられた。

1.  **ゲノムDNAの調製とシーケンシング:**
    *   ゲノムDNA（gDNA）は、Genomic-tip 100/Gカラム（Qiagen）を用いて単離された。
    *   ロングリードシーケンシング：初期のクリーンアップは、D. firmibasisゲノムDNA（9.3 μg）250 μlをAMPure XPビーズ（Beckman Coulter）112.5 μlに添加して行われ、高分子量DNAを選択した。DNAは磁気ビーズに結合させ、5分間回転させ、70% EtOH 200 μlで2回洗浄し、37 °Cで10分間インキュベートして62 μl H2Oで溶出した。高分子量DNAのさらなる選択は、Short Read Eliminator（SRE）キット（PacBio）を用いて行われた。
    *   ロングリードシーケンシングライブラリは、gDNA 1.5 μgからSQK-LSK112ライゲーションシーケンシングキット（Oxford Nanopore）を用いて調製された。得られたライブラリの100 ngは、MinION Mk1C（Oxford Nanopore）上のR10.4フローセルで72時間シーケンシングされた。ベースコールはGuppy v6.3.2（Oxford Nanopore）を用いてr104_e81_sup_g610モデルで行われた。合計で7.5 Gbpのロングリード配列が生成された。
2.  **ゲノムアセンブリ:**
    *   ゲノムはFlye v2.9.1を用いてデフォルト設定でアセンブリされた。
    *   ポリッシングはMedaka v1.6.0を用いてr104_e81_sup_g610モデルで行われた。
    *   さらなるポリッシングは、Pilon v1.23を用いてIlluminaショートリードで行われた。
    *   アセンブリはBandage v0.8.1を用いて手動でキュレーションされた。
3.  **アノテーション:**
    *   遺伝子予測はAugustus v3.3.3を用いて行われた。
    *   機能アノテーションは、NCBI非冗長タンパク質データベースに対するBLASTを用いて行われた。
    *   tRNA遺伝子はtRNAscan-SE v2.0を用いて予測された。
    *   rRNA遺伝子はBarrnap v0.9を用いて予測された。
    *   反復要素はRepeatMasker v4.1.2を用いて同定された。

**セントロメアの同定:**

この論文では、新しいアセンブリには未決定の塩基がなく、主に染色体を表す6つの大きなコンティグで構成されていると述べられている。テロメアの分析とD. discoideumゲノムとの比較により、6つの主要なコンティグがほぼ完全な染色体を表していることが確認された。セントロメアのアセンブリや同定について明示的に詳述されていないが、染色体レベルの完全なアセンブリにはセントロメア領域が含まれていることが示唆されている。

### 2.3. The multicellularity genes of dictyostelid social amoebas [3]

**アセンブリ手法の概要:**

この研究では、Dictyostelid social amoebasの多細胞性遺伝子を解析するために、以下のゲノムシーケンシングとアセンブリ手法が用いられた。

1.  **ゲノムシーケンシングとアセンブリ:**
    *   DLゲノムは、454 Rocheプラットフォームを用いて29倍のカバー率でシーケンシングされた。
    *   アセンブリは、平均挿入サイズ32.5 kbの末端シーケンスされたフォスミドの詳細なマップによって支援された。
    *   スキャフォールドに残ったギャップは、プライマーウォーキングによって閉じられ、最終的に54個のギャップフリーコンティグのアセンブリが得られた。
    *   初期アセンブリはNewblerアセンブラーで行われ、500塩基を超えるコンティグはStadenアセンブリパッケージに入力された。
    *   フォスミド末端はABI BigDyeキットを用いてシーケンシングされ、標準フォワードプライマー5′-GTACAACGACACCTAGAC-3′とリバースプライマー5′-CAGAAACAGCCTAGGAA-3′が使用された。アセンブリ前の配列トリミングが行われた。

**セントロメアの同定:**

この論文では、D. discoideumがA/Tリッチな反復配列からなる特異な染色体末端を持つことが言及されており、これは染色体外リボソームDNAパリンドローム末端にも存在するとされている。D. firmibasisとP. pallidumの染色体とパリンドロームは通常の真核生物のGTリッチなテロメア反復配列を持つ一方、DLは通常の真核生物の染色体テロメアとD. discoideumのようなA/Tリッチなパリンドローム末端を持つ中間的な状況を示している。テロメアとその特性については議論されているが、セントロメア領域のアセンブリや同定については明示的に詳述されていない。

## 3. 比較分析と結論

上記3つの論文を比較すると、Dictyostelium属のゲノムアセンブリは、その高い反復配列含有量とA+Tリッチ性という共通の課題に直面していることがわかる。これらの課題に対処するため、各研究は異なるアプローチを採用している。

*   **Centromere sequence and dynamics in Dictyostelium discoideum [1]** は、特定のセントロメア領域に焦点を当て、BLAST、GAPアセンブラー、プライマーウォーキング、手動キュレーションを組み合わせた詳細なアプローチを採用している。この研究は、DIRS型トランスポゾンがセントロメア機能に寄与している可能性を示唆している。

*   **Chromosome-level genome assembly and annotation of the social amoeba Dictyostelium firmibasis [2]** は、ロングリードシーケンシング（Oxford Nanopore）と、Flye、Medaka、Pilon、Bandageといった最新のアセンブリおよびポリッシングツールを組み合わせることで、染色体レベルの高品質なアセンブリを達成している。このアプローチは、セントロメア領域を明示的にアセンブリしたとは述べていないものの、完全な染色体レベルのアセンブリがこれらの領域を含んでいることを示唆している。

*   **The multicellularity genes of dictyostelid social amoebas [3]** は、454 Rocheプラットフォームによるシーケンシングと、フォスミドマップ、プライマーウォーキング、Newblerアセンブラーを組み合わせた手法を用いている。この論文もテロメアの特性に言及しているが、セントロメア領域のアセンブリについては詳細な記述がない。

全体として、初期の研究ではショットガンシーケンシングとギャップ充填のための手動アプローチが用いられていたのに対し、最近の研究ではロングリードシーケンシング技術と高度なアセンブリソフトウェアの進歩が、より完全な染色体レベルのゲノムアセンブリを可能にしていることが明らかになった。セントロメア領域の特性（反復配列の多さなど）は依然としてアセンブリの課題であるが、新しい技術はこれらの困難な領域の解明に貢献している。今後の研究では、これらの領域の機能的検証と、異なるDictyostelium種間でのセントロメア構造の進化に関する比較分析がさらに進むことが期待される。

## 4. 参考文献

[1] Glöckner, G., & Heidel, A. J. (2009). Centromere sequence and dynamics in Dictyostelium discoideum. *Nucleic Acids Research*, *37*(6), 1809–1816. [https://pmc.ncbi.nlm.nih.gov/articles/PMC2665215/](https://pmc.ncbi.nlm.nih.gov/articles/PMC2665215/)

[2] Edelbroek, B., Kjellin, J., Jerlström-Hultqvist, J., Koskiniemi, S., & Söderbom, F. (2024). Chromosome-level genome assembly and annotation of the social amoeba Dictyostelium firmibasis. *Scientific Data*, *11*(1), 678. [https://www.nature.com/articles/s41597-024-03513-8](https://www.nature.com/articles/s41597-024-03513-8)

[3] Glöckner, G., Lawal, H. M., Felder, M., Singh, R., Singer, G., Weijer, C. J., & Schaap, P. (2016). The multicellularity genes of dictyostelid social amoebas. *Nature Communications*, *7*(1), 12085. [https://www.nature.com/articles/ncomms12085](https://www.nature.com/articles/ncomms12085)



`assembly2/assembly-results/ragtag_flye_scaffold/ragtag_polished_round1.fasta` に対して、**conda を使って TRF と EDTA を実行する手順**をまとめます。

---

# 1. Conda 環境の準備

まずは作業用の conda 環境を作っておくと安全です。

```bash
conda create -n repeats -c bioconda -c conda-forge trf edta
conda activate repeats
```

これで **TRF** と **EDTA** の両方が入ります。  
（もし片方でエラーが出たら個別にインストールするのもOKです）

---

# 2. TRF の実行

### コマンド

```bash
trf \
  assembly2/assembly-results/ragtag_flye_scaffold/ragtag_polished_round1.fasta \
  2 7 7 80 10 50 500 -d -h
```

### 出力

- `ragtag_polished_round1.fasta.2.7.7.80.10.50.500.dat`  
    → 機械可読フォーマット（リピート位置、コピー数、リピート長など）
    
- `ragtag_polished_round1.fasta.2.7.7.80.10.50.500.html`  
    → ブラウザで見れる整形済みレポート
    

> **ポイント**
> 
> - **period size**（繰り返し単位の長さ）が 150–200bp 前後でコピー数が多い領域はセントロメア候補
>     
> - この `.dat` を BED/GFF に変換して genome browser に読み込むと可視化しやすい
>     

---

# 3. EDTA の実行

### コマンド

```bash
EDTA.pl \
  --genome assembly2/assembly-results/ragtag_flye_scaffold/ragtag_polished_round1.fasta \
  --species others \
  --step all \
  --threads 16
```

### 出力（主要なもの）

- `*.fasta.mod.EDTA.TEanno.gff3` → TE アノテーション（IGV/JBrowse にそのままロード可能）
    
- `*.fasta.mod.EDTA.TElib.fa` → 構築されたリピートライブラリ
    
- `*.fasta.mod` → 繰り返しマスク済みゲノム配列
    

> **ポイント**
> 
> - GFF3 をロードして「DIRS, LTR retrotransposon が連続する領域」を探す
>     
> - TRF 結果と重ねて「tandem repeat ＋ TE が密集していて遺伝子が少ない領域」＝セントロメア候補
>     

---

# 4. 実行後の次ステップ

1. **TRF の .dat → BED/GFF 変換**（スクリプトが必要なら書けます）
    
2. **IGV/JBrowse で TRF 結果＋EDTA GFF3＋遺伝子アノテーション**を重ねる
    
3. TE クラスター & tandem repeat が重なる領域を抽出
    

---

👉 Akinobu さん、次のアクションは

- 「実際に上のコマンドを流して出てきたファイルの見方を教えてほしい」
    
- それとも「TRF の .dat を BED/GFF に変換する具体的なスクリプトが欲しい」
    

どちらに進めたいですか？