

## 🧬 発表タイトル（仮）

**「ロングリードデータを用いたDictyostelium discoideum ゲノムの段階的アセンブリ戦略」**

― _Cell Press Protocol に準拠した実践的ガイドラインと最適戦略の検討_

---

## 1. 🎯 研究の最終ゴール（目指す姿）

本研究の最終的なゴールは、以下の論文と同様に、

> 高精度なドラフトゲノムの構築と構造変異（SV）の解析可能な状態に仕上げること
> 
> （参照：Kim & Kim, _A Cell Press journal_, July 2022）

にあります。そのために必要な作業フローは、大きく以下の5段階に分けられます：

1. **ロングリード分布の可視化と評価**
2. **ドラフトアセンブリの構築**
3. **アセンブリ精度の評価（N50・BUSCOなど）**
4. **アセンブリベースでの構造変異（SV）検出**
5. **RNA-seq による遺伝子アノテーション**

---

## 2. 📍 現在の到達地点（どこまで進んだか）

|ステップ|現状|使用データ・ツール|
|---|---|---|
|リード評価|✅ 完了|ONTデータによるread-length分布（N50: 12,777）|
|アセンブリ比較|✅ 完了|Canu, Flye, Raven, Shasta で比較。**Canu（50%）が最良（14 contigs）**|
|Polishing（前段）|✅ 一部完了|ONT+Illuminaリードを使った Pilon による2段階修正|

### ✅ 結論：**Canu（50%）出力をベースにPolishingを進行中**

---

## 3. 🚀 今後の戦略とロードマップ

### 🛠 Step 1: polishing の完了（現在進行中）

- Pilon を2段階（`snps+indels` → `gaps+local`）で適用し、塩基精度を最大化。
- 次の段階に進む前に、**BUSCO**で完成度を数値評価する。

---

### 📐 Step 2: 高精度なアセンブリ評価と構造解析（次の到達点）

|タスク|方法|目的|
|---|---|---|
|BUSCO解析|`busco -m genome -l dictyostelium_odb10`|シングルコピー遺伝子の完全性確認|
|Cumulative plot作成|`assembly-stats`, `ggplot2`|contig分布の可視化|
|参照比較・scaffold化|`RagTag`|理想的な**8リプリコンへの近似**|
|SV検出|`SVIM-asm`|オリジナルゲノムとの差異（欠失・挿入など）把握|

---

### 🧬 Step 3: 遺伝子アノテーション（最終段階）

|タスク|ツール|備考|
|---|---|---|
|繰り返し領域のマスキング|`RepeatModeler`, `RepeatMasker`|正確な予測の前処理|
|RNA-seqマッピング|`HISAT2`|アセンブリとの対応づけ|
|遺伝子構造予測|`BRAKER2`|`GeneMark` + `Augustus` によるエビデンスベース予測|

---

## 4. 🧭 成果目標（成功の定義）

|指標|成果目標値|達成状況|
|---|---|---|
|Contig数|≦ 8|現状14（Canu）→ RagTag適用予定|
|N50長|≧ 1Mb|✅ クリア（Canu N50 2–3 Mb）|
|BUSCO 完全性|≧ 95%|⏳ 評価予定|
|SV 検出数|適切な検出精度|⏳ SVIM-asm適用予定|
|遺伝子アノテーション|GTF生成|⏳ BRAKER2適用予定|

---

## 5. 💡 まとめ

- **Cell Press 論文に沿った実践プロトコル**を自前のDictyosteliumデータに適用しつつ、最適な戦略を探索。
- 現時点では、**Canu（50%）が最良**のアセンブリ出力であるため、これを軸に polishing → 評価 → 構造解析 → アノテーションまで進める。
- 今後は、**BUSCO・RagTag・SVIM-asm・BRAKER2**を用いた統合解析により、**研究利用可能な完全ドラフトゲノム**の構築を目指す。

---

## 📚 発表全体構成案

**「ロングリードシーケンシングを用いたDictyostelium discoideumのゲノムアセンブリとPolishing戦略」**

（参考：Kim & Kim, _Cell Press journal_, 2022）

---

### 0. タイトル & イントロダクション

- 発表タイトル
    
- 発表者名・日付
    
- 本研究の参照プロトコル：
    
    _A beginner’s guide to assembling a draft genome and analyzing structural variants with long-read sequencing technologies_（Kim & Kim, 2022, Cell Press）
    
