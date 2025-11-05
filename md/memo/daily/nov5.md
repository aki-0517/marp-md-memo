```bash
delta-filter -m ${PREFIX}.delta > ${PREFIX}.m-to-m.filter.delta && \
show-coords -THrd ${PREFIX}.m-to-m.filter.delta > ${PREFIX}.m-to-m.coords && \
syri -c ${PREFIX}.m-to-m.coords \
     -d ${PREFIX}.m-to-m.filter.delta \
     -r ${REF} \
     -q ${QRY} \
     --prefix ${OUT_PREFIX} --no-chrmatch && \
plotsr --sr ${OUT_PREFIX}syri.out \ --genomes genomes.txt \ -H 10 -W 15 \ -o flye_vs_canu_plot_mtom.pdf
```

を行った。


canuとflyeの２つのアセンブリ結果がある。その２つをシンテニーを調べたら、反復領域のところにシンテニーがなかった。
nanoporeデータでmappingを調べると、flyeの反復領域の中の真ん中の少しの範囲がmappingが欠損してた。
これが原因で、その欠損している部分だけでなく、シンテニーがあるその周りの反復領域全てのシンテニーがないことになってしまうことはある？
使ったこのソフトウェアのアルゴリズムや仕組みをもとに、根拠を上げながら詳しく




---
---
はい — 結論を先に短く書くと：

