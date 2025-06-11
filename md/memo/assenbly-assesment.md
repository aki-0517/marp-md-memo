### 3479769.pts-1.tardis
```bash
canu \
  -p dicty -d assembly_canu \
  genomeSize=34m \
  -nanopore-raw Dicty_filtered.fastq \
  useGrid=false \
  maxThreads=16 \
  maxMemory=64
```
### 3131286.pts-1.tardis
```bash
flye \
  --nano-raw Dicty_filtered.fastq \
  --genome-size 34m \
  --out-dir assembly_flye \
  --threads 16
```

### 2686688.pts-1.tardis
```bash
shasta \
  --input Dicty_filtered.fastq \
  --assemblyDirectory assembly_shasta \
  --threads 16 \
  --config Nanopore-UL-Phased-Nov2022
```

