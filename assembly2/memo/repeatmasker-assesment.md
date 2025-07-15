Flye アセンブリに RepeatMasker を適用した前後での BUSCO 比較結果から、以下のような変化が確認できます。

---

## 1. 完全性スコア（C/S/D/F/M）

| 指標                         | Before RM       | After RM        | 変化       |
| -------------------------- | --------------- | --------------- | -------- |
| C (Complete)               | 94.9% (242/255) | 94.9% (242/255) | 同一       |
| – S (Single-copy)          | 93.3% (238)     | 93.3% (238)     | 同一       |
| – D (Duplicated)           | 1.6% (4)        | 1.6% (4)        | 同一       |
| F (Fragmented)             | 1.2% (3)        | 1.2% (3)        | 同一       |
| M (Missing)                | 3.9% (10)       | 3.9% (10)       | 同一       |
| E (Errors: internal stops) | 1.2% (3/242)    | 0.8% (2/242)    | 減少 (3→2) |

**ポイント**

* Complete〜Missing の主要なスコアは全く変わらず、RepeatMasker 前後で遺伝子の「検出率」自体には影響しません。
* ただし内部終止コドンを含む件数（E）は 3 → 2 と軽微に改善しています。

---

## 2. コンティグ／スキャフォールド統計

| 指標                  | Before RM  | After RM   | 変化             |
| ------------------- | ---------- | ---------- | -------------- |
| Number of scaffolds | 11         | 11         | 同一             |
| Number of contigs   | 16         | 12         | 減少 (16→12)     |
| Total length (bp)   | 34,875,698 | 34,807,692 | 減少 (\~68 kb)   |
| Percent gaps        | 0.001%     | 0.000%     | ギャップ完全解消       |
| Scaffold N50        | 8 MB       | 8 MB       | 同一             |
| Contig N50          | 3 MB       | 6 MB       | 増加 (3 MB→6 MB) |

**ポイント**

* Contig 数が 16→12 と減少し、小さなコンティグがマスク後に統合または除去された可能性を示唆します。
* Contig N50 が 3 MB→6 MB と大きく伸び、局所的な連続性（アセンブリの断片化）が改善しています。
* スキャフォールドレベルでは変化が見られず、全体構造は維持されています。
* ギャップ率が 0.001%→0.000% となり、N（ギャップ）マスク処理により報告上は完全にギャップが消えています。

---

### 総括

* **遺伝子検出率（BUSCO C/S/D/F/M）** に大きな影響はないものの、**内部終止コドン**のあるモデルがわずかに減少。
* **コンティグの連続性**（Contig N50 と数）に改善が見られ、小規模断片の減少と長いコンティグの増加が確認できました。
* **スキャフォールド構造** 自体は前後でほぼ変わらず、RepeatMasker は主にベースレベルの配列置換・注釈に留まる一方、下流解析に向けたクリーンアップとして有効だったと言えます。


Flye アセンブリに RepeatMasker を適用した前後での BUSCO 比較結果から、以下のような変化が確認できます。

---

## 1. 完全性スコア（C/S/D/F/M）

|指標|Before RM|After RM|変化|
|---|---|---|---|
|C (Complete)|94.9% (242/255)|94.9% (242/255)|同一|
|– S (Single-copy)|93.3% (238)|93.3% (238)|同一|
|– D (Duplicated)|1.6% (4)|1.6% (4)|同一|
|F (Fragmented)|1.2% (3)|1.2% (3)|同一|
|M (Missing)|3.9% (10)|3.9% (10)|同一|
|E (Errors: internal stops)|1.2% (3/242)|0.8% (2/242)|減少 (3→2)|

**ポイント**

- Complete〜Missing の主要なスコアは全く変わらず、RepeatMasker 前後で遺伝子の「検出率」自体には影響しません。
    
- ただし内部終止コドンを含む件数（E）は 3 → 2 と軽微に改善しています。
    

---

## 2. コンティグ／スキャフォールド統計

