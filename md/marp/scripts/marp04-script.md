**Slide 1**

Hello everyone. Today, I will talk about my project to build a high-quality genome sequence for *Dictyostelium discoideum*. 

My goal is to assemble the genome at the chromosome level. (A chromosome is a long, organized package of DNA inside each cell.)

Having a chromosome-level genome means I want to know the exact order of DNA across all of these packages. 

This is important for two main reasons. First, it helps us study the structure of centromeres, (which are special regions on each chromosome that keep DNA copies together during cell division. )
Second, it lets us analyze how different parts of the genome evolved together over time, giving insight into the organism’s biology and history. 

In this presentation, I will explain the steps how to achieve this goal.

---

**Slide 2**

Let’s begin with some background about *Dictyostelium discoideum*. It is used as a model organism because it switches between a single-cell form and a multicellular form under certain conditions. Its nuclear genome is about 34 million base pairs long;( each base pair is one “letter” in the DNA code. )
The genome is divided into six main chromosomes. In addition to these chromosomes, there are about 100 copies of an 88-kilobase rDNA, and there is a 56-kilobase circular mitochondrial genome. 

Three key features of this genome influence how I assemble it. 

First, the genome is extremely AT-rich—about 77.6 percent (of the DNA consists of the letters A and T. )

Because low complexity sequences like long runs of A’s or T’s can cause sequencing machines to slip or make mistakes, I need long-read sequencing that can span these difficult regions. 
Second, *Dictyostelium discoideum* contains around 390  tRNA genes scattered throughout the genome. 

Because tRNA are nearly identical, a short-read assembly might collapse them into one location, so having reads that cover large regions helps to place each tRNA gene correctly. 

Third, more than 11 percent of the genome consists of simple sequence repeats(SSR), which are short patterns of DNA that repeat many times in a row. These repeats can confuse short-read assembly tools, but they can also serve as useful markers for tracking genetic differences among strains. 

Taken together, these characteristics justify using a hybrid approach: long reads to achieve contiguity and short reads for accuracy.

---

**Slide 3**

Next, I will compare the two sequencing I used: ONT long reads and Illumina short reads. 

ONT, produces very long reads—while Illumina produces short reads. 

ONT long reads have lower per-base accuracy, meaning they make more errors such as insertions and deletions,  indels, and mismatches. 

By contrast, Illumina short reads have very high accuracy, and their errors are mostly single-base substitutions. 

The strength of ONT is that it can resolve large repeats and structural variants, because a single read can span through long repeated regions. 

Illumina is ideal for correcting small errors, because of its high accuracy. 

However, ONT’s lower accuracy requires extra steps to correct mistakes, and Illumina’s short reads cannot span long repetitive regions, so they struggle to assemble through those parts. By combining ONT for structure and Illumina for base-level accuracy, I leverage the strengths of both technologies to create a high-quality assembly.

---

**Slide3.5**

This slide shows the overall steps I followed to assemble the genome. 

First, I collected two kinds of DNA sequence data: long reads from ONT and short reads from Illumina. 

Next, I checked the quality of those reads to make sure they were good enough for assembly. After confirming quality, I ran four different assembly programs—Canu, Flye, Raven, and Shasta—and then compared their outputs using a tool QUAST to see which assembly looked best.

Once I picked good promising assemblies, I polished them by running Pilon and Medaka to correct small errors. 

Finally, I evaluated the polished assemblies again with QUAST, and if needed, I plan to reassemble or add scaffolding to connect contigs into larger sequences.

---

**Slide 4**

Here, I show the read length distribution of ONT data of *Dictyostelium discoideum*. 

There are about 8 billion bases in total from ONT reads, and there were 934,886 reads. 

On average, each ONT read was about 8,941 base pairs long. The longest read I obtained is 139,714 base pairs. 

The N50 is a metric that indicating the length at which 50 percent of the total bases are in reads at least that long

This was 12,777 base pairs. 

A higher N50 value suggests I have more long reads, which helps assemble large, continuous pieces of the genome. In the distribution graph, you can see that most reads fall between 1,000 and 50,000 base pairs, but there are also some very long reads exceeding 100,000 base pairs. In the zoomed-in view for reads up to 20,000 base pairs, you can clearly see that the average read length is around 9,000 base pairs and the N50 is also in that range. Having many short reads and some very long reads allows to piece together repeated regions while still covering the genome well.

