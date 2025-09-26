```bash
cat RM_2139892.FriSep191125262025/round-2/family-*.fa \
RM_2139892.FriSep191125262025/round-3/family-*.fa \
RM_2139892.FriSep191125262025/round-4/family-*.fa > custom_repeats_canu.fa
```

```bash
RepeatMasker -pa 4 -lib custom_repeats_canu.fa -dir RM_masked_output_canu assembly2/assembly-results/ragtag_canu_scaffold/canu_polished_round1.fasta
```

