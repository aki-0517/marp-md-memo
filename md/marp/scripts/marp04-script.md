**Slide 1**

Hello everyone. Today, I will talk about my project to build a high-quality genome sequence for *Dictyostelium discoideum*. *Dictyostelium discoideum* is a social amoeba—a tiny single-celled organism that can come together to form a multicellular body under certain conditions. Scientists use it as a model to study how cells work and how they communicate. 

My goal is to assemble the genome at the chromosome level. (A chromosome is a long, organized package of DNA inside each cell.)

Having a chromosome-level genome means we want to know the exact order of DNA across all of these packages. This is important for two main reasons. First, it helps us study the structure of centromeres, (which are special regions on each chromosome that keep DNA copies together during cell division. )
Second, it lets us analyze how different parts of the genome evolved together over time, giving us insight into the organism’s biology and history. In this presentation, I will explain the steps I took to achieve this goal.

---

**Slide 2**

Let me begin with some background about *Dictyostelium discoideum*. It is used as a model organism because it switches between a single-cell form and a multicellular form under certain conditions. Its nuclear genome is about 34 million base pairs long;( each base pair is one “letter” in the DNA code. )
The genome is divided into six main chromosomes. In addition to these chromosomes, there are about 100 copies of an 88-kilobase rDNA, and there is a 56-kilobase circular mitochondrial genome. 

Three key features of this genome influence how I assemble it. 

First, the genome is extremely AT-rich—about 77.6 percent of the DNA consists of the letters A and T. Because low complexity sequences like long runs of A’s or T’s can cause sequencing machines to slip or make mistakes, I need long-read sequencing that can span these difficult regions. 
Second, *Dictyostelium discoideum* contains around 390 transfer RNA genes scattered throughout the genome. Transfer RNA genes produce molecules that help build proteins. Because these genes are nearly identical, a short-read assembly might collapse them into one location, so having reads that cover large regions helps us place each tRNA gene correctly. Third, more than 11 percent of the genome consists of simple sequence repeats, which are short patterns of DNA that repeat many times in a row. These repeats can confuse short-read assembly tools, but they can also serve as useful markers for tracking genetic differences among strains. Taken together, these characteristics justify using a hybrid approach: long reads to achieve contiguity and short reads for accuracy.

---

**Slide 3**

Next, I will compare the two sequencing technologies we used: ONT long reads and Illumina short reads. 

ONT, produces very long reads—while Illumina produces short reads. 

ONT long reads have lower per-base accuracy, meaning they make more errors such as insertions and deletions, also known as indels, and mismatches. 

By contrast, Illumina short reads have very high accuracy, and their errors are mostly single-base substitutions. 

The strength of ONT is that it can resolve large repeats and structural variants, because a single read can span through long repeated regions. Illumina is ideal for correcting small errors, because of its high accuracy. 

However, ONT’s lower accuracy requires extra steps to correct mistakes, and Illumina’s short reads cannot span long repetitive regions, so they struggle to assemble through those parts. By combining ONT for structure and Illumina for base-level accuracy, we leverage the strengths of both technologies to create a high-quality assembly.

---

**Slide 4**

This slide shows the overall steps I followed to assemble the genome. First, I collected two kinds of DNA sequence data: long reads from ONT and short reads from Illumina. Next, I checked the quality of those reads to make sure they were good enough for assembly. After confirming quality, I ran four different assembly programs—Canu, Flye, Raven, and Shasta—and then compared their outputs using a tool called QUAST to see which assembly looked best. Once I picked the most promising assemblies, I polished them by running Pilon and Medaka to correct small errors. Finally, I evaluated the polished assemblies again with QUAST, and if needed, I plan to reassemble or add scaffolding to connect contigs into larger sequences.

---

**Slide 5**

Here, I show the read length distribution for our ONT data of *Dictyostelium discoideum*. We collected about 8.36 billion bases in total from ONT reads, and there were 934,886 reads. On average, each ONT read was about 8,941 base pairs long. The longest read we obtained was 139,714 base pairs. 

