
## ref data
https://www.ncbi.nlm.nih.gov/datasets/genome/GCF_000004695.1/

## 📊 NanoPlot QC 評価（定量的）

| 指標 | 結果 | 比較基準 | 評価 | コメント |
|------|------|----------|------|----------|
| **総リード数** | **934,886** | ONTシーケンスでは10万以上が一般的 | ✅ 十分 | 高カバレッジのアセンブリが可能 |
| **総塩基数** | **8.36 Gb** | *D. discoideum* のゲノム \~34 Mb → 必要カバレッジ=34 Mb × 50 = **1.7 Gb** | ✅ **246×カバレッジ** | 十分以上の冗長性。フィルタ後でも問題なし |
| **平均リード長** | **8,941 bp** | ONTリード平均 5–15 kb が一般的 | ⭕ | |
| **リードN50** | **12,777 bp** | RavenやFlye推奨N50 >10 kb | ✅ 適正範囲 | 構造的に強いアセンブリに貢献 |
| **最大リード長** | **139,714 bp** | Flye推奨 >50 kb あればよい | ✅ 非常に良好 | 長距離構造決定に有利 |
| **平均品質スコア** | **13.8** | ONT通常分布 10–15 | ⭕ | |
| **Q>15 リード割合** | **44.2%** | Medaka推奨：Q>15で50%以上あれば良い | ❌ やや低め | フィルタしてからアセンブリ or polishing重視 |
| **Q>20 リード割合** | **1.0%** | 理想は10%以上（次世代フローセルで達成可能） | ❌ 低 | これはONTの限界。Pilonなどで補正必須 |

---

## 📌 評価まとめ（定量的）

| 指標カテゴリ        | 結論                                                 |
| ------------- | -------------------------------------------------- |
| **量（カバレッジ）**  | **246×** → 十分以上。フィルタ後も余裕                           |
| **長さ（N50）**   | **12.7 kb** → 十分。Flye, Raven対応水準                   |
| **精度（品質スコア）** | **Q15超 = 44.2%** → polishing必須。Medakaだけでは不十分な可能性あり |
| **構造解像度**     | **最大リード = 139 kb** → 良好。リピート領域も越えられる可能性大           |

---

## 🔧 推奨アクション（定量評価に基づく）

1. **リードフィルタリング**（`Filtlong`等）

   * 条件例：長さ > 2,000 bp、Q > 10 または >Q15
   * **仮にQ15未満を除去してもカバレッジ >100× 残る**

2. **アセンブリツール選定**

   * `Flye` or `Raven`：10 kb以上のN50と高カバレッジに適している
   * `Canu`：高精度で時間コストはやや高
   * → **推奨：Flye**

3. **ポリッシング**

   * ONTのみ：`Medaka` → **限界あり**
   * Illumina併用可：`Pilon` or `NextPolish` → **精度向上期待**
   * → **併用を強く推奨**

4. **QC再評価**

   * アセンブリ後：`QUAST`, `BUSCO`で定量評価継続
   * ゴール：**BUSCO Complete > 95%**, **N50 > 1 Mb**, **Contig数 ≈ 染色体数（8）**


---

## 📉 なぜ「過剰カバレッジ」がアセンブリに悪影響を与えるか？

| 問題             | 説明                                                             |
| -------------- | -------------------------------------------------------------- |
| **リードエラーの蓄積**  | ONTリードは誤差（主にインデル）が多いため、カバレッジが高すぎると**エラーの誤った合意**が起き、アセンブリ精度が下がる |
| **リピート領域の誤解釈** | リピートに富んだゲノムでは、**短い低品質リードが重複領域を誤って橋渡し**して誤結合する可能性が高まる           |
| **リソース消費の増大**  | 計算コストとメモリが増え、処理が不安定に。特に `Canu` や `Shasta` で顕著                  |
| **ノイズリードの影響**  | 短くて低品質なリードが多数混在すると、誤ったコンセンサスを導きやすい                             |

🧠 一般に、**ONTリードでの最適カバレッジは 30–60× 程度**です（FlyeやCanuもこの範囲を推奨）。

---

## ✅ フィルタ戦略（目標：高品質なサブセットで50–60×カバレッジ）

