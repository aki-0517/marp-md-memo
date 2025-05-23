# Dictyostelium discoideum Genome Assembly & Polishing Summary

## Table of Contents
- [Dictyostelium discoideum Genome Assembly \& Polishing Summary](#dictyostelium-discoideum-genome-assembly--polishing-summary)
  - [Table of Contents](#table-of-contents)
  - [Background Information](#background-information)
    - [Overview of Dictyostelium discoideum Genome](#overview-of-dictyostelium-discoideum-genome)
      - [Genome Sequence Features](#genome-sequence-features)
  - [Data Used](#data-used)
  - [Visualization of ONT Read Length Distribution](#visualization-of-ont-read-length-distribution)
    - [Data](#data)
  - [Assembly Methods](#assembly-methods)
    - [Evaluation by Quast](#evaluation-by-quast)
  - [Polishing Methods](#polishing-methods)
  - [Current Status](#current-status)
  - [Future Directions](#future-directions)

## Background Information

- **Number of nuclear chromosomes**: 6
- **Extrachromosomal palindrome (rDNA)**: ~88 kb linear palindrome, ~100 copies, accounts for ~20% of nuclear DNA
- **Mitochondrial genome**: ~55.6 kb, ~200 copies
- **Number of replicons**: 6 nuclear chromosomes + rDNA palindrome + mitochondrial genome = 8 (ideal number of contigs)

---

### Overview of Dictyostelium discoideum Genome

- **Total length**: ~33.89 Mb (chromosome assembly: 33,817,471 bp; unassigned contigs: 69,589 bp)
- **Number of chromosomes**: 6
- **Coverage**: 9.1×–10.3× for chromosomes 1–6, overall average 8.3×
- **Number of contigs**: 309 on chromosome assembly (155 gaps), 34 unassigned contigs
- **Estimated total gap length**: 155,750 bp (including sequence, repeat, and clone gaps)
- **Assembly method**: Whole-Chromosome Shotgun (WCS) + HAPPY mapping (3,902 markers, average 8.7 kb apart)

#### Genome Sequence Features
- **AT-rich**: 77.57% A+T (G+C is 22.43%)
- **CpG depletion**: CpG frequency is 62% of GpC, suggesting cytosine methylation
- **Simple sequence repeats (SSR)**: >11% of genome bases (especially high in non-coding regions, many homopolymers)
- **Transposons**: Many non-LTR retrotransposons, frequent insertions near tRNA genes
- **tRNA genes**: 390 (among the highest in eukaryotes), nearly all sense codons covered, many tandem duplications

## Data Used

- ONT: `Dicty_gDNA_NEB-2.fastq`
- Illumina: `./Dicty-genome/Stationary_S1_R1.fastq.gz`, `./Dicty-genome/Stationary_S1_R2.fastq.gz`
- Subset: `Dicty_gDNA_NEB-2_half.fastq.gz`

## Visualization of ONT Read Length Distribution

### Data

- ONT

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

![read-length](../public/read-length/image.png)

*This project focuses on genome assembly of Dictyostelium discoideum, mainly using ONT data as the primary sequencing data.*

## Assembly Methods

- **Flye**
- **Canu**
- **Raven**
- **Shasta**

### Evaluation by Quast

| Metric              | Raven   | Flye    | Shasta   | Canu     |
|---------------------|---------|---------|----------|----------|
| # contigs           | 28      | 33      | 36       | 14       |
| Largest contig      | 5,803,050 | 5,006,538 | 12,082,896 | 8,736,265 |
| Total length        | 35,504,430 | 34,319,985 | 33,471,541 | 34,572,512 |
| N50                 | 2,694,088 | 2,870,802 | 6,719,217 | 3,611,137 |
| L50                 | 5       | 5       | 2        | 3        |
| GC (%)              | 22.77   | 22.98   | 22.90    | 23.06    |

- Canu has the fewest contigs (14), with high N50 and largest contig, showing good overall balance.
- Shasta has the largest contig and N50, but more contigs.
- Flye and Raven have more contigs and slightly lower N50.

→ Overall, Canu is considered the most accurate.

## Polishing Methods

- **Pilon** (2 rounds with ONT and Illumina reads)
  - Limited improvement in accuracy
- **Medaka** (1 round with ONT reads)

## Current Status

- Number of contigs: 14
- Goal: 8 (6 chromosomes + 1 rDNA + 1 mitochondria)
- Current best: Canu (50%), Pilon (ONT50% + Illumina50%)

## Future Directions

1. **Try polishing with other libraries**
   - Consider polishing tools and parameters other than the current best (Canu→Pilon)
2. **Aim to reduce the number of contigs by scaffolding**
   - Apply scaffolding tools to the best assembly results to approach the target of 8
3. **Consider multi-assembly strategies**
   - Integrate multiple assemblies with different breakpoints to obtain longer contigs 