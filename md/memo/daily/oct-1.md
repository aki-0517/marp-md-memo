https://pmc.ncbi.nlm.nih.gov/articles/PMC5633606/

1. DIRS1エレメントを比べて、canuとflyeのどちらの方が精度が高いか調べる
2. その後、他のcentromere全てのdirs1以外のエレメントをアノテーションする
   
これが１つのゴール





---

## 解析方法の変更

### 1️⃣ GFFの Target 列を使う

GFFの9列目（attribute列）に含まれる `Target` 情報を抽出して、DIRS1ライブラリ配列に対応するIDを調べます。

```bash
# Flye
awk -F'\t' '$3=="dispersed_repeat" {print $1, $4, $5, $9}' DIRS_masked_output/ragtag_polished_round1.fasta.out.gff | head
```

- これで `Target "Motif:M11339.1"` が出ます。
    
- `dd_dirs1.fasta` 内で M11339.1 が DIRS1 の配列か確認してください。
    

---

### 2️⃣ DIRS1に対応するMotif IDを抽出して解析

例：もし `DIRS1` のライブラリIDが `M11339.1` なら：

```bash
# Flye DIRS1コピー数
awk -F'\t' '$3=="dispersed_repeat" && $9 ~ /M11339.1/ {count++} END{print count}' DIRS_masked_output/ragtag_polished_round1.fasta.out.gff

# Canu DIRS1コピー数
awk -F'\t' '$3=="dispersed_repeat" && $9 ~ /M11339.1/ {count++} END{print count}' DIRS_masked_output_canu/canu_polished_round1.fasta.out.gff
```

---

### 3️⃣ コピー長の確認

```bash
# Flye DIRS1コピー長
awk -F'\t' '$3=="dispersed_repeat" && $9 ~ /M11339.1/ {print $5-$4+1}' DIRS_masked_output/ragtag_polished_round1.fasta.out.gff | \
awk 'BEGIN{sum=0;min=1e9;max=0;count=0} {count++; sum+=$1; if($1<min) min=$1; if($1>max) max=$1} END{if(count>0) print "min="min,"max="max,"avg="sum/count; else print "No hits"}'

# Canu DIRS1コピー長
awk -F'\t' '$3=="dispersed_repeat" && $9 ~ /M11339.1/ {print $5-$4+1}' DIRS_masked_output_canu/canu_polished_round1.fasta.out.gff | \
awk 'BEGIN{sum=0;min=1e9;max=0;count=0} {count++; sum+=$1; if($1<min) min=$1; if($1>max) max=$1} END{if(count>0) print "min="min,"max="max,"avg="sum/count; else print "No hits"}'
```

---

```bash
(base) [aki@tardis ~]$ awk -F'\t' '$3=="dispersed_repeat" && $9 ~ /M11339.1/ {count++} END{print count}' DIRS_masked_output/ragtag_polished_round1.fasta.out.gff
453
(base) [aki@tardis ~]$ awk -F'\t' '$3=="dispersed_repeat" && $9 ~ /M11339.1/ {count++} END{print count}' DIRS_masked_output_canu/canu_polished_round1.fasta.out.gff
447
(base) [aki@tardis ~]$ awk -F'\t' '$3=="dispersed_repeat" && $9 ~ /M11339.1/ {print $5-$4+1}' DIRS_masked_output/ragtag_polished_round1.fasta.out.gff | \
> awk 'BEGIN{sum=0;min=1e9;max=0;count=0} {count++; sum+=$1; if($1<min) min=$1; if($1>max) max=$1} END{if(count>0) print "min="min,"max="max,"avg="sum/count; else print "No hits"}'
min=29 max=7061 avg=1253.58
(base) [aki@tardis ~]$ awk -F'\t' '$3=="dispersed_repeat" && $9 ~ /M11339.1/ {print $5-$4+1}' DIRS_masked_output_canu/canu_polished_round1.fasta.out.gff | \
> awk 'BEGIN{sum=0;min=1e9;max=0;count=0} {count++; sum+=$1; if($1<min) min=$1; if($1>max) max=$1} END{if(count>0) print "min="min,"max="max,"avg="sum/count; else print "No hits"}'
min=29 max=7061 avg=1231.77
```



|アセンブリ|DIRS1コピー数|長さ min|長さ max|平均長|
|---|---|---|---|---|
|Flye|453|29|7061|1253.6|
|Canu|447|29|7061|1231.8|

---

## 分析ポイント

1. **コピー数**
    
    - Flye: 453, Canu: 447
        
    - Flyeの方が少し多く検出されている → 欠損が少ない可能性がある
        
2. **コピー長**
    
    - min, maxは両者同じ
        
    - 平均長はFlyeが少し長い（1253 vs 1231） → 分断が少なく、より完全なコピーを保持している可能性がある
        
3. **差の大きさ**
    
    - コピー数の差は 6/450 ほど（約1.3%）
        
    - 平均長の差は 22bp ほどで、非常に小さい
        

---

## 結論（DIRS1に限った場合）

- **Flyeの方が若干優勢**
    
    - コピー数が多い
        
    - 平均長が少し長い
        
- ただし差は非常に小さく、実務的には **両者ともほぼ同等レベル** と言える
    

💡 追加で精度を判断したい場合は：

- コピーの**分布や断片化のパターン**をスキャフォールド上で可視化
    
- `%div` やスコア情報を考慮して、正確性も比較
    




---
