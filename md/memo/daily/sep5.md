
https://www.dfam.org/releases/Dfam_3.9/families/FamDB/


---

# 377189.pts-54.tardis
---

## 1️⃣ 前提

- ゲノム FASTA: assembly2/assembly-results/ragtag_flye_scaffold/ragtag_polished_round1.fasta
    
- Dfam ライブラリ: `/home/aki/Dfam-RepeatMasker.lib`
    
- DIRS 検出結果の出力先: `DIRS_masked_output/`
    

---

## 2️⃣ RepeatMasker 実行

```bash
RepeatMasker -pa 4 -lib /home/aki/Dfam-RepeatMasker.lib   -gff -dir DIRS_masked_output -engine ncbi assembly2/assembly-results/ragtag_flye_scaffold/ragtag_polished_round1.fasta
```

- `-pa 4` → 並列 4 スレッド
    
- `-lib` → Dfam ライブラリを指定
    
- `-gff` → 出力を GFF3 形式に
    
- `-dir` → 出力ディレクトリ
    

これで `DIRS_masked_output/genome.fasta.out.gff` が作られます。

---

## 3️⃣ DIRS のみ抽出

GFF の `type` カラムに `DIRS` が含まれている場合、それだけ抜き出します：

```bash
grep -i "DIRS" DIRS_masked_output/ragtag_polished_round1.fasta.out.gff > DIRS_masked_output/ragtag.DIRS.gff
```

---

## 4️⃣ BED 形式に変換（必要なら）

BED 形式は start/end が必須なので、`gff2bed` を使います：

```bash
gff2bed < DIRS_masked_output/ragtag.DIRS.gff > DIRS_masked_output/ragtag.DIRS.bed
```

- これで BED ファイルが完成
    
- IGV などで可視化可能
    

---

## 5️⃣ 確認

```bash
head DIRS_masked_output/ragtag.DIRS.bed
```

- start, end, scaffold/chromosome, DIRS 名が出力されるはずです
    

---

💡 注意点

- Dfam ライブラリに **DIRS モデルが含まれていないと検出されません**
    
- `grep "DIRS"` の代わりに `grep -i "DIRS"` にすると大文字小文字を無視できます
    


```bash
(base) [aki@tardis ~]$ gff2bed < DIRS_masked_output/ragtag.DIRS.gff > DIRS_masked_output/ragtag.DIRS.bed
```


# chromesome1 - DIRS
---

## 1️⃣ RepeatMasker 実行

```bash
RepeatMasker -pa 4 -lib /home/aki/Dfam-RepeatMasker.lib -gff -dir DIRS_masked_output_chr1 -engine ncbi chromosome1.fna
```

- `-pa 4` → 並列 4 スレッド
    
- `-lib` → Dfam ライブラリを指定
    
- `-gff` → GFF3 形式で出力
    
- `-dir` → 出力ディレクトリ
    

出力は `DIRS_masked_output_chr1/chromosome1.fna.out.gff` になります。

---

## 2️⃣ DIRS のみ抽出

GFF 出力から `DIRS` タイプの行だけを抽出します：

```bash
grep -i "DIRS" DIRS_masked_output_chr1/chromosome1.fna.out.gff > DIRS_masked_output_chr1/chromosome1.DIRS.gff
```

---

💡 ポイント：

- chromosome1.fna のみを対象にしているので、出力 GFF の `seqid` も chromosome 1 のみに対応します。
    
- 出力ディレクトリは chromosome ごとに分けておくと整理しやすいです。
    

---


```bash

(base) [aki@tardis ~]$ gff2bed < DIRS_masked_output_chr1/chromosome1.DIRS.gff > DIRS_masked_output_chr1/chromosome1.DIRS.bed
```





---

### 重要な観察点

```
ID=1;Target "Motif:(A)n" 1 28
ID=8;Target "Motif:TE2335_SO2_FAM0398" 23 147
```

- `Target` のあとに `"Motif:～"` が書かれている
    
- これが **リピートのファミリー/モチーフ名** になります
    
- GFF の 3列目は全部 `dispersed_repeat` なので無視でOK
    
- 欲しいのは `"Motif:..."` の部分だけ
    

---

### 「DIRS 以外に何があるか」を見る正しいやり方

**Motif 名（リピート名）を取り出して集計する**のが正解です。

```bash
grep -v "^#" DIRS_masked_output_chr1/chromosome1.fna.out.gff \
  | cut -f9 \
  | sed -n 's/.*Target "\(Motif:[^"]*\)".*/\1/p' \
  | sort \
  | uniq -c \
  | awk '$1 >= 10'
```

```bash
grep -v "^#" DIRS_masked_output_chr1/chromosome1.fna.out.gff \
  | cut -f9 \
  | sed -n 's/.*Target "\(Motif:[^"]*\)".*/\1/p' \
  | sort \
  | uniq -c

```

