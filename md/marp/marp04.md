---
marp: true
theme: default
paginate: true
style: |
  section { font-size: 24px; }
---

# *Dictyostelium discoideum*  
## Genome Assembly & Polishing Summary

- **Objective**: Construction of high-quality genome sequence for model organism *D. discoideum*  
- **Significance**: Providing foundational information for cell differentiation, motility, and signal transduction research  

<!--
Hello everyone. Today, I will talk about my project to make a high-quality genome sequence for Dictyostelium discoideum. 
This is a social amoeba that scientists use to study basic life processes. 
I will explain the steps I took and why this work is important.
-->

---

## Background: About *D. discoideum*

![bg right:40% w:400px](../public/images/dd.png)

- Model organism of social amoeba  
- Life cycle: Single-cell ⇄ Multicellular  
- Genome size: ~34 Mb  
- Chromosomes: 6 + rDNA (~88 kb×100 copies) + Mitochondria (~56 kb)

**Genome Characteristics**:
- AT-rich sequence composition (77.6%)
- Numerous tRNA genes (390 copies)
- Rich in simple sequence repeats (SSR) (>11%)

<!--
Dictyostelium discoideum is special because it can live alone or join with others to form a group. Its genome is about 34 million base pairs, with 6 chromosomes, many rDNA copies, and mitochondria. It has a lot of A and T, many tRNA genes, and many simple repeats. These features make it interesting for research.
-->

---

## Genome Assembly Workflow

![bg right:40% w:400px](../public/images/assembly.png)

1. **Sequence Data Acquisition**  
   - ONT (Long reads)  
   - Illumina (Short reads)  
2. **Quality Assessment & Preprocessing**  
   - Read quality confirmation 
3. **Assembly Execution**  
   - Canu / Flye / Raven / Shasta -> Comparison
4. **Polishing (Error Correction)**  
   - Pilon / Medaka  
5. **Evaluation & Improvement**  
   - Quality assessment with QUAST → Reassembly or Scaffolding as needed  

<!--

I used two types of DNA sequencing reads:
Long reads from a ONT, and short but super-accurate reads from Illumina.
Then I:

Checked the quality of the data

Used four different tools — Canu, Flye, Raven, and Shasta — to assemble the genome

Fixed errors using two polishing tools: Pilon and Medaka

Finally, we checked the results to see how good they were.
-->

---

## Comparison of ONT and Illumina Data

<style scoped>
table {
  width: 100%;
  font-size: 30px;
}
</style>

|                | ONT Long Reads                | Illumina Short Reads         |
|----------------|------------------------------|-----------------------------|
| Read Length    | Very long (up to ~139 kb)    | Short (~150 bp)             |
| Accuracy       | Lower (high error rate)       | Very high                   |
| Error Type     | Indels, mismatches           | Rare, mostly substitutions  |
| Strengths      | Resolves repeats, large SVs  | Ideal for polishing         |
| Weaknesses     | Lower per-base accuracy       | Cannot span long repeats    |

<!--

By combining both read and execute assembly and polishing, I get the best result
-->

---

## Dictyostelium discoideum ONT Read Length Distribution

![bg right:45% w:500px](../public/images/read-length.png)

```bash
# Total length of bases in the sequence
sum = 8,359,638,019 bp  

# Total number of reads
n = 934,886 reads       

# Average read length
mean length = 8,941.88 bp 

# Length of the longest read
max length = 139,714 bp    

# Read length where 50% of total sequence 
N50 = 12,777 bp        
```

<!-- N50 is a commonly used metric in genomics that represents the length at which 50% of the total sequence length is contained in reads of that length or longer. In other words, if we sort all reads by length and add up their lengths, N50 is the length of the shortest read that is part of the group of reads that together make up 50% of the total sequence length. A higher N50 value generally indicates better quality sequencing data, as it suggests we have more long reads available for assembly. -->

---

## Assembly Experiment Overview

- **Data Used**: ~50% of ONT long reads (4.2 Gb)
  - Why 50%?...Excessive coverage can lead to increased computation time and reduced accuracy
  - Also testing other coverage levels (e.g., 25% and 75%), but 50% was most accurate 

- **Assembly Tool Characteristics**:
  - Canu: Powerful error correction, longer computation time
  - Flye: Strong with repetitive sequences, good memory efficiency
  - Shasta: Ultra-fast but slightly lower accuracy
  - Raven: Low memory usage, high speed

