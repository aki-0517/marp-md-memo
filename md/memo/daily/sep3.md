ltr見つけたい

### EDTA
```bash
(EDTA2) [aki@tardis ~]$ EDTA.pl --genome chromosome1.fna \
>         --species Others \
>         --step ltr \
>         --threads 8 \
>         --overwrite 1

########################################################
##### Extensive de-novo TE Annotator (EDTA) v2.0.0  ####
##### Shujun Ou (shujun.ou.1@gmail.com)             ####
########################################################



2025年  9月  3日 水曜日 14:30:27 JST    Dependency checking:
                                All passed!

Can't find label LTR at /home/aki/miniconda3/envs/EDTA2/share/EDTA/EDTA.pl line 383.
```


### LTRharvest を使った LTR 検出
```bash
gt suffixerator -db chromosome1.fna -indexname chromosome1 -tis -suf -lcp -des -ssp -sds -dna

gt ltrharvest -index chromosome1 -minlenltr 100 -maxlenltr 5000 \
  -mindistltr 1000 -maxdistltr 20000 -similar 85 -mintsd 4 -maxtsd 6 \
  -out ltr_candidates.fasta -outinner ltr_inner.fasta -gff3 ltr_candidates.gff3
```

```bash
grep -v '^#' ltr_candidates.gff3 | awk '$3=="LTR_retrotransposon" {
  chrom="NC_007087.3"; 
  name=$9;
  split(name,a,";");
  id=""; sim="";
  for(i in a){
    if(a[i] ~ /^ID=/) id=a[i];
    if(a[i] ~ /^ltr_similarity=/) sim=a[i];
  }
  gsub("ID=","",id);
  gsub("ltr_similarity=","",sim);
  strand="+";
  if($7=="-") strand="-";
  print chrom"\t"$4-1"\t"$5"\t"id"("sim")\t0\t"strand
}' > ltr_candidates_for_IGV.bed
```

Figure 3の説明から、_Dictyostelium discoideum_ の **chromosome 1** のリピート領域（特にDIRSやSkipper要素がある領域）の特徴を整理すると以下のようになります。

---

### 1. **場所とアセンブリ状況**

- このリピート領域は **chromosome 1 のおそらく centromere 付近** に存在すると考えられている。
    
- クロモソーム末端（telomere）近くの繰り返し配列は複雑すぎて完全にはアセンブルできなかったが、**chromosome 1 の大部分のクラスターはアセンブルできている**。
    

---

### 2. **構造的特徴**

- 領域内には **DIRS 型レトロトランスポゾン** と **Skipper 型レトロトランスポゾン** が存在。
    
- これらの要素は **同じタイプ同士で集まる傾向** がある（type-specific clustering）。
    
- 頻繁な **挿入や部分的な欠失** が起こっており、これによって **長距離の規則性はほとんどなく、モザイク状の構造** になっている。
    

---

### 3. **配列方向性と複雑性**

- Figure 3 では、完全なリピートユニットは **色付き三角形** で表現され、三角形のサイズはリピートユニットの長さに対応。
    
- 三角形の **左下が 5′末端、右上が 3′末端** を表す。
    
- 三角形の向きはリピートの向きを示しており、部分リピートや完全リピートが複雑に混ざった **モザイク状の配列パターン** を形成している。
    
- 縦軸（リピートユニットの長さ）と横軸（染色体上の距離）が同じスケールで示されているため、**ユニットサイズと染色体距離の関係も一目でわかる**。
    

---

### 4. **まとめると**

- **長距離の規則性がないモザイク状リピート領域**
    
- **DIRS と Skipper が混在**し、タイプごとにまとまる
    
- 頻繁な挿入・欠失で多様性が高い
    
- 配列方向性が複雑で、リピート長と位置が多様
    

---

💡 簡単に言うと、この領域は **centromere の可能性がある複雑なリピートの塊で、部分欠失や挿入でモザイク状になっている**、ということです。

---


了解です。1列目（chrom）を **必ず `NC_007087.3` に固定** して、ID・LTR 類似度・ファミリー・内部配列も含めた BED を作るコマンドをまとめます。

---

### 1️⃣ LTR 全体用（詳細版）

```bash
grep -v '^#' ltr_candidates.gff3 | awk '
$3=="LTR_retrotransposon" {
    chrom="NC_007087.3"        # 1列目を固定
    start=$4-1                 # BEDは0-based
    end=$5
    strand=$7
    if(strand=="." || strand=="") strand="+"

    # 属性列から ID, ltr_similarity, family を取得
    name=$9
    split(name,a,";")
    id=""; sim=""; family=""
    for(i in a){
        if(a[i] ~ /^ID=/) id=a[i]
        if(a[i] ~ /^ltr_similarity=/) sim=a[i]
        if(a[i] ~ /^Family=/) family=a[i]
    }
    gsub("ID=","",id)
    gsub("ltr_similarity=","",sim)
    gsub("Family=","",family)

    print chrom"\t"start"\t"end"\t"id"("sim"):"family"\t0\t"strand
}' > ltr_candidates_detailed_for_IGV.bed
```





