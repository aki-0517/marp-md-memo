# test

flyeとcanuでアセンブリした2つのfasta（flye_assembly_FIXED_v4.5.fasta, canu_polished_round1.fasta）をquickmergeでマージし、最終的なベストなアセンブリを得る手順を、最新情報に基づきまとめます。

---

## quickmergeのインストール（conda）

1. conda環境（もしくはmamba）をセットアップします。
    
    - MinicondaやAnacondaが未インストールの場合は公式サイトから入手。
        
2. biocondaチャンネルを有効にして、quickmergeをインストールします。
    

bash

`conda install -c conda-forge -c bioconda quickmerge`

---

## アセンブリファイルの準備

- flye: `flye_repaired_assembly_v4.5_final/flye_assembly_FIXED_v4.5.fasta`
    
- canu: `assembly2/assembly-results/ragtag_canu_scaffold/canu_polished_round1.fasta`
    

（mummerも依存として必要ですが、bioconda経由でインストールされます）

---

## マージ作業の推奨手順

1. **nucmerによるアライメント**
    
    - まず、2つのfastaをnucmerでアライメントします。
        
    - "self assembly"（canu）と"hybrid assembly"（flye）という順で指定。
        
    - オプションは適宜調整。
        

bash

`nucmer -l 100 -prefix out assembly2/assembly-results/ragtag_canu_scaffold/canu_polished_round1.fasta flye_repaired_assembly_v4.5_final/flye_assembly_FIXED_v4.5.fasta`

2. **delta-filterで繰り返しや重複のフィルタリング**
    

bash

`delta-filter -r -q -l 10000 out.delta > out.rq.delta`

3. **quickmergeによるマージ**
    
    - ハイブリッド（flye）を-q, self（canu）を-rに指定。
        
    - 詳細パラメータやcutoff値はN50やリピートの量に応じて調整。
        
    - 一例（N50=2Mbの場合）
        

bash

`quickmerge -d out.rq.delta -q flye_repaired_assembly_v4.5_final/flye_assembly_FIXED_v4.5.fasta -r assembly2/assembly-results/ragtag_canu_scaffold/canu_polished_round1.fasta -hco 5.0 -c 1.5 -l 2000000 -ml 10000 -p final_merge`

- "-hco"（anchor contig overlap cutoff）と"-c"（contig extension cutoff）はデフォルト値で十分ですが、N50やrepeat量に合わせて調節。
    
- "-l"（anchor contig length cutoff）は自己アセンブリ（canu）のN50推奨。
    
- "-ml"（min alignment length）はrepeat多い場合は高め推奨。
    

[github+2](https://github.com/mahulchak/quickmerge)​

```bash
delta-filter -r -q -l 20000 out.delta > out.rq.delta

quickmerge -d out.rq.delta \
  -q flye_repaired_assembly_v4.5_final/flye_assembly_FIXED_v4.5.fasta \
  -r assembly2/assembly-results/ragtag_canu_scaffold/canu_polished_round1.fasta \
  -hco 7.0 -c 2.0 -l 2000000 -ml 15000 -p final_merge_strict

```

---

## 補足

- 順序としてself assembly（canu）→hybrid（flye）となります（mergeコマンドでは-q/-rの順）。
    
- マージ後は、検証のためmummerplotでdot plotを確認するとよいでしょう。
    
- 出力アセンブリを最終チェック・ポリッシュしたい場合、FinisherSCやQuiver, Pilonなどを併用可能です。
    
- 結果のfastaヘッダーに空白がある場合は除去（自動修正機能あり）。
    

[kazumaxneo.hatenablog+1](https://kazumaxneo.hatenablog.com/entry/2017/10/26/010151)​

---

この手順で最新quickmergeを使ったflyeとcanuのアセンブリ統合が可能です。パラメータ調整や出力検証も忘れず行ってください。

Add to follow-up

Check sources

1. [https://github.com/mahulchak/quickmerge](https://github.com/mahulchak/quickmerge)
2. [https://kazumaxneo.hatenablog.com/entry/2017/10/26/010151](https://kazumaxneo.hatenablog.com/entry/2017/10/26/010151)
3. [https://www.nature.com/articles/s41587-020-0422-6](https://www.nature.com/articles/s41587-020-0422-6)
4. [https://anaconda.org/bioconda/quickmerge](https://anaconda.org/bioconda/quickmerge)
5. [https://bioconda.github.io/recipes/quickmerge/README.html](https://bioconda.github.io/recipes/quickmerge/README.html)
6. [https://actg-wgaa.readthedocs.io/en/latest/Setup/software/conda.html](https://actg-wgaa.readthedocs.io/en/latest/Setup/software/conda.html)
7. [https://juser.fz-juelich.de/record/916537/files/The%20Plant%20Journal%20-%202022%20-%20Rengs%20-%20A%20chromosome%20scale%20tomato%20genome%20built%20from%20complementary%20PacBio%20and%20Nanopore%20sequences.pdf](https://juser.fz-juelich.de/record/916537/files/The%20Plant%20Journal%20-%202022%20-%20Rengs%20-%20A%20chromosome%20scale%20tomato%20genome%20built%20from%20complementary%20PacBio%20and%20Nanopore%20sequences.pdf)
8. [https://omicstutorials.com/step-by-step-guide-combining-fasta-files/](https://omicstutorials.com/step-by-step-guide-combining-fasta-files/)
9. [https://pypi.org/project/conda-merge/](https://pypi.org/project/conda-merge/)
10. [https://www.biorxiv.org/content/biorxiv/early/2020/10/28/2020.10.28.358903/DC4/embed/media-4.docx?download=true](https://www.biorxiv.org/content/biorxiv/early/2020/10/28/2020.10.28.358903/DC4/embed/media-4.docx?download=true)
11. [https://stackoverflow.com/questions/79712446/merge-many-fasta-files](https://stackoverflow.com/questions/79712446/merge-many-fasta-files)
12. [https://stackoverflow.com/questions/42352841/how-to-update-an-existing-conda-environment-with-a-yml-file](https://stackoverflow.com/questions/42352841/how-to-update-an-existing-conda-environment-with-a-yml-file)
13. [https://qiita.com/satoshi_kawato/items/01c2d9d3b7e68039cff8](https://qiita.com/satoshi_kawato/items/01c2d9d3b7e68039cff8)
14. [https://academic.oup.com/nar/article/44/19/e147/2468393](https://academic.oup.com/nar/article/44/19/e147/2468393)
15. [https://docs.conda.io/projects/conda/en/stable/user-guide/install/index.html](https://docs.conda.io/projects/conda/en/stable/user-guide/install/index.html)
16. [http://emersonlab.org/documents/ChakrabortyChang2020bioRxiv_simcomplex_v3.pdf](http://emersonlab.org/documents/ChakrabortyChang2020bioRxiv_simcomplex_v3.pdf)
17. [https://www.biostars.org/p/207178/](https://www.biostars.org/p/207178/)
18. [https://www.anaconda.com/docs/getting-started/miniconda/main](https://www.anaconda.com/docs/getting-started/miniconda/main)
19. [https://help.galaxyproject.org/t/how-to-merge-or-generate-fasta-file-with-the-same-chromosome/190](https://help.galaxyproject.org/t/how-to-merge-or-generate-fasta-file-with-the-same-chromosome/190)
20. [https://biohpc.cornell.edu/lab/userguide.aspx?a=software&i=574](https://biohpc.cornell.edu/lab/userguide.aspx?a=software&i=574)




# merged_final_merge.fasta