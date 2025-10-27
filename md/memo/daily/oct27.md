ナイス！これは *Dictyostelium discoideum* の3Dゲノム構造をHi-C解析で明らかにした、非常に詳細な論文（おそらく NAR のやつ）ですね。
あなたがすでに Flye や Canu でアセンブリ済みのゲノム配列を持っているなら、ここで紹介されている **Hi-C解析パイプラインを自前のアセンブリに適用する**ことが可能です。

以下に、論文での方法をベースに「Flye/Canuアセンブリ結果にHi-Cデータを適用する流れ」を整理します👇

---

### 🧬 全体フロー

1. **自分のアセンブリをリファレンスとして使用**

   * Flye や Canu の出力（`assembly.fasta`）を、BWAやdistiller-nfでリファレンスとして指定します。
   * 論文では *AX4 strain reference genome* にマッピングしてますが、代わりにあなたの `assembly.fasta` を使います。

   ```bash
   bwa index assembly.fasta
   ```

---

2. **Hi-Cリードのマッピング**

   * 論文では `distiller-nf` パイプラインを使用しています（Hi-Cデータ処理を自動化）。
   * あなたも同様に使えます（Singularity/Dockerが必要）。
   * コマンド例：

   ```bash
   nextflow run mirnylab/distiller-nf \
       --bwa_index assembly.fasta \
       --reads 'hic_data/*_{R1,R2}.fastq.gz' \
       --outdir distiller_output \
       --reference_name my_assembly
   ```

   出力に `.pairs` や `.cool` ファイルが生成されます。

---

3. **ペア情報のフィルタリング**

   * `pairtools` を使って PCR duplicates や低品質マッピングを除去します。
   * 論文同様、MAPQ≥30を推奨。

   ```bash
   pairtools parse ... | pairtools select '(mapq1>=30) and (mapq2>=30)' | pairtools sort | pairtools dedup > filtered.pairs
   ```

---

4. **コンタクトマップ作成**

   * `.pairs` を `cooler` でバイニングして `.cool` ファイルを生成します。

   ```bash
   cooler cload pairs assembly.chrom.sizes:2000 filtered.pairs my_assembly_2kb.cool
   cooler balance my_assembly_2kb.cool
   ```

   → これで Hi-C マトリクス（各ゲノム領域間の相互作用頻度）が得られます。

---