# RagtagでLTRharvest を使った LTR 検出

`ragtag_polished_round1.fasta.masked` **全コンティグに対してLTRharvestを走らせて、IGVで可視化できるBEDを作る**なら、先ほどのワンライナーをそのまま使えば大丈夫です。

最終的なコマンドはこうです👇

```bash
# 1. index作成
gt suffixerator -db /home/aki/RM_masked_output/ragtag_polished_round1.fasta.masked \
  -indexname ragtag_polished_round1 -tis -suf -lcp -des -ssp -sds -dna

# 2. LTR検出
gt ltrharvest -index ragtag_polished_round1 \
  -minlenltr 100 -maxlenltr 5000 \
  -mindistltr 1000 -maxdistltr 20000 -similar 85 -mintsd 4 -maxtsd 6 \
  -out ragtag_ltr_candidates.fasta \
  -outinner ragtag_ltr_inner.fasta \
  -gff3 ragtag_ltr_candidates.gff3

# 3. IGV用BEDファイル作成（全コンティグ対応）
grep -v '^#' ragtag_ltr_candidates.gff3 | awk '$3=="LTR_retrotransposon" {
  chrom=$1;   # gff3からcontig名をそのまま利用
  name=$9;
  split(name,a,";");
  id=""; sim="";
  for(i in a){
    if(a[i] ~ /^ID=/) id=a[i];
    if(a[i] ~ /^ltr_similarity=/) sim=a[i];
  }
  gsub("ID=","",id);
  gsub("ltr_similarity=","",sim);
  strand="+";
  if($7=="-") strand="-";
  print chrom"\t"$4-1"\t"$5"\t"id"("sim")\t0\t"strand
}' > ragtag_ltr_candidates_for_IGV.bed
```

---

📌 ポイント

- `chrom=$1` にしているので、`NC_000895.1_RagTag_pilon` や `NC_007087.3_RagTag_pilon` など **全コンティグ名が保持される** → IGVでゲノム全体を見られる。
    
- 出力は標準的な **BED6形式** (chrom, start, end, name, score, strand) になるので、そのままIGVに読み込み可能。
    
- `ltr_similarity` が `name` の後ろに括弧付きで追加される（例: `gi|2_copy54076(85)`）。
    

---

ナイス確認です！⚡  
結果からするとこういう状況ですね：

- **FASTAヘッダー** →
    
    ```
    >NC_000895.1_RagTag_pilon
    >NC_007087.3_RagTag_pilon
    >NC_007088.5_RagTag_pilon
    >NC_007089.4_RagTag_pilon
    >NC_007091.3_RagTag_pilon
    >NC_007092.3_RagTag_pilon
    >contig_12_pilon_pilon
    ...
    ```
    
- **LTRharvest GFF3** →
    
    ```
    ##sequence-region   seq1 ...
    ##sequence-region   seq2 ...
    ##sequence-region   seq3 ...
    ...
    ```
    

👉 つまり `ltrharvest` は contig名を `seq1, seq2, ...` に置き換えてしまっています。  
このままでは **IGVに読み込んでも位置が一致しない** ので、必ず対応付けが必要です。

---

## ✅ 対応表の作り方

FASTAの順番と `seqN` の対応をつければOK。

```bash
grep ">" /home/aki/RM_masked_output/ragtag_polished_round1.fasta.masked \
  | sed 's/>//' \
  | nl -w1 -s$'\t' > contig_map.txt
```

📄 `contig_map.txt` の例：

```
1   NC_000895.1_RagTag_pilon
2   NC_007087.3_RagTag_pilon
3   NC_007088.5_RagTag_pilon
4   NC_007089.4_RagTag_pilon
5   NC_007091.3_RagTag_pilon
6   NC_007092.3_RagTag_pilon
7   contig_12_pilon_pilon
...
```

---

## ✅ BEDファイルを置換

```bash
awk 'NR==FNR{map["seq" $1]=$2; next} {if($1 in map) $1=map[$1]; print}' \
  contig_map.txt ragtag_ltr_candidates_for_IGV.bed \
  > ragtag_ltr_candidates_for_IGV_renamed.bed
```

これで `ragtag_ltr_candidates_for_IGV_renamed.bed` は  
`NC_000895.1_RagTag_pilon` など **FASTAヘッダーそのままのcontig名** になります。

---

## ✅ IGVで読み込む前に毎回チェックすること

1. `grep ">" fasta | head` → FASTAヘッダー確認
    
2. `head gff3` → `seq1` 形式か、ヘッダーそのままか確認
    
3. 違っていたら `contig_map.txt` を作って置換
    