### 📌 1. フィルタ条件の決定

| 指標            | 推奨カットオフ                             | 理由                                      |
| ------------- | ----------------------------------- | --------------------------------------- |
| **リード長**      | `--min_length 3,000`                | 短すぎるリードは誤マッピング・誤接続の元                    |
| **クオリティ**     | `--min_mean_q 12〜15`                | Q15で約97%正確（1 error/50bp）→ polishingしやすい |
| **ターゲットベース数** | `--target_bases 2000000000`（= 2 Gb） | ゲノム34 Mb × 60× = 2.04 Gb                |


1. **`--min_length 3000`**
    
    - 目的：極端に短いリード（<3 kb）は反復領域の誤ブリッジやノイズ原因になりやすいため除去。
        
    - 効果：構造的に意味のある長リードを優先的に残し、アセンブリの正確性を向上。
        
2. **`--min_mean_q 12`**
    
    - 目的：平均品質スコアが12未満のリードはエラー率が高く（誤り率 >6％）アセンブリ精度低下を招くため除去。
        
    - 効果：Polishing（Medaka/Pilon）前のデータ品質を底上げし、後続ステップの効率・精度を向上。
        
3. **`--target_bases 2000000000`（≈2 Gb）**
    
    - 目的：_D. discoideum_ （約34 Mb）のゲノムに対して約60×カバレッジを維持するため。
        
    - 効果：過剰カバレッジによるエラーの誤った合意や計算リソース浪費を防ぎつつ、十分な冗長性を確保。
        


Filtlong のフィルター条件

1. **長さが 3,000 bp 未満のリードを捨てる**
    
    - Nanopore リードには短いもの（たとえば数百 bp）のノイズも混ざっています。
        
    - 短すぎるリードはゲノムの大きな構造（リピート領域など）を正しくつなげるのにあまり役立たないので、3 kb（3,000 bp）以上の長さのリードだけを残します。
        
2. **平均品質スコアが 12 未満のリードを捨てる**
    
    - 「品質スコア」は塩基読み取りの精度を表し、スコア 12 は約 94 % の正確さを示します。
        
    - 平均が 12 未満だと誤り（特にインデル）が多くなるので、12 以上のスコアを持つキレイなリードだけを残します。
        
3. **出力合計を 2 Gb（約 60×カバレッジ）に制限する**
    
    - _Dictyostelium_ のゲノムは約 34 Mb（メガベース）なので、34 Mb × 60 = 2,040 Mb（約 2 Gb）あれば十分な深さ（60×）になります。
        
    - Filtlong は「長さ × 品質」の点数順で高得点リードを選んで、合計が 2 Gb になるまで絞り込んでくれます。
        
4. **入力ファイル**
    
    - フィルタ前の生データが入っているのは `Dicty_gDNA_NEB-2.fastq` のような FASTQ ファイルです。
        
    - ここから上記ルールでキレイなリードだけを抜き出して、新しい FASTQ（たとえば `Dicty_filtered.fastq`）に保存します。
        

---

**まとめると**、

- 「ある程度長くて」＋「ある程度精度が高い」リードだけを選び、
    
- 必要なだけ（60×相当）を残してカバーできればOK、
    
- 残りはアセンブリやポリッシングの邪魔になるので捨てよう、  
    というフィルタリング設定です。これで次のアセンブリ工程がスムーズになります！

下端が滑らかな曲線になっているのは、実は「リード長が長くなるほど平均クオリティは下がりやすい」というシーケンステクノロジー固有の性質が反映されているためです。具体的には：

- **エラー蓄積の影響**  
    ONTリードでは１塩基あたり一定の誤り率があるため、リードが長くなるほど「誤りを含む塩基の割合」が上がり、その結果として平均 Qスコアが下がっていきます。
    
- **平均値の“希釈”効果**  
    平均クオリティは全塩基のスコアの単純平均なので、長いリードほど低品質塩基の影響を受けやすく、短いリードでは高品質塩基だけで平均を押し上げやすい。
    
- **曲線状の下限分布**  
    各リード長における最小・最大の平均クオリティ分布を点でプロットすると、下限（error 多め）の値が滑らかに落ちる「カーブ」を描きます。これが散布図下端の曲線です。
    

