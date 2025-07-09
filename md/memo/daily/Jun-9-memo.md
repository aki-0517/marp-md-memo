```bash
quast.py \
  -o quast_ragtag_comparison \
  -R /home/aki/ncbi_dataset/ncbi_dataset/data/GCF_000004695.1/GCF_000004695.1_dicty_2.7_genomic.fna \
  --threads 16 \
  /home/aki/assembly2/assembly-results/flye/assembly.fasta \
  /home/aki/assembly2/assembly-results/medaka_flye_round2/consensus.fasta \
  /home/aki/assembly2/assembly-results/ragtag_flye_scaffold/ragtag.scaffold.fasta \
  /home/aki/assembly2/assembly-results/ragtag_flye_scaffold/ragtag_polished_round1.fasta \
  /home/aki/assembly2/assembly-results/ragtag_flye_scaffold/ragtag_medaka_polished.fasta \
  -l raw_flye,medaka_round2,scaffold_only,pilon_polished,medaka_polished
  ```