```bash
cat RM_2139892.FriSep191125262025/round-2/family-*.fa \
RM_2139892.FriSep191125262025/round-3/family-*.fa \
RM_2139892.FriSep191125262025/round-4/family-*.fa > custom_repeats_canu.fa
```

```bash
RepeatMasker -pa 4 -lib custom_repeats_canu.fa -dir RM_masked_output_canu assembly2/assembly-results/ragtag_canu_scaffold/canu_polished_round1.fasta
```

![[Pasted image 20250926150234.png]]

LTR retrieverとTEsolverで漏れた要素（特にDNAトランスポゾンや非LTR型、novel構造）は、以下の追加ツールで検出・補強できます:[nature+2](https://www.nature.com/articles/s42003-023-05322-y)

## 追加で有効なde novo検出ツール

- **RepeatModeler2**
    
    - 幅広いTEタイプ（DNAトランスポゾン、非LTR型、未知repeat含む）に対応。[pnas](https://www.pnas.org/doi/10.1073/pnas.1921046117)
        
    - LTRharvest・LTR_retrieverモジュールも内蔵されており、annotation精度が高い。[pnas](https://www.pnas.org/doi/10.1073/pnas.1921046117)
        
- **EDTA（Extensive de novo TE Annotator）**
    
    - LTR型（LTR_FINDER, LTRharvest, LTR_retriever）だけでなく、TIR型（DNAトランスポゾン）、Helitronなど複数タイプのde novo検出を統合したパイプライン。[onlinelibrary.wiley+1](https://onlinelibrary.wiley.com/doi/10.1111/1755-0998.13489)
        
    - 非LTR型や領域横断的なrepeatも感度高く検出。
        
- **RepeatScout / RECON**
    
    - RepeatModeler2で自動利用されるコアde novo検出（高頻度k-merベース）で、まれなnovel要素やDNA型に強い。[nature](https://www.nature.com/articles/s42003-023-05322-y)
        

---

## カバー範囲比較

|ツール|LTR型|DNA型|非LTR型|未知/novel|
|---|---|---|---|---|
|LTR retriever|○|×|×|×|
|TEsolver|○|○|△|△|
|RepeatModeler2|○|○|○|○|
|EDTA|○|○|○|○|

LTR retriever＋TEsolver＋RepeatModeler2 or EDTA の組み合わせでほぼ全て網羅可能です。[onlinelibrary.wiley+2](https://onlinelibrary.wiley.com/doi/10.1111/1755-0998.13489)  
特に、**RepeatModeler2**と**EDTA**は、今の方法でカバーできていないDNA型・非LTR型・未知repeatのde novo同定に有効です。

---

**補足**

- Dictyostelium discoideumではTdd、DDTなどDNA型トランスポゾンやtRNA関連非LTR型（TRE5/3など）も重要マーカーです。[academic.oup+1](https://academic.oup.com/nar/article/39/15/6608/1019502)
    
- 非LTR型（例：TREファミリー）は一般のLTRツールでは検出困難なので、RepeatModeler2かEDTAの追加利用が必須です。
    

---

**結論:**  
LTR retriever＋TEsolver後の漏れは、**RepeatModeler2**または**EDTA**を使うことで、網羅的に取得できます。LTR retrieverとTEsolverで検出できなかったトランスポゾン（主にDNA型や非LTR型）は、RepeatModeler2やEDTA（Extensive de novo TE Annotator）で補完可能です。これらはDNA型や未知repeatにも強く、多様なタイプのTE検出に対応しています。[nature+2](https://www.nature.com/articles/s42003-023-05322-y)

- RepeatModeler2：DNA型、非LTR型、未知repeatも網羅
    
- EDTA：LTR型、DNA型、Helitron含む多様なカテゴリを統合検出
    

これらのツールを追加活用することで、漏れたセントロメア構成要素まで高精度でde novo検出できます。[pnas+2](https://www.pnas.org/doi/10.1073/pnas.1921046117)

1. [https://www.nature.com/articles/s42003-023-05322-y](https://www.nature.com/articles/s42003-023-05322-y)
2. [https://www.pnas.org/doi/10.1073/pnas.1921046117](https://www.pnas.org/doi/10.1073/pnas.1921046117)
3. [https://onlinelibrary.wiley.com/doi/10.1111/1755-0998.13489](https://onlinelibrary.wiley.com/doi/10.1111/1755-0998.13489)
4. [https://academic.oup.com/nar/article/39/15/6608/1019502](https://academic.oup.com/nar/article/39/15/6608/1019502)
5. [https://www.jstage.jst.go.jp/article/ggs/94/6/94_18-00024/_html/-char/ja](https://www.jstage.jst.go.jp/article/ggs/94/6/94_18-00024/_html/-char/ja)
6. [https://kazumaxneo.hatenablog.com/entry/2019/06/12/073000](https://kazumaxneo.hatenablog.com/entry/2019/06/12/073000)
7. [https://www.frontiersin.org/journals/plant-science/articles/10.3389/fpls.2023.1237426/full](https://www.frontiersin.org/journals/plant-science/articles/10.3389/fpls.2023.1237426/full)
8. [https://kazumaxneo.hatenablog.com/entry/2020/11/06/073000](https://kazumaxneo.hatenablog.com/entry/2020/11/06/073000)
9. [https://kazumaxneo.hatenablog.com/entry/2020/11/07/165013](https://kazumaxneo.hatenablog.com/entry/2020/11/07/165013)
10. [https://pmc.ncbi.nlm.nih.gov/articles/PMC6547461/](https://pmc.ncbi.nlm.nih.gov/articles/PMC6547461/)
11. [https://bibgraph.hpcr.jp/abst/pubmed/31892957](https://bibgraph.hpcr.jp/abst/pubmed/31892957)
12. [https://www.frontiersin.org/journals/microbiology/articles/10.3389/fmicb.2017.01869/full](https://www.frontiersin.org/journals/microbiology/articles/10.3389/fmicb.2017.01869/full)
13. [https://kazumaxneo.hatenablog.com/archive/category/transposon](https://kazumaxneo.hatenablog.com/archive/category/transposon)
14. [https://pmc.ncbi.nlm.nih.gov/articles/PMC10876081/](https://pmc.ncbi.nlm.nih.gov/articles/PMC10876081/)
15. [https://pmc.ncbi.nlm.nih.gov/articles/PMC311061/](https://pmc.ncbi.nlm.nih.gov/articles/PMC311061/)
16. [https://kazumaxneo.hatenablog.com/entry/2021/08/02/231056](https://kazumaxneo.hatenablog.com/entry/2021/08/02/231056)
17. [https://www.nature.com/articles/s41467-025-63889-y](https://www.nature.com/articles/s41467-025-63889-y)
18. [https://pmc.ncbi.nlm.nih.gov/articles/PMC1635306/](https://pmc.ncbi.nlm.nih.gov/articles/PMC1635306/)
19. [https://pmc.ncbi.nlm.nih.gov/articles/PMC5813529/](https://pmc.ncbi.nlm.nih.gov/articles/PMC5813529/)
20. [https://juganizo.com/?_=%2Fplphys%2Farticle%2F176%2F2%2F1410%2F6117145%23lirKEvczM7%2FdEMoEhcUYFLBxKorJTA%2Fm](https://juganizo.com/?_=%2Fplphys%2Farticle%2F176%2F2%2F1410%2F6117145%23lirKEvczM7%2FdEMoEhcUYFLBxKorJTA%2Fm)