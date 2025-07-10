以下に、scaffolding後のアセンブリに RepeatMasker を適用する**目的**と、適用することで**期待できる効果**をまとめます。要約としては、RepeatMasker によってリピート領域を正確に検出・マスクし、下流の解析（アノテーション、アラインメント、構造変異検出、品質評価など）でのアーティファクトを低減させることが主目的です。その結果、偽陽性の遺伝子予測や誤ったアライメント、構造変異の誤検出を減らし、解析精度と信頼性を大きく向上させることが期待されます。

## 1. 目的 (Purpose)

### 1.1 リピート領域の特定とマスク

RepeatMasker は、ゲノム中のインタースパーストリピートや低複雑性配列を検出し、N 置換（ハードマスク）や小文字化（ソフトマスク）を行うツールです ([training.galaxyproject.org][1])。
マスク処理により、リピート配列が後続の解析で誤って信号として扱われるのを防止します ([Biostars][2])。

### 1.2 アノテーションパイプラインの前処理

遺伝子予測ツール（e.g. Augustus, GENSCAN）は、トランスポゾンの ORF をしばしば「本来の遺伝子」と誤認します ([Biostars][2])。
事前にリピートをマスクすることで、偽陽性の遺伝子モデル生成を大幅に抑制し、アノテーションの正確性を向上させます ([PMC][3])。

### 1.3 アライメントや比較解析の精度向上

複数ゲノム間比較やリードのマッピングでは、マスクされていないリピートが数万件のスプリアスアライメントを生む原因になります ([PMC][4])。
RepeatMasker を適用することで、マッピングエラーを減らし、真の相同性領域の検出感度を高めます ([ACS ESS][5])。

## 2. 期待できる効果 (Expected Effects)

### 2.1 遺伝子予測の精度向上

* **偽陽性削減**：トランスポゾン配列由来の ORF が誤ってスキャンされるのを防ぎ、真の遺伝子構造予測の信頼性を高める ([Biostars][2])。
* **アノテーション時間短縮**：余分な候補領域が減ることで、予測ツールの計算負荷が軽減します ([biostar.galaxyproject.org][6])。

### 2.2 比較ゲノム解析・構造変異検出の改善

* **真の変異検出**：リピート由来の誤検出（偽陽性転座や逆位）が減り、SyRI や MUMmer 等による構造変異解析の精度が向上します ([Nature][7])。
* **多重マッピング回避**：独自コンセンサスに基づくリピートブラウザなどでも、複数箇所への非特異的マッピングを抑制できます ([mobilednajournal.biomedcentral.com][8])。

### 2.3 アセンブリ品質評価の向上

* **QUAST 等での評価精度**：リピートに起因するギャップや重複によるアーティファクトを取り除き、実際のギャップ数や N50 の評価をより正確に反映 ([training.galaxyproject.org][1])。
* **BUSCO スコアの信頼度**：リピート混入による予測欠損を防ぎ、真の遺伝子被覆率を評価できます ([PMC][3])。

### 2.4 下流データベースの品質向上

* **公開データセットの準備**：UCSC Genome Browser などで公開する場合、マスクデータ付きのリリースが標準となっており、利用者側の解析負荷を軽減 ([biostar.galaxyproject.org][6])。
* **二次解析への対応**：変異検出、転写産物解析、エピジェネティック解析など、多彩な下流解析のベースとして、意図せぬリピートノイズを排除したクリーンなゲノム配列が得られます ([docs.omicsbox.biobam.com][9])。

---

以上を踏まえ、Flye や Canu の scaffolding 後アセンブリに RepeatMasker を適用することは、アセンブリ評価から遺伝子予測、構造変異解析、さらにはデータ共有に至るまで、ゲノム解析パイプライン全体の精度・効率を大きく向上させる重要な前処理ステップです。

[1]: https://training.galaxyproject.org/training-material/topics/genome-annotation/tutorials/repeatmasker/tutorial.html?utm_source=chatgpt.com "Genome Annotation / Masking repeats with RepeatMasker / Hands-on"
[2]: https://www.biostars.org/p/169981/?utm_source=chatgpt.com "how do I run repeat masker - Biostars"
[3]: https://pmc.ncbi.nlm.nih.gov/articles/PMC7565776/?utm_source=chatgpt.com "Review on the Computational Genome Annotation of Sequences ..."
[4]: https://pmc.ncbi.nlm.nih.gov/articles/PMC11185745/?utm_source=chatgpt.com "ULTRA-Effective Labeling of Repetitive Genomic Sequence - PMC"
[5]: https://acsess.onlinelibrary.wiley.com/doi/pdf/10.1002/tpg2.20204?utm_source=chatgpt.com "A multiple alignment workflow shows the effect of repeat masking ..."
[6]: https://biostar.galaxyproject.org/p/25470/index.html?utm_source=chatgpt.com "when is repeatmasker used in data analysis pipelines? - Galaxy Help"
[7]: https://www.nature.com/articles/s42003-023-05322-y?utm_source=chatgpt.com "Repetitive DNA sequence detection and its role in the human genome"
[8]: https://mobilednajournal.biomedcentral.com/articles/10.1186/s13100-020-00208-w?utm_source=chatgpt.com "The UCSC repeat browser allows discovery and visualization of ..."
[9]: https://docs.omicsbox.biobam.com/latest/Repeat-Masking/?utm_source=chatgpt.com "Repeat Masking - OmicsBox User Manual - BioBam"