要するに、「リードが長いほどエラーが累積して平均品質が非線形に低下する」ことが、下端のカーブとして見えている、というわけです。

---

## 🛠️ 実行例：Filtlongによるフィルタリング

```bash
filtlong \
  --min_length 3000 \
  --min_mean_q 12 \
  --target_bases 2000000000 \
  Dicty_gDNA_NEB-2.fastq \
  > Dicty_filtered.fastq
```

* `--min_length`: 3 kb未満のリードを除外（assemblyに不要）
* `--min_mean_q`: 平均Q12未満を除外（error-prone）
* `--target_bases`: 全体で約60×カバレッジを残すように自動選択（優先順位はスコアに従う）

🎯 **Filtlongは長くて高品質なリードを優先して選別**します。

---

## 📈 フィルタ後に期待される結果

| 指標   | フィルタ前   | フィルタ後（目安）            |
| ---- | ------- | -------------------- |
| 総塩基数 | 8.36 Gb | \~2.0 Gb             |
| リード数 | 934,886 | \~200,000 以下（≒高品質のみ） |
| 平均長  | \~9 kb  | より高くなる（スコア優先のため）     |
| 平均Q  | 13.8    | 上昇（低品質除去のため）         |

---

## 🧭 まとめ：戦略の意図

* **目標**：FlyeやRavenなどのアセンブラが**高精度な構造構築をしやすい**ように、良質なリードだけを残す
* **カバレッジ制限**は、**誤ったコンセンサスの形成を防ぐための“ノイズ除去”**
* この処理で、**assembly → polishing → BUSCO/QUAST評価**という流れの精度が最大化される

---

## ✅ Filtlong フィルタリングコマンド

```bash
filtlong \
  --min_length 3000 \
  --min_mean_q 12 \
  --target_bases 2000000000 \
  Dicty_gDNA_NEB-2.fastq \
  > Dicty_filtered.fastq
```

---

## 🔍 各オプションの意味

| オプション                       | 内容                                   |
| --------------------------- | ------------------------------------ |
| `--min_length 3000`         | 3,000 bp 未満のリードを除外（短すぎて構造構築に不利）      |
| `--min_mean_q 12`           | Q12 未満のリードを除外（誤り率 >6.3%）             |
| `--target_bases 2000000000` | 出力合計を2 Gb（= \~60×）に制限。スコア（長さ×品質）で選抜  |
| `Dicty_gDNA_NEB-2.fastq`    | 入力FASTQファイル                          |
| `>`                         | 高品質リードのみを `Dicty_filtered.fastq` に出力 |

---

以下は、**Filtlong によるフィルタリング後の NanoPlot 結果（`NanoPlot-report02.html`）** に基づく定量的評価と、**期待値との比較**です。

---

## ✅ フィルタ後 NanoPlot QC 結果（定量評価）

| 指標             | フィルタ後の結果       | フィルタ前     | 期待値（目標）        | 評価 | コメント                     |
| -------------- | -------------- | --------- | -------------- | -- | ------------------------ |
| **総リード数**      | **186,107**    | 934,886   | \~200,000 以下   | ✅  | 高品質なリードのみ選抜されている         |
| **総塩基数**       | **2.05 Gb**    | 8.36 Gb   | \~2.0 Gb（=60×） | ✅  | 目標カバレッジにぴったり             |
| **平均リード長**     | **11,036 bp**  | 8,941 bp  | 高いほどよい         | ✅  | 長くて構造決定に有利なリード中心に構成されている |
| **リードN50**     | **13,741 bp**  | 12,777 bp | >10 kb         | ✅  | より長いリードが選ばれている           |
| **最大リード長**     | **139,714 bp** | 同じ        | >50 kb         | ✅  | リピート領域をまたぐ能力あり           |
| **平均品質スコア**    | **14.7**       | 13.8      | ≥14 推奨         | ✅  | 精度向上。polishing効率も良化      |
| **Q>15 リード割合** | **66.3%**      | 44.2%     | ≥50%           | ✅  | Medaka要件を満たしている          |
| **Q>20 リード割合** | **2.5%**       | 1.0%      | ≥10%（理想）       | ❌  | ONT限界。Illumina併用で補正推奨    |