The N50, which is a metric indicating the length at which 50 percent of the total bases are in reads at least that long, was 12,777 base pairs. 

A higher N50 value suggests we have more long reads, which helps assemble large, continuous pieces of the genome. In the distribution graph, you can see that most reads fall between 1,000 and 5,000 base pairs, but there are also some very long reads exceeding 100,000 base pairs. In the zoomed-in view for reads up to 20,000 base pairs, you can clearly see that the average read length is around 9,000 base pairs and the N50 is also in that range. Having many short reads alongside some very long reads allows us to piece together repeated regions while still covering the genome well.

---

**Slide 6**

Next, I will describe our assembly experiment. We used about 50 percent of the ONT reads—approximately 4.2 gigabases of data—because using all the data can increase computation time and sometimes reduce accuracy in certain assemblers. We also tested other coverage levels, such as 25 percent and 75 percent, but we found that using 50 percent gave the most accurate results in our trials. We compared four assembly tools: Canu, Flye, Shasta, and Raven. Canu has powerful error-correction steps but takes longer to run. Flye handles repetitive sequences well and is memory-efficient. Shasta is ultra-fast but has slightly lower base accuracy. Raven uses very little memory and is also quite fast. By testing all four tools on the same 50 percent dataset, we can see which one offers the best combination of contiguity, accuracy, and resource usage.

---

**Slide 7**

Now, let’s compare the assembly results from Raven, Flye, Shasta, and Canu using four key metrics. First, the number of contigs measures how many continuous pieces the assembly is broken into. A lower number of contigs is better because it indicates a more complete, less fragmented assembly. Canu produced the fewest contigs, with only 14, which is the best result in this category. Raven produced 28 contigs, Flye produced 33, and Shasta produced 36. 

Second, we look at the size of the largest contig. A larger largest contig is better because it shows the ability to assemble long continuous regions. Shasta produced the longest contig at 12 megabases, which is the best performance for this metric. Canu’s largest contig was 8.7 megabases, Raven’s was 5.8 megabases, and Flye’s was 5.0 megabases. Third, we compare the total length of all contigs. The expected reference genome size is 34.2 megabases, so closer to that number is better. Flye produced a total length of 34.3 megabases, and Canu produced 34.6 megabases, both very close to the reference. Raven produced a total length of 35.5 megabases, which is slightly larger than expected and suggests it may have duplicated or included extra regions. Shasta produced a total length of 33.5 megabases, which is slightly smaller than expected, indicating it may have collapsed some repeats. Finally, we consider the N50 value, which measures contig length distribution. A higher N50 indicates a more continuous assembly. Shasta achieved the highest N50 at 6.7 megabases, which is the best result. Canu’s N50 was 3.6 megabases, Flye’s was 2.8 megabases, and Raven’s was 2.7 megabases. In summary, Canu gave the fewest contigs and a solid N50, while Shasta produced the largest contig and highest N50.

---

**Slide 8**

Next, we performed dotplot analyses to assess assembly accuracy. A dotplot compares each assembled contig against a reference genome by plotting matching regions. 

A perfectly linear diagonal line means the contig aligns perfectly along a chromosome. For Shasta, the dotplot shows almost perfect linear alignment with minimal breaks or inversions. This indicates that Shasta’s contigs match the reference chromosomes very well, capturing chromosome structure accurately. For Canu, the dotplot is also very good, but there are a few gaps and minor inversions. Some contigs map to the same chromosome, suggesting a slightly more fragmented assembly. For Flye, the dotplot shows similar alignment to Shasta in several regions but with noticeable misalignments and breaks around certain chromosomes, indicating that some regions were misplaced or inverted. For Raven, the dotplot is the most fragmented. Many short contigs align in pieces, with frequent gaps and orientation changes, showing poor continuity. In conclusion, Shasta performed best at preserving chromosome structure, followed by Canu, then Flye, and Raven was the most fragmented.