|指標|Before RM|After RM|変化|
|---|---|---|---|
|Number of scaffolds|11|11|同一|
|Number of contigs|16|12|減少 (16→12)|
|Total length (bp)|34,875,698|34,807,692|減少 (~68 kb)|
|Percent gaps|0.001%|0.000%|ギャップ完全解消|
|Scaffold N50|8 MB|8 MB|同一|
|Contig N50|3 MB|6 MB|増加 (3 MB→6 MB)|

**ポイント**

- Contig 数が 16→12 と減少し、小さなコンティグがマスク後に統合または除去された可能性を示唆します。
    
- Contig N50 が 3 MB→6 MB と大きく伸び、局所的な連続性（アセンブリの断片化）が改善しています。
    
- スキャフォールドレベルでは変化が見られず、全体構造は維持されています。
    
- ギャップ率が 0.001%→0.000% となり、N（ギャップ）マスク処理により報告上は完全にギャップが消えています。
    

---

### 総括

- **遺伝子検出率（BUSCO C/S/D/F/M）** に大きな影響はないものの、**内部終止コドン**のあるモデルがわずかに減少。
    
- **コンティグの連続性**（Contig N50 と数）に改善が見られ、小規模断片の減少と長いコンティグの増加が確認できました。
    
- **スキャフォールド構造** 自体は前後でほぼ変わらず、RepeatMasker は主にベースレベルの配列置換・注釈に留まる一方、下流解析に向けたクリーンアップとして有効だったと言えます。


概要として、Flye（約34.8 Mb）とCanu（約35.9 Mb）のアセンブリ長の差分は主に重複コンティグ（haplotig）や未解決リピート、ギャップ埋めの不足によるものです。このギャップを縮め、真のゲノム長（約34.2 Mb）に近づけるには、以下のようなステップを検討するとよいでしょう。

## 1. 重複コンティグ（haplotig）の除去・統合

### 1.1 Purge Haplotigs や Purge Dups の活用

- **Purge Haplotigs** や **Purge Dups** は、過剰に重複したコンティグ（特にヘテロ接合領域の両アレル）を検出・削除し、ホモロガス部分を一本化します。
    
- カバレッジ分布を用いて「単コピー vs 重複コピー」を分類し、高カバレッジの冗長コンティグを削除することで、総長を減少させられます。
    

```bash
# 例: Purge Dups
minimap2 -x map-ont -t 16 assembly.fasta assembly.fasta > aln.paf
purge_dups -2 -T cutoffs.txt -c coverage.hist aln.paf > dups.bed
seqkit subseq -v -i dups.bed assembly.fasta > assembly.purged.fasta
```

### 1.2 HaploMerger2 の適用

- 真の一重コピーゲノムを再構築するために、HaploMerger2 を使って、異なるアレル間の重複を統合・マージする手法もあります。
    

## 2. ギャップフィリングとスキャフォールディングの改善

### 2.1 LR GapCloser や TGS-GapCloser

- 長リードを用いたギャップクローズツール（例：LR_GapCloser, TGS-GapCloser）で、スキャフォールド中のNギャップを埋めることで、総長を参照により近い値に調整できます。
    

```bash
# 例: LR_GapCloser
LR_GapCloser -i scaffold.fasta -l ont_reads.fastq -o scaffold.gapclosed.fasta
```

### 2.2 RagTag/Reference‐guided Scaffolding

- 既知の染色体レベル参照を使って、RagTag などのリファレンスガイドスキャフォールディングを再度行い、不足している領域をリファレンス由来で補完する。
    

```bash
ragtag.py scaffold reference.fna assembly.fasta -o ragtag_scaffolded
```

## 3. さらなるポリッシングとベースレベル精度向上

### 3.1 多段階ポリッシング（Medaka → Pilon → Racon）

- 複数のポリッシングツールを段階的に組み合わせると、塩基エラーによるアセンブリの膨張（誤インデルやギャップ拡大）を抑えられます。
    

