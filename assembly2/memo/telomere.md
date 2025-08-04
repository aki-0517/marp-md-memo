テロメア配列とは、**染色体の末端にある「フタ」のような構造（テロメア）を構成する、特徴的なDNA配列**のことです。

この配列は、特定の短い塩基配列が何千回も繰り返されている「反復配列」です。ヒトの場合、テロメア配列は「**TTAGGG**」という6つの塩基の組み合わせが繰り返されています。



---

### テロメア配列の重要な役割

テロメア配列は、靴ひもの先端にあるプラスチックのキャップによく例えられます。このキャップがあるおかげで、靴ひもはほつれずにその役目を果たせます。テロメアも同様に、染色体にとって不可欠な2つの重要な役割を担っています。

#### 1. 遺伝情報の保護
細胞が分裂する際、染色体も複製されます。しかし、複製の仕組み上、染色体の末端は毎回少しずつ短くなってしまいます。テロメア配列は遺伝情報を含まない「捨て石」のような部分なので、この部分が代わりに短くなることで、**内側にある重要な遺伝情報が削られてしまうのを防いでいます。**

#### 2. 染色体の安定化
もし染色体の末端がむき出しになっていると、細胞はそれを「壊れたDNA」と勘違いして、他の染色体とくっつけてしまうことがあります。テロメア配列が末端を覆っていることで、**染色体どうしの異常な融合を防ぎ、ゲノム全体の安定性を保っています。**

### テロメアと「老化」「がん」の関係
細胞分裂のたびにテロメアは短くなるため、**テロメアの長さは「細胞の寿命の回数券」**とも言われます。テロメアが一定の短さに達すると、細胞はそれ以上分裂できなくなり、「細胞老化」という状態になります。これが個体の老化の一因と考えられています。

一方で、がん細胞の多くは「テロメラーゼ」という特殊な酵素を活性化させ、短くなったテロメアを修復して無限に分裂し続ける能力を獲得しています。このため、テロメアやテロメラーゼは、老化やがんの研究において非常に重要なターゲットとなっています。

_Dictyostelium discoideum_のゲノムに関する論文によると、この生物は他の多くの真核生物で見られる典型的なテロメア配列を持っていません。代わりに、染色体の末端には、染色体外リボソームDNA（rDNA）エレメントの末端に見られるのと同じ**(G+A)に富むリピートモチーフ**が存在します 1111。

---

### テロメア構造の特徴

この研究では、典型的な(G+T)に富むテロメア様のモチーフはゲノム配列中に見つかりませんでした 2。その代わり、以下の点が明らかになりました。

- **rDNA配列の利用**: 染色体の末端には、染色体外に多数存在するrDNAパインドロームの部分的なコピーが存在し、これがテロメアとして機能していると考えられています 3333。
    
- **末端構造**: 染色体の末端領域とrDNA様配列との接合部が12箇所確認されており、これらが各染色体の末端（テロメア）であると結論付けられています 4444。
    
- **共通の維持機構**: この構造は、染色体の末端と染色体外rDNAの両方を維持・安定化させるために、共通のメカニズムが用いられている可能性を示唆しています 5。
    

要約すると、_Dictyostelium_は一般的なテロメア配列（例：ヒトのTTAGGG）を持たず、リボソームDNAの末端配列を染色体のテロメアとして利用するという、非常にユニークな構造を持っています。

はい、承知いたしました。アセンブリ後のFASTAファイル (`ragtag_polished_round1.fasta`) から、リボソームDNA (rDNA) の末端にある反復配列（テロメア様配列）を特定するためのコマンド手順を、段階的に解説します。

この手順は、既知の

_Dictyostelium discoideum_のrDNA配列をリファレンスとして使用し、あなたのアセンブリデータから該当する領域を見つけ出し、その末端を解析するという流れに基づいています。これは、_Dictyostelium_のテロメアがrDNAの末端反復配列によって構成されているという生物学的知見を利用したアプローチです 1111。

---

### 必要なツール

あらかじめ、以下のバイオインフォマティクスツールがインストールされ、パスが通っていることを確認してください。

