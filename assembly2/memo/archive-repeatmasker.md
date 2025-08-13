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




以下のような手順で、`ragtag_medaka_polished.fasta`（Flye‐scaffold）と `canu_medaka_polished.fasta`（Canu‐scaffold）の “どこが違うか” を調べられます。代表的な方法を２つ挙げます。

---

## 方法① MUMmer（NUCmer + show-coords / show-snps）

1. **アライメント作成**
    
    ```bash
    # ディレクトリはご自身のパスに合わせてください
    nucmer --maxmatch -p flye_vs_canu \
      assembly2/assembly-results/ragtag_flye_scaffold/ragtag_medaka_polished.fasta \
      assembly2/assembly-results/ragtag_canu_scaffold/canu_medaka_polished.fasta
    ```
    
2. **相違座標の出力**
    
    ```bash
    # アライメント結果の座標リスト（大きなギャップや未整列域）を確認
    show-coords -rcl flye_vs_canu.delta > flye_vs_canu.coords.txt
    
    # SNP／小さいインデルを一覧化
    show-snps -Clr flye_vs_canu.delta > flye_vs_canu.snps.txt
    ```
    
    - `flye_vs_canu.coords.txt`…整列領域やギャップ長が出力されるので、“Flye にはないが Canu にある大きなギャップ” などが一目でわかります。
        
    - `flye_vs_canu.snps.txt`…塩基レベルの違い（SNP/インデル）を詳細にリストアップします。
        
3. **結果の確認**
    
    - `coords.txt` のうち、`%IDY`（一致度）が低い行や、`Len2`（対象配列のギャップ長）が大きい行を grep してみると、差分領域が絞り込めます。
        
    - `snps.txt` はタブ区切りなので、R や Python、Excel などでフィルタリング・集計してもよいでしょう。
        

---

## 方法② Minimap2 + dotplot / bedtools

1. **Minimap2 でマッピング**
    
    ```bash
    minimap2 -t 16 -ax asm5 \
      assembly2/assembly-results/ragtag_flye_scaffold/ragtag_medaka_polished.fasta \
      assembly2/assembly-results/ragtag_canu_scaffold/canu_medaka_polished.fasta \
      > canu_to_flye.sam
    
    samtools view -bS canu_to_flye.sam \
      | samtools sort -o canu_to_flye.bam
    samtools index canu_to_flye.bam
    ```
    
2. **カバレッジ差分の検出**
    
    ```bash
    # Flye に整列しない（Canu 特有の）領域を抽出
    bedtools genomecov -ibam canu_to_flye.bam -bga \
      | awk '$4==0' > canu_unique_regions.bed
    
    # その配列をFASTAで取り出す
    bedtools getfasta \
      -fi assembly2/assembly-results/ragtag_canu_scaffold/canu_medaka_polished.fasta \
      -bed canu_unique_regions.bed \
      -fo canu_only_sequences.fa
    ```
    
    こうすると、Canu スキャフォールドにだけ含まれるシーケンス塊が抜き出せます。
    
3. **dotplot で可視化（オプション）**
    
    ```bash
    # paftools を使った dotplot
    minimap2 -t 16 -x asm5 \
      assembly2/assembly-results/ragtag_flye_scaffold/ragtag_medaka_polished.fasta \
      assembly2/assembly-results/ragtag_canu_scaffold/canu_medaka_polished.fasta \
      | paftools.js dotplot - > flye_vs_canu.dotplot.svg
    ```
    
    大きな直線ギャップや倒立　（inversion）など、構造変異の有無が一目でわかります。
    

---

### Tips

- **dnadiff**（MUMmer のラッパー）を使うと、`report.html` やサマリーが自動生成されて便利です。
    
    ```bash
    dnadiff -p diff_report \
      assembly2/assembly-results/ragtag_flye_scaffold/ragtag_medaka_polished.fasta \
      assembly2/assembly-results/ragtag_canu_scaffold/canu_medaka_polished.fasta
    ```
    
- **QUAST** で両方をまとめて評価すれば、欠落率や不整合位置をレポートしてくれます（参照ゲノムがある場合は `-R` オプションも）。
    
    ```bash
    quast.py \
      -o quast_diff \
      assembly2/assembly-results/ragtag_flye_scaffold/ragtag_medaka_polished.fasta \
      assembly2/assembly-results/ragtag_canu_scaffold/canu_medaka_polished.fasta
    ```
    

――以上を使えば、Flye vs Canu の polished スキャフォールド間で「どの領域が増減しているのか」「どの塩基で差があるのか」を詳細に特定できます。


