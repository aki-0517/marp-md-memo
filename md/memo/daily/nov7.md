
## chromesome4 gtog
```bash
#!/bin/bash

# --- 変数の設定 ---
REF_ORIG="assembly2/assembly-results/ragtag_flye_scaffold/ragtag_polished_round1.fasta"
QRY_ORIG="assembly2/assembly-results/ragtag_canu_scaffold/canu_polished_round1.fasta"
REF_CONTIG="NC_007089.4_RagTag_pilon"
QRY_CONTIG="NC_007090.3_RagTag_pilon"
REF_TARGET="flye_NC_007089.4.fasta"
QRY_TARGET="canu_NC_007090.3.fasta"
# 逆相補鎖にしたクエリFASTA用の新しいファイル名
QRY_TARGET_RC="canu_NC_007090.3.rc.fasta"
PREFIX_TARGET="flye89_vs_canu90"
OUT_PREFIX_TARGET="syri_flye89_vs_canu90"
GENOMES_FILE="genomes_target.txt"
OUTPUT_PDF="plot_flye89_vs_canu90_gtog.pdf"

# --- 以下、コマンド本体 ---
echo "--- Step 1: Extracting FASTA sequences and applying reverse complement ---" && \
samtools faidx ${REF_ORIG} ${REF_CONTIG} > ${REF_TARGET} && \
samtools faidx ${QRY_ORIG} ${QRY_CONTIG} > ${QRY_TARGET} && \

echo "--- Applying reverse complement to query contig ---" && \
seqtk seq -r ${QRY_TARGET} > ${QRY_TARGET_RC} && \

echo "--- Step 2: Creating genomes file for plotsr ---" && \
echo -e "${QRY_TARGET_RC}\n${REF_TARGET}" > ${GENOMES_FILE} && \

echo "--- Step 3: Running nucmer, syri, and plotsr pipeline (g-to-g mode) ---" && \
nucmer --maxmatch --prefix=${PREFIX_TARGET} ${REF_TARGET} ${QRY_TARGET_RC} && \

# --- g-to-g filtering ---
delta-filter -g ${PREFIX_TARGET}.delta > ${PREFIX_TARGET}.g-to-g.filter.delta && \
show-coords -THrd ${PREFIX_TARGET}.g-to-g.filter.delta > ${PREFIX_TARGET}.g-to-g.coords && \

# --- Run SyRI ---
syri -c ${PREFIX_TARGET}.g-to-g.coords \
     -d ${PREFIX_TARGET}.g-to-g.filter.delta \
     -r ${REF_TARGET} \
     -q ${QRY_TARGET_RC} \
     --prefix ${OUT_PREFIX_TARGET} \
     --no-chrmatch && \

# --- Visualization with plotsr ---
plotsr --sr ${OUT_PREFIX_TARGET}syri.out \
       --genomes ${GENOMES_FILE} \
       -H 10 -W 15 \
       -o ${OUTPUT_PDF} && \

echo "--- Pipeline finished successfully! Check ${OUTPUT_PDF} ---"
```


6つの染色体を持つ生物の生物のgenome assemblyをして(discoideum ax2)、flyeとcanuで行なって、canuは6つちゃんとできたが、反復領域のせいで、flyeはchromesome3と4がマージしてしまった。
flye: assembly2/assembly-results/ragtag_flye_scaffold/ragtag_polished_round1.fasta
canu: assembly2/assembly-results/ragtag_canu_scaffold/canu_polished_round1.fasta

なので、これを適切な場所で切り離したい

canuもしくはref(反復領域は含まれてない)を使って、適切なところで切り取る方法は？

手動じゃなくて、プログラムで自動に切りたい。

canuのデータを使って、flye切る。

chromesome3がflyeでは逆向きになってchromesome4とくっついてしまってるので、それも考慮して、手順を教えて。condaで。コマンド上で完結させて
chromesome3のヘッダ: >NC_007089.4_RagTag_pilon
chromesome4のヘッダ: >NC_007090.3_RagTag_pilon


![[Pasted image 20251107104415.png]]
![[Pasted image 20251107104554.png]]

## canu chromesome3
![[Pasted image 20251107104936.png]]

## canu chromesome4
![[Pasted image 20251107105129.png]]




flye-fixed
![[Pasted image 20251107151302.png]]


```bash
minimap2 -ax map-ont \
  flye_repaired_assembly_v4.5_final/flye_assembly_FIXED_v4.5.fasta \
  Dicty_gDNA_NEB-2.fastq | \
  samtools view -bS - | \
  samtools sort -o mapping_flye_fixed.sorted.bam && \
samtools index mapping_flye_fixed.sorted.bam
```

https://www.sciencedirect.com/science/article/pii/S0960982225005081?via%3Dihub#abs0015



次
scp aki@100.84.9.80:mapping_flye_fixed.sorted.bam .
scp aki@100.84.9.80:mapping_flye_fixed.sorted.bam.bai .