5. **ループ・ドメイン検出**

   * 論文では “LASCA pipeline” を使用しています。
   * GitHub: [ArtemLuzhin/LASCA_pipeline](https://github.com/ArtemLuzhin/LASCA_pipeline)

   ```bash
   python3 Get_pvalue_v7.py -i my_assembly_2kb.cool -o lasca_output/
   ```

   → これでループ構造（loop anchorsやloop interiors）が検出されます。

---

6. **RNA-seqとの統合**

   * 論文と同じく、発現データ（TPMなど）をHi-Cループ内部と比較できます。
   * 例えば `TPM` vs `loop region` で統計解析すると、
     “ループ内部の方が発現が高い” という結果が得られるかもしれません。

---

### 💡Tips

* **アセンブリ品質チェック**：
  Hi-Cを適用する前に、`QUAST` や `BUSCO` で品質確認しておくと良いです。
* **Scaffoldの順序補正にもHi-Cが使える**：
  もしアセンブリがコンティグレベルなら、`3D-DNA` や `SALSA` を使ってHi-Cでスキャフォルディングすることも可能です。

---

もし今「Hi-Cリードデータ」も手元にあるなら、
次にやるべきステップは：

> ✅ Step 1: Flye/CanuアセンブリをBWAでindex
> ✅ Step 2: distiller-nfまたはJuicerでHi-Cマッピング
> ✅ Step 3: cooler/LASCAでloop構造を解析

---

 じゃあ、あなたのように **Flye/Canu両方のアセンブリ結果があって、Hi-CリードをNCBIから取得できる** 状況では、  
次のような実践的なパイプラインで「どちらのアセンブリが染色体構造により整合しているか」を比較できます👇

---

## 🧬 全体の流れ（実践版）

|ステップ|内容|主なツール|
|---|---|---|
|①|Hi-Cリード取得（NCBI SRA）|`prefetch`, `fasterq-dump`|
|②|Flye/Canuアセンブリをリファレンス化|`bwa index`|
|③|Hi-Cリードを各アセンブリにマッピング|`distiller-nf` or `Juicer`|
|④|コンティグ間の相互作用を可視化|`cooler`, `Juicebox`|
|⑤|Hi-Cベースのscaffold構築|`3D-DNA` or `SALSA`|
|⑥|Flye/Canu間のHi-C整合度比較|contact map / misjoin pattern|
|⑦|ポリッシュ → BUSCO / QUASTで評価|`pilon`, `busco`, `quast`|

---

## 🧩 ステップ詳細

### ✅ ① Hi-Cリードの取得（NCBI SRA）

Hi-C実験データは通常 `SRR` アクセッションとして登録されています。  
まずはダウンロード：

```bash
# 例: SRR12345678 が Hi-C データ
prefetch SRR12345678
fasterq-dump SRR12345678 --split-files -O hic_data/
```

結果として：

```
hic_data/SRR12345678_1.fastq
hic_data/SRR12345678_2.fastq
```

が得られます。

---

### ✅ ② Flye/Canuアセンブリをリファレンス化

```bash
bwa index flye_assembly.fasta
bwa index canu_assembly.fasta
```

---

### ✅ ③ Hi-Cリードをマッピング（例：distiller-nf）

```bash
nextflow run mirnylab/distiller-nf \
  --bwa_index flye_assembly.fasta \
  --reads 'hic_data/*_{1,2}.fastq.gz' \
  --reference_name flye \
  --outdir hic_flye_out \
  --nthreads 16
```

Canu版も同様に：

```bash
nextflow run mirnylab/distiller-nf \
  --bwa_index canu_assembly.fasta \
  --reads 'hic_data/*_{1,2}.fastq.gz' \
  --reference_name canu \
  --outdir hic_canu_out \
  --nthreads 16
```

これで `.pairs` や `.cool` ファイルが出力されます。

---

### ✅ ④ Hi-Cマップの可視化

`cooler` で 2kb～10kb のビン幅でバイニングします：

```bash
cooler cload pairs flye_assembly.chrom.sizes:5000 hic_flye_out/filtered.pairs flye_5kb.cool
cooler balance flye_5kb.cool
```

次に、`HiGlass` や `Juicebox` でマップを可視化すると：

- 染色体ごとに対角線がきれいに出ていれば → 正しい連結
    
- 斜線や交差があれば → misjoin の可能性
    

---

### ✅ ⑤ Hi-Cベースのスキャフォルディング

`3D-DNA` はHi-Cデータを用いた自動スキャフォルディングツールです。  
Flye/Canuアセンブリのどちらかを選び、Hi-Cで順序付けできます。

```bash
run-asm-pipeline.sh flye_assembly.fasta hic_flye_out/merged_nodups.txt
```

出力：

- `flye_assembly.final.assembly`（染色体スケールのscaffold）
    
- `.hic` ファイル（Juiceboxで可視化可能）
    

その後 `Juicebox` で手動修正もできます。

---

### ✅ ⑥ Flye vs Canu の整合性比較

Hi-Cコンタクトマップを比較すると：

|パターン|意味|
|---|---|
|対角線が途切れない|コンティグの正しい順序|
|クロスや断線|誤結合（misjoin）|
|飛び地状の点|誤って異なる染色体を接続している|

→ より滑らかで一貫した対角線が見える方が「Hi-C整合度が高いアセンブリ」。

---

### ✅ ⑦ 最終評価

Scaffold結果をポリッシュ＆評価。

```bash
# long read polishing
racon reads.fastq flye_assembly.final.assembly > polished.fasta
# short read polishing
pilon --genome polished.fasta --frags illumina.bam --output pilon_polished

# 評価
busco -i pilon_polished.fasta -l dictyostelium_db -m genome
quast -r reference.fasta pilon_polished.fasta
```

BUSCOで完全遺伝子割合、QUASTでN50やmisassembly数を比較すれば、  
Hi-C整合性＋構造的品質の両方が評価できます。

---

## 🧠 まとめ（Flye/Canu＋Hi-C比較の狙い）

|比較項目|Flye|Canu|
|---|---|---|
|N50・断片化度|やや長い傾向|より保守的|
|繰り返し解像度|強い（repeat graph）|低めだが堅実|
|Hi-C整合性|染色体の正しい順序・向き|誤join検出に強い|

**→ Hi-Cマップで一方が滑らかに対角線を示せば、それが最も信頼できる染色体スケールアセンブリ。**

---



# やること

- flyeとcanuで構造の違いが生まれた理由を調べる。どちらの方が正確なデータかをverifyする方法を探してできるだけ多く試す(hi-c)。
- scaffoldして最も正確なデータができるならそれを試す

https://academic.oup.com/nar/article/53/2/gkaf006/7973901?login=true#502628887


短く結論：**はい — Hi-C は「その反復領域がどの位置にあり、どのように周囲の配列と繋がっているか（大局的な配置や誤結合／挿入の有無）」を判定するのに有力です。ただし**

- Hi-C 単体では**反復ユニットの正確なコピー数**や細かな配列構造（どのリピートが何回並んでいるか）を高精度に決めるのは苦手、
    
- 反復領域はマッピング曖昧性を生みやすく、Hi-C の接触信号がアーチファクトになることもある、  
    という制約があります。したがって **Hi-C を使って「どちらのアセンブリが大局的に正しいか（misjoin／誤スキャフォルディング／挿入場所の違い）」を評価し、最終判断は長鎖リードのマッピング（split reads / coverage）や k-mer/optical map 等の追加証拠で裏付ける** のがベストです。
    

以下、実用的なワークフロー（やること／コマンド例／解釈のコツ）をステップごとに示します。

# 1) 全体戦略（要点）

- Hi-C：**位置と隣接関係**（例：「リピート領域がそのコンティグ内で孤立しているのか、別コンティグに強く接触するか」「境界にmisjoinの兆候があるか」）を検出。
    
- 長鎖リード（Nanopore/PacBio）：**コピー数の検証／スプリットアライメント（境界のread support）**の確認。
    
- depth/k-mer：**リピートのcollapse（コピー数不足）や過剰展開**の疑いを補助的に評価。
    
- （可能なら）Optical map / Bionano：リピート長の大局的なサイズ確認に強力。
    

# 2) Hi-C を用いた具体的手順（AX2 の Flye / Canu の両アセンブリで比較）

前提：`AX2_flye.fasta` と `AX2_canu.fasta`、AX4 の Hi-C FASTQ (`HiC_R1.fastq.gz`, `HiC_R2.fastq.gz`) があるとする。

### A. Hi-C を両方のアセンブリにマップして contact map を作る

推奨：Juicer → Juicebox の流れ（GUI で直感的に比べられる）。Juicer が重ければ pairtools + cooler + HiGlass / Juicebox でもOK。

簡易 pipeline（pairtools + bowtie2 / samtools / cooler）例：

```bash
# 1) index
bowtie2-build AX2_flye.fasta AX2_flye
bowtie2-build AX2_canu.fasta AX2_canu

# 2) map (例: flye)
bowtie2 --very-sensitive -x AX2_flye -1 HiC_R1.fastq.gz -2 HiC_R2.fastq.gz -S flye_hic.sam -p 12
samtools view -bS flye_hic.sam | samtools sort -o flye_hic.sorted.bam
samtools index flye_hic.sorted.bam

# 3) pairtools でペア作成・dedup
pairtools parse --chroms-path AX2_flye.chrom.sizes flye_hic.sorted.bam \
  | pairtools sort -o flye_pairs.sorted.bedpe
pairtools dedup flye_pairs.sorted.bedpe > flye_pairs.dedup.bedpe

# 4) cooler に入れて matrix 作成（例：10kb bin）
cooler cload pairs --assembly AX2_flye.chrom.sizes --chunksize 1000000 flye_pairs.dedup.bedpe AX2_flye_10kb.cool
```

同様に `AX2_canu.fasta` に対しても実行して比較用 `.cool` / `.hic` を得る。

**注意点**：反復領域はマルチマップを生むので、`pairtools` や mapping の段階で MAPQ でフィルタ (`samtools view -q 30`) するなど、ユニークマップのみで解析して「真の接触」を見やすくする。

### B. 可視化・観察

- Juicebox / HiGlass / HiCExplorer で Flye と Canu の contact map を並べて比較。
    
- 見るポイント：
    
    - その centromeric repeat 領域付近で主対角が途切れているか？ → misjoin の可能性
        
    - その領域の内部で異常に高いオフダイアゴナル信号（別染色体/別コンティグと強く接触）→ 位置の違い（転座／誤配置）やマルチマッピングの影響
        
    - 領域の境界で X 型や交差パターンがあるか → inversion のサイン
        

> 解釈ヒント：**もし Flye の assembly ではその領域が一続きのコンティグ内で接触が滑らかに減衰しているが、Canu では接触が途切れて別コンティグと強く接触する**—> Flye の方がその locus の大局的配置をより正しく表している可能性が高い（ただしマッピングの信頼度確認必須）。

# 3) Hi-C の限界とそれを補う検証（必ずやる）

Hi-C が示す「差」は 3 つの原因で生じうる：A) 本当の構造差、B) アセンブリ誤り、C) Hi-C マッピングアーチファクト。したがって下をやる：

