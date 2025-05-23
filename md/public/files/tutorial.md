Step-by-step method details
Visualizing read-length distribution
Timing: 1 h
The continuous long-read (CLR) mode of Pacific Biosciences (PacBio) or Oxford Nanopore Technologies (ONT) will generate reads of varying sizes, thus necessitating the use of statistics to determine whether or not the sequencing was successful. It is important to visualize read-length distributions and ensure that your reads were properly generated. N50, a read- or contig-length distribution statistic, can be used for assessing the read-length quality. N50 is the shortest read or contig length obtained when the cumulative length of the longest read or contig length equals 50% of the total read or assembly length. We present scripts to visualize the read-length distributions of the three long-read datasets.

Note: The following scripts contain seven symbols, such as `, ', ", ‘, ’, “, and ”. These seven symbols appear similar to each other; however, they serve distinct functions in a script. To accurately use the scripts, please do not copy and paste them in MS Word; otherwise, Word may automatically transform one symbol into another, and the script may not function at all.
Run the following scripts in your terminal to create a read-length table:
#!/usr/bin/env bash

# Create a new file and generate a header line

echo "platform,length" > length.csv

# Add each read length into the length.csv file.

bioawk -c fastx '{print "PacBio_CLR," length($seq)}' SRR11906525_WGS_of_drosophila_melanogaster_female_adult_subreads.fastq.gz >> length.csv

bioawk -c fastx '{print "PacBio_HiFi," length($seq)}' SRR12473480_Drosophila_PacBio_HiFi_UltraLow_subreads.fastq.gz >> length.csv

bioawk -c fastx '{print "ONT," length($seq)}' SRR13070625_1.fastq.gz >> length.csv

Visualize the read-length distribution data using R ggplot2. Save this script as a new file and run it, or type the following script directly into R or Rstudio. The output will be similar to that presented in Figure 2A:
Fig2.jpg
Figure 2. De novo genome assembly read-length distribution and quality assessment
(A) Read-length distributions of the three publicly available datasets used in this study. Each vertical dotted line represents the mean value of each dataset.

(B) Cumulative coverage plot for the contig/scaffold length.

(C) BUSCO analysis used to determine the number of single-copy orthologs known in a lineage.

#!/usr/bin/env Rscript

# Please specify your working directory using setwd

setwd("/path/to/Input_CSV_file")

library(ggplot2)

library(dplyr)

library(cowplot)

# Import the read-length distribution table

read_length_df <- read.csv("length.csv")

# Organize the imported read-length table

# You can replace the level arguments for your platform, species, or strains

read_length_df$platform <- as.factor(read_length_df$platform)

read_length_df$platform <- factor(read_length_df$platform,level = c("PacBio_CLR","PacBio_HiFi","ONT"))

# Calculate the average read-lengths for each platform

summary_df <- ddply(read_length_df, "platform", summarise, grp.mean=mean(length))

# Draw a read-length distribution plot for all reads

total.length.plot <- ggplot(read_length_df, aes(x=length, fill=platform, color=platform)) +

  geom_histogram(binwidth=100, alpha=0.5, position="dodge") +

  geom_vline(data=summary_df, aes(xintercept=grp.mean, color=platform), linetype="dashed", size =0.2) +

  scale_x_continuous(labels = comma) +

  scale_y_continuous(labels = comma) +

  labs(x = "Read length (bp)", y = "Count") +

  theme_bw()

# Draw a read-length distribution plot for reads ≤ 20 kb in length

20 kb.length.plot <- ggplot(read_length_df, aes(x=length, fill=platform, color=platform)) +

  geom_histogram(binwidth=50, alpha=0.5, position="dodge") +

  geom_vline(data=summary_df, aes(xintercept=grp.mean, color=platform), linetype="dashed", size=0.2) +

  scale_x_continuous(labels = comma, limit = c(0,20000)) +

  scale_y_continuous(labels = comma) +

  labs(x = "Read length (bp)", y = "Count") +

  theme_bw()

# Merge both the read-length distribution plots

plot <- plot_grid(total.length.plot, 20 kb.length.plot, ncol = 1)

# Save the figure using the file name, “read.length.pdf”

pdf("read.length.pdf",width=6,height=8,paper='special')

print(plot)

dev.off()

Calculate N50 statistics using assembly-stats. You can save or type this script in your terminal to run it:
#!/usr/bin/env bash

# Unzipped FASTA/Q files are required for assembly-stats

# You can unzip your fastq.gz files using the command “gzip -d file_name.fastq.gz”

# For general usage, specify the read or contig file names after “assembly-stats”

# Calculate summary stats and save the output as an “N50_stat” file

assembly-stats SRR11906525_WGS_of_drosophila_melanogaster_female_adult_subreads.fastq >> N50_stat

assembly-stats SRR12473480_Drosophila_PacBio_HiFi_UltraLow_subreads.fastq >> N50_stat

assembly-stats SRR13070625_1.fastq >> N50_stat

# You can see the output of assembly-stats by typing “cat N50_stat” in your terminal

> cat N50_stat

# The following is the output of “cat N50_stat” command (result of assembly-stats)

stats for SRR11906525_WGS_of_drosophila_melanogaster_female_adult_subreads.fastq

sum = 12016661679, n = 1437524, ave = 8359.28, largest = 99345

N50 = 13094, n = 321336

N60 = 11342, n = 419876

N70 = 9489, n = 535376

N80 = 7388, n = 678061

N90 = 4902, n = 874814

N100 = 50, n = 1437524

N_count = 0

Gaps = 0

stats for SRR12473480_Drosophila_PacBio_HiFi_UltraLow_subreads.fastq

sum = 25600110705, n = 2301518, ave = 11123.14, largest = 26462

N50 = 11151, n = 976954

N60 = 10586, n = 1212627

N70 = 10055, n = 1460775

N80 = 9530, n = 1722273

N90 = 8996, n = 1998694

N100 = 369, n = 2301518

N_count = 0

Gaps = 0

stats for SRR13070625_1.fastq

sum = 7133020037, n = 640215, ave = 11141.60, largest = 417450

N50 = 21491, n = 83878

N60 = 16642, n = 121685

N70 = 12824, n = 170598

N80 = 9526, n = 235039

N90 = 6112, n = 327186

N100 = 1, n = 640215

N_count = 0

Gaps = 0

Note: For the PacBio CLR mode and ONT, high-quality DNA would have >10-kb N50 read lengths, and a high-quality genome assembly would have >1-Mb N50 contig lengths (Kim et al., 2019a, 2020, 2021; ; ).
Approximate genome-size estimation
Timing: 5 h
This part of the protocol is required when generating data for a novel species. After estimating the genome size, you can determine the required sequencing throughput for your species. A high-quality genome assembly necessitates more than 20× sequencing coverage. We propose three methodologies that can be employed depending on the situation. You can skip this step if you are analyzing public datasets.

The estimated genome size of your species can be found in public databases:
Animal: Animal Genome Size Database (http://www.genomesize.com/index.php)
Plant: Plant DNA C-values Database (https://cvalues.science.kew.org/
If you have short-read DNA sequencing data, the k-mer-based genome size estimation can be applied:
#!/usr/bin/env bash

# KAT is a toolkit for addressing assembly completeness through k-mer counts (Mapleson et al., 2017)

# More information about KAT in: https://github.com/TGAC/KAT

# You can use the short-read DNA sequencing data provided in the Key Resource Table (Accession number: SRX8624462) to run the following script

# You need to provide the file path to the sequencing data or run this script in the same folder where the sequencing data is saved

kat hist -o prefix -t 10 SRR12099722∗ 1> kat.output.txt

echo dme_size >> genome_size.txt

grep -i "Estimated" kat.output.txt >> genome_size.txt

# hist: a kat module for drawing histograms and estimating genome size

# -o: output prefix; you can specify “prefix” for your species or strain names

# -t: the number of threads that will be used to run the kat program

# You can replace SRR12099722∗ with your short-read DNA sequencing data# You can replace dme_size with the name of your species

# You can check the kat output by typing “cat genome_size.txt” in your terminal

> cat genome_size.txt

# Genome size can be estimated using the short-read DNA sequencing data

dme_size

Estimated genome size: 166.18 Mbp

Estimated heterozygous rate: 0.41%

Calculate the transcript-based coverage using short-read DNA/RNA sequencing data.
Critical: For an accurate estimation, high-quality transcriptome assembly is required.
Conduct de novo transcriptome assembly using Trinity (Grabherr et al., 2011):
#!/usr/bin/env bash

# Trinitiy is a package for conducting de novo transcriptome assembly from RNA-seq data

# For more information: https://github.com/trinityrnaseq/trinityrnaseq/wiki

# You can use the short-read RNA sequencing data provided in the Key Resource Table (Accession number: GSM5452671, GSM5452672) to run the following script

# You need to provide the file path to the sequencing data or run this script in the same folder where the sequencing data are saved

Trinity --seqType fq --max_memory 120G --left /home/assembly/analysis/00_STARprotocol/SRR15130841_GSM5452671_Control_CM1_Drosophila_melanogaster_RNA-Seq_1.fastq.gz,/home/assembly/analysis/00_STARprotocol/SRR15130842_GSM5452672_Control_CM2_Drosophila_melanogaster_RNA-Seq_1.fastq.gz --right /home/assembly/analysis/00_STARprotocol/SRR15130841_GSM5452671_Control_CM1_Drosophila_melanogaster_RNA-Seq_2.fastq.gz,/home/assembly/analysis/00_STARprotocol/SRR15130842_GSM5452672_Control_CM2_Drosophila_melanogaster_RNA-Seq_2.fastq.gz --CPU 8 --output Dmel.trinity

# --seqType: sequence type; as short-read sequencing data are typically present in the FASTQ format, you can specify this as “fq”

# # --max_memory: maximum memory required to run the Trinity. “120G” indicates 120 GB

# --left and --right: input files required for trinity analysis. Currently, short-read sequencing is mainly performed in a “paired-end” mode. Each DNA molecule is sequenced at both the ends, producing two paired files. You should specify one as “--left” and the other as “—right”

# --CPU: the number of threads required for Trinity analysis

# --output: output prefix

# Trinity output should be in the “Dmel.trinity” (or “your_species_trinity”) folder

# Assembled transcript FASTA file will be “Dmel.trinity.Trinity.fasta” (or “your_species_trinity.Trinity.fasta”)

# You can assess the assembled quality of transcriptomes using assembly-stat

> assembly-stats Dmel.trinity.Trinity.fasta

# The following is the output of the “assembly-stats Dmel.trinity.Trinity.fasta” command (result of assembly-stats)

stats for Dmel.trinity.Trinity.fasta

sum = 72662995, n = 67038, ave = 1083.91, largest = 27780

N50 = 2454, n = 8357

N60 = 1816, n = 11781

N70 = 1180, n = 16695

N80 = 653, n = 25022

N90 = 364, n = 40185

N100 = 201, n = 67038

N_count = 0

Gaps = 0

Map the short DNA reads to the transcriptome using HISAT2:
#!/usr/bin/env bash

# HISAT2 was used to map DNA sequencing reads to the assembled transcripts, and SAMtools was used to process the alignment data

# For more information about HISAT2: http://daehwankimlab.github.io/hisat2/

# HISAT2 ref: https://www.nature.com/articles/s41587-019-0201-4

# For more information about SAMtools: http://www.htslib.org/

# SAMtools ref: https://www.ncbi.nlm.nih.gov/pmc/articles/PMC2723002/

# Index your assembled transcript FASTA file using the prefix “Dmel”

hisat2-build Dmel.trinity.fa Dmel

# Map your short-read DNA sequences to the assembled transcript using the index

# You can use the short-read DNA sequencing data provided in the Key Resource Table (Accession number: SRX8624462) to run the following script

# You need to provide the file path to the sequencing data or run this script in the same folder where the sequencing data are saved

hisat2 -x Dmel -p 10 -1 SRR12099722∗_1∗ -2 SRR12099722∗_2∗ --very-sensitive | samtools sort -@ 10 -o Dmel.very_sensitive.bam

# For HISAT2, the parameters are as follows:

# -x: index prefix

# -p: the number of threads required by HISAT2

# -1 and -2: paired-end files; you can change the name of your sequencing data

# --very-sensitive: sensitivity option

# For SAMtools, the parameters are as follows:

# sort: SAMtools module to sort the mapped read information

# -@: the number of threads required by SAMtools

# -o: output file name

# Index your read mapping file

samtools index Dmel.very_sensitive.bam

Estimate the genome size:
#!/usr/bin/env bash

# Calculate average coverage of each transcript

samtools coverage Dmel.very_sensitive.bam | awk '{print $7}' | tail -n +2 | grep -vw "0" | awk '{sum+=$1}END{print sum/NR}' > average.coverage.txt

# coverage: SAMtools module to calculate the coverage of each transcript (or contig, scaffold, etc.)

# awk '{print $7}': select the coverage column in the output of SAMtools coverage

# tail -n +2: remove the header line

# grep -vw "0": remove “0” coverage rows

# awk '{sum+=$1}END{print sum/NR}': calculate the average coverage with non-zero values

# Calculate the total read length of the DNA sequencing file

bioawk -c fastx '{sum+=length($seq)}END{print sum}' SRR12099722_WGS_Drosophila_melanogaster_adult_ISCs_1.fastq.gz > total.read.length.txt

# Estimate the genome size

paste average.coverage.txt total.read.length.txt | awk '{print "Estimated genome size = " $2∗2/$1/1000000 " Mb"}'

# After running the preceding script, the following result will be displayed in your terminal

Estimated genome size = 185.04 Mb

Long-read sequencing-based genome assembly