---

**Slide 5**

Next, I will describe my assembly experiment. 

I used about 50 percent of the ONT reads—approximately 4.2 gigabases of data—because using all the data can increase computation time and sometimes reduce accuracy in certain assemblers. 

I also tested other coverage levels, such as 25 percent and 75 percent, but I found that using 50 percent gave the most accurate results. 

I compared four assembly tools: Canu, Flye, Shasta, and Raven. 

Canu has powerful error-correction steps but takes longer to run. 

Flye handles repetitive sequences well and memory-efficient. 

Shasta is ultra-fast but has slightly lower base accuracy. 

Raven uses very little memory and is also quite fast. 

By testing all four tools on the same dataset, I can see which one offers the best combination of contiguity, accuracy.

---

**Slide 6**

Now, let’s compare the assembly results using four key metrics. 

First, the number of contigs measures how many continuous pieces the assembly is broken into. A lower number of contigs is better because it indicates a more complete, less fragmented assembly. 

Canu produced the fewest contigs, with only 14, which is the best result in this category. 

Second, I look at the size of the largest contig. 

A larger largest contig is better because it shows the ability to assemble long continuous regions. Shasta produced the longest contig at 12 megabases, which is the best performance for this metric. 

Third, I compare the total length of all contigs. 

The expected reference genome size is 34.2 megabases, but this does not include repetitive regions, so the actual assembly may be longer. Still, the closer it is to that number, the better.

Flye produced a total length of 34.3 megabases, and Canu produced 34.6 megabases, both very close to the reference. Raven produced a total length of 35.5 megabases, which is slightly larger than expected and suggests it may have duplicated or included extra regions. 

Shasta produced a total length of 33.5 megabases, which is smaller than expected, indicating it may have collapsed some repeats. 

Finally, I consider the N50 value, which measures contig length distribution. 

Shasta achieved the highest N50 at 6.7 megabases, which is the best result. 

In summary, Canu gave the fewest contigs and a solid N50, while Shasta produced the largest contig and highest N50.

---

**Slide 7**

Next, I performed dotplot analyses to assess assembly accuracy. 
A dotplot compares each assembled contig against a reference genome by plotting matching regions. 

A perfectly linear diagonal line means the contig aligns perfectly along a chromosome. 

For Shasta, the dotplot shows almost perfect linear alignment with minimal breaks or inversions. This indicates that Shasta’s contigs match the reference chromosomes very well, capturing chromosome structure accurately. 

For Canu, the dotplot is also very good, but there are a few gaps and minor inversions. 

Some contigs map to the same chromosome, suggesting a slightly more fragmented assembly. For Flye, the dotplot shows similar alignment to Shasta in several regions but with noticeable misalignments and breaks around certain chromosomes, indicating that some regions were misplaced or inverted. 

For Raven, the dotplot is the most fragmented. Many short contigs align in pieces, with frequent gaps and orientation changes, showing poor continuity. 

In conclusion, Shasta performed best at preserving chromosome structure, followed by Canu, then Flye, and Raven was the most fragmented.

---

**Slide 8**

### I also ran a BUSCO analysis.

It checks how complete an assembly is by looking for genes expected to appear exactly once in the genome. Here's a summary of key results for each assembler, focusing on a standout metric in each category.

- **Completeness**:
    
    Canu, Raven, and Flye each captured 94.9% of the expected genes completely — the highest among the four.
    
- **Single-copy genes**:
    
    Canu and Flye both had the highest number of single-copy genes, at 236 (92.5%), showing low redundancy.
    
- **Duplicated genes**:
    
    Shasta had the lowest duplication rate, with only 4 duplicated genes (1.6%).
    
- **Fragmented genes**:
    
    All four assemblers had 3 fragmented genes (1.2%), showing no major differences.
    
- **Missing genes**:
    
    Shasta had the highest number of missing genes, 19 (7.5%), which is above the ideal threshold.
    
- **Stop-codon errors**:
    
    Shasta had the fewest internal stop codon errors — only 1 (0.4%).
    