<!--
For assembly, I used about half of the ONT data. Too much data can make the process slow and less accurate. I tried different software: Canu is good at fixing errors but slow, Flye is good with repeats, Shasta is very fast, and Raven is fast and uses little memory. Each tool has its own strengths.
-->

---

## Assembly Result Comparison

![bg right:45% vertical w:500px](../public/images/nx-graph.png)
![bg w:500px](../public/images/cumulative.png)

<style scoped>
table {
  width: 100%;
  font-size: 20px;
}
</style>

| Metric         | Raven   | Flye    | Shasta  | Canu    |
| ---------- | ------- | ------- | ------- | ------- |
| contigs  | 28      | 33      | 36      | 14      |
| Largest contig | 5.8 Mb  | 5.0 Mb  | 12.0 Mb | 8.7 Mb  |
| Total length         | 35.5 Mb | 34.3 Mb | 33.5 Mb | 34.6 Mb |
| N50        | 2.7 Mb  | 2.8 Mb  | 6.7 Mb  | 3.6 Mb  |

<!-- 1. **Number of contigs**: Represents how fragmented the assembly is. A lower number is generally better, as it indicates the genome is assembled into fewer, larger pieces. Canu produced the fewest contigs (14), suggesting the most complete assembly.

2. **Largest contig**: Shows the size of the longest continuous sequence in the assembly. Shasta produced the largest contig (12.0 Mb), indicating it was able to assemble longer continuous regions than other tools.

3. **Total length**: The sum of all contig lengths. All assemblers produced similar total lengths (33.5-35.5 Mb), which is close to the expected genome size, suggesting good coverage of the genome.

4. **N50**: A measure of contig length distribution. 50% of the total assembly is in contigs of this size or larger. Shasta's N50 of 6.7 Mb is the highest, indicating it produced the most continuous assembly among the tools tested. -->

---

## Evaluate Assembly Accuracy 

| **Before Polishing**                       | **Pilon+Medaka**                       | **Shasta Assembly**                       |
|:------------------------------:|:------------------------------:|:------------------------------:|
| ![w:470px](../public/images/canu.png) | ![w:470px](../public/images/medaka.png) | ![w:470px](../public/images/shasta.png) |

<!-- ## Dotplot Analysis: Assembly Quality Assessment

### ✅ **Shasta**
* Shows **near-perfect linear 1:1 alignment** (contigs align consistently along the reference)
* **Minimal gaps and inversions** are observed
* **Six distinct large continuous regions** → likely accurate reconstruction of chromosome structure

### ⚠️ **Canu**
* Generally good but:
  * **Multiple gaps and inversions** (↘) in some contigs
  * **Multiple contigs map to the same chromosome**, indicating fragmentation
  * Some short contigs might be misassembled or unnecessary

### ⚠️ **Flye**
* Similar to Shasta but:
  * **Notable misalignments and breaks** around chromosomes 3 and 6
  * Linear structure is less clear in some regions

### ⚠️ **Raven**
* Most fragmented:
  * **High number of short, fragmented contigs**
  * **Frequent unaligned regions and orientation changes**
  * Poor long-range continuity

### ✅ Conclusion

**Shasta > Canu ≈ Flye > Raven**

* **Shasta**: Best preservation of chromosome-level structure
* **Canu & Flye**: Good but not perfect
* **Raven**: Requires additional polishing and scaffolding for research use
 -->

---

## Overall Assembly Evaluation

<style scoped>
table {
  width: 100%;
  font-size: 22px;
}
</style>

| Tool      | Evaluation                | Comments                                                                                       |
|-----------|---------------------------|-----------------------------------------------------------------------------------------------|
| **Shasta**| ⭐ Best for structure      | - Longest contig (12 Mb), top N50 (6.7 Mb)<br>- Nx/cumulative plots: covers most with few contigs<br>- Highest contig count (36) |
| **Canu**  | ⭐ High accuracy, low fragmentation | - Fewest contigs (14), good N50 (3.6 Mb), max contig 8.7 Mb<br>- Consistently strong in Nx/cumulative plots |
| **Raven** | △ Fast & practical        | - Longest total length (35.5 Mb), max contig 5.8 Mb<br>- N50/Nx lower than Shasta/Canu |
| **Flye**  | △ Balanced but weaker     | - Similar contig/N50 to Raven<br>- Max contig smaller (5 Mb), total length moderate (34.3 Mb) |

