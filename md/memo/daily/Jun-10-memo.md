
数値、referenceは全部書く
## Data

#### ONT
- Dicty_gDNA_NEB-2.fastq
#### Illumina
- ./Dicty-genome/Stationary_S1_R1.fastq.gz
- ./Dicty-genome/Stationary_S1_R2.fastq.gz

ONT, Illuminaの仕組みを理解する

DIcty
https://www.nature.com/articles/nature03481

reference genome...AX4
https://www.ncbi.nlm.nih.gov/datasets/genome/GCF_000004695.1/

Papers
https://genomebiology.biomedcentral.com/articles/10.1186/s13059-025-03578-7



- カバレッジが多いとエラーの回数が増えてそれが正しいと誤認識してしまう。

## QC

**pycoQC**

- Oxford Nanopore Technologies（ONT）のシーケンスデータの品質評価を行うツール。
- 主にONTの出力ファイル（sequencing_summary.txtなど）を入力として、リード長、リード数、クオリティスコア、各チャンネルごとのパフォーマンスなど、ランの品質を可視化・レポート化する。
- HTML形式のインタラクティブなレポートを出力できる。
- ONT特有の情報を詳細に解析できるのが特徴。

**NanoPlot**

- 長鎖リード（主にOxford NanoporeやPacBio）のシーケンスデータの可視化・品質評価ツール
- 入力ファイルとしてFASTQ/FASTA、BAM、albacore summary、sequencing_summary.txtなどを受け付け、多様なプロット（リード長ヒストグラム、累積リード数プロット、リード長×クオリティのバイバリアントプロット、バイオリンプロットなど）を作成する
- サンプルごとの比較（NanoComp）や、リードのダウンサンプリング、クオリティ・長さによるフィルタリングも可能
- ウェブサービスやコマンドラインで利用でき、HTMLレポートも出力できる

- pycoQCとNanoPlotは特にロングリード（ONTやPacBio）向け、FastQCはショートリード向けですが、用途によって使い分けや併用が可能です。



- 違うsoftwareの結果でcompare
- SNP