**「はい。Flye の反復領域の中で一部がマッピング欠損している（アセンブリが欠けている／崩れている）と、その小さな欠損が原因で SyRI がその領域の周囲にある（本来はシンテニックな）反復領域全体を『シンテニー無し／非共線（rearranged/un-aligned）』として扱ってしまうことは十分にあり得ます。」** ([PMC](https://pmc.ncbi.nlm.nih.gov/articles/PMC6913012/?utm_source=chatgpt.com "SyRI: finding genomic rearrangements and local sequence ..."))

以下、なぜそうなるのかを使ったソフトウェア（NUCmer/MUMmer の出力→delta-filter→show-coords→SyRI→plotsr）のアルゴリズム的仕組みと根拠、発生しやすい具体的なケース、そして検証・対処法を順を追って詳しく説明します。重要点には該当ソースを付けます。

---

# 1) 仕組み：SyRI が「シンテニック」をどう決めるか（要点）

- SyRI は **全ゲノムアライメント（WGA）から“共線（co-linear）な経路（syntenic path）”をまず見つけ、残りを“再配列（rearranged）/未整列” と決める**方式です。つまり「連続した（もしくは接続可能な）整列ブロックの集合」を探してそれがシンテニーと見なされます。小さなギャップや断片化があると“連続性”が壊れ、そこから周囲が再配列扱いになることがあります。([PMC](https://pmc.ncbi.nlm.nih.gov/articles/PMC6913012/?utm_source=chatgpt.com "SyRI: finding genomic rearrangements and local sequence ..."))
    
- SyRI は WGA（NUCmer 等の .delta / coords や PAF）を入力に取るため、**WGA の出力にあったアライメントの“分断”や“欠落”がそのまま SyRI の解析グラフに反映**されます（SyRI 自体はアライメントを補完して新しい短いマッチを生成するような魔法はしません）。([PMC](https://pmc.ncbi.nlm.nih.gov/articles/PMC6913012/?utm_source=chatgpt.com "SyRI: finding genomic rearrangements and local sequence ..."))
    

---

# 2) どうして「部分的欠損（真ん中の小さい範囲）」が周囲全部のシンテニー消失に波及するのか（具体的メカニズム）

1. **反復領域は多重ヒットを生む（複数の候補アライメント）**
    
    - 反復配列では参照・クエリの複数箇所にほぼ同じ配列が存在するため、WGA（NUCmer）が複数の等価なアライメント候補を出します。どの候補を“優先”するかで、得られる連続パスが変わります。([toolshed.g2.bx.psu.edu](https://toolshed.g2.bx.psu.edu/repository/display_tool?changeset_revision=6753195df9e0&render_repository_actions_for=tool_shed&repository_id=4c8d49ef284b7a1f&tool_config=database%2Fcommunity_files%2F000%2Frepo_46%2FMUMmer%2Fmummer_utilities_tool.xml&utm_source=chatgpt.com "MUMmer utilities"))
        
2. **小さな欠損が“連続性”を断ち切る**
    
    - 例えば Flye のアセンブリ上で反復の中間に数百〜数kbの欠損があった場合、WGA 上ではその領域に対応する長い連続アライメントが**切れて**短い断片に分かれるか、あるいはまったくアライメントが出てこない（アラインしない）ブロックができます。SyRI は“長くコリニア（連続）な経路”を探すため、その断片化で **その領域をシンテニー候補パスから外してしまう**可能性が高くなります。([PMC](https://pmc.ncbi.nlm.nih.gov/articles/PMC6913012/?utm_source=chatgpt.com "SyRI: finding genomic rearrangements and local sequence ..."))
        
3. **フィルタ処理（delta-filter や show-coords）で代替アライメントが落ちる**
    
    - 実行しているパイプラインで `delta-filter -m` を使う／その他で「best-only」「unique-only」的なフィルタをかけると、**反復での“補助的に連結できる”アライメント候補が消される**ことがあります（最良候補だけ残すと、欠損を跨ぐ別解が不要になり、結果的に大きな区間が“未整列”扱いになる）。MUMmer の delta-filter は長さ・一意性・consistency などでフィルタするため、この影響を受けます。([toolshed.g2.bx.psu.edu](https://toolshed.g2.bx.psu.edu/repository/display_tool?changeset_revision=6753195df9e0&render_repository_actions_for=tool_shed&repository_id=4c8d49ef284b7a1f&tool_config=database%2Fcommunity_files%2F000%2Frepo_46%2FMUMmer%2Fmummer_utilities_tool.xml&utm_source=chatgpt.com "MUMmer utilities"))
        
4. **局所的ギャップがグラフ構造を複雑化 → SyRI は保守的に扱う**
    
    - SyRI は genome graph 上でコリニア経路を最適化／抽出します。反復領域の多重マッチ＋小さなギャップは“複雑なネットワーク”になり、SyRI はそこを再配列／重複扱いとして分類したり、未整列として扱ったりしがちです（論文でその方法論が説明されています）。([PMC](https://pmc.ncbi.nlm.nih.gov/articles/PMC6913012/?utm_source=chatgpt.com "SyRI: finding genomic rearrangements and local sequence ..."))
        

まとめると、**小さな欠損が「連続した一致」を断ち切り、さらにフィルタ段階で橋渡しするような補助アラインメントが捨てられると、その周りの反復領域全体が“シンテニー無し”になる** — という流れです。これがあなたの観察（Flye の反復中心でマッピング欠損 → その領域周辺のシンテニーがない）と整合します。([PMC](https://pmc.ncbi.nlm.nih.gov/articles/PMC6913012/?utm_source=chatgpt.com "SyRI: finding genomic rearrangements and local sequence ..."))

---

# 3) 「マッピング欠損」が意味する生物学的/技術的原因（nanopore mapping を踏まえて）

- Nanopore リードを Flye アセンブリにマップして中央が欠損しているなら、**Flye 側でその短区間が欠けている（欠失）、あるいは誤組立て（collapse）されている可能性**が高いです。リードがそこにマップしない＝シーケンスが存在しない、または配列がかなり変化している。
    
- 反復は長さ差やコピー数差を生みやすく、長尺リードでもマッピングが難しいケースがあります（特に低カバレッジ・高エラー率の領域）。これがアセンブリ間のズレと合わさると WGA が断片化します。([PMC](https://pmc.ncbi.nlm.nih.gov/articles/PMC6913012/?utm_source=chatgpt.com "SyRI: finding genomic rearrangements and local sequence ..."))
    

---

# 4) delta-filter/show-coords 側の影響（あなたのコマンドで該当しそうな点）

- あなたのコマンド列では `delta-filter -m` を使っています。delta-filter は **“out of .delta file を条件でフィルタ”** するツールで、`-m` 等のオプションは“複数マッチの扱い／ベストマッチの選択”に影響します。フィルタで“最良の”対にならないアライメントが落ちると、SyRI に渡る WGA 情報が簡略化され、結果的に反復をブリッジする別解が消えてしまうことがあります。([toolshed.g2.bx.psu.edu](https://toolshed.g2.bx.psu.edu/repository/display_tool?changeset_revision=6753195df9e0&render_repository_actions_for=tool_shed&repository_id=4c8d49ef284b7a1f&tool_config=database%2Fcommunity_files%2F000%2Frepo_46%2FMUMmer%2Fmummer_utilities_tool.xml&utm_source=chatgpt.com "MUMmer utilities"))
    
- `show-coords -THrd` で生成する coords は “要約” なので、**細かいサブアライメントの情報（断片化された多くの小アライメント）が見えにくくなる**ことがあります。元の .delta を直接見たり、`show-aligns` や `--coords -B` 相当の出力で詳細確認すると良いです。([mummer4.github.io](https://mummer4.github.io/manual/manual.html?utm_source=chatgpt.com "Running MUMmer4.x"))
    

---

# 5) 検証方法（実際に確かめるべきコマンドや手順）

以下を順にやることで「欠損が本当にアセンブリ由来か、フィルタの振る舞いか」を切り分けられます。コマンド例／意図付きで示します。

1. **生のアライメント（.delta）を詳細に見る**
    
    - `.delta` を `show-aligns` で開くか、`show-coords` を最小フィルタで出力して断片化の様子を確認。  
        例：`show-coords -rclTH ${PREFIX}.m-to-m.filter.delta > ${PREFIX}.all.coords`（既存オプションの組合せで）｡([mummer4.github.io](https://mummer4.github.io/manual/manual.html?utm_source=chatgpt.com "Running MUMmer4.x"))
        
2. **delta-filter のフィルタを変えてみる（-m を外す / -1 を試す / identity/length閾値を変更）**
    
    - 例：`delta-filter -1`（one-to-one best matches にする）や、逆にフィルタを弱めて全アライメントを SyRI に渡す。これで SyRI のシンテニー判定が変わるか確認。`-1` は“一対一（best）”にするオプション（MUMmer 系でよく使われる）。([mummer4.github.io](https://mummer4.github.io/manual/manual.html?utm_source=chatgpt.com "Running MUMmer4.x"))
        
3. **NUCmer を `--maxmatch` で再実行して全ての短い一致を拾う**
    
    - 反復領域では `--maxmatch`（全てのマッチを検出）にしておくと代替マッチが出やすく、SyRI に渡る情報が豊富になります。例：`nucmer --maxmatch -l 100 -c 100 -p prefix ref.fa qry.fa`。その後 delta-filter の挙動を比較。([mummer4.github.io](https://mummer4.github.io/tutorial/tutorial.html?utm_source=chatgpt.com "2. Examples - MUMmer"))
        
4. **Flye アセンブリに対して長リード（nanopore）を再マップしてカバレッジを可視化**
    
    - `minimap2` でマップして `samtools depth` / IGV 等で中心部のカバレッジ低下を確認。カバレッジが本当に落ちているならアセンブリ欠失の可能性が高い。  
        例：`minimap2 -ax map-ont flye.fa reads.fastq | samtools sort -o aln.bam` → `samtools depth -a aln.bam | awk '$3<…'`。（具体閾値はデータ依存）｡
        
5. **SyRI に PAF 入力（minimap2 の PAF）で入れてみる**
    
    - SyRI は PAF も読めます（`-F P` オプション）。minimap2 の PAF を入れると、NUCmer とは違うアライメント戦略による結果が得られ、判断材料になります。([GitHub](https://github.com/schneebergerlab/syri "GitHub - schneebergerlab/syri: Synteny and Rearrangement Identifier"))
        
6. **元の .delta を可視化（dotplot）して、どの範囲で断裂が起きているかを確認**
    
    - `mummerplot` や `plotsr`（syri.out を受け取ってプロット）で該当染色体・領域を拡大表示。plotsr は SyRI の out を受けて綺麗に描画できます。([Schneeberger Lab](https://schneebergerlab.github.io/syri/plotsr.html?utm_source=chatgpt.com "Plotting genomic structure using plotsr | syri - GitHub Pages"))
        
7. **Flye のアセンブリグラフを確認**（可能なら）
    
    - Flye は assembly graph 情報を出すので（`assembly_graph.gfa` 等）、GFA を見てその反復領域が collapse しているかどうか確認できます。collapse ならコピー数の不一致や未解決分岐が見えるはず。
        

---

# 6) 対処・解析のための実践的アドバイス

- **最初にやること（低コスト）**
    
    1. 当該領域の `.delta` と `show-coords`（フィルタ無し）の原本を抜き出して、断片化の詳細を確認。
        
    2. Nanopore リードのカバレッジプロットでそこが本当に“欠損”しているか確認。欠損＝アセンブリの問題。カバレッジ低下であれば、Flye の該当 contig の該当範囲を手で検査（consensus が短くなっている等）する。([mummer4.github.io](https://mummer4.github.io/manual/manual.html?utm_source=chatgpt.com "Running MUMmer4.x"))
        
- **次に試す（解析パイプライン調整）**
    
    - `nucmer --maxmatch` → delta-filter を弱める（`-m` を外す／`-1` を試す）→ SyRI を再実行。これでSyRIが“本来のシンテニック経路”を再認識するか確認。([mummer4.github.io](https://mummer4.github.io/tutorial/tutorial.html?utm_source=chatgpt.com "2. Examples - MUMmer"))
        
- **もし Flye が短区間を本当に欠いているなら**
    
    - Flye を再アセンブル（異なるパラメータ）や、Polishing（短リードで Pilon 等）で修正、あるいはハイブリッドアセンブルや長リードの追加で欠損を埋める。補間できない場合は SyRI の結果を「この領域は assembly-level の欠損が原因である」と注釈して扱うのが現実的。([PMC](https://pmc.ncbi.nlm.nih.gov/articles/PMC6913012/?utm_source=chatgpt.com "SyRI: finding genomic rearrangements and local sequence ..."))
        
- **SyRI のパラメータ**
    
    - SyRI にも `--invgaplen` や `--maxsize` 等、検出の閾値を変えるパラメータがあります（GitHub README）。反復や小断裂の扱いをチューニングしてみてください。([GitHub](https://github.com/schneebergerlab/syri "GitHub - schneebergerlab/syri: Synteny and Rearrangement Identifier"))
        

---

# 7) 具体的チェックリスト（あなたがすぐ実行できる短い順番）

1. （すぐ）Nanopore → Flye の当該 contig に再マップして `samtools depth` でカバレッジ確認。
    
2. （すぐ）元の `.delta`（フィルタ前）を `show-coords` / `show-aligns` で調べ、該当領域でアライメントが分断されていないか確認。
    
3. （次）`nucmer --maxmatch` で再アライン → delta-filter のフィルタを弱めて SyRI 再実行。違いを比較。
    
4. （次）SyRI を `-F P` で minimap2 の PAF 入力でも動かしてみる（アライナ選択の差を見る）。
    
5. plotsr / dotplot で当該領域を拡大して可視化（どの範囲が未整列になっているかを把握）。([mummer4.github.io](https://mummer4.github.io/tutorial/tutorial.html?utm_source=chatgpt.com "2. Examples - MUMmer"))
    

---

# 参考（主要ソース）

- SyRI 論文・概要（アルゴリズムの説明）。([PMC](https://pmc.ncbi.nlm.nih.gov/articles/PMC6913012/?utm_source=chatgpt.com "SyRI: finding genomic rearrangements and local sequence ..."))
    
- SyRI GitHub README（パラメータや既知の注意点、PAF 対応など）。([GitHub](https://github.com/schneebergerlab/syri "GitHub - schneebergerlab/syri: Synteny and Rearrangement Identifier"))
    
- MUMmer / NUCmer / show-coords / delta-filter のマニュアル（.delta のフィルタ動作と show-coords の用途）。([mummer4.github.io](https://mummer4.github.io/manual/manual.html?utm_source=chatgpt.com "Running MUMmer4.x"))
    
- plotsr（SyRI の出力を可視化するツール、zoom 描画に便利）。([GitHub](https://github.com/schneebergerlab/plotsr?utm_source=chatgpt.com "schneebergerlab/plotsr: Tool to plot synteny and structural ..."))
    

---

# 最後に（まとめ）

- 小さなマッピング欠損が **WGA の断片化 → フィルタで橋渡しアラインメントが消える → SyRI の syntenic path が切れる** 流れで、周囲の反復領域までシンテニーが消えるのは**十分に起こり得る**現象です。まずは（A）マッピング（カバレッジ）確認、（B）.delta（フィルタ前）確認、（C）フィルタ／アライナの設定変更で再解析、の順で切り分けるのを強く推奨します。([PMC](https://pmc.ncbi.nlm.nih.gov/articles/PMC6913012/?utm_source=chatgpt.com "SyRI: finding genomic rearrangements and local sequence ..."))
    

もしよければ、あなたが見つけた「反復領域の座標（参照上の start–end）」と、可能なら `show-coords` の当該行（周辺数行）や `samtools depth` の該当領域のカバレッジサマリをここに貼ってくれれば、具体的にその断片化のパターンを一緒に読んで、どのフィルタ/パラメータが効きそうか（例：`delta-filter -1` を試す／`--maxmatch` で re-run）をステップごとに指示できます。迅速に解析できるように手順も書きます。





いいね、状況よく分かった — やりたいのは「もっと緩く（寛容に）アラインメントを拾って SyRI に渡し、反復領域でのシンテニー検出感度を上げる」ことですね。結論を先に短く書くと：

**やること（優先順）**

1. **アライメントを“より多く”拾う**（`nucmer --maxmatch`，`-l`/`-c` を下げる）。([mummer4.github.io](https://mummer4.github.io/manual/manual.html?utm_source=chatgpt.com "Running MUMmer4.x"))
    
2. **delta-filter のフィルタを緩める（または通さない）**：`-g` は「最も互いに整合する（長く一貫した）セット」を選ぶ保守的モード。反復を拾いたければ `-m`（many-to-many）やフィルタ無し、あるいは `-1`（one-to-one）を試して比較する。([Biostars](https://www.biostars.org/p/356518/?utm_source=chatgpt.com "mummerplot for genome self alignment NOT yielding full ..."))
    
3. **SyRI のしきい値を調整**（`--allow-offset` や `--invgaplen` など）で「小さなギャップ/重複」を許容する。さらに PAF（minimap2）入力も試すとアライナ依存のバイアスを確認できる。([GitHub](https://github.com/schneebergerlab/syri "GitHub - schneebergerlab/syri: Synteny and Rearrangement Identifier"))
    

以下、理由・具体的コマンド・チェックポイントを根拠付きで示します。

---

# 理由（簡潔）

- `nucmer` はデフォルトだと**一意なマッチを優先**する（反復の複数候補を全部拾わない）。`--maxmatch` を使えば _全ての_マッチを拾うので、反復領域での代替アラインメントが増える → SyRI のグラフが繋がりやすくなる。([mummer4.github.io](https://mummer4.github.io/manual/manual.html?utm_source=chatgpt.com "Running MUMmer4.x"))
    
- `delta-filter -g` は「長く整合する（mutually consistent）セット」を選ぶため、反復で断片的にしかマッチしない領域は落ちやすい。`-m`（many）やフィルタを弱めると反復由来の複数マッチを残せる。BiostarsやMUMmerの議論でも `-g` は比較的保守的に使われている。([Biostars](https://www.biostars.org/p/356518/?utm_source=chatgpt.com "mummerplot for genome self alignment NOT yielding full ..."))
    
- SyRI は与えられたアライメント群から「最大の矛盾しない syntenic path」を選ぶアルゴリズムなので（DAG＋動的計画法）、入力に多くの短い補助アライメントがあるほど反復で“つなげる”可能性が上がる。SyRI 自身にも重複・ギャップ系の許容パラメータがある。([BioMed Central](https://genomebiology.biomedcentral.com/articles/10.1186/s13059-019-1911-0?utm_source=chatgpt.com "SyRI: finding genomic rearrangements and local sequence ..."))
    

---

# 具体的に試すコマンド例（順番どおりに、差分を比較していくと分かりやすい）

### A) nucmer をより許容的に再実行（反復での橋渡し候補を拾う）

```bash
# 例：--maxmatch で全マッチを拾い、min cluster/lenを少し下げる
nucmer --maxmatch -l 50 -c 40 -p ${PREFIX} ${REF} ${QRY}
# （-l は最小マッチ長、-c は min cluster。必要に応じさらに下げると短い一致も拾える）
```

理由：`--maxmatch` は反復での代替アラインメントを増やす。-l/-c を下げれば短めのアンカーも拾えます（ただしノイズも増える）。([mummer4.github.io](https://mummer4.github.io/manual/manual.html?utm_source=chatgpt.com "Running MUMmer4.x"))

---

### B) delta-filter の振る舞いを変えて SyRI に渡す（比較用に3通り）

```bash
# 保守的（現状）: -g (longest mutually consistent set)
delta-filter -g ${PREFIX}.delta > ${PREFIX}.g-to-g.filter.delta

# より寛容（many-to-many）
delta-filter -m ${PREFIX}.delta > ${PREFIX}.m-to-m.filter.delta

# 最小フィルタ（one-to-one best） - 別用途で試す
delta-filter -1 ${PREFIX}.delta > ${PREFIX}.one2one.filter.delta

# またはフィルタを通さずに生 delta を syri に渡して比較（ただしサイズ大）
cp ${PREFIX}.delta ${PREFIX}.no_filter.delta
```

使い分け：`-g` は「最長一貫セット」を選びやすく、反復では落ちることがある。`-m` は多対多を残すので反復の“別コピー”へのマッチを残しやすく、SyRI がつなげられる可能性が上がる。Biostars/MUMmer の議論でも実践されている。([Biostars](https://www.biostars.org/p/356518/?utm_source=chatgpt.com "mummerplot for genome self alignment NOT yielding full ..."))

---

### C) show-coords（出力オプション）で細かい断片も確認

```bash
show-coords -THrd ${PREFIX}.m-to-m.filter.delta > ${PREFIX}.m-to-m.coords
# あるいはフィルタ無しの delta に対しても show-coords して比較
show-coords -rclTH ${PREFIX}.no_filter.delta > ${PREFIX}.no_filter.coords
```

目的：どの程度アラインメントが断片化しているかを目で見る（長さ/identity/ギャップ）。`-rclTH` など色々なオプションで詳細を見るとヒントが得られます。([mummer4.github.io](https://mummer4.github.io/manual/manual.html?utm_source=chatgpt.com "Running MUMmer4.x"))

---

### D) SyRI のパラメータを調整して実行

代表的に効くパラメータ：

- `--allow-offset <INT>`：連続するアライメント間で「どれだけ重なり/オフセット」を許すか。小さなずれを無視してブロック化したければ値を大きくする。
    
- `--invgaplen <INT>`：inversion を一まとまりとして扱う際に許すギャップ長。反復中の小さな欠損で inversion と判定されないように（または許容してつなげるため）増やせる。
    
- `--maxsize`：SyRI が出力する SV の最大サイズ制限（出力振る舞いの調整用）。
    
- `-F P`：minimap2 の PAF を入力にする（別アライナの観点で比較）。
    
- `--all`：重複領域でも短変異を全部報告する（出力量が増える）。([GitHub](https://github.com/schneebergerlab/syri "GitHub - schneebergerlab/syri: Synteny and Rearrangement Identifier"))
    

例（`-m` を使った delta-filter の結果を SyRI に入れて、allow-offset を緩める）：

```bash
syri -c ${PREFIX}.m-to-m.coords \
     -d ${PREFIX}.m-to-m.filter.delta \
     -r ${REF} -q ${QRY} \
     --prefix ${OUT_PREFIX} --no-chrmatch \
     --allow-offset 500 --invgaplen 2000 --maxsize 200000
```

（`--allow-offset` / `--invgaplen` の値はデータ依存なので、まずは 100–1000 の範囲で増やして試すのが良いです。）([GitHub](https://github.com/schneebergerlab/syri "GitHub - schneebergerlab/syri: Synteny and Rearrangement Identifier"))

---

### E) 別戦略：minimap2(PAF) → SyRI (`-F P`) を試す

```bash
# minimap2で全体アライメント（PAF）
minimap2 -x asm5 -t 16 ${REF} ${QRY} > ${PREFIX}.paf

# syri に PAF を渡す（-F P）
syri -r ${REF} -q ${QRY} -F P -c ${PREFIX}.paf -o ${OUT_PREFIX}_paf.syri.out ...
```

理由：アライナを変えると、反復でのアラインメント分布が変わることがある（minimap2 は長読に強い）。SyRI は PAF を読めるので（v1.6+）、アライナの違いを比較してみると良い。([GitHub](https://github.com/schneebergerlab/syri "GitHub - schneebergerlab/syri: Synteny and Rearrangement Identifier"))

---

# 検証と比較のやり方（必ずやる）

1. **3パターン以上で比較する**（現状：`-g` のまま、`-m`、`--maxmatch`+`-m`、`minimap2 PAF` など）。
    
2. **plotsr / dotplot で問題領域を拡大表示**（SyRI の出力を可視化して“どのフィルタで繋がったか”を確認）。([GitHub](https://github.com/schneebergerlab/plotsr?utm_source=chatgpt.com "schneebergerlab/plotsr: Tool to plot synteny and structural ..."))
    
3. **元データに戻ってカバレッジ確認**：Flye の当該 contig に nanopore リードを再マップして `samtools depth` で中心部の欠損が本物か確認（欠損＝アセンブリ起因の可能性高）。もしカバレッジが低ければ「アセンブリを直す or 注釈を残す」が必要。
    
4. **ノイズ注意**：アライメントを緩めすぎると偽陽性（実際は別コピーに誤マップ）が増える。短すぎるアンカーや低アイデンティティは慎重に扱う。
    

---

# 推奨フロー（実作業）

1. `nucmer --maxmatch -l 50 -c 40` で再アライン。
    
2. delta-filter で `-m` と `-g` をそれぞれ作る（`-m` を本命）。
    
3. `show-coords` の出力を比較（断片化の有無）。
    
4. SyRI を `--allow-offset 200~500` `--invgaplen 1000~2000` で走らせる（`-m` の出力を使う）。
    
5. `plotsr` で当該反復領域をズームして比較。
    
6. 必要なら minimap2(PAF) を作って `-F P` で SyRI を再実行し、結果を突き合わせる。([mummer4.github.io](https://mummer4.github.io/manual/manual.html?utm_source=chatgpt.com "Running MUMmer4.x"))
    

---

# 参考（主要）

- MUMmer / nucmer / delta-filter 動作（`--maxmatch` の効果、delta-filter の `-g`/`-m` 挙動）。([mummer4.github.io](https://mummer4.github.io/manual/manual.html?utm_source=chatgpt.com "Running MUMmer4.x"))
    
- SyRI README / マニュアル（`--invgaplen`,`--allow-offset`,`-F P` など）。([GitHub](https://github.com/schneebergerlab/syri "GitHub - schneebergerlab/syri: Synteny and Rearrangement Identifier"))
    
- plotsr の使い方（SyRI 結果の可視化）。([GitHub](https://github.com/schneebergerlab/plotsr?utm_source=chatgpt.com "schneebergerlab/plotsr: Tool to plot synteny and structural ..."))
    

---

必要なら、あなたの環境に合わせて「**具体的に値を置いた実行コマンド**（`--allow-offset=○○` 等）を 2〜3 種類 作ってその差分を比較し、plotsr の画像をどう読むか」まで作ります。もしよければ、**（A）現在の .delta（もしくは filter した .delta）ファイルのサイズ**と、**（B）反復領域の参照上の start–end（例: chr5:1,234,000-1,240,000）**を教えてください。そうすれば、私が推奨する具体的なパラメータの“試行セット”を一発で出します。



---

結論から言うと──

> ✅ **はい、結果が異なります。**
> 
> 特に「`--maxmatch` を使って再アライメントしたかどうか」で、  
> **synteny の見え方（反復領域の扱い）が変わります。**

---

### 🔍 詳しく説明すると：

あなたが過去に実行したスクリプト👇

```bash
delta-filter -m ${PREFIX}.delta > ${PREFIX}.m-to-m.filter.delta
```

このとき使っている `${PREFIX}.delta` は、  
おそらく **`nucmer` のデフォルトモード（= unique match ベース）** で作成した delta です。

つまり：

```bash
nucmer ${REF} ${QRY} -p ${PREFIX}
```

で作られた delta ファイルをフィルタしている。

---

### 🧠 重要な違い：`--maxmatch` の有無

|モード|コマンド例|検出される一致の種類|反復領域の扱い|
|---|---|---|---|
|**デフォルト**|`nucmer`|unique match のみ（重複領域は除外）|反復領域はほぼ無視される|
|**maxmatch**|`nucmer --maxmatch`|すべての一致（反復含む）|反復領域の synteny も検出可能|

`--maxmatch` をつけると、  
同じ配列が複数箇所に存在する場合でも、それぞれにアライメントを張ります。  
一方、デフォルトでは「重複している一致」は切り捨てられるため、  
**反復領域で synteny が途切れる現象**が起きやすいんです。

---

### 🧬 つまりあなたの過去のコマンドは：

- `delta-filter -m` で many-to-many にしてはいる  
    　→ しかしその元の `.delta` がそもそも unique match 限定。
    
- 結果として、**反復内部のシンテニー情報自体が `.delta` に含まれていなかった**。
    

---

### 💡 改善される理由（`--maxmatch` 付きの場合）

- `.delta` に反復領域の対応関係も含まれる
    
- `delta-filter -m` でそれを保持
    
- `syri` がその複数対応を元に、反復領域内の synteny をより多く検出できる  
    → **結果的にプロット（plotsr）で反復領域のシンテニーが復活して見える**
    

---

### ✅ 結論まとめ

|比較項目|過去のコマンド|今回（--maxmatch 版）|
|---|---|---|
|`.delta` の生成|unique match のみ|全一致を含む|
|反復領域の対応|ほぼ欠損|部分的に復活|
|syri 出力の差|シンテニー断片的|より連続的（反復含む）|
|実行時間・ファイルサイズ|軽い|やや重い（特に大規模リピートで）|

# 873341.pts-53.tardis
```
# --- 変数の設定（過去の実行と混ざらないよう別PREFIXに変更） ---
PREFIX="flye_vs_canu_maxmatch"
REF="assembly2/assembly-results/ragtag_flye_scaffold/ragtag_polished_round1.fasta"
QRY="assembly2/assembly-results/ragtag_canu_scaffold/canu_polished_round1.fasta"
OUT_PREFIX="flye_vs_canu_syri_maxmatch"  # 出力プレフィックスも変更

# --- NUCmer alignment with maxmatch ---
nucmer --maxmatch -l 50 -c 40 -p ${PREFIX} ${REF} ${QRY}

# --- Many-to-many filtering (keep all matches) ---
delta-filter -m ${PREFIX}.delta > ${PREFIX}.m-to-m.filter.delta

# --- Generate coords for syri ---
show-coords -THrd ${PREFIX}.m-to-m.filter.delta > ${PREFIX}.m-to-m.coords

# --- Run SyRI with relaxed parameters (for repeats) ---
syri -c ${PREFIX}.m-to-m.coords \
     -d ${PREFIX}.m-to-m.filter.delta \
     -r ${REF} \
     -q ${QRY} \
     --prefix ${OUT_PREFIX} \
     --no-chrmatch \
     --allow-offset 400 \
     --invgaplen 2000

# --- Visualize with plotsr ---
plotsr --sr ${OUT_PREFIX}syri.out \
       --genomes genomes.txt \
       -H 10 -W 15 \
       -o ${OUT_PREFIX}_plot.pdf

```





---
# 23551.pts-53.tardis

### ✅ 完成版：**g-to-g（1:1 genome-to-genome synteny）版スクリプト**

```bash
#!/bin/bash

# --- 変数の設定 ---
REF_ORIG="assembly2/assembly-results/ragtag_flye_scaffold/ragtag_polished_round1.fasta"
QRY_ORIG="assembly2/assembly-results/ragtag_canu_scaffold/canu_polished_round1.fasta"
REF_CONTIG="NC_007089.4_RagTag_pilon"
QRY_CONTIG="NC_007090.3_RagTag_pilon"
REF_TARGET="flye_NC_007089.4.fasta"
QRY_TARGET="canu_NC_007090.3.fasta"
# 逆相補鎖にしたクエリFASTA用の新しいファイル名
QRY_TARGET_RC="canu_NC_007090.3.rc.fasta"

# g-to-g 比較用プレフィックス
PREFIX_TARGET="flye89_vs_canu90_gtog"
OUT_PREFIX_TARGET="syri_flye89_vs_canu90_gtog"

GENOMES_FILE="genomes_target_gtog.txt"
OUTPUT_PDF="plot_flye89_vs_canu90_gtog.pdf"

# --- Step 1: FASTA抽出と逆相補変換 ---
echo "--- Step 1: Extracting FASTA sequences and applying reverse complement ---" && \
samtools faidx ${REF_ORIG} ${REF_CONTIG} > ${REF_TARGET} && \
samtools faidx ${QRY_ORIG} ${QRY_CONTIG} > ${QRY_TARGET} && \
\
echo "--- Applying reverse complement to query contig ---" && \
seqtk seq -r ${QRY_TARGET} > ${QRY_TARGET_RC} && \

# --- Step 2: plotsr用 genomes ファイル作成 ---
echo "--- Step 2: Creating genomes file for plotsr ---" && \
echo -e "${QRY_TARGET_RC}\n${REF_TARGET}" > ${GENOMES_FILE} && \

# --- Step 3: nucmer, syri, plotsr pipeline (g-to-g) ---
echo "--- Step 3: Running nucmer + delta-filter (-g) + syri + plotsr ---" && \
nucmer --maxmatch --prefix=${PREFIX_TARGET} ${REF_TARGET} ${QRY_TARGET_RC} && \

# ★ g-to-g モードのdelta-filterを使用
delta-filter -g ${PREFIX_TARGET}.delta > ${PREFIX_TARGET}.g-to-g.filter.delta && \
show-coords -THrd ${PREFIX_TARGET}.g-to-g.filter.delta > ${PREFIX_TARGET}.g-to-g.coords && \

# SyRI 実行
syri -c ${PREFIX_TARGET}.g-to-g.coords \
     -d ${PREFIX_TARGET}.g-to-g.filter.delta \
     -r ${REF_TARGET} \
     -q ${QRY_TARGET_RC} \
     --prefix ${OUT_PREFIX_TARGET} \
     --no-chrmatch \
     --allow-offset 400 \
     --invgaplen 2000 && \

# 可視化
plotsr --sr ${OUT_PREFIX_TARGET}syri.out \
       --genomes ${GENOMES_FILE} \
       -H 10 -W 15 \
       -o ${OUTPUT_PDF} && \

echo "--- Pipeline finished successfully! Check ${OUTPUT_PDF} ---"
```

---

### 🧠 補足（違いのポイント）

|処理|m-to-m版|g-to-g版|
|---|---|---|
|`delta-filter`|`-m` （多対多）|`-g`（genome-to-genome best hit）|
|シンテニー感度|高い（反復も拾いやすい）|低め（重複や反復を除く）|
|ノイズ（偽陽性）|多め|少ない・構造的に信頼性高い|
|推奨用途|反復や局所的変異の探索|グローバルな構造比較、方向性確認|

---

もし「g-to-g版をベースにしつつ、もう少し感度を上げたい」場合は、  
以下のように SyRI にパラメータを追加して再実行すると良いです：

```bash
syri -c ${PREFIX_TARGET}.g-to-g.coords \
     -d ${PREFIX_TARGET}.g-to-g.filter.delta \
     -r ${REF_TARGET} \
     -q ${QRY_TARGET_RC} \
     --prefix ${OUT_PREFIX_TARGET} \
     --no-chrmatch \
     --allow-offset 400 \
     --invgaplen 2000
```

これで、反復領域付近の“途切れたシンテニー”をある程度補完できます。

---

必要ならこの `g-to-g` スクリプトをベースに、  
「m-to-m」「raw」「paf(minimap2)」モードも同じ形で自動実行する統合スクリプトも作れます。  
（例：`MODE=g2g` / `m2m` / `raw` / `paf` で切り替えられる1本化版）  
希望しますか？