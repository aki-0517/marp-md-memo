RepeatModelerのラウンド2～4で作成されたモデルファイル（`*.fa`）を使ったRepeatMaskerの実行手順は以下の通りです。

***

## 1. カスタムのリピートライブラリを作成する

ラウンドごとのfamily系ファイルをまとめて1つのファイルに結合します。例:

```bash
cat RM_3794378.TueAug191046142025/round-2/family-*.fa \
   RM_3794378.TueAug191046142025/round-3/family-*.fa \
   RM_3794378.TueAug191046142025/round-4/family-*.fa > custom_repeats.fa
```

***

## 2. RepeatMaskerにカスタムライブラリを指定して実行

RepeatMaskerをカスタムライブラリで実行するには、以下のコマンド例を参考にしてください。

```bash
RepeatMasker -pa 4 -lib custom_repeats.fa -dir RM_masked_output assembly2/assembly-results/ragtag_flye_scaffold/ragtag_polished_round1.fasta
```

- `-pa 4`：4コア使用（環境に合わせて調整）
- `-lib custom_repeats.fa`：自作のリピートライブラリ指定
- `-dir RM_masked_output`：出力ディレクトリ指定
- assembly2/assembly-results/ragtag_flye_scaffold/ragtag_polished_round1.fasta：マスクしたいゲノム配列ファイル

***

## 3. 実行後の確認

`RM_masked_output`ディレクトリにマスク結果ファイル（*.masked、*.outなど）が生成されます。

***

```bash
(base) [aki@tardis ~]$ ls -l RM_masked_output/
合計 73620
-rw-rw-r-- 1 aki aki 19180723  8月 25 12:22 ragtag_polished_round1.fasta.cat.gz
-rw-rw-r-- 1 aki aki 35558579  8月 25 12:22 ragtag_polished_round1.fasta.masked
-rw-rw-r-- 1 aki aki     8786  8月 25 12:22 ragtag_polished_round1.fasta.ori.out
-rw-rw-r-- 1 aki aki 20623783  8月 25 12:22 ragtag_polished_round1.fasta.out
-rw-rw-r-- 1 aki aki     2454  8月 25 12:22 ragtag_polished_round1.fasta.tbl
```

```bash
(base) [aki@tardis ~]$ head RM_masked_output/ragtag_polished_round1.fasta.out
   SW   perc perc perc  query                     position in query     matching   repeat          position in repeat
score   div. del. ins.  sequence                  begin end    (left)   repeat     class/family  begin  end    (left)  ID

   13   20.0  2.4  2.4  NC_000895.1_RagTag_pilon    567   608 (54945) + (TAGTAA)n  Simple_repeat      1     42    (0)   1  
   13   27.9  0.0  0.0  NC_000895.1_RagTag_pilon    819   861 (54692) + (TAACAA)n  Simple_repeat      1     43    (0)   2  
   12   11.9  0.0  6.9  NC_000895.1_RagTag_pilon   3985  4015 (51538) + (AAAAT)n   Simple_repeat      1     29    (0)   3  
   15   24.4  0.0  3.9  NC_000895.1_RagTag_pilon   4190  4242 (51311) + (ATAAAAA)n Simple_repeat      1     51    (0)   4  
   15   21.8  5.7  0.0  NC_000895.1_RagTag_pilon   4654  4706 (50847) + (AAAAGT)n  Simple_repeat      1     56    (0)   5  
   15   29.9  3.1  0.0  NC_000895.1_RagTag_pilon   6951  7015 (48538) + (ACAAAA)n  Simple_repeat      1     67    (0)   6  
   11   17.2  7.7  0.0  NC_000895.1_RagTag_pilon   8948  8986 (46567) + (ATAAAT)n  Simple_repeat      1     42    (0)   7  
```