```bash
racon -t 16 ont_reads.fq assembly.fasta aln.sam > assembly.racon.fasta
medaka_consensus -i ont_reads.fq -d assembly.racon.fasta -o medaka_out
pilon --genome medaka_out/consensus.fasta --frags illumina.bam --output pilon_out
```

## 4. アセンブリ統合（Assembly Merging）

### 4.1 Meta‐assembler の利用

- Flye と Canu の双方の長所を生かすため、**Metassembler** や **Quickmerge** を使い、両アセンブリを統合して長さと精度を向上させる。
    

```bash
quickmerge -d canu.fasta -q flye.fasta -hco 5.0 -lco 5.0 -ml 20000 -o merged.fasta
```

---

これらの手法を組み合わせることで、不要な重複やギャップを削減し、真のゲノム長（約34.2 Mb）への収束を図ることができます。



以下に、_Dictyostelium discoideum_ AX2（GCA_000004695.1）の反復配列（repetitive regions）を取り扱う（検出・マスク・解決する）手法に言及した主要な論文・リソースをまとめました。これらの研究では、RepeatMasker や de novo 検出ツール、参照ガイド型アセンブリ、手動フィニッシングなど、さまざまなアプローチで反復領域への対応策を講じています。

---

## 1. RepeatMasker を用いた反復配列の検出とマスク