以下、それぞれワンライナーで流し込める例です。パスは適宜読み替えてください。

```bash
# 方法① MUMmer (NUCmer→show-coords→show-snps)
nucmer --maxmatch -p flye_vs_canu \
  assembly2/assembly-results/ragtag_flye_scaffold/ragtag_medaka_polished.fasta \
  assembly2/assembly-results/ragtag_canu_scaffold/canu_medaka_polished.fasta && \
show-coords -rcl flye_vs_canu.delta > flye_vs_canu.coords.txt && \
show-snps -Clr flye_vs_canu.delta > flye_vs_canu.snps.txt
```

```bash
# 方法② Minimap2→samtools→bedtools (差分領域抽出)
minimap2 -t16 -ax asm5 \
  assembly2/assembly-results/ragtag_flye_scaffold/ragtag_medaka_polished.fasta \
  assembly2/assembly-results/ragtag_canu_scaffold/canu_medaka_polished.fasta | \
samtools view -bS - | samtools sort -o canu_to_flye.bam && \
samtools index canu_to_flye.bam && \
bedtools genomecov -ibam canu_to_flye.bam -bga | awk '$4==0' > canu_unique_regions.bed && \
bedtools getfasta \
  -fi assembly2/assembly-results/ragtag_canu_scaffold/canu_medaka_polished.fasta \
  -bed canu_unique_regions.bed \
  -fo canu_only_sequences.fa
```

この “完全に未マップ領域”232 kb の抽出結果からわかること、そしてそれをどう活かせるかを整理します。

---

## 1. 評価ポイント

### A. 完全未マップ領域（232 kb）の意義

- **アセンブリ間の「まったく重ならない」シーケンス量**  
    → Flye スキャフォールドには存在せず、Canu スキャフォールドにだけ含まれる配列
    
- **232 kb** という数字は、「アセンブリ間の構造的な差分」のうち、真に新規／消失した（あるいは極端に異なる）領域の大きさを示す
    
- これが妥当かどうかは、
    
    - 参照ゲノム中のギャップや未解決リピート領域の長さ
        
    - Genic／Intergenic 領域の重要度  
        などと比較して検討します。
        

### B. 小規模差分（約900 kb相当の indel・リピート長変動）

- “完全未マップ領域” 以外に、SNP／小インデルやリピート解像の違いが ~1.1 Mb まで総長差を増加させている
    
- これらはベースエラーやツールのアルゴリズム差（リピート展開のしきい値など）によるものと考えられ、
    
    - **Polishing** の不足
        
    - **リピート領域の扱い**  
        を改善ポイントとして挙げられます。
        

---

## 2. どう役立つか

1. **構造変異・挿入配列の同定**
    
    - 232 kb のユニーク領域を遺伝子アノテーションして、「特定の遺伝子／転写産物」が欠落・重複していないかをチェック
        
    - プラスミドやゲノム挿入シーケンスなど、バクテリアであればモバイルエレメント探索にも利用
        
2. **アセンブリ品質改善の指標**
    
    - “真に異なる”232 kb に対して再マッピング・再ポリッシュを行い、
        
        - ミスアセンブル（Misjoin）の有無
            
        - リピート領域の再展開  
            を試みる
            
3. **リピート・インデル差分の定量**
    
    - `show-snps` や QUAST のレポートと合わせ、小規模 indel の合計長を算出 → ポリッシュラウンドの効果測定に活用
        
    - 「どの contig／領域にエラーが集中しているか」から、特定 contig の再アセンブル優先度を決定
        
4. **下流解析へのブレークダウン**
    
    - 抽出した FASTA を BLAST して “何がそこにあるか” を同定
        
    - Annotation pipeline（Prokka など）で ORF を予測し、
        
        - コーディング領域か
            
        - 非コーディング領域か  
            を判断  
            → データ解釈（例：病原性遺伝子の有無、代謝経路の違い）に直結
            

---

### ▶︎ 次のステップ例

1. **BLAST/Prokka 解析** でユニーク領域の機能を解明
    
2. **再ポリッシュ or 再スキャフォールド** で misjoin の修正
    
3. **リピート領域の特定ツール**（RepeatModeler, TAREAN など）を走らせ、
    
    - リピートタイプごとの差分を把握
        
    - Assembly graph の可視化（Bandage など）で構造異常を確認
        

これらにより、Flye vs Canu の差分が「単なるツール間のパラメータ差」なのか「生物学的に意義ある構造差」なのかを切り分け、ゲノムアセンブリの最終品質をさらに高められます。