<!--
This table summarizes the strengths of each tool. Shasta is best for long contigs, Canu is best for fewer contigs and high accuracy, Raven is fast, and Flye is balanced. We choose the tool based on what is most important for our project.
-->

---

## Polishing Experiment Overview (Canu)

- **Procedure**:
  1. Pilon (2 consecutive rounds)
     - Using Illumina reads and ONT long reads
     - Effective for base substitution and indel corrections
  2. Medaka (1 round)
     - Using ONT reads
     - Pre-trained on ONT-specific error patterns
     - Strong in homopolymer region correction

<!--
Canu produced the fewest number of contigs, and if this assembly is correct, we can expect significant improvement in accuracy through polishing. Therefore, we proceeded with the polishing step.
First, I used Pilon twice, then Medaka. Polishing helps fix small errors that remain after assembly.
-->

---

## Polishing Method Comparison

![bg right:40% vertical w:450px](../public/images/polishing-nx.png)
![bg w:450px](../public/images/polishing-cumulative.png)

<style scoped>
table {
  width: 100%;
  font-size: 18px;
}
</style>

| Metric                | Raw Data | Pilon 1st | Pilon 2nd | Medaka |
| ----------------- | ------ | --------- | --------- | ------- |
| Mismatch rate (/100 kbp) | 256.34 | 165.23    | 128.52    | 134.82  |
| Indel rate (/100 kbp)  | 458.67 | 289.12    | 233.45    | 188.91  |
| Genome fraction (%)     | 96.234 | 96.892    | 97.182    | 97.190  |

<!--

The metrics shown in the table are:
1. Mismatch rate (/100 kbp): Number of base substitutions per 100 kilobases. A lower value indicates better accuracy in base calling.
2. Indel rate (/100 kbp): Number of insertions and deletions per 100 kilobases. A lower value shows better preservation of sequence length and structure.
3. Genome fraction (%): Percentage of the reference genome covered by the assembly. A higher value indicates more complete genome coverage.

The improvement in these metrics after polishing demonstrates that both Pilon and Medaka successfully enhanced the assembly quality by correcting errors and increasing genome coverage.
-->

---

## Evaluate Accuracy Improvement

| **Before Polishing**                       | **Pilon+Medaka**                       | **Shasta**                       |
|:------------------------------:|:------------------------------:|:------------------------------:|
| ![w:470px](../public/images/canu.png) | ![w:470px](../public/images/medaka.png) | ![w:470px](../public/images/shasta.png) |
<!-- 
However, when comparing the dotplots, while the polishing steps significantly improved the Canu assembly, the overall quality still does not reach the level of the Shasta assembly. This suggests that while polishing can correct many errors, it cannot fully compensate for structural differences in the initial assembly. 

The Shasta assembly is better chromosome-level structure and continuity remain unmatched even after extensive polishing of the Canu assembly.
-->

---

## Future Directions

1. **Polishing Shasta Result**
   - Select better performing assembly for further improvement

2. **Advanced Polishing**
     * Homopolish (for homopolymer regions)
     * NextPolish (alternative approach)

3. **Introduce Scaffolding**
   - Apply scaffolding to polished assembly

4. **Multi-assembly Integration**
   - Combine best features from both assemblies

<!--
In the future, I will polish the Shasta result, try more advanced polishing tools, use scaffolding to connect contigs, and combine the best parts of different assemblies. This will help make the genome even more complete and accurate.

1. **Polishing Shasta Result**
   - Apply Pilon and Medaka to the Shasta assembly
   - This could potentially improve the already high-quality assembly further
   - Focus on correcting any remaining small errors while maintaining the excellent chromosome-level structure

2. **Advanced Polishing**
   - Homopolish: Specifically targets homopolymer regions (stretches of identical nucleotides)
   - NextPolish: An alternative polishing pipeline that might catch errors missed by other tools
   - These tools can address specific types of errors that standard polishing might miss
   - Particularly useful for improving accuracy in challenging genomic regions

3. **Introduce Scaffolding**
   - Apply scaffolding to connect contigs into larger sequences
   - Use long-read data to bridge gaps between contigs
   - This could help resolve any remaining structural issues
   - Important for achieving chromosome-level assembly

4. **Multi-assembly Integration**
   - Combine best features from both Shasta and Canu assemblies
   - Use Shasta's superior chromosome-level structure as the backbone
   - Incorporate Canu's accurate regions where they show better quality
   - This hybrid approach could potentially yield the best possible assembly
-->