- **UCSC Genome Browser: RepeatMasker Track for D. discoideum AX4**  
    AX4／AX2 の参照アセンブリ（2009年版）に対し、Arian Smit の **RepeatMasker** プログラムでインタースプレースドリピートおよび低複雑度配列を検出・マスクしている ([hgdownload.soe.ucsc.edu](https://hgdownload.soe.ucsc.edu/hubs/GCA/000/004/695/GCA_000004695.1/html/GCA_000004695.1_dicty_2.7.repeatMasker.html?utm_source=chatgpt.com "Untitled"))。
    
    - コマンド例:
        
        ```bash
        RepeatMasker -species "dictyostelium discoideum" -s -no_is -frag 20000 assembly.fasta
        ```
        
    - 出力として、反復領域を N でマスクした FASTA と詳細アノテーションを得られる。
        
- **Frith, M. C. & Hamada, M. “A new repeat-masking method enables specific detection of homologous sequences” (2010)**  
    AT-richゲノム（マラリア原虫や粘菌を含む）で標準的なリピートマスキングが困難な点を指摘し、**tantan** という確率モデルに基づく新手法を提案 ([ResearchGate](https://www.researchgate.net/publication/49637590_A_new_repeat-masking_method_enables_specific_detection_of_homologous_sequences?utm_source=chatgpt.com "(PDF) A new repeat-masking method enables specific detection of ..."))。
    
    - slime-mould ゲノムでの性能改善を報告。
        
    - D. discoideum のようなリピート多発・AT-rich ゲノム解析に応用可能。
        

---

## 2. 反復配列のデノボ検出と分類

- **Ponger, L. et al. “The complex repeats of _Dictyostelium discoideum_” (2001)**  
    ディクティオステリウムに存在する多様な転移因子（DIRS-1, TRE5, TRE3, skipper など）を de novo で同定し、転位様配列の分類とコピー数推定を行った ([PubMed](https://pubmed.ncbi.nlm.nih.gov/11282973/?utm_source=chatgpt.com "The complex repeats of Dictyostelium discoideum - PubMed"))。
    
    - 反復要素が塩基配列の約10 %を占めることを定量化し、アセンブリ中の断片化や未配置コンティグ（floating contigs）の原因としてリピートが関与する点を示唆。
        
- **Weissenbach, J. et al. “Characterization of a new repetitive sequence in the _Dictyostelium_ genome” (1985)**  
    BglII断片ライブラリから 1 kb 程度の新規反復配列を単離・シーケンスし、その構造を報告 ([ScienceDirect](https://www.sciencedirect.com/science/article/abs/pii/0003986185904746?utm_source=chatgpt.com "Characterization of a new repetitive sequence in the Dictyostelium ..."))。
    
    - Tandem repeat motif や配列境界の特徴を解析し、リピート要素のゲノム分布への示唆を提供。
        

---

## 3. 参照ガイド型アセンブリ／手動フィニッシングによる解決

- **Eichinger, L. et al. “The genome of the social amoeba _Dictyostelium discoideum_” (2005)**  
    HAPPY mapping や手動フィニッシングを多用し、浮動コンティグ（floating contigs）の大半が反復領域に挟まれていることを指摘 ([PMC](https://pmc.ncbi.nlm.nih.gov/articles/PMC1352341/?utm_source=chatgpt.com "The genome of the social amoeba Dictyostelium discoideum - PMC"))。
    
    - Chromosome 2 の再アセンブリで、リピート境界における組み換えやクローンフィニッシングで高精度化。
        
- **Thompson, C. R. et al. “Centromere sequence and dynamics in _Dictyostelium discoideum_” (2007)**  
    クロモソーム末端に存在する DIRS-1 リピートクラスターを手動で再構築し、セントロメア領域の構造を解明 ([PMC](https://pmc.ncbi.nlm.nih.gov/articles/PMC2665215/?utm_source=chatgpt.com "Centromere sequence and dynamics in Dictyostelium discoideum"))。
    
    - アセンブリ困難なセントロメア反復を特異的に扱うため、PCR およびクローンベースのアプローチを併用。
        
- **Maeda, M. et al. “Crawling into a new era—the _Dictyostelium_ genome project” (EMBO J, 2002)**  
    初期リードを BLAST ベースでリピートアセンブリサイクルに振り分け、組み換え的にアセンブリを更新する手法を記述 ([EMBO Press](https://www.embopress.org/doi/10.1093/emboj/cdg214?utm_source=chatgpt.com "Crawling into a new era—the Dictyostelium genome project"))。
    
    - 反復領域リードの誤配置を抑制するため、seed clones→BLAST→binning のループを実装。
        

---

## 4. ストック間での重複配列（大型リピート）の実態調査

- **Bloomfield, D. S. et al. “Widespread duplications in the genomes of laboratory stocks of _Dictyostelium discoideum_” (2008 & 2009)**  
    Ax2／Ax3 等の実験株間で 15 kb 以上の大型重複（segmental duplications）が頻出し、アセンブリやマッピングの誤解釈を招きやすいことを報告 ([PMC](https://pmc.ncbi.nlm.nih.gov/articles/PMC2643946/?utm_source=chatgpt.com "Widespread duplications in the genomes of laboratory stocks of ..."), [BioMed Central](https://genomebiology.biomedcentral.com/articles/10.1186/gb-2008-9-4-r75?utm_source=chatgpt.com "Widespread duplications in the genomes of laboratory stocks of ..."))。
    
    - 反復や重複配列を同定することで、株間変異とアセンブリ精度の評価に寄与。
        

---

## 5. データベース・ブラウザでのアノテーション活用

- **dictyBase “The Dictyostelium Sequencing Project”**  
    ゲノム中のレトロトランスポゾン由来要素（約5 %）やDNAトランスポゾン要素をカタログ化し、可視化ブラウザでマスク済み配列を提供 ([Dictybase](https://dictybase.org/genomeseq.htm?utm_source=chatgpt.com "The Dictyostelium Sequencing Project - dictyBase"))。
    
    - RepeatMasker や de novo 検出結果を組み合わせた総合アノテーションをダウンロード可能。
        

---

## 結論

_Dictyostelium discoideum_ AX2 の反復配列への対応法としては、

1. **RepeatMasker** など既存ツールによるマスク
    
2. **tantan** のような確率モデルベースの改善法
    
3. **de novo** 検出＆分類による要素リスト化
    
4. **参照ガイド型アセンブリ**（RagTag, BLAST-based binning）や手動フィニッシングによる問題領域の解決
    
5. **株間重複配列の把握**
    

といった多層的なアプローチが報告されています。これらを組み合わせることで、AX2 ゲノムの高品質アセンブリおよび下流解析が可能となります。