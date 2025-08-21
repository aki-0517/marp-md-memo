承知しました。前と後で1つずつ、主要指標のみの表にまとめました。

### 前（`assembly2/quast/quast_polished2/report.txt`）
| Parameter                | Flye_Medaka | Canu_Medaka |
| ------------------------ | ----------: | ----------: |
| # contigs (>= 0 bp)      |          16 |          41 |
| Total length (>= 0 bp)   |     34875198 |     36010564 |
| N50                      |      3394368 |      3355856 |
| Genome fraction (%)      |      97.171 |      97.082 |
| Duplication ratio        |       1.034 |       1.067 |
| # mismatches per 100 kbp |      141.98 |      144.04 |
| # indels per 100 kbp     |      211.43 |      217.18 |
| # misassemblies          |         669 |         726 |
| # N's per 100 kbp        |        0.00 |        0.00 |

### 後（`assembly2/quast/scaffolding-pilon/report.txt`）
| Parameter                | Flye_Polished | Canu_Polished |
| ------------------------ | ------------: | ------------: |
| # contigs (>= 0 bp)      | <span style="color: green">11 (-5)</span> | <span style="color: green">29 (-12)</span> |
| Total length (>= 0 bp)   | <span style="color: green">34861082 (-14116)</span> | <span style="color: green">35997303 (-13261)</span> |
| N50                      | <span style="color: green">8060236 (+4665868)</span> | <span style="color: green">5606788 (+2250932)</span> |
| Genome fraction (%)      | <span style="color: green">97.172 (+0.001)</span> | <span style="color: red">97.059 (-0.023)</span> |
| Duplication ratio        | <span style="color: red">1.035 (+0.001)</span> | <span style="color: green">1.066 (-0.001)</span> |
| # mismatches per 100 kbp | <span style="color: green">139.15 (-2.83)</span> | <span style="color: green">142.25 (-1.79)</span> |
| # indels per 100 kbp     | <span style="color: green">109.09 (-102.34)</span> | <span style="color: green">117.35 (-99.83)</span> |
| # misassemblies          | <span style="color: red">675 (+6)</span> | <span style="color: red">741 (+15)</span> |
| # N's per 100 kbp        | <span style="color: red">1.26 (+1.26)</span> | <span style="color: red">2.18 (+2.18)</span> |

- 前後それぞれで、FlyeとCanuの主要精度指標を最小限で比較できるように整理しました。



- 染色体レベルのAX2アセンブリが完成すれば、それを基準として、AX4株やその他の実験株、さらには野外から単離された野生株との間で、ゲノム構造を詳細に比較することが可能になる。これにより、重複、逆位、転座といった大規模な構造多型（SV）を染色体全体にわたって正確に同定できる 。例えば、AX2株に特有の染色体1上の増幅 のような違いを、ゲノムワイドに、かつ塩基レベルの解像度で明らかにすることができる。これは、実験室環境下での株の適応進化や、株間で観察される表現型の違いの遺伝的基盤を理解する上で、極めて重要な情報を提供する。


tgs gap closer
https://github.com/BGI-Qingdao/TGS-GapCloser

chip-seq

Hi-C