---

**Slide 9**

We also ran a BUSCO analysis.

It checks how complete an assembly is by looking for genes expected to be present exactly once in a given lineage. Here are the results for each assembler. 

First, we look at the percentage of complete genes found. Canu, Raven, Shasta captured 94.9 percent of the expected genes completely. 

Shasta captured 91.4 percent, which is still good but slightly lower. 

Second, we consider how many of those complete genes are single-copy, meaning they appear exactly once. Canu found 236 single-copy genes, which is 92.5 percent of the total, indicating low redundancy. Raven found 235 single-copy genes, or 92.2 percent. Flye found 236 single-copy genes, or 92.5 percent. Shasta found 229 single-copy genes, or 89.8 percent. Third, we check the number of duplicated genes—those found more than once. A low duplication rate is desirable. Canu had 6 duplicated genes, which is 2.4 percent of the total. Raven had 7 duplicated genes, or 2.7 percent. Flye had 6 duplicated genes, or 2.4 percent. Shasta had 4 duplicated genes, or 1.6 percent. Fourth, we look at fragmented genes, which are only partially detected. All four assemblers had 3 fragmented genes, or 1.2 percent, which is acceptable but ideally under 2 percent. Fifth, we check missing genes, which were not detected at all. Canu, Raven, and Flye each missed 10 genes, representing 3.9 percent, while Shasta missed 19 genes, representing 7.5 percent. A missing rate under 5 percent is ideal, so Canu, Raven, and Flye meet that criterion, while Shasta is slightly above. Finally, we look at stop-codon errors, which are complete genes that contain internal stop codons. Canu had 2 such errors, or 0.8 percent. Raven had 2 errors, or 0.8 percent. Flye had 3 errors, or 1.2 percent. Shasta had 1 error, or 0.4 percent. Ideally, we want that rate to be under 1 percent. Overall, Canu, Raven, and Flye each show 94.9 percent completeness, with Canu having the best balance of low missing genes and low duplication. Shasta is slightly lower in completeness, at 91.4 percent, but still respectable.

---

**Slide 10**

Here is an overall evaluation summary of the four assembly tools. For Canu, we conclude that it has high accuracy and low fragmentation. It produced the fewest contigs—only 14—and achieved a good N50 of 3.6 megabases. Its maximum contig length was 8.7 megabases, and it showed consistently strong performance in the Nx and cumulative plots. Therefore, we rate Canu as having high accuracy and low fragmentation. For Shasta, we note that it is good for structure. It produced the longest contig at 12 megabases and achieved the top N50 of 6.7 megabases. Although it had the highest number of contigs, at 36, indicating more fragmentation, its ability to preserve long continuous sequences made it stand out. We rate Shasta as good for structural continuity. For Flye, we observe that it is balanced. It shows performance similar to Raven in terms of the number of contigs and N50, with a maximum contig size of 5 megabases and a total assembly length of 34.3 megabases, very close to the reference. Thus, Flye offers a balanced tradeoff between contiguity and accuracy. Finally, Raven is fast and practical. It produced a total length of 35.5 megabases, which is slightly larger than the reference, and had a maximum contig length of 5.8 megabases. Its N50 was lower than those of Shasta and Canu, indicating more fragmentation, but Raven is quick to run and uses little memory. Based on these results, we decided to focus our next steps on Canu and Shasta, as they showed the best performance in terms of contig number and assembly continuity.

---

**Slide 11**

