# Slide 1: Title
Hello everyone. Today, I want to talk about the genome of a small organism called Dictyostelium discoideum. This organism is a type of social amoeba. In my presentation, I will explain how I made a high-quality genome sequence for this organism. 

# Slide 2: Purpose
The main goal of my research is to get a very accurate and high-quality genome sequence of Dictyostelium discoideum. 

# Slide 3: Background
Dictyostelium discoideum is called a model organism. This means scientists use it to study basic life processes, because it is easy to grow and has interesting features. It can live as a single cell, but it can also join with other cells to make a group. Its genome, which is all of its DNA, is about 34 million base pairs long. It has 6 chromosomes, many copies of a special DNA called rDNA, and also has mitochondria, which are like the cell's batteries. The genome has a lot of A and T letters, many tRNA genes, and many simple repeated sequences.

# Slide 4: Genome Assembly Workflow
To make the genome, I first get DNA sequence data using two methods. One is called ONT, which is long pieces of DNA, and the other is called Illumina, which is short but accurate pieces. After I get the data, I check if the quality is good. Then, I use computer programs to assembly the DNA pieces. After that, I polish, or correct, the genome to fix any mistakes. Finally, I check the quality again and try to improve it if needed.

# Slide 5: ONT vs Illumina
ONT gives me very long DNA reads, but these have more mistakes. 
Illumina gives me short reads, but they are very accurate. I use both types of data to get the best results. The long ONT reads help me see big parts of the genome, and the short Illumina reads help me fix small mistakes.


# Slide 7: ONT Read Length
In Dictyostelium discoideum ONT data, I got about 8.3 billion base pairs in total, from about 930,000 reads. The average read was about 9,000 base pairs long. The longest read was about 140,000 base pairs. The N50 was about 13,000 base pairs. This means that half of all the DNA is in reads longer than 13,000 base pairs.

# Slide 8: Assembly Experiment
To assemble the genome, I used about half of the ONT data. because If I use too much data, the process can become slow and less accurate. 

I tried different softowares for assembly. 
Canu is very good at fixing errors, but it is slow. Flye is good with repeated sequences and uses less memory. Shasta is very fast, but a bit less accurate. Raven is also fast and uses little memory.

# Slide 9: Assembly Results
Let's compare the results of the different assembly tools.

1. **Number of contigs (degree of fragmentation):**

   Fewer contigs mean the genome is less fragmented and more continuous.  
   Canu produced the fewest contigs, which is closest to the target chromosome number.

2. **Maximum contig length:**

   A longer maximum contig means larger chromosome fragments were assembled.  
   Shasta achieved the longest contig by far.

3. **Total assembly length:**

   The closer to the known genome size (about 33.8 Mb), the better.  
   Raven is a bit over, which may mean redundancy.  
   Shasta is slightly short, which may mean missing regions.  
   Flye and Canu are both close to the ideal.

4. **N50 (measure of continuity):**

   A higher N50 means more of the genome is in longer contigs.  
   Shasta is the best for continuity.

5. **GC content:**

   All are close to the known value of 23%, so there is no problem here.

# Slide 10: Overall Trends and Selection

From the plots:
- Shasta and Canu have a few long contigs that quickly reach the total genome length.
- Shasta stands out for continuity, while Canu is best for having fewer, larger contigs.

**How to choose:**
1. If you want the highest continuity (longest fragments): Shasta
2. If you want fewer contigs, close to the chromosome number, and good coverage: Canu
3. If you want to avoid redundancy but maximize coverage: Flye
4. If you want maximum coverage, even if a bit redundant: Raven

**Conclusion:**
- Canu is a balanced choice with fewer contigs and fewer errors.

# Slide 11: Evaluate Assembly Accuracy
To check how good my assembly is, I use bandage. 

# Slide 12: Polishing Experiment
After assembling the genome, I polish it to fix mistakes of canu result. First, I use Pilon two times. Pilon uses both Illumina and ONT reads to fix errors. 
Then, I use Medaka, with ONT data.

# Slide 13: Polishing Comparison
Pilon helps reduce the number of mistakes, and Medaka helps even more. After polishing, the genome becomes more complete and accurate.

# Slide 14: Current Status
Now, after using Canu and polishing, I have 14 contigs. The total length of the genome is 35 million base pairs. The N50 is about 3.7 million. The GC content is 22.8%. The genome is about 97% complete, and there are much fewer mistakes than before.

# Slide 15: Future Directions
In the future, I want to check if Canu is really the best program. I will also try more polishing tools to make the genome even better. I plan to scaffolding to connect the contigs. 

Thank you very much for listening to my presentation.
