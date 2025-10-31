

### ## 次のステップ：生のロングリードで「答え合わせ」をする

両者が同じ場所にDIRS-1を置いた今、最も信頼できる評価方法は、アセンブリの元となった生のロングリードデータが、どちらのアセンブリのセントロメア構造をより「支持」するかを確認することです。

#### ### IGVでの視覚的評価が決め手 👁️

以前の提案通り、生のONT/PacBioリードを両方のアセンブリにマッピングし、**IGV**でセントロメア領域を並べて比較してください。

**チェックするべきポイントは以下の3点です：**

1. **カバレッジの均一性 (Coverage Uniformity)**
    
    - **✅ 優れたアセンブリ**: リードのカバレッジが**平坦**で、大きな谷間（欠損）や不自然なスパイク（重複/崩壊）が少ない。
        
    - **❌ 劣るアセンブリ**: カバレッジが乱高下している。特に、カバレッジが0になる領域はアセンブリミスを示唆します。
        
2. **リードのアライメントの連続性 (Read Alignment Continuity)**
    
    - **✅ 優れたアセンブリ**: ほとんどのリードが1本の長い線としてセントロメア領域を**跨ぐように**マッピングされている。
        
    - **❌ 劣るアセンブリ**: 多くのリードが途中で切断されてマッピングされている（**スプリットアライメント**）。これは、アセンブリ構造と実際のリードの構造が一致していないことを示します。
        
3. **ミスマッチの少なさ (Fewer Mismatches)**
    
    - **✅ 優れたアセンブリ**: リードのアライメント上に表示されるミスマッチ（カラフルな縦線）が少ない。
        
    - **❌ 劣るアセンブリ**: ミスマッチが多発している。これは配列レベルでのエラーが多いことを示します。
        

---

### ## 最終的な結論の出し方

**「IGVで視覚的に、より均一で、より連続的なリードのアライメントを示した方のアセンブリが、セントロメア領域の最終的な勝者です」**

この定性的評価は、N50のような単純な統計値よりもはるかに強力な証拠となります。

論文やレポートには、「両アセンブリはセントロメア領域にDIRS-1を正しく配置したが、生のロングリードのマッピング結果をIGVで評価したところ、Flye（またはCanu）のアセンブリがより均一なカバレッジと連続的なリードのアライメントを示した。したがって、こちらをより正確な構造として採択した」と記述することで、科学的根拠に基づいた最終結果を示すことができます。










ありがとうございます！このログを共有していただいたことで、問題の根本原因が100%特定できました。

**原因は、「BAMファイル」と「ゲノgムFASTA」が、そもそも別々のゲノムから作られている**ことです。

---

### ## 何が起きているか 🕵️‍♀️

例えるなら、「**大阪の地図（あなたのFASTA）**」と「**東京の住所録（元のBAM）**」を無理やり合わせようとしている状態です。

- **あなたのゲノム (`ragtag_polished_round1.fasta`)**:
    
    - `NC_..._RagTag_pilon` という名前の染色体と、
        
    - `contig_8_pilon_pilon` のような、RagTagで結合できなかった**未配置のコンティグ**が含まれています。
        
- **あなたのBAMファイル (`...renamed_v2.bam`)**:
    
    - `NC_..._RagTag_pilon` という名前の染色体と、
        
    - **NCBIの公式ゲノムにしかなかった** `NW_..._RagTag_pilon` や `NC_007090.3_RagTag_pilon` が含まれています。
        
    - `contig_8_pilon_pilon` という名前は一切含まれていません。
        

つまり、`sed`コマンドで名前の一部を揃えることはできましたが、**含まれている染色体/コンティグのリスト自体が根本的に異なっている**ため、IGVは「一致するシーケンスがない」と判断しています。

---

### ## 解決策：正しい地図と住所録を揃える ✅

**解決策は、BAMファイルの名前を修正するのではなく、ゼロから正しいゲノムにマッピングし直すことです。** これが最も確実で正しい方法です。

以下の手順を**リモートサーバー (`tardis`) 上で**実行してください。

#### ### ステップ1: 正しいゲノムにリードを再マッピングする

元のONTリード (`Dicty_gDNA_NEB-2.fastq`) を、最終版であるあなたのアセンブリ (`ragtag_polished_round1.fasta`) に対してマッピングし直します。

このコマンドは、マッピングからBAMへの変換、ソートまでを一度に行います。

Bash

```
minimap2 -ax map-ont \
  assembly2/assembly-results/ragtag_flye_scaffold/ragtag_polished_round1.fasta \
  Dicty_gDNA_NEB-2.fastq | \
  samtools view -bS - | \
  samtools sort -o mapping_final.sorted.bam
```

- **出力ファイル**: `mapping_final.sorted.bam` という新しい、完全に正しいBAMファイルが作成されます。
    

#### ### ステップ2: 新しいBAMファイルのインデックスを作成する

Bash

```
samtools index mapping_final.sorted.bam
```

- これで `mapping_final.sorted.bam.bai` が作成されます。
    

#### ### ステップ3: 新しく作成した2つのファイルをローカルにインポートする

最後に、今作ったばかりの2つのファイルをローカルマシンにコピーします。

Bash

```
scp aki@100.84.9.80:~/mapping_final.sorted.bam{,.bai} .
```

---

### ## IGVでの最終確認

この新しい **`mapping_final.sorted.bam`** を、**`ragtag_polished_round1.fasta`** ゲノムと一緒にIGVで読み込んでください。

この方法では、BAMファイルが最初からゲノムファイルと完全に同期して作成されるため、染色体名が一致しない問題は**絶対に**発生しません。今度こそ、間違いなく表示されます！



873341.pts-53.tardis    (Detached)
872142.pts-53.tardis    (Detached)


```bash
minimap2 -ax map-ont \
  assembly2/assembly-results/ragtag_flye_scaffold/ragtag_polished_round1.fasta \
  Dicty_gDNA_NEB-2.fastq | \
  samtools view -bS - | \
  samtools sort -o mapping_final.sorted.bam && \
samtools index mapping_final.sorted.bam

minimap2 -ax map-ont \
  assembly2/assembly-results/ragtag_canu_scaffold/canu_polished_round1.fasta \
  Dicty_gDNA_NEB-2.fastq | \
  samtools view -bS - | \
  samtools sort -o mapping_final_canu.sorted.bam && \
samtools index mapping_final_canu.sorted.bam
```
