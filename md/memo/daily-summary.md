# Daily Memo Summary

This document summarizes the daily memos from the `md/memo/daily/` directory, containing research notes on genome assembly analysis for Dictyostelium discoideum.

## Overview

The daily memos document a comprehensive genome assembly project comparing different assembly tools (Flye vs Canu) and analyzing various aspects of genome structure including:

- Genome assembly quality assessment
- Transposable element (TE) and telomere identification
- Assembly polishing and scaffolding workflows
- Comparative genomics analysis

## Key Research Topics

### 1. Genome Assembly Comparison (Aug 4-5, 2025)
**Files:** `aug-4.md`, `aug-5.md`

- **Focus:** Comparison between Flye and Canu assemblers for D. discoideum genome
- **Key findings:** Flye_Polished showed superior performance with:
  - Better contiguity (N50: 8.06 Mb vs 5.61 Mb)
  - Fewer contigs (11 vs 29)
  - Lower duplication ratio (1.035 vs 1.066)
  - Reduced misassemblies

### 2. Transposable Element Analysis (Jul 16-17, 2025)
**Files:** `jul-16.md`, `jul-17.md`

- **Methods:** Used BLAST-based approaches with Repbase database
- **Key elements identified:**
  - DIRS-1 retrotransposon (concentrated at chromosome ends)
  - Various LTR and non-LTR retrotransposons
  - DNA transposons (DDT-A, Tdd-4)

### 3. Telomere Identification (Jul 14, 2025)
**Files:** `Jul-14-memo.md`

- **D. discoideum specificity:** Unlike typical organisms, D. discoideum lacks standard (G+T)-rich telomeres
- **Alternative structure:** Ribosomal DNA (rDNA) elements function as telomeres
- **Methods:** Junction analysis between complex repeats and rDNA sequences

### 4. Quality Control and Data Processing (Jun 10-25, 2025)
**Files:** `Jun-10-memo.md`, `Jun-25-memo.md`, `Jul-9-memo.md`

- **Tools used:**
  - pycoQC and NanoPlot for ONT data quality assessment
  - QUAST for assembly comparison
  - RepeatMasker for repetitive element masking
- **Data sources:**
  - ONT: Dicty_gDNA_NEB-2.fastq
  - Illumina: Stationary_S1_R1/R2.fastq.gz
  - Reference: D. discoideum AX4 genome (GCF_000004695.1)

## Assembly Workflow Summary

### Data Processing Pipeline
1. **Quality Control:** pycoQC/NanoPlot for ONT, FastQC for Illumina
2. **Assembly:** Flye (recommended) or Canu for long-read assembly
3. **Polishing:** 
   - Medaka for long-read polishing
   - Pilon for short-read polishing
4. **Scaffolding:** RagTag for reference-guided scaffolding
5. **Quality Assessment:** QUAST for comprehensive evaluation

### Repetitive Element Analysis
1. **De novo prediction:** EDTA with sensitive parameters
2. **Homology search:** RepeatMasker with Repbase database
3. **Telomere identification:** 
   - Tandem Repeat Finder (TRF)
   - Junction analysis for D. discoideum-specific structures

## Key Insights

### Assembly Quality
- Flye consistently outperformed Canu in multiple metrics
- Proper polishing significantly improved assembly accuracy
- Reference-guided scaffolding enhanced contiguity

### D. discoideum Genome Features
- Unique telomere structure involving rDNA elements
- High AT content affecting assembly strategies
- Abundant transposable elements, particularly DIRS-1

### Technical Considerations
- Coverage optimization important to avoid misassembly
- Species-specific repeat libraries improve annotation accuracy
- Multiple validation approaches necessary for quality assessment

## Files Analyzed
- `aug-5.md` - Latest TE/telomere identification protocols
- `aug-4.md` - Assembly comparison results
- `jul-17.md` - Repeat element masking workflows
- `jul-16.md` - Assembly evaluation commands
- `Jul-14-memo.md` - Telomere analysis methodology
- `Jul-9-memo.md` - QUAST comparison and prokka analysis
- `Jun-25-memo.md` - Comparative genomics visualization
- `Jun-10-memo.md` - Data overview and QC methodology

## References
- Nature paper on D. discoideum genome: https://www.nature.com/articles/nature03481
- Genome Biology assembly paper: https://genomebiology.biomedcentral.com/articles/10.1186/s13059-025-03578-7
- NCBI reference genome: https://www.ncbi.nlm.nih.gov/datasets/genome/GCF_000004695.1/