---

## 📌 フィルタ後の達成状況（まとめ）

| 評価軸            | 達成度                   | コメント                       |
| -------------- | --------------------- | -------------------------- |
| **量（カバレッジ）**   | ✅ **60×を維持**          | 無駄なくリード削減できた               |
| **リード長**       | ✅ **平均・N50ともに向上**     | アセンブリに有利な構造的長さ確保           |
| **品質スコア**      | ✅ **平均Q14.7、Q15超66%** | polishingに理想的な分布           |
| **超高品質（Q>20）** | ❌ **2.5%**            | ONT由来の制限、Illumina必須ではないが推奨 |

---

## ✅ 結論と今後の方針

🎯 **Filtlong によるフィルタリングは目標通りに成功**し、今後のアセンブリステップに最適なデータセットが得られています。

### 🔧 次ステップ

1. **アセンブリ（推奨：Flye）**

   * `--nano-raw Dicty_filtered.fastq` を入力に

2. **Polishing**

   * ONTのみ：`Medaka`
   * 可能ならIllumina併用 → `Pilon`, `NextPolish`

3. **評価**

   * `BUSCO`, `QUAST`, `dotplot` による構造評価と完全性チェック

---

## ## 定量的評価（フィルタ後）

|指標|フィルタ後の結果|目標値・基準|評価|
|---|---|---|---|
|**カバレッジ**|**≈ 60×**|ONTアセンブリは30–60×が最適69, 75([biostars.org](https://www.biostars.org/p/363651/?utm_source=chatgpt.com "Good coverage value for de novo assembly with NanoPore Reads?"), [royalsocietypublishing.org](https://royalsocietypublishing.org/doi/10.1098/rstb.2020.0160?utm_source=chatgpt.com "Benchmarking Oxford Nanopore read assemblers for high-quality ..."))|✅|
|**平均リード長**|**11,036 bp**|ONT理想5–15 kb、N50>10 kb76([gensoft.pasteur.fr](https://gensoft.pasteur.fr/docs/Flye/2.9/USAGE.html?utm_source=chatgpt.com "Flye manual"))|✅|
|**リードN50**|**13,741 bp**|Flye推奨N50>10 kb76([gensoft.pasteur.fr](https://gensoft.pasteur.fr/docs/Flye/2.9/USAGE.html?utm_source=chatgpt.com "Flye manual"))|✅|
|**最大リード長**|**139,714 bp**|>50 kbあればリピート越え可能87([nanoporetech.com](https://nanoporetech.com/applications/investigations/genome-assembly?utm_source=chatgpt.com "Assembly \| Oxford Nanopore Technologies"))|✅|
|**平均品質スコア（Q）**|**14.7**|MedakaではQ>15が理想だが最低Q12以上75([cloud-span.github.io](https://cloud-span.github.io/nerc-metagenomics04-polishing/aio.html?utm_source=chatgpt.com "Polishing: Aio - GitHub Pages"))|⭕|
|**Q>15 リード割合**|**66.3 %**|Medaka推奨：50 %超78([cloud-span.github.io](https://cloud-span.github.io/nerc-metagenomics04-polishing/aio.html?utm_source=chatgpt.com "Polishing: Aio - GitHub Pages"))|✅|
|**Q>20 リード割合**|**2.5 %**|ONT限界だが理想は10 %超89([reddit.com](https://www.reddit.com/r/bioinformatics/comments/1e9kpw6/help_with_bacteria_long_read_closed_genome/?utm_source=chatgpt.com "Help with bacteria long read closed genome assembly - Reddit"))|❌|

---

## 各指標の理由と参考文献

### 1. カバレッジ（量）

- **最適カバレッジ**：ONTアセンブリでは 30–60×が推奨され、乱れの少ない構造的正確性を得やすい79([biostars.org](https://www.biostars.org/p/363651/?utm_source=chatgpt.com "Good coverage value for de novo assembly with NanoPore Reads?"), [royalsocietypublishing.org](https://royalsocietypublishing.org/doi/10.1098/rstb.2020.0160?utm_source=chatgpt.com "Benchmarking Oxford Nanopore read assemblers for high-quality ..."))。
    
- **過剰カバレッジのリスク**：エラーの誤った合意やリピート誤接続、計算負荷増大の原因となる79([biostars.org](https://www.biostars.org/p/363651/?utm_source=chatgpt.com "Good coverage value for de novo assembly with NanoPore Reads?"))。
    

### 2. リード長（構造分解能）

- **平均長 5–15 kb**：Oxford Nanoporeのリード長として一般的な範囲で、10 kb以上あれば大半のリピート領域を越えやすい80([nanoporetech.com](https://nanoporetech.com/applications/investigations/genome-assembly?utm_source=chatgpt.com "Assembly | Oxford Nanopore Technologies"))。
    
- **N50>10 kb**：FlyeやRavenの初期disjointig構築に十分な長さとされる76([gensoft.pasteur.fr](https://gensoft.pasteur.fr/docs/Flye/2.9/USAGE.html?utm_source=chatgpt.com "Flye manual"))。
    

### 3. 品質スコア（精度）

- **平均Qスコア**：Q12（誤り率約6％）以上がリードフィルタリングの下限目安75([cloud-span.github.io](https://cloud-span.github.io/nerc-metagenomics04-polishing/aio.html?utm_source=chatgpt.com "Polishing: Aio - GitHub Pages"))。
    
- **Q>15割合**：Medakaがモデル化を安定させるために50 %以上を推奨78([cloud-span.github.io](https://cloud-span.github.io/nerc-metagenomics04-polishing/aio.html?utm_source=chatgpt.com "Polishing: Aio - GitHub Pages"))。
    
- **Q>20割合**：ONTフローセル技術の現状では10 %超は難しいため、Illumina併用で補正するのが一般的82([reddit.com](https://www.reddit.com/r/bioinformatics/comments/1e9kpw6/help_with_bacteria_long_read_closed_genome/?utm_source=chatgpt.com "Help with bacteria long read closed genome assembly - Reddit"))。
    

### 4. 長距離構造（最大リード）

- **最大リード長**：>50 kbのリードは大きなリピートや構造変異を正確に橋渡し可能88([nanoporetech.com](https://nanoporetech.com/applications/investigations/genome-assembly?utm_source=chatgpt.com "Assembly | Oxford Nanopore Technologies"))。
    

---

## 推奨フィルタリング設定（Filtlong）

```bash
filtlong \
  --min_length 3000 \
  --min_mean_q 12 \
  --target_bases 2000000000 \
  Dicty_gDNA_NEB-2.fastq \
  > Dicty_filtered.fastq
```

1. **--min_length 3000**  
    短いリードは誤ブリッジやノイズ原因となるため除去80([melbournebioinformatics.org.au](https://www.melbournebioinformatics.org.au/tutorials/tutorials/hybrid_assembly/nanopore_assembly/?utm_source=chatgpt.com "Hybrid genome assembly - Nanopore and Illumina"))。
    
2. **--min_mean_q 12**  
    誤り率約6％未満の安定したリードだけを残し、polishing効率を向上75([cloud-span.github.io](https://cloud-span.github.io/nerc-metagenomics04-polishing/aio.html?utm_source=chatgpt.com "Polishing: Aio - GitHub Pages"))。
    
3. **--target_bases 2000000000**  
    34 Mb×60×=2.04 Gbに合わせ、過剰カバレッジを回避79([biostars.org](https://www.biostars.org/p/363651/?utm_source=chatgpt.com "Good coverage value for de novo assembly with NanoPore Reads?"))。
    

---

## まとめ：この戦略の適切性

- **量・長さ・精度**の各指標がFlyeやMedakaの要件を満たし、**リピート解析も有利な長さ**を確保。
    
- **Illumina併用polishing**（Pilon, NextPolish）で、残ったエラーも十分に補正可能。
    
- **BUSCO Complete >95 %**、**Contig数≒染色体数**が達成できれば、染色体レベルのアセンブリとして十分な品質です。
    

このQC戦略は、Flye推奨のカバレッジ・長さ、Medakaのpolishing要件を満たしつつ、過剰データを適切に絞り込むことで計算リソースとアセンブリ精度を最適化する、**最善かつ適切な方法**と言えます。