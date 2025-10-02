
ここでは、以下の2つのFASTAファイルを比較する前提で進めます。

  * **参照ゲノム (Reference):** `assembly2/assembly-results/ragtag_flye_scaffold/ragtag_polished_round1.fasta`
  * **クエリゲノム (Query):** `assembly2/assembly-results/ragtag_canu_scaffold/canu_polished_round1.fasta`

ワークフローは大きく分けて「**1. 全ゲノムアライメント**」「**2. SyRiによる構造再編の同定**」「**3. plotsrによる可視化**」の3つのステップで構成されます。

-----

# screen -r 377189.pts-54.tardis
### **前提条件：必要なツールのインストール**

まず、以下のツールがインストールされ、パスが通っている（コマンドラインから呼び出せる）ことを確認してください。

  * **MUMmer4:** 高速な全ゲノムアライメントツール。SyRiはMUMmerのアライメント結果を入力として利用します。
  * **SyRi:** ゲノムの構造再編を同定するメインツール。
  * **samtools:** FASTAファイルのインデックスを作成するために使用します。
  * **plotsr:** SyRiの結果を可視化するためのツール。

これらのツールは、多くの場合`conda`を使ってインストールするのが簡単です。

```bash
# conda環境の作成（推奨）
conda create -n syri_env
conda activate syri_env

# ツールのインストール
conda install -c bioconda mummer4 syri samtools python=3
conda install -c bioconda plotsr
```

-----

### **手順**

#### **ステップ1：MUMmer4による全ゲノムアライメント**

最初に、`nucmer`コマンドを使って2つのFASTAファイルをアライメントします。これにより、両ゲノム間の相同な領域が特定されます。

1.  **nucmerの実行**
    `--maxmatch`オプションで、ユニークでない繰り返し配列なども含めた、できるだけ多くのアライメントを見つけます。

    ```bash
    # 分かりやすいようにファイルパスを変数に入れておきます
    REF="assembly2/assembly-results/ragtag_flye_scaffold/ragtag_polished_round1.fasta"
    QRY="assembly2/assembly-results/ragtag_canu_scaffold/canu_polished_round1.fasta"
    PREFIX="flye_vs_canu"

    nucmer --maxmatch -c 100 -b 500 -l 50 ${REF} ${QRY} -p ${PREFIX}
    ```

      * `${REF}`: 参照ゲノムのパス
      * `${QRY}`: クエリゲノムのパス
      * `-p ${PREFIX}`: 出力ファイルの接頭辞（プレフィックス）を指定します。ここでは `flye_vs_canu` となります。
      * このコマンドにより、`flye_vs_canu.delta` というファイルが生成されます。

2.  **アライメントのフィルタリング**
    次に、`delta-filter`を使って、1対1の最良のアライメントのみを抽出します。これは、後のSyRiの解析でノイズを減らすために重要なステップです。

    ```bash
    delta-filter -1 -q -r ${PREFIX}.delta > ${PREFIX}.filter.delta
    ```

      * `-1`: 1対1のアライメントのみを保持します。
      * このコマンドにより、フィルタリングされた `flye_vs_canu.filter.delta` が生成されます。

3.  **座標情報の抽出**
    最後に、`show-coords`を使って、フィルタリングされたdeltaファイルからSyRiが読み込める形式の座標ファイル（`.coords`）を生成します。

    ```bash
    show-coords -THrd ${PREFIX}.filter.delta > ${PREFIX}.coords
    ```

      * `-THrd`: タブ区切り、ヘッダーなし、参照とクエリ両方の情報を出力するオプションです。
      * このコマンドにより、`flye_vs_canu.coords` が生成されます。

-----

#### **ステップ2：SyRiによる構造再編の同定**

ステップ1で作成した座標ファイル（`.coords`）と元のFASTAファイルを使って、SyRiを実行します。

```bash
syri -c ${PREFIX}.coords -r ${REF} -q ${QRY} --prefix ${PREFIX}_syri
```

  * `-c`: 入力となる`.coords`ファイルを指定します。
  * `-r`: 参照ゲノムのFASTAファイルを指定します。
  * `-q`: クエリゲノムのFASTAファイルを指定します。
  * `--prefix`: SyRiが出力するファイルの接頭辞を指定します。

このコマンドが完了すると、以下のようなファイルが生成されます。

  * **`flye_vs_canu_syri.out`**: SyRiの主要な出力ファイル。同定されたシンテニーブロック（共直線的な領域）や構造再編（転座、逆位、重複など）に関する詳細な情報が含まれています。
  * **`flye_vs_canu_syri_sr.gff`**: 構造再編の情報をGFFフォーマットで出力したファイル。ゲノムブラウザでの表示に便利です。
  * その他、いくつかのVCFファイルなど。

-----

#### **ステップ3：plotsrによる可視化**

最後に、SyRiの出力結果を`plotsr`を使って可視化し、ゲノム構造の違いを視覚的に理解します。

1.  **FASTAファイルのインデックス作成**
    `plotsr`は各染色体（またはコンティグ）の長さを知るために、FASTAインデックスファイル（`.fai`）を必要とします。`samtools faidx`で作成します。

    ```bash
    samtools faidx ${REF}
    samtools faidx ${QRY}
    ```

      * これにより、各FASTAファイルと同じディレクトリに `.fai` ファイルが作成されます。

2.  **ゲノム情報ファイルの作成**
    `plotsr`に、どのゲノムを描画するかを伝えるための設定ファイル（ここでは`genomes.txt`という名前にします）を作成します。

    以下の内容で `genomes.txt` というファイルを作成してください。**ファイルパスはご自身の環境に合わせて修正してください。**

    ```
    # [表示名]  [FASTAファイルへのパス]  [.faiファイルへのパス]
    Flye_Polished    assembly2/assembly-results/ragtag_flye_scaffold/ragtag_polished_round1.fasta    assembly2/assembly-results/ragtag_flye_scaffold/ragtag_polished_round1.fasta.fai
    Canu_Polished    assembly2/assembly-results/ragtag_canu_scaffold/canu_polished_round1.fasta    assembly2/assembly-results/ragtag_canu_scaffold/canu_polished_round1.fasta.fai
    ```

      * 各カラムはタブで区切ってください。

3.  **plotsrの実行**
    準備が整ったら、`plotsr`を実行してプロットを作成します。

    ```bash
    plotsr --sr ${PREFIX}_syri.out --genomes genomes.txt -H 10 -W 15 -o ${PREFIX}_plot.pdf
    ```

      * `--sr`: SyRiの出力ファイル (`.out`) を指定します。
      * `--genomes`: 先ほど作成したゲノム情報ファイルを指定します。
      * `-H`, `-W`: 出力する画像の高さと幅をインチ単位で指定します。適宜調整してください。
      * `-o`: 出力ファイル名を指定します。PDF形式 (`.pdf`) を推奨します（拡大しても綺麗です）。

これで、`flye_vs_canu_plot.pdf` という名前のファイルが生成されます。このファイルを開くと、2つのゲノムアセンブリ間の対応関係や、逆位、転座といった構造的な違いが視覚的に表現された図（ドットプロットや染色体ペインティング）を確認できます。

以上がSyRiを用いたゲノム構造比較と可視化の基本的な手順です。ゲノムのサイズや複雑さによっては、計算に時間がかかる場合があります。