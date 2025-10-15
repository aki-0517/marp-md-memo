flyeのNC_007089.4_RagTag_pilonはcanuのNC_007089.4_RagTag_pilonとNC_007090.3_RagTag_pilonがマージされてしまっているので、下の手順でやるとNC_007090.3_RagTag_pilonの部分の相同性が見れないので、ここもみれるようにする方法はある？

```bash

PREFIX="flye_vs_canu"
REF="assembly2/assembly-results/ragtag_flye_scaffold/ragtag_polished_round1.fasta"
QRY="assembly2/assembly-results/ragtag_canu_scaffold/canu_polished_round1.fasta"
OUT_PREFIX="flye_vs_canu_syri_gtog" # 出力プレフィックスを g-to-g に変更

delta-filter -g ${PREFIX}.delta > ${PREFIX}.g-to-g.filter.delta && \
show-coords -THrd ${PREFIX}.g-to-g.filter.delta > ${PREFIX}.g-to-g.coords && \
syri -c ${PREFIX}.g-to-g.coords \
     -d ${PREFIX}.g-to-g.filter.delta \
     -r ${REF} \
     -q ${QRY} \
     --prefix ${OUT_PREFIX} --no-chrmatch && \
plotsr --sr ${OUT_PREFIX}syri.out \
       --genomes genomes.txt \
       -H 10 -W 15 \
       -o flye_vs_canu_plot_gtog.pdf 
```



---

### ## 手順1：オリジナルのアラインメント結果（.delta）に相同性が含まれているか確認する

まず、`delta-filter` をかける前の、`nucmer` が出力した生のアラインメントファイル (`${PREFIX}.delta`) に、Flye の `NC_007089.4` と Canu の `NC_007090.3` の対応関係がそもそも含まれているかを確認します。

`show-coords` を使って、該当するコンティグ間のアラインメント情報だけを抽出してみます。

Bash

```
# フィルタリング前の .delta ファイルに対して show-coords を実行し、
# grep で Flye の NC_007089.4 と Canu の NC_007090.3 の組み合わせを探す
show-coords -THrd ${PREFIX}.delta | grep "NC_007089.4_RagTag_pilon" | grep "NC_007090.3_RagTag_pilon"
```

#### **【結果の解釈】**

- 何か出力された場合：
    
    おめでとうございます！🎉 nucmer はアラインメントを正しく見つけています。問題は delta-filter か syri の段階にあります。この場合は 手順3 へ進んでください。
    

---

### ## 手順3：（アラインメントがある場合）`mummerplot`で視覚的に確認する

手順1でアラインメントの存在が確認できた場合、`syri` や `plotsr` がそれをどう解釈しているかが問題になります。

その前に、**最も確実な方法**として、フィルタリング前の `.delta` ファイルを `mummerplot` で直接プロットし、本当に1対2のアラインメントが起きているかを目で見て確認することをお勧めします。

Bash

```
# フィルタリング前の.deltaファイルを使い、PNG形式でドットプロットを出力
mummerplot --png -p ${PREFIX}_unfiltered ${PREFIX}.delta --layout
```

このプロットで、横軸のFlye `NC_007089.4` の領域から、縦軸のCanu `NC_007089.4` と `NC_007090.3` の2つの領域に向かって斜めの線が伸びていれば、アラインメントは存在しています。

もしこの図でアラインメントが確認できるのに `plotsr` で描画されないのであれば、それは `syri` がこの構造変化を「転座 (Translocation)」や「重複 (Duplication)」などと解釈し、`plotsr` のデフォルトのシンテニープロットでは意図した通りに表示されていない可能性があります。

### ## まとめ

まずは **手順1** の `grep` コマンドで、アラインメントの有無をチェックするのが最も効率的です。そこから、`nucmer` に戻るべきか、`syri` の挙動を詳しく見るべきかの判断ができます。


## mergeのやり方

quickmerge
https://github.com/mahulchak/quickmerge

longstitch
https://github.com/bcgsc/LongStitch

ragtag
https://github.com/malonge/RagTag

galaxyのtutorialにやり方あるかも？