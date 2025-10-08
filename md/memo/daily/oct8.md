# Todo
- chromesome1の反復領域がなんで違うか調べる
- NC_007089.4_RagTag_pilonのかけてるところをcanuでもvisualizeできるようにする
- 足りてない他の小さいcontigがvisualizeされてないのはなぜ？

visualize前のoutput fileがあるので、それを見ればわかる(名前の違いかも)



ご指摘の通り、添付されたプロットはFlyeのアセンブリにおいて、本来は別々であるべき領域が単一のコンティグに誤ってマージされてしまった可能性を示唆していますね。このFlyeの「余分な」部分が、Canuアセンブリのどこに由来するのかを突き止め、可視化するための手順をご提案します。

現在実行された手順では、`delta-filter`に`-1`オプション（1対1の最良のアライメントのみを保持）を使用しているため、1つのFlyeコンティグが複数のCanuコンティグにまたがって相同性を持つ場合、最もスコアの高いペア以外の情報が失われてしまいます。

この問題を解決するには、**1対1ではなく多対多（many-to-many）のアライメントを許容する**ことで、単一のコンティグ内に存在する転座（translocation）や大規模な再編成を検出できます。これにより、まさに知りたかった「Flyeのコンティグの余分な部分と、それに対応するCanuのコンティグ」の関係性を明らかにできます。

以下に、既存のワークフローを少しだけ変更した、具体的な手順を示します。

---

### **解決策：多対多アライメントを用いた構造再編解析**

基本的な流れは同じですが、`delta-filter`のオプションを変更し、その後のSyRiとplotsrを再実行します。

#### **ステップ1：アライメントのフィルタリング（変更箇所）**

MUMmer4によるアライメント (`nucmer`) は同じものを再利用できます。変更するのは`delta-filter`のコマンドです。`-1`オプションの代わりに`-m`オプションを使用します。

Bash

```
# nucmerで生成した .delta ファイルは再利用
PREFIX="flye_vs_canu"

# delta-filterのオプションを -1 から -m に変更
# これにより、1対多や多対多のアライメントが保持される
delta-filter -m -q -r ${PREFIX}.delta > ${PREFIX}.m-to-m.filter.delta

# 座標情報の抽出（入力ファイル名を変更）
show-coords -THrd ${PREFIX}.m-to-m.filter.delta > ${PREFIX}.m-to-m.coords
```

- `-m`: 1対1（one-to-one）ではなく、多対多（many-to-many）のアライメントを許容してフィルタリングします。これにより、1つの参照コンティグが複数のクエリコンティグに対応するような、転座した領域のアライメントも保持されます。
    

#### **ステップ2：SyRiによる構造再編の同定（再実行）**

ステップ1で作成した新しい座標ファイル (`.m-to-m.filter.delta`, `.m-to-m.coords`) を使って、SyRiを再実行します。

Bash

```
# 分かりやすいように、出力の接頭辞も変更しておきましょう
REF="assembly2/assembly-results/ragtag_flye_scaffold/ragtag_polished_round1.fasta"
QRY="assembly2/assembly-results/ragtag_canu_scaffold/canu_polished_round1.fasta"
OUT_PREFIX="flye_vs_canu_syri_mtom"

syri -c ${PREFIX}.m-to-m.coords \
     -d ${PREFIX}.m-to-m.filter.delta \
     -r ${REF} \
     -q ${QRY} \
     --prefix ${OUT_PREFIX} \
     --no-chrmatch
```

- SyRiが多対多のアライメント情報を基に解析を行うため、コンティグ間の転座 (`TRANS`) として構造再編が正しく同定されることが期待されます。
    

#### **ステップ3：plotsrによる可視化（再実行）**

最後に、新しいSyRiの出力ファイルを使って`plotsr`を再実行します。`genomes.txt`やFASTAインデックスはそのまま使えます。

Bash

```
plotsr --sr ${OUT_PREFIX}.out \
       --genomes genomes.txt \
       -H 10 -W 15 \
       -o flye_vs_canu_plot_mtom.pdf
```

### **期待される結果**

この手順で生成された新しいPDFファイル (`flye_vs_canu_plot_mtom.pdf`) を確認してください。

元のプロットでFlye側だけが長く、比較相手がいなかった灰色領域（例：`NC_007089.4_RagTag_pilon`の右半分）に、**別のCanuコンティグとの相同性を示す色付きのブロック**（転座として検出されるため、多くはオレンジ色や青色で表示されます）が描画されるはずです。

これにより、「Flyeの長大なコンティグは、CanuのAというコンティグとBというコンティグが結合したものである」という仮説を視覚的に直接確認・証明することができます。ドットプロットを避けて、ご希望の形式で共線性（シンテニー）を示しつつ、問題の領域の対応関係を明らかにすることが可能です。


```bash
# --- 変数の設定（ご自身の環境に合わせてパスを修正してください） ---
PREFIX="flye_vs_canu"
REF="assembly2/assembly-results/ragtag_flye_scaffold/ragtag_polished_round1.fasta"
QRY="assembly2/assembly-results/ragtag_canu_scaffold/canu_polished_round1.fasta"
OUT_PREFIX="flye_vs_canu_syri_mtom"

# --- 以下を1コマンドとして実行 ---
delta-filter -m -q -r ${PREFIX}.delta > ${PREFIX}.m-to-m.filter.delta && \
show-coords -THrd ${PREFIX}.m-to-m.filter.delta > ${PREFIX}.m-to-m.coords && \
syri -c ${PREFIX}.m-to-m.coords \
     -d ${PREFIX}.m-to-m.filter.delta \
     -r ${REF} \
     -q ${QRY} \
     --prefix ${OUT_PREFIX} --no-chrmatch && \
plotsr --sr ${OUT_PREFIX}.out \
       --genomes genomes.txt \
       -H 10 -W 15 \
       -o flye_vs_canu_plot_mtom.pdf
```