---

### 処理の流れ

1. `cut -f9` → 属性フィールドだけ取る
    
2. `sed -n 's/.*Target "\(Motif:[^"]*\)".*/\1/p'`
    
    - `Target "Motif:XXXX"` の `Motif:XXXX` 部分を抽出
        
3. `sort | uniq -c` → 種類ごとの出現数をカウント
    
4. `awk '$1 >= 10'` → 10以上のものだけ表示
    

---

### 出力イメージ

```
   45 Motif:(A)n
   37 Motif:(ATT)n
   22 Motif:(TATT)n
   85 Motif:TE2335_SO2_FAM0398
   14 Motif:DIRS-N2
```

---

👉 こうすれば **DIRS 以外に何が繰り返し出ているか（モチーフ名・ファミリー名ごと）** がはっきり見えます。

---



# chromwsome1 - ltr
`chromosome1.fna.out.gff` から **LTR 系だけを抽出して BED に変換** する流れを説明します。

---

## 1️⃣ GFF の中身を確認

RepeatMasker の GFF はこんな感じでしたね：

```
NC_007087.3  RepeatMasker  dispersed_repeat  1033 1060  0.0  +  .  ID=1;Target "Motif:(A)n" 1 28
NC_007087.3  RepeatMasker  dispersed_repeat  3361 3423  7.1  +  .  ID=4;Target "Motif:DIRS-N2" 1 64
...
```

9列目 (`Target "Motif:..."`) のところに `LTR` という語が入ったモチーフが LTR 系です。

---

## 2️⃣ LTR だけを抽出

```bash
grep -i "LTR" DIRS_masked_output_chr1/chromosome1.fna.out.gff > DIRS_masked_output_chr1/chromosome1.LTR.gff
```

---

## 3️⃣ BED 形式に変換

BED の基本形式は：

```
seqid  start  end  name  score  strand
```

なので GFF から必要なカラムを切り出して整形します。

```bash
grep -i "LTR" DIRS_masked_output_chr1/chromosome1.fna.out.gff \
  | awk '{OFS="\t"; print $1,$4-1,$5,$9,$6,$7}' \
  > DIRS_masked_output_chr1/chromosome1.LTR.bed
```

---

### 補足

- `$1` → seqid (例: NC_007087.3)
    
- `$4-1` → GFF は1-based、BEDは0-basedなので -1
    
- `$5` → end 座標
    
- `$9` → Motif名（ID/Target 情報）
    
- `$6` → スコア
    
- `$7` → ストランド
    

---

👉 これで `chromosome1.LTR.bed` が作れます。

---


なるほど、流れはほぼ正しいです 👍

ただ、いくつか確認ポイントがあります：

---

### 1️⃣ Skipper の BED が空っぽ？

```bash
head DIRS_masked_output_chr1/chromosome1.Skipper.bed
```

で何も出ていないということは、**chromosome1 の RepeatMasker 出力に Skipper がなかった**可能性が高いです。

- もしくは `"Skipper"` の大文字/小文字が合っていないか
    
- 別の Motif 名で出力されている（例: `Skipper-1` など）
    

確認方法：

```bash
grep -i "Skipper" DIRS_masked_output_chr1/chromosome1.fna.out.gff
```

ここでヒットする行があるか確認してください。

---

### 2️⃣ LINE の BED

```bash
grep -i "LINE" `DIRS_masked_output_chr1/chromosome1.fna.out.gff` \
  | awk '{OFS="\t"; print $1,$4-1,$5,$9,$6,$7}' \
  > chromosome1.LINE.bed
```

```bash
(base) [aki@tardis ~]$ head -10 chromosome1.LINE.bed
NC_007087.3     11770   11814   ID=12;Target    13.6    +
NC_007087.3     26263   26307   ID=26;Target    15.9    +
NC_007087.3     32022   32066   ID=31;Target    15.9    +
NC_007087.3     46276   46320   ID=61;Target    15.9    -
NC_007087.3     50456   50514   ID=64;Target    15.8    +
NC_007087.3     61727   61771   ID=72;Target    15.9    -
NC_007087.3     74713   74757   ID=91;Target    15.9    -
NC_007087.3     84344   84388   ID=105;Target   15.9    -
NC_007087.3     90005   90049   ID=111;Target   15.9    +
NC_007087.3     107928  107972  ID=131;Target   15.9    +
```

---

### 3️⃣ BED 生成のポイント

- 3列目（type）は RepeatMasker では `dispersed_repeat` になっているので、抽出は 9列目の Motif 名で行う
    
- Motif 名が `"Skipper"` と正確に書かれているか確認
    
- 大文字/小文字の違いに注意 → `-i` オプションで無視可能
    

---

💡 まとめると：

