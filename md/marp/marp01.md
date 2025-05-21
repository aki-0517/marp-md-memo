---
marp: true
paginate: true
---

# Genome Assembly and Polishing Strategy for *Dictyostelium discoideum*

**Using Long-Read Sequencing and the Canu Assembler**  
*Based on Cell Press Protocol by Kim & Kim (2022)*

---

## 🧬 Background

- Genome assembly aims to reconstruct full-length DNA sequences
- Short-read sequencing (e.g., Illumina) struggles with repeats and structural variants
- Long-read platforms (e.g., ONT, PacBio) are better for de novo assembly
- *D. discoideum* has 6 chromosomes + extrachromosomal rDNA + mitochondrial genome → **8 replicons**

---

## 🎯 Objective

- Use ONT data to assemble a high-quality, low-contig draft genome
- Evaluate multiple assemblers to find the best-performing output
- Use the best output (Canu 50% data) as the base for polishing

---

## 🧪 Workflow Overview (based on Kim & Kim, 2022)

1. Visualize ONT read distribution
2. Estimate genome size (Flye)
3. Compare assemblers (Canu, Flye, Shasta, Raven)
4. Select best assembly (Canu 50%)
5. Polish using ONT + Illumina reads (Pilon)
6. Evaluate and scaffold (BUSCO, RagTag, etc.)

---

## 📊 ONT Read Quality and Genome Size

- N50 = 12,777 bp, Max = 139 kb → High-quality ONT reads
- Genome size (Flye) ≈ **33.5 Mb**

---

## ⚙️ Assembly Comparison (QUAST)

| Assembler | Contigs | N50 | Length | GC Content |
|-----------|---------|-----|--------|------------|
| **Canu (50%)** | **14** | High | 33.5 Mb | 22.8% |
| Shasta | 36 | 6.7 Mb | 33.5 Mb | 22.9% |
| Flye | ~40 | 2.8 Mb | 34.3 Mb | 22.98% |
| Raven | ~60 | 2.7 Mb | 35.5 Mb | 22.77% |

✅ **Canu (50%)** shows the best balance: compact, high-contiguity, close to 8 replicons

---

## 🛠️ Polishing Strategy (Pilon)

- Base assembly: Canu (50% reads)
- Reads used: ONT + Illumina (short-read)
- 2-Step Pilon polishing:
  1. `--fix snps,indels`
  2. `--fix gaps,local`
- Outputs: Corrected FASTA, VCF

---

## 🧪 Next Steps in Analysis

| Step | Tool | Purpose |
|------|------|---------|
| Quality check | BUSCO | Evaluate single-copy gene coverage |
| Scaffolding | RagTag | Align to reference genome |
| SV detection | SVIM-asm | Detect large variants |
| Gene annotation | BRAKER2 | Predict gene structure using RNA-seq |

---

## 🔍 Position in the Kim & Kim Protocol

| Feature | This Study | Kim & Kim (2022) |
|--------|------------|------------------|
| Species | *D. discoideum* | *D. melanogaster* |
| Platform | ONT + Illumina | ONT + PacBio + Illumina |
| Best Assembly | Canu (50%) | ONT/Shasta |
| Polishing | Pilon | Pilon |

---

## 🧾 Summary and Outlook

- ✅ Canu (50%) produced **14 contigs**, highest quality result
- ✅ Polishing is in progress using Pilon
- 🔜 Evaluate gene completeness (BUSCO), perform scaffolding (RagTag)
- 🎯 Final goal: **Accurate 8-replicon genome assembly**

---

## 🙏 Acknowledgment

Protocol reference:  
Kim & Kim (2022). A beginner’s guide to genome assembly using long-read sequencing. *Cell Press Protocol*.

Thank you!