### A. 長鎖リードマッピング（必須）

- `minimap2` で long reads を各アセンブリにマップし、**境界付近に split reads や chimeric alignment が存在するか**を確認。
    

```bash
minimap2 -ax map-ont AX2_flye.fasta longreads.fastq | samtools sort -o flye_long.bam
samtools index flye_long.bam

minimap2 -ax map-ont AX2_canu.fasta longreads.fastq | samtools sort -o canu_long.bam
samtools index canu_long.bam
```

- IGV で該当領域を見て、**多数のスプリットアライメントが境界を支持しているか**（→ assembly の誤り）を判断。逆に長鎖リードが綺麗に連続して並ぶならそのアセンブリが支持される。
    

### B. coverage（コピー数のチェック）

- `samtools depth` で該当領域のcoverageを比較。大きく低い／高い差があれば collapse/expansion の示唆。
    

```bash
samtools depth -a flye_long.bam | awk '{sum[$1]+=$3;cnt[$1]++} END{for(c in sum) print c, sum[c]/cnt[c]}'  # per-chrom coverage簡易
```

（より正確には region ベースで平均 depth を取る）

### C. assembly graph と dotplot

- リピートがどのようにつながっているかは assembly graph（例えば Flye の `assembly_graph.gfa`）や `Bandage` で可視化すると分かる。Canu も似た情報を吐く。
    
