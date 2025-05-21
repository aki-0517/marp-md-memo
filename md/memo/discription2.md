
---

### 1. 🔬 研究背景と目的

- ゲノムアセンブリは、**高精度かつ少数contigで完結するドラフトゲノム**が理想
    
- Dictyostelium discoideumでは**8リプリコン（6染色体+2 extra）**が目標形
    
- Illuminaのみでは困難 ⇒ ロングリード（ONT）+補完的ショートリード（Illumina）を用いる
    
- 本研究の目的：
    
    **ONTリードによるアセンブリ比較・評価を行い、最良の出力に対してpolishing・解析を進める**
    

---

### 2. 🧪 実験全体フロー（Kim & Kim プロトコル準拠）

```
(1) リード評価・可視化
(2) ゲノムサイズ推定
(3) 各アセンブラによるドラフトアセンブリ作成
(4) Quastによる比較評価（N50, contig数など）
(5) 最良アセンブリ（Canu）をベースに polishing
(6) BUSCO / RagTag / SVIM-asm / BRAKER へと展開

```

---

### 3. 📊 リード評価・ゲノムサイズ推定

- 使用リード：ONT（Oxford Nanopore）
    - **N50 = 12,777 bp**, 最大長 139 kb
- Flye によるゲノムサイズ推定：約 **33.5 Mb**
- 長さ分布ヒストグラムも提示可能

---

### 4. ⚙️ 各アセンブラの性能比較（Quast）

|Assembler|Contig数|N50|総長|GC含量|
|---|---|---|---|---|
|**Canu（50%）**|**14**|約3Mb|33.5 Mb|22.8%|
|Shasta|36|6.7 Mb|33.5 Mb|22.9%|
|Flye|40程度|2.8 Mb|34.3 Mb|22.98%|
|Raven|60程度|2.7 Mb|35.5 Mb|22.77%|

> ✅ Canu（50%）が最良：contig数が最も少なく、N50も高く、8リプリコンへの近似性が高い

---

### 5. 🧬 polishing戦略：Pilonによる2段階補正

- ベースアセンブリ：Canu（50%リード使用）
- 使用リード：ONT（長距離）、Illumina（高精度短距離）
- Pilon による2段階修正：
    1. `-fix snps,indels`
    2. `-fix gaps,local`

> Pilon適用後のFASTA, VCFから修正の度合いを確認可能

---

### 6. 📈 今後の解析ステップ

|ステップ|使用ツール|目的|
|---|---|---|
|アセンブリ精度評価|BUSCO|シングルコピー遺伝子の網羅率|
|リファレンス比較|RagTag|8リプリコンスキャフォールディング|
|構造変異検出|SVIM-asm|contigベースのSV検出|
|遺伝子アノテーション|BRAKER2 + HISAT2|RNA-seqに基づく構造予測|

---

### 7. 🧭 研究の位置づけと参考プロトコルとの比較

|項目|本研究|Kim & Kim (2022)|
|---|---|---|
|生物種|_D. discoideum_|_D. melanogaster_|
|リード|ONT + Illumina|PacBio HiFi + ONT + Illumina|
|アセンブラ|Canu 他|Canu, Hifiasm, Shasta|
|polishing|Pilon|Pilon|
|BUSCO / SVIM等の適用|今後予定|実施済み|

> ✅ プロトコルはKim & Kimと同様のフローを踏襲中。今後はBUSCO等による評価フェーズへ進む

---

### 8. 📝 まとめと今後の展望

- ✅ Canu（50%）が最も良好なアセンブリ結果（14 contigs）
- ✅ Pilon による polishing を開始・進行中
- 🔜 BUSCO / RagTag による精度評価・参照ベース補完へ
- 🎯 最終目標：**8リプリコン構成の高精度・高連続性ゲノムの完成**

---

### 🔧 補足スライド（必要なら）

- Canu / Flye / Shasta / Raven コマンド比較
- Pilon 出力ファイル例（.vcf, .fasta）
- Cell Press プロトコルの引用図や対応表（図2A/B/Cなど）
