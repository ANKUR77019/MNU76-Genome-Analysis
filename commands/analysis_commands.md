# MNU76 Genome Analysis — Command Documentation

This document records the main commands used in the independent reanalysis of publicly available *Streptomyces* sp. MNU76 sequencing data.

## 1. Download sequencing data

SRA accessions:

- Illumina: `SRR8841085`
- Oxford Nanopore: `SRR8841084`

```bash
prefetch SRR8841085
prefetch SRR8841084

fasterq-dump SRR8841085 --split-files -O ~/igib_project/data -e 2
fasterq-dump SRR8841084 -O ~/igib_project/data -e 2
```

## 2. Illumina quality control

```bash
fastqc ~/igib_project/data/SRR8841085_1.fastq \
       ~/igib_project/data/SRR8841085_2.fastq \
       -o ~/igib_project/results
```

FastQC was used to assess per-base quality, GC content, sequence length, adapter content and other read-quality characteristics.

## 3. Illumina preprocessing

```bash
trimmomatic PE -threads 2 \
  ~/igib_project/data/SRR8841085_1.fastq \
  ~/igib_project/data/SRR8841085_2.fastq \
  ~/igib_project/data/SRR8841085_1_paired.fastq \
  ~/igib_project/data/SRR8841085_1_unpaired.fastq \
  ~/igib_project/data/SRR8841085_2_paired.fastq \
  ~/igib_project/data/SRR8841085_2_unpaired.fastq \
  SLIDINGWINDOW:4:20 MINLEN:50
```

The sliding-window filter trims low-quality read ends, and the minimum-length filter removes reads shorter than 50 bases.

## 4. Nanopore read assessment

A Python script was used to calculate read count, total bases, read-length statistics and per-read quality summaries from the FASTQ file.

Input file: `SRR8841084.fastq`

The reported read N50 is a read-length statistic; it is distinct from assembly N50.

## 5. Long-read assembly

```bash
canu -p mnu76 \
  -d ~/igib_project/results/canu \
  genomeSize=10.5m \
  -nanopore ~/igib_project/data/SRR8841084.fastq \
  maxThreads=4 maxMemory=9g useGrid=false
```

Canu was used to assemble the Oxford Nanopore reads into draft genomic contigs.

## 6. Illumina mapping and polishing

Illumina reads were mapped to the Canu draft assembly using BWA-MEM2. SAMtools was used to process and sort alignments before Pilon polishing.

```bash
pilon -Xmx7g -Xms1g \
  --genome ~/igib_project/results/canu/mnu76.contigs.fasta \
  --frags ~/igib_project/results/mnu76_illumina.sorted.bam \
  --output mnu76_pilon \
  --outdir ~/igib_project/results/pilon \
  --threads 2
```

Pilon used the Illumina alignments to identify assembly differences supported by the short reads.

## 7. Assembly quality assessment

```bash
quast \
  ~/igib_project/results/canu/mnu76.contigs.fasta \
  ~/igib_project/results/pilon/mnu76_pilon.fasta \
  -o ~/igib_project/results/quast_pilon
```

QUAST was used to compare contig count, assembly length, N50, N90, largest contig and GC content.

## 8. Genome annotation

```bash
bakta \
  --db /home/ankur/db-light \
  --output ~/igib_project/results/bakta \
  --prefix MNU76 \
  --threads 2 \
  --force \
  ~/igib_project/results/pilon/mnu76_pilon.fasta
```

Bakta predicted coding sequences and other genomic features.

## 9. Biosynthetic gene cluster prediction

```bash
antismash \
  ~/igib_project/results/pilon/mnu76_pilon.fasta \
  --output-dir ~/igib_project/results/antismash_run1 \
  --genefinding-tool prodigal \
  --cpus 2
```

antiSMASH was used to identify genomic regions associated with secondary-metabolite biosynthetic potential.

## Important reproducibility notes

- Paths correspond to the working environment used for this analysis and may need adjustment on another computer.
- CheckM did not produce a final completeness/contamination estimate because its pplacer step was terminated.
- antiSMASH regions are computational predictions and do not establish metabolite production or biological activity.
- Raw FASTQ files are not included in the repository; retrieve them from the NCBI SRA using the accessions listed above.