- **BLAST+**: `blastn`, `makeblastdb` コマンドを使用します。
    
- **Samtools**: `samtools faidx` コマンドで特定の配列を抽出します。
    
- **Entrez Direct (EDirect)**: NCBIからリファレンス配列をダウンロードするために使用します（推奨）。
    
- **Tandem Repeats Finder (TRF)**: 配列中の縦列反復配列を検出するための専門ツールです。
    

---

### ステップ1：リファレンスrDNA配列の準備

まず、検索の元となる_Dictyostelium discoideum_の完全な染色体外rDNA配列をNCBIからダウンロードします。

Bash

```
# EDirectを使ってアクセッション番号 M11158 の配列を取得する
efetch -db nucleotide -id M11158 -format fasta > dicty_rDNA_ref.fasta

# 取得したファイルを確認
head dicty_rDNA_ref.fasta
```

**コマンド解説**:

- `efetch`: NCBIのデータベースからデータを取得するコマンドです。
    
- `-db nucleotide -id M11158 -format fasta`: ヌクレオチドデータベースから指定したアクセッション番号のデータをFASTA形式で取得します。
    
- `> dicty_rDNA_ref.fasta`: 取得した結果をファイルに保存します。
    

---

### ステップ2：BLASTによるrDNAコンティグの特定

次に、ダウンロードしたリファレンス配列をクエリとして、あなたのゲノムアセンブリに対してBLAST検索を実行し、rDNAを含むコンティグ（またはスカフォールド）を特定します。

Bash

```
# 1. あなたのアセンブリファイルからBLASTデータベースを作成
makeblastdb -in ragtag_polished_round1.fasta -dbtype nucl -out my_assembly_db

# 2. blastnを実行して、アセンブリ内のrDNA配列を検索
blastn -query dicty_rDNA_ref.fasta -db my_assembly_db -outfmt 6 -out blast_results.txt

# 3. BLASTの結果を確認
head blast_results.txt
```

**コマンド解説**:

- `makeblastdb`: FASTAファイルからBLASTが検索できる形式のデータベースを作成します。
    
- `blastn`: 塩基配列同士を比較するBLAST検索です。
    
- `-outfmt 6`: 結果をタブ区切りのシンプルな形式で出力します。見やすく、後続の処理で扱いやすいです。出力ファイルの2列目に、ヒットしたあなたのアセンブリ内のコンティグ名が表示されます。
    

---

### ステップ3：rDNAコンティグ配列の抽出

BLASTの結果から、rDNAを含むコンティグの名前をリストアップし、そのコンティグの完全な塩基配列をアセンブリ全体から抽出します。

Bash

```
# 1. BLAST結果の2列目（ヒットしたコンティグ名）を抽出し、重複を除いてリスト化
cut -f2 blast_results.txt | sort -u > rdna_contig_names.txt

# 2. samtoolsでアセンブリファイルをインデックス化
samtools faidx ragtag_polished_round1.fasta

# 3. リストにある名前のコンティグ配列をすべて抽出する
xargs samtools faidx ragtag_polished_round1.fasta < rdna_contig_names.txt > rdna_contigs.fasta

# 4. 抽出されたFASTAファイルを確認
head rdna_contigs.fasta
```

**コマンド解説**:

- `cut -f2 | sort -u`: タブ区切りファイルの2列目だけを切り出し、並べ替えて重複を削除します。これにより、rDNAが乗っているコンティグの名前リストが作成されます。
    
- `samtools faidx`: FASTAファイルをインデックス化し、特定の配列を高速に抽出できるようにします。
    
- `xargs samtools faidx ...`: `rdna_contig_names.txt` に書かれたコンティグ名を一つずつ`samtools faidx`に渡し、該当する配列をすべて抽出して`rdna_contigs.fasta`に出力します。
    

---

### ステップ4：末端の反復配列の解析

最後に、抽出したrDNAコンティグ (`rdna_contigs.fasta`) の末端領域に、特徴的な(G+A)リッチな反復配列があるかを確認します。これには**Tandem Repeats Finder (TRF)**を使用するのが最も確実です。

