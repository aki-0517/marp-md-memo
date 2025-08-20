`ragtag_polished_round1.fasta` ファイルに対して、RepeatModelerで繰り返し配列（リピート配列）のカスタムライブラリを作成し、そのライブラリを使ってRepeatMaskerでゲノム配列をマスクするまでの標準的な手順を以下にまとめます。

-----

## 概要

このプロセスは大きく2つのステップに分かれます。

1.  **RepeatModelerの実行**: ゲノム配列から繰り返し配列を *de novo*（ゼロから）で同定・分類し、カスタムライブラリを作成します。
2.  **RepeatMaskerの実行**: ステップ1で作成したカスタムライブラリを用いて、ゲノム配列上の繰り返し配列領域を特定し、マスクします（通常は小文字に変換）。

-----

### ステップ1: RepeatModelerによるカスタムリピートライブラリの構築(3357778.pts-53.tardis)

まず、ゲノムアセンブリから繰り返し配列のデータベースを構築し、それをモデル化します。

#### 1-1. データベースの構築 (BuildDatabase)

RepeatModelerが解析できるように、FASTAファイルを専用のデータベース形式に変換します。

  * **目的**: `ragtag_polished_round1.fasta` ファイルからNCBI-BLAST用のデータベースを構築します。
  * **コマンド**:

```bash
BuildDatabase -name ragtag_db -engine ncbi ragtag_polished_round1.fasta
```

  * **説明**:
      * `-name ragtag_db`: 出力されるデータベースの接頭辞（プレフィックス）を指定します。ここでは `ragtag_db` としています。
      * `-engine ncbi`: 使用する検索エンジンを指定します。`ncbi` (NCBI-BLAST) が一般的です。
      * `ragtag_polished_round1.fasta`: 入力となるゲノムアセンブリのFASTAファイルです。

このコマンドを実行すると、`ragtag_db.nhr`, `ragtag_db.nin`, `ragtag_db.nsq` などの複数のファイルが生成されます。

#### 1-2. リピート配列の同定とモデル化 (RepeatModeler)

次に、構築したデータベースを用いて繰り返し配列を同定し、分類します。この処理は計算に非常に時間がかかる場合があります。

  * **目的**: 繰り返し配列ファミリーを同定し、コンセンサス配列のライブラリ（カスタムライブラリ）を作成します。
  * **コマンド**:

```bash
RepeatModeler -database ragtag_db -engine ncbi -pa 8
```

  * **説明**:
      * `-database ragtag_db`: ステップ1-1で作成したデータベースの接頭辞を指定します。
      * `-engine ncbi`: データベース構築時と同じエンジンを指定します。
      * `-pa 8`: 並列計算に使用するCPUコア（スレッド）数を指定します。マシンのスペックに合わせてこの数値を調整してください（例: `-pa 16`、`-pa 32`など）。

このコマンドが正常に完了すると、`RM_....` という名前のディレクトリ（例: `RM_12345.20250807-112233`）が生成されます。その中に、最も重要なファイルであるカスタムリピートライブラリが格納されています。

  * **重要な出力ファイル**: `[生成されたRMディレクトリ]/consensi.fa.classified`
      * これが、このゲノムに特異的な繰り返し配列のFASTA形式ライブラリです。RepeatMaskerで使用します。

-----

### ステップ2: RepeatMaskerによるゲノム配列のマスキング

RepeatModelerで作成したカスタムライブラリを使い、元のゲノム配列上のリピート領域をマスクします。

  * **目的**: `consensi.fa.classified` を辞書として使い、`ragtag_polished_round1.fasta` 内のリピート配列を特定してマスクします。
  * **コマンド**:

<!-- end list -->

```bash
RepeatMasker -lib [生成されたRMディレクトリ]/consensi.fa.classified -gff -pa 8 ragtag_polished_round1.fasta
```

  * **説明**:
      * `-lib [パス]`: マスキングに使用するカスタムライブラリファイルを指定します。ステップ1-2で得られた `consensi.fa.classified` のパスを正確に入力してください。
      * `-gff`: マスキング結果をGFF形式でも出力します。これは後の遺伝子予測などで役立ちます。
      * `-pa 8`: 並列計算に使用するCPUコア（スレッド）数を指定します。`RepeatModeler` と同様に、環境に合わせて調整してください。
      * `ragtag_polished_round1.fasta`: マスク対象のゲノムFASTAファイルです。

#### 主要な出力ファイル

このコマンドにより、以下のようなファイルが生成されます。

  * `ragtag_polished_round1.fasta.masked`: 繰り返し配列領域が小文字（ソフトマスキング）または'N'（ハードマスキング、デフォルトはソフト）に置換されたFASTAファイル。
  * `ragtag_polished_round1.fasta.out`: マスクされた各リピート領域の詳細なアラインメント情報。
  * `ragtag_polished_round1.fasta.tbl`: マスクされたリピートのサマリーテーブル。
  * `ragtag_polished_round1.fasta.gff`: GFF3形式のマスク情報。

以上が、`ragtag_polished_round1.fasta`に対してRepeatModelerとRepeatMaskerを実行する一連の手順です。