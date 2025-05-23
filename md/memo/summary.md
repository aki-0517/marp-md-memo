# Dictyostelium discoideum ゲノムアセンブリ・ポリッシングまとめ

## 背景知識

- **核染色体数**: 6本
- **extrachromosomal palindrome（rDNA）**: 約88kbの線状パリンドロームが約100コピー、核DNAの約20%を占める
- **ミトコンドリアゲノム**: 約55.6kbが約200コピー
- **リプリコン数**: 6本の核染色体 + rDNAパリンドローム + ミトコンドリアゲノム = 計8本（理想的なcontig数）

## 使用データ

- ONT: `Dicty_gDNA_NEB-2.fastq`
- Illumina: `./Dicty-genome/Stationary_S1_R1.fastq.gz`, `./Dicty-genome/Stationary_S1_R2.fastq.gz`
- サブセット: `Dicty_gDNA_NEB-2_half.fastq.gz`

## ONTリード長分布の可視化

### Data

- ONT

![read-length](../public/read-length/image.png)

### Result

```bash
stats for Dicty_gDNA_NEB-2.fastq
sum = 8359638019, n = 934886, ave = 8941.88, largest = 139714
N50 = 12777, n = 188908
N60 = 10380, n = 261703
N70 = 8415, n = 351177
N80 = 6569, n = 463244
N90 = 4537, n = 614607
N100 = 19, n = 934886
N_count = 0
Gaps = 0
```
※ 現在のプロジェクトはDictyostelium discoideumのゲノムアセンブリに焦点を当てており、ONTデータが主要なシーケンスデータとして使用されています。

## アセンブリ手法

- **Flye**
- **Canu**
- **Raven**
- **Shasta**

### Quastによる評価

| 指標                | Raven   | Flye    | Shasta   | Canu     |
|---------------------|---------|---------|----------|----------|
| # contigs           | 28      | 33      | 36       | 14       |
| Largest contig      | 5,803,050 | 5,006,538 | 12,082,896 | 8,736,265 |
| Total length        | 35,504,430 | 34,319,985 | 33,471,541 | 34,572,512 |
| N50                 | 2,694,088 | 2,870,802 | 6,719,217 | 3,611,137 |
| L50                 | 5       | 5       | 2        | 3        |
| GC (%)              | 22.77   | 22.98   | 22.90    | 23.06    |

- Canuはcontig数が最も少なく（14本）、N50や最大長も高水準で、全体のバランスが良い。
- Shastaは最大contig長・N50が最も大きいが、contig数が多い。
- FlyeやRavenはcontig数が多く、N50もやや低い。

→ 総合的にCanuが最も精度が高いと判断される。

## ポリッシング手法

- **Pilon**（ONT, Illuminaリードで2ラウンド）
  - 精度向上は限定的
- **Medaka**（ONTリードで1ラウンド）

## 現状

- contig数: 14
- 目標: 8（6染色体+1rDNA+1ミトコンドリア）
- 現状ベスト: Canu（50%）、Pilon（ONT50%+Illumina50%）

## 今後の方針

1. **他のライブラリでpolishingを試す**
   - 現状ベスト（Canu→Pilon）以外のpolishingツールやパラメータを検討
2. **Scaffoldingでcontig数削減を目指す**
   - ベストなアセンブリ結果をもとにscaffoldingツールを適用し、目標の8本に近づける
3. **マルチアセンブリ戦略の検討**
   - 切れ目の異なる複数アセンブリを統合し、より長いcontigを目指す

---

### 参考: 主なアセンブリ・ポリッシングコマンド

#### Flye
```bash
flye --nano-raw Dicty_gDNA_NEB-2.fastq.gz -g 34m -o flye_assembly -t 8
```
#### Canu
```bash
canu -p dicty_canu -d canu_assembly genomeSize=34m -nanopore-raw Dicty_gDNA_NEB-2_half.fastq.gz maxThreads=8 maxMemory=32g useGrid=false gnuplotTested=true
```
#### Raven
```bash
raven -t 8 Dicty_gDNA_NEB-2_half.fastq.gz > raven_assembly.fasta
```
#### Shasta
```bash
shasta --input Dicty_gDNA_NEB-2_half.fastq --config Nanopore-Dec2019 --threads 8 --assemblyDirectory shasta_assembly
```
#### Quast
```bash
quast.py ./raven_assembly.fasta flye_assembly/assembly.fasta shasta_assembly/Assembly.fasta canu_assembly/dicty_canu.contigs.fasta -o quast_results_no_ref -t 8 -l Raven,Flye,Shasta,Canu
```
#### Pilon
```bash
java -Xmx64G -jar pilon.jar --genome canu_assembly/dicty_canu.contigs.fasta --bam merged.bam --output pilon_round1_snps_indels --fix snps,indels
```
#### Medaka
```bash
medaka_consensus -i Dicty_gDNA_NEB-2.fastq -d pilon_round2_polished.fasta -o medaka_out -t 8 -m r941_min_sup_g507
```

---

（このファイルは今後の進捗や考察も随時追記してください） 