👍 了解です！
Akinobu さんが最新に実行した `RM_378377.ThuAug281714082025/` の内容は

```
consensi.fa  
round-1/  
round-2/
```

だけですね。
つまり **round-3 と round-4 がまだ生成されていない**ので、利用できるカスタムライブラリは

* `round-1/family-*.fa`
* `round-2/family-*.fa`
* `consensi.fa`（その時点までにまとめられたコンセンサス配列）

になります。

---

## 実行手順

### 1. カスタムリピートライブラリの作成

`round-1` と `round-2` のファミリーFASTAを結合し、さらに `consensi.fa` を追加してカスタムライブラリを作ります：

```bash
cat RM_378377.ThuAug281714082025/round-1/family-*.fa \
    RM_378377.ThuAug281714082025/round-2/family-*.fa \
    RM_378377.ThuAug281714082025/consensi.fa > custom_repeats.fa
```

---

### 2. RepeatMasker を実行

作成したライブラリを指定して RepeatMasker を実行します：

```bash
RepeatMasker -pa 4 \
  -lib custom_repeats.fa \
  -dir RM_masked_output_ref_chromesome1 \
  chromosome1.fna
```

---

### 3. 出力確認

`RM_masked_output/` に以下のファイルが出ます：

```bash
chromosome1.fna.cat  chromosome1.fna.masked  chromosome1.fna.ori.out  chromosome1.fna.out  chromosome1.fna.tbl
```

---

👉 ポイント

* 今回は **ラウンド2までのモデル**しかないので、それを反映したカスタムライブラリになります。
* 将来的に `round-3` 以降を回して完成度を上げたら、同じ手順でファイルを結合し直せば OK です。

---




---

# 1️⃣ 参照 FASTA のインデックス作成

```bash
samtools faidx RM_masked_output_ref_chromesome1/chromosome1.fna.masked
```

---

# 2️⃣ Simple repeat の BED作成（RepeatMasker .out から）

```bash
mkdir -p igv_tracks_chromosome1

awk 'NR>3 && $11 ~ /Simple_repeat/ {
  print $5"\t"$6-1"\t"$7"\t"$10"_"$11"\t"$1"\t"$9
}' RM_masked_output_ref_chromesome1/chromosome1.fna.out > igv_tracks_chromosome1/simple_repeats.bed
```

---

# 3️⃣ Tandem repeat の BED作成（TRF から）

```bash
# TRF 実行（出力を直接 .dat にリダイレクト）
mkdir -p RM_masked_output_ref_chromesome1/trf_results
trf RM_masked_output_ref_chromesome1/chromosome1.fna.masked 2 5 5 80 10 40 2000 -d -h -ngs > RM_masked_output_ref_chromesome1/trf_results/chromosome1.fna.masked.trf.dat

# 生成された .dat ファイルを変数に設定
TRF_DAT=RM_masked_output_ref_chromesome1/trf_results/chromosome1.fna.masked.trf.dat

# Tandem repeat の BED
mkdir -p igv_tracks_chromosome1
awk 'BEGIN{OFS="\t"} /^@/ {chr=substr($0,2); next} /^[0-9]+/ {
  print chr, $1-1, $2, $14"("$4"x)", $8, "."
}' "$TRF_DAT" > igv_tracks_chromosome1/tandem_repeats.bed

# 高スコアのみ（オプション）
awk 'BEGIN{OFS="\t"} /^@/ {chr=substr($0,2); next} /^[0-9]+/ && $8 > 500 {
  print chr, $1-1, $2, $14"_score"$8, $8, "."
}' "$TRF_DAT" > igv_tracks_chromosome1/high_score_tandems.bed

```

---

# 4️⃣ LTR repeat の BED作成（EDTA のGFFから）



---

### 次にやるべきこと

1. **EDTA を実行して TE 注釈ファイルを作成する**
    

```bash
# EDTA 環境をアクティベート
conda activate EDTA2

# 出力ディレクトリ作成
mkdir -p RM_masked_output_ref_chromesome1/edta_results
cd RM_masked_output_ref_chromesome1/edta_results

# EDTA 実行（CPUコア数は適宜変更）
EDTA.pl --genome ../chromosome1.fna.masked --species Others --step all --anno 1 --threads 8
```