Bash

```
# TRFを実行して、抽出したrDNAコンティグ内の反復配列を検出
# パラメータは一般的なものです。データに応じて調整が必要な場合があります。
trf rdna_contigs.fasta 2 7 7 80 10 50 500 -f -d -m

# TRFは複数の出力ファイルを出力します。
# 最も重要なのは ".dat" ファイルです。
less rdna_contigs.fasta.2.7.7.80.10.50.500.dat
```

**コマンド解説と結果の解釈**:

- `trf`: Tandem Repeats Finderの実行コマンドです。
    
- `2 7 7 80 10 50 500`: マッチ、ミスマッチ、インデルのスコアや、検出する反復の最小スコアなどを指定するパラメータです。
    
- `-f -d -m`: 出力形式などを指定するオプションです。
    
- **`.dat` ファイルの確認**: このファイルには、検出された反-復配列の位置、長さ、コンセンサス配列などが表形式でまとめられています。
    
    - **見るべきポイント**:
        
        1. `Sequence:` の行でコンティグ名を確認します。
            
        2. `Indices:` の列で、反復配列の開始位置と終了位置を確認します。**開始位置が1に近いもの**（コンティグの先頭）や、**終了位置がコンティグの全長に近いもの**（コンティグの末尾）を探します。
            
        3. `Consensus pattern:` の列で、その反復配列の単位となる配列を確認します。_Dictyostelium_の場合、(G+A)に富む配列（例：`AG`, `AAG`, `AAAG`など）が見つかるはずです。
            

この手順により、あなたのアセンブリデータにおけるrDNAの末端反復配列を特定することができます。










---
# 以降アーカイブ


## 🧬ゲノムアセンブリからテロメア繰り返し配列を探索・同定するパイプライン

### はじめに

このドキュメントは、ゲノムアセンブリデータから未知のテロメア繰り返し配列を探索し、その候補配列が本当にテロメアであるかを検証するための一連の標準的な手順をまとめたものです。このワークフローは、ツールのバージョン互換性の問題を回避し、堅牢に動作するよう最適化されています。

### 🔬 必要ツール

この解析には、以下のコマンドラインツールが必要です。事前にインストールしておいてください。

  * **seqkit**: 高速なFASTA/Q操作ツール
  * **bedtools**: ゲノム領域の操作ツール
  * **Tandem Repeats Finder (TRF)**: 縦列反復配列の探索ツール
  * **bowtie2**: 高速なリードマッピングツール
  * **samtools**: SAM/BAMファイルの操作ツール
  * **IGV (Integrative Genomics Viewer)**: (任意) 結果を可視化するためのゲノムブラウザ

-----

## ✅ 解析フロー

### ステップ1：染色体末端配列の抽出

ゲノム全体のFASTAファイルから、各染色体（またはScaffold）の両端10kbの配列を抽出します。ここでは`bedtools`を用いることで、確実な抽出を行います。

#### 1\. 各配列の長さ情報を取得する

`seqkit`を使い、FASTAファイル内の各配列のIDと長さを取得します。`-i`オプションでIDを最初の空白までで切り出すのが重要です。

```bash
seqkit fx2tab -n -l -i your_genome.fasta > lengths.txt
```

#### 2\. 抽出領域の座標ファイル (BED形式) を作成する

`lengths.txt`を元に、各配列の先頭10kbと末尾10kbの座標情報を`awk`で生成します。

```bash
awk 'BEGIN{OFS="\t"} {
    len=$2;
    end_len=10000;
    if (len > 0) print $1, 0, (len > end_len ? end_len : len);
    if (len > end_len) {
        start = len - end_len;
        print $1, start, len;
    }
}' lengths.txt > regions.bed
```

#### 3\. 配列を抽出する

`bedtools getfasta`を使い、`regions.bed`で指定された領域の配列をゲノムから一括で抽出します。

```bash
bedtools getfasta -fi your_genome.fasta -bed regions.bed -fo telomere_ends.fasta
```