- `nucmer` / `mummerplot` で周辺領域の dotplot を作り、反復の整列状態を見る。
    

```bash
nucmer --prefix=region_compare flye_region.fasta canu_region.fasta
mummerplot --png region_compare.delta
```

### D. k-mer / Merqury / KAT による copy number の推定（Illumina がある場合）

- Illumina short reads（もしある）を使い k-mer 分布から反復の copy 数が assembly 中で反映されているか（collapse していないか）を評価。Merqury や KAT を使う。
    

### E. （オプション）Optical map / Bionano

- 可能なら Bionano 等の物理マップで反復領域の長さを確認できると非常に説得力がある。
    

# 4) 実用的な判断ルール（まとめ）

- **Hi-C が示す「大局的接続（隣接関係／misjoin）」** + **長鎖リードの split/支持** が一致すれば、そのアセンブリが正しいと判断して良い。
    
- **Hi-C だけで copy number（ユニット数）を確定するのは不可** → depth / k-mer / optical map で補う。
    
- 反復領域はマッピング曖昧性を生むため、**MAPQ フィルタやユニークマップのみで再解析**してアーティファクトを除外してから解釈する。
    

# 5) 具体的に私が今できること（あなたが欲しければ）

- あなたの `AX2_flye.fasta` / `AX2_canu.fasta` / Hi-C FASTQ 名 / longreads ファイル名 を教えてもらえれば、上の pipeline をあなたのファイル名に合わせて具体コマンド列（Juicer 用 or pairtools+cooler 用）と、観察すべき Juicebox のスクリーンショット例（どのパターンを misjoin と読むか）を作ります。
    
- もしくは、あなたが既に作った `pairs.dedup.bedpe` や `.hic`、または longread の BAM を共有（要ダウンロード可能な場所のURL）してくれれば、観察ポイントを具体的に解釈します。
    