1. Skipper が本当に chromosome1 にあるかまず確認
    
2. なければ BED は空になるのが正しい
    
3. LINE や Simple_repeat は同じ方法で BED 化可能
    




### Dictyostelium firmibasisで試してみる

DIRSが多いchr1でやってみる

https://www.ncbi.nlm.nih.gov/datasets/genome/GCA_036169595.1/

```
ls GCA_036169595.1/ncbi_dataset/data/GCA_036169595.1

GCA_036169595.1_ASM3616959v1_genomic.fna  cds_from_genomic.fna  genomic.gff  protein.faa  sequence_report.jsonl

(base) [aki@tardis GCA_036169595.1]$ grep -n "^>" GCA_036169595.1_ASM3616959v1_genomic.fna
1:>CM069765.1 Dictyostelium firmibasis strain TNS-C-14 chromosome 1, whole genome shotgun sequence
117665:>CM069766.1 Dictyostelium firmibasis strain TNS-C-14 chromosome 2, whole genome shotgun sequence
181170:>CM069767.1 Dictyostelium firmibasis strain TNS-C-14 chromosome 3, whole genome shotgun sequence
242956:>CM069768.1 Dictyostelium firmibasis strain TNS-C-14 chromosome 4, whole genome shotgun sequence
297856:>CM069769.1 Dictyostelium firmibasis strain TNS-C-14 chromosome 5, whole genome shotgun sequence
351332:>CM069770.1 Dictyostelium firmibasis strain TNS-C-14 chromosome 6, whole genome shotgun sequence
388637:>JAVFKY010000007.1 Dictyostelium firmibasis strain TNS-C-14 contig_33_np1212, whole genome shotgun sequence
390119:>JAVFKY010000008.1 Dictyostelium firmibasis strain TNS-C-14 contig_23_np1212, whole genome shotgun sequence
391317:>JAVFKY010000010.1 Dictyostelium firmibasis strain TNS-C-14 contig_9_np1212, whole genome shotgun sequence
391790:>JAVFKY010000011.1 Dictyostelium firmibasis strain TNS-C-14 contig_16_np1212, whole genome shotgun sequence
392236:>JAVFKY010000012.1 Dictyostelium firmibasis strain TNS-C-14 contig_63_np1212, whole genome shotgun sequence
392533:>CM069771.1 Dictyostelium firmibasis strain TNS-C-14 mitochondrion, complete sequence, whole genome shotgun sequence

fir-chromesome1.fna
```

# fir-chromesome1.fna

了解！`fir-chromesome1.fna` 用に書き換えるとこんな感じになります👇

---

## 1️⃣ RepeatMasker 実行

```bash
RepeatMasker -pa 4 -lib /home/aki/Dfam-RepeatMasker.lib -gff -dir DIRS_masked_output_chr1 -engine ncbi fir-chromesome1.fna
```

- `-pa 4` → 並列 4 スレッド
    
- `-lib` → Dfam ライブラリを指定
    
- `-gff` → GFF3 形式で出力
    
- `-dir` → 出力ディレクトリ
    

出力は  
`DIRS_masked_output_chr1/fir-chromesome1.fna.out.gff`  
になります。

---

## 2️⃣ DIRS のみ抽出

```bash
grep -i "DIRS" DIRS_masked_output_chr1/fir-chromesome1.fna.out.gff > DIRS_masked_output_chr1/fir-chromesome1.DIRS.gff
```

---

💡 ポイント：

- `fir-chromesome1.fna` のみを対象にしているので、出力 GFF の `seqid` も **chromosome 1 のみ** に対応します。
    
- 出力ディレクトリ `DIRS_masked_output_chr1` に染色体1関連の結果をまとめて管理できます。
    

---

OK👌 では `fir-chromesome1.DIRS.gff` を BED に変換しましょう。

GFF → BED の基本変換は：

- GFF の **start**（4列目）は 1-based
    
- BED の **start**（2列目）は 0-based
    

なので `start - 1` して出力します。

---

### 変換コマンド

```bash
awk '{OFS="\t"; if($0 !~ /^#/){print $1, $4-1, $5, $9, $6, $7}}' \
DIRS_masked_output_chr1/fir-chromesome1.DIRS.gff > DIRS_masked_output_chr1/fir-chromesome1.DIRS.bed
```

---

### 出力 BED の列

1. 染色体ID（seqid）
    
2. start (0-based)
    
3. end (1-based)
    
4. 属性（attribute、GFFの9列目）
    
5. スコア（GFFの6列目）
    
6. ストランド（+/-）
    

---

👉 出力が正しくできてるか確認するには：

```bash
head DIRS_masked_output_chr1/fir-chromesome1.DIRS.bed
```

---

もし BED をさらにシンプルにして **chr, start, end, repeat_family** だけにしたい？それともフル情報のままでOK？