このステップの完了後、すべての末端配列が`telomere_ends.fasta`という一つのファイルに保存されます。

-----

### ステップ2：候補となる繰り返し配列の探索

抽出した末端配列の中から、タンデムリピート（縦列反復配列）を`TRF`を使って網羅的に探索します。

```bash
trf telomere_ends.fasta 2 7 7 80 10 50 500 -d -h
```

-----

### ステップ3：最有力候補の同定

`TRF`の出力ファイル (`.dat`) は非常に大きいことがあるため、コマンドを使って**コピー数が最も多い繰り返し配列**を効率的に見つけ出します。

```bash
grep -v -e "Sequence:" -e "Parameters:" telomere_ends.fasta.*.dat \
| awk '{print "Copy#:", $4, "\tPeriod:", $3, "\tConsensus:", $14}' \
| sort -k2 -nr \
| head -n 20
```

このコマンドの出力リストの最上位に表示される、**コピー数が突出して多く、かつ単純なパターン**の配列が、テロメア配列の最有力候補となります。（例：`(TAA)n`など）

-----

### ステップ4：ゲノム全体での分布による最終検証

最有力候補となった配列が、本当にテロメアであるか（染色体末端にのみ存在するか）を`bowtie2`によるマッピングで最終確認します。

#### 1\. 検索用の配列（クエリ）を作成する

候補配列（例：`TAA`）を複数回繰り返したFASTAファイルを作成します。

```bash
# 例：候補がTAAの場合
CANDIDATE="TAA"
echo -e ">telomere_candidate_${CANDIDATE}\n$(for i in {1..15}; do echo -n $CANDIDATE; done)" > candidate_query.fasta
```

#### 2\. ゲノム全体へのマッピング

`bowtie2`を使い、候補配列がゲノム上のどこに存在するかを網羅的に検索します。

```bash
# ゲノムインデックスの作成（初回のみ）
bowtie2-build your_genome.fasta genome_index

# マッピングの実行
bowtie2 -x genome_index -f candidate_query.fasta -k 10000 --very-sensitive-local -S telomere_hits.sam
```

#### 3\. 可視化用ファイルへの変換

マッピング結果を`samtools`でソート・インデックス化し、ゲノムブラウザで表示できる形式にします。

```bash
samtools view -bS telomere_hits.sam | samtools sort -o telomere_hits.sorted.bam
samtools index telomere_hits.sorted.bam
```

### 結論と結果の解釈

最終的に生成された`telomere_hits.sorted.bam`ファイルを、`IGV`などのゲノムブラウザでゲノムアセンブリと一緒に表示します。

マッピングされたヒット（リード）が、**ゲノムのすべての染色体（または長いScaffold）の末端部分にだけ特異的に集中していれば、その候補配列が真のテロメア配列であると結論できます。**

```bash
samtools view telomere_hits.sorted.bam | \
awk '
  BEGIN {
    # まず最初に、各Scaffoldの全長を lengths.txt から読み込んで記憶する
    while ((getline < "lengths.txt") > 0) {
      len_map[$1] = $2
    }
    # 末端領域の幅を10kbに設定
    margin = 10000
    print "Scaffold_ID\tSTART_HITS\tMIDDLE_HITS\tEND_HITS"
  }
  {
    # BAM/SAMファイルの3列目(Scaffold ID)と4列目(マッピング開始位置)を変数に格納
    scaffold = $3
    position = $4
    total_len = len_map[scaffold]

    # マッピング位置に応じて、各領域のヒット数をカウントアップ
    if (position <= margin) {
      start_hits[scaffold]++
    } else if (position >= total_len - margin) {
      end_hits[scaffold]++
    } else {
      middle_hits[scaffold]++
    }
  }
  END {
    # 最後に、集計結果をScaffoldごとに出力
    for (scaffold in len_map) {
      printf "%s\t%d\t%d\t%d\n", scaffold, start_hits[scaffold]+0, middle_hits[scaffold]+0, end_hits[scaffold]+0
    }
  }
'
```

