RepeatModeler itself **does not** provide BAM or BED files - it only creates the repeat library (FASTA consensus sequences). The **visualization** comes from RepeatMasker and TRF outputs, which need to be converted to visualization-friendly formats.

## Getting Visualization Files

### **RepeatMasker Output Conversion**

**RepeatMasker produces:**

- `.out` file (detailed annotations)
- `.gff` file (if requested)
- No direct BED/BAM output

**Convert RepeatMasker .out to BED:**

```bash
# Basic conversion
awk 'NR>3 {print $5"\t"$6-1"\t"$7"\t"$10"_"$11"\t"$1"\t"$9}' \
    custom_repeats/genome.fasta.out > all_repeats.bed

# Color-coded by repeat class
awk 'NR>3 {
    if($11 ~ /LINE/) color="255,0,0"        # Red for LINEs
    else if($11 ~ /SINE/) color="0,255,0"   # Green for SINEs  
    else if($11 ~ /LTR/) color="0,0,255"    # Blue for LTRs
    else if($11 ~ /Satellite/) color="255,0,255" # Magenta for satellites
    else color="128,128,128"                 # Gray for others
    print $5"\t"$6-1"\t"$7"\t"$10"_"$11"\t"$1"\t"$9"\t"$6-1"\t"$7"\t"color
}' custom_repeats/genome.fasta.out > repeats_colored.bed
```

**Generate GFF from RepeatMasker:**

```bash
# Use RepeatMasker's built-in GFF conversion
RepeatMasker -lib mygenome-families.fa -gff -pa 8 genome.fasta

# This creates genome.fasta.out.gff
```

### **TRF Output Conversion**

**Convert TRF to BED:**

```bash
# Basic tandem repeats BED
awk '{print $1"\t"$2-1"\t"$3"\t"$14"("$4"x)"\t"$8"\t."}' \
    trf_results/genome.fasta.2.5.5.80.10.40.2000.dat > tandem_repeats.bed

# High-scoring tandem repeats only
awk '$8 > 500 {print $1"\t"$2-1"\t"$3"\t"$14"_score"$8"\t"$8"\t."}' \
    trf_results/genome.fasta.2.5.5.80.10.40.2000.dat > high_score_tandems.bed
```

## Creating IGV-Ready Files

### **Multi-Track BED File**

```bash
# Create separate tracks for different repeat types
mkdir igv_tracks

# Satellite repeats
awk 'NR>3 && $11 ~ /Satellite/ {print $5"\t"$6-1"\t"$7"\t"$10"\t"$1"\t"$9}' \
    custom_repeats/genome.fasta.out > igv_tracks/satellites.bed

# Transposable elements  
awk 'NR>3 && ($11 ~ /LINE|SINE|LTR|DNA/) {print $5"\t"$6-1"\t"$7"\t"$10"\t"$1"\t"$9}' \
    custom_repeats/genome.fasta.out > igv_tracks/transposons.bed

# Unknown/Novel repeats (potential centromeric)
awk 'NR>3 && $11 ~ /Unknown|Unclassified/ {print $5"\t"$6-1"\t"$7"\t"$10"\t"$1"\t"$9}' \
    custom_repeats/genome.fasta.out > igv_tracks/novel_repeats.bed

# High-copy repeats (potential centromeric)
awk 'NR>3 && $1 > 1000 {print $5"\t"$6-1"\t"$7"\t"$10"_copy"$1"\t"$1"\t"$9}' \
    custom_repeats/genome.fasta.out > igv_tracks/high_copy_repeats.bed
```

### **Create BigWig for Repeat Density**

```bash
# Install required tools if needed
# conda install -c bioconda bedtools ucsc-bedgraphtobigwig

# Create genome file
awk '{print $1"\t"length($2)}' genome.fasta.fai > genome.sizes

# Calculate repeat density in windows
bedtools makewindows -g genome.sizes -w 10000 > windows.bed
bedtools coverage -a windows.bed -b all_repeats.bed > repeat_density.bedgraph

# Convert to BigWig for smooth visualization
bedGraphToBigWig repeat_density.bedgraph genome.sizes repeat_density.bw
```

## IGV Visualization Setup

### **Loading Files in IGV**

1. **Load genome:**
    
    ```bash
    # Create .fai index if not exists
    samtools faidx genome.fasta
    ```
    
    - Load `genome.fasta` as reference
2. **Load repeat tracks:**
    
    - `satellites.bed` - Satellite repeats
    - `tandem_repeats.bed` - Tandem repeats from TRF
    - `high_copy_repeats.bed` - High-copy elements
    - `repeat_density.bw` - Overall repeat density
3. **Configure track display:**
    
    - Right-click tracks → Set color
    - Adjust height and squish/expand modes
    - Use "Collapsed" view for dense regions

### **Advanced Visualization**

**Create repeat heat map:**

```bash
# Combine all repeat data with intensity
awk 'NR>3 {print $5"\t"$6-1"\t"$7"\t"$10"\t"$1}' \
    custom_repeats/genome.fasta.out | \
bedtools map -a windows.bed -b stdin -c 5 -o sum > repeat_heatmap.bedgraph

bedGraphToBigWig repeat_heatmap.bedgraph genome.sizes repeat_heatmap.bw
```

**Centromere candidate regions:**

```bash
# Identify regions with both high tandem repeats AND high interspersed repeats
bedtools intersect -a high_score_tandems.bed -b high_copy_repeats.bed -wa -wb | \
bedtools merge -d 5000 > centromere_candidates.bed
```

## Example IGV Session

**Track organization from top to bottom:**

1. **Genes/annotations** (if available)
2. **Centromere candidates** (thick track, red)
3. **Satellite repeats** (blue)
4. **Tandem repeats** (green)
5. **Novel/Unknown repeats** (orange)
6. **All transposons** (gray, collapsed)
7. **Repeat density** (BigWig heatmap)

**IGV tips for repeat visualization:**

- Use **"Squished"** view for repeat-dense regions
- **Color-code** by repeat family/type
- **Group tracks** by repeat category
- Use **BigWig** for genome-wide overview
- **Zoom levels**:
    - Chromosome view: See overall repeat distribution
    - 100kb view: Identify repeat clusters
    - 10kb view: See individual repeat elements
    - 1kb view: Examine repeat structure detail

## Alternative Visualization Tools

**If IGV is too slow with large repeat datasets:**

```bash
# Use JBrowse2 for web-based visualization
# Convert to JBrowse-compatible formats

# Or use UCSC Genome Browser format
# Upload custom tracks to genome.ucsc.edu
```

**Command-line visualization:**

```bash
# Quick text-based visualization of repeat density
bedtools makewindows -g genome.sizes -w 100000 | \
bedtools coverage -a stdin -b all_repeats.bed | \
awk '{print $1":"$2"-"$3"\t"$7}' > repeat_density_summary.txt
```

The key is that RepeatModeler creates the library, RepeatMasker maps it back to create annotations, and then you convert those annotations to visualization formats. The workflow gives you comprehensive repeat visualization capabilities for identifying centromeric regions.

**Circos plots** (for circular genome view):

bash

```bash
# Install circos and create repeat density plots
# Great for showing repeat distribution across whole genome
conda install -c bioconda circos

# Create circos config for repeat visualization
# Shows repeat density as heatmaps around chromosomes
```

**Plotly/Bokeh** (interactive plots):

bash

```bash
# Create interactive plots for exploring repeat distributions
# Good for publication-quality figures
```