Overall, Canu, Raven, and Flye performed similarly well in completeness, with Canu showing the best overall balance. 

Shasta performed slightly lower, but still within an acceptable range.

---

**Slide 9**

Here is an overall evaluation summary of the four assembly tools. 

For Canu, I conclude that it has high accuracy and low fragmentation. It produced the fewest contigs and achieved a good N50. 

Its maximum contig length was longest, and it showed consistently strong performance in the Nx and cumulative plots. Therefore, we rate Canu as having high accuracy and low fragmentation. For Shasta, it is good for structure. It produced the longest contig and achieved the top N50 of 6.7 megabases. Although it had the highest number of contigs, at 36, indicating more fragmentation, its ability to preserve long continuous sequences made it stand out. 

I rate Shasta as good for structural continuity. 

For Flye, we observe that it is balanced. It shows performance similar to Raven in terms of the number of contigs and N50, with a maximum contig size of 5 megabases and a total assembly length of 34.3 megabases, very close to the reference. Thus, Flye offers a balanced tradeoff between contiguity and accuracy. Finally, Raven is fast and practical. It produced a total length of 35.5 megabases, which is slightly larger than the reference, and had a maximum contig length of 5.8 megabases. Its N50 was lower than those of Shasta and Canu, indicating more fragmentation, but Raven is quick to run and uses little memory. Based on these results, we decided to focus our next steps on Canu and Shasta, as they showed the best performance in terms of contig number and assembly continuity.

---

**Slide 10**

Now, I will explain polishing experiment. Polishing is the process of correcting small errors in an assembled genome to improve its accuracy. 

I used two polishing tools: Pilon and Medaka. 

Pilon uses Illumina short reads and ONT long reads to correct both base substitutions, which are wrong letters in the DNA sequence, and indels, which are insertions and deletions of letters. Because Illumina reads have very high accuracy, they are ideal for fixing small mistakes. 

Medaka, on the other hand, works only with ONT reads and is pre-trained to recognize common error patterns specific to ONT sequencing. 

It is especially strong at correcting errors in homopolymer regions, which are stretches of the same letter repeated many times.  My process is to first run Pilon, which would correct many base-level errors, and then follow with Medaka to address any remaining ONT-specific mistakes. 

This two-step approach usually results in significant improvements in accuracy and genome coverage.

---

**Slide 12**

Now I will compare the Canu assembly before and after polishing using dotplots. Before polishing, the Canu contigs align reasonably well to the reference genome, but you can see occasional small gaps and minor misalignments. 

After applying Pilon twice and then running Medaka, the dotplot shows contigs aligned almost perfectly along a straight diagonal line with very few gaps. 

This visual improvement corresponds to the reduced mismatch and indel rates that we observed in the numeric metrics. 

Polishing made the assembly align more closely with the reference, confirming that the small errors were successfully corrected.

---

**Slide 13**

Similarly, let me compare the Shasta assembly before and after polishing.

Before polishing, Shasta’s contigs already show long, continuous alignments to the reference. However, you can still see small alignment breaks and some misaligned segments, especially in regions where ONT errors are common.

After applying Pilon, the dotplot shows higher identity in some regions — reflected by darker colors — but the small alignment breaks and misaligned fragments noted earlier still remain. Many small contig fragments are also visible. These issues are expected to be resolved in the subsequent scaffolding step.

---

**Slide 14**

Finally, let me outline our future directions. 
First, I will compare the polished results to determine which assembly performs better overall in terms of both completeness and accuracy. 
Second, I plan to apply advanced polishing tools such as Homopolish, which is specifically designed to correct errors in homopolymer regions, and NextPolish, which is an alternative polishing pipeline. 

These tools may catch errors that Pilon and Medaka miss. 

Third, I will introduce scaffolding to connect contigs into larger sequences. 

Scaffolding uses long-read or additional data to bridge gaps between contigs, which can help resolve remaining structural issues and bring closer to a chromosome-level assembly. 

This step also validates contig order and orientation while correcting potential misjoins.

Fourth, I will pursue multi-assembly integration by combining the best features of different assemblies. This hybrid approach could yield the most complete and accurate final genome. 

Thank you for listening. That concludes my presentation on genome assembly and polishing for *Dictyostelium discoideum*.