- 実行後、`chromosome1.fna.mod.EDTA.intact.gff3` や `chromosome1.fna.mod.EDTA.TEanno.gff3` が生成されます。
    

2. **GFF ファイルがあるか確認**
    

```bash
ls -lh *.gff3
```

3. **BED 変換**
    

先ほどの AWK スクリプトで `GFF_FILE` をそのファイルに指定して実行します。

---

💡 ポイント：

- 今はまだ EDTA を実行していないので、`awk` が止まった原因は「そもそも入力ファイルが存在しない」ことです。
    
- まず EDTA を完了させてから BED 作成に進めば OK です。
    

必要であれば、**chromosome1 用に最短で IGV 向け BED を作るコマンド集** をこちらでまとめることもできます。作りますか？

```bash
GFF_FILE=RM_masked_output_ref_chromesome1/chromosome1.fna.mod.EDTA.intact.gff3

# 古い BED があれば削除
rm -f igv_tracks_chromosome1/ltr_repeats.bed

awk -F'\t' 'BEGIN{OFS="\t"}
{
  attr=$9; cls="";
  n=split(attr,parts,";");
  for(i=1;i<=n;i++){
    p=parts[i]; lp=tolower(p);
    if(lp ~ /^classification=/ || lp ~ /^class=/){
      split(p,kv,"=");
      cls=kv[2];
      break;
    }
  }
  if(cls=="" || tolower($3)!="repeat_region") next;
  if(tolower(cls) ~ /ltr/){
    print $1, $4-1, $5, cls, ".", "." >> "igv_tracks_chromosome1/ltr_repeats.bed"
  }
}' "$GFF_FILE"
```

### 結果


```bash
Traceback (most recent call last):
  File "/home/aki/miniconda3/envs/EDTA2/share/EDTA/bin/TIR-Learner2.5/Module3/GetAllSeq.py", line 63, in <module>
    pool.map(GetListFromFile,fileList) #shujun
  File "/home/aki/miniconda3/envs/EDTA2/lib/python3.6/multiprocessing/pool.py", line 266, in map
    return self._map_async(func, iterable, mapstar, chunksize).get()
  File "/home/aki/miniconda3/envs/EDTA2/lib/python3.6/multiprocessing/pool.py", line 644, in get
    raise self._value
FileNotFoundError: [Errno 2] No such file or directory: 'TIR-Learner_FinalAnn.gff3'
mv: 'TIR-Learner/*FinalAnn*.gff3' を stat できません: そのようなファイルやディレクトリはありません
mv: 'TIR-Learner/*FinalAnn*.fa' を stat できません: そのようなファイルやディレクトリはありません
Can't open ./TIR-Learner-Result/TIR-Learner_FinalAnn.fa: そのようなファイルやディレクトリはありません at /home/aki/miniconda3/envs/EDTA2/share/EDTA/util/rename_tirlearner.pl line 19.
Warning: LOC list chromosome1.fna.masked.mod.TIR.ext30.list is empty.
Error: Could not load sequence. Empty file or bad format.
Can't open ./TIR-Learner-Result/TIR-Learner_FinalAnn.gff3: そのようなファイルやディレクトリはありません.
Warning: The TIR result file has 0 bp!

2025年  8月 29日 金曜日 11:37:54 JST    Start to find Helitron candidates.

2025年  8月 29日 金曜日 11:37:54 JST    Identify Helitron candidates from scratch.

Error: Could not load sequence. Empty file or bad format.

        perl make_bed_with_intact.pl EDTA.intact.fa > EDTA.intact.bed 

2025年  8月 29日 金曜日 11:39:15 JST    Warning: The Helitron result file has 0 bp!

2025年  8月 29日 金曜日 11:39:15 JST    Execution of EDTA_raw.pl is finished!

ERROR: Raw LTR results not found in chromosome1.fna.masked.mod.EDTA.raw/chromosome1.fna.masked.mod.LTR.raw.fa
        If you believe the program is working properly, this may be caused by the lack of intact LTRs in your genome. Consider to use the --force 1 parameter to overwrite this check
```





- chromesome1でltrが見つからないのはおかしい。
- 他のやり方をpaperで探して、戦略を練る。
- chromesome1で確かめてからassembled genomeに適用しても良い。
- assembled genomeで試すときも、１つのchromesomeだけ抽出してから実行することで時間短縮できる。