Now, I will explain our polishing experiment. Polishing is the process of correcting small errors in an assembled genome to improve its accuracy. We used two polishing tools: Pilon and Medaka. Pilon uses Illumina short reads and ONT long reads to correct both base substitutions, which are wrong letters in the DNA sequence, and indels, which are insertions and deletions of letters. Because Illumina reads have very high accuracy, they are ideal for fixing small mistakes. Medaka, on the other hand, works only with ONT reads and is pre-trained to recognize common error patterns specific to ONT sequencing. It is especially strong at correcting errors in homopolymer regions, which are stretches of the same letter repeated many times, such as “AAAAA.” Our procedure was to first run Pilon twice, which would correct many base-level errors, and then follow with Medaka to address any remaining ONT-specific mistakes. This two-step approach usually results in significant improvements in accuracy and genome coverage.

---

**Slide 12**

Here are the polishing results for the Canu assembly. We assessed three metrics at four stages: before any polishing, after the first Pilon run, after the second Pilon run, and after Medaka. The first metric is the mismatch rate, reported as the number of base substitutions per 100 kilobases. Initially, the raw assembly had a mismatch rate of 256.34, which is quite high. After the first Pilon run, the mismatch rate dropped to 165.23. After the second Pilon run, it further dropped to 128.52, representing the best improvement in our pipeline. Following Medaka, the mismatch rate was 134.82, which is still very good. The second metric is the indel rate, reported as the number of insertions and deletions per 100 kilobases. The raw assembly’s indel rate was 458.67, which is high. After the first Pilon run, it dropped to 289.12. After the second Pilon run, it dropped further to 233.45. Finally, after Medaka, it reached 188.91, which is the lowest indel rate observed and the best outcome for indel correction. The third metric is the genome fraction, which measures the percentage of the reference genome covered by the assembly. Initially, the raw assembly covered 96.234 percent of the reference. After the first Pilon run, coverage increased to 96.892 percent. After the second Pilon run, coverage went up to 97.182 percent. Finally, after Medaka, coverage reached 97.190 percent. These results show that each polishing step consistently improved accuracy by reducing both mismatches and indels, while also increasing the fraction of the genome covered.

---

**Slide 13**

Now I will compare the Canu assembly before and after polishing using dotplots. Before polishing, the Canu contigs align reasonably well to the reference genome, but you can see occasional small gaps and minor misalignments. After applying Pilon twice and then running Medaka, the dotplot shows contigs aligned almost perfectly along a straight diagonal line with very few gaps. This visual improvement corresponds to the reduced mismatch and indel rates that we observed in the numeric metrics. Polishing made the assembly align more closely with the reference, confirming that the small errors were successfully corrected.

---

**Slide 14**

Similarly, let me compare the Shasta assembly before and after polishing. Before polishing, Shasta’s contigs already show long, continuous alignments to the reference. However, you can still see small alignment breaks and some misaligned segments, especially in regions where ONT errors are common. After applying Pilon twice and then Medaka, the dotplot shows a cleaner, straighter alignment with fewer breaks. The long contigs remain intact, but now they also match the reference more precisely. This indicates that polishing fixed many small errors while preserving Shasta’s strength in large-scale continuity. In both the Canu and Shasta assemblies, polishing successfully improved base accuracy without compromising large-scale structural quality.

---

**Slide 15**

Finally, let me outline our future directions. First, we will compare the polished results from Canu and Shasta to determine which assembly performs better overall in terms of both completeness and accuracy. Second, we plan to apply advanced polishing tools such as Homopolish, which is specifically designed to correct errors in homopolymer regions, and NextPolish, which is an alternative polishing pipeline. These tools may catch errors that Pilon and Medaka miss, especially in challenging genomic regions. Third, we will introduce scaffolding to connect contigs into larger sequences. Scaffolding uses long-read or additional data to bridge gaps between contigs, which can help resolve remaining structural issues and bring us closer to a chromosome-level assembly. Fourth, we will pursue multi-assembly integration by combining the best features of different assemblies. For example, we might use Shasta’s superior chromosome-level structure as a backbone and incorporate Canu’s accurately assembled regions in areas where it outperforms Shasta. This hybrid approach could yield the most complete and accurate final genome. Thank you for listening. That concludes my presentation on genome assembly and polishing for *Dictyostelium discoideum*.