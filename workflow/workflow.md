# MNU76 Genome Analysis — Workflow and Methodology

## 1. Project Overview

This project is an independent computational reanalysis of publicly available Illumina and Oxford Nanopore sequencing data from *Streptomyces* sp. MNU76.

The objective was to understand how sequencing reads can be assembled into a draft genome, polished using short-read evidence, assessed for quality, annotated and analysed for potential secondary-metabolite biosynthetic regions.

The strain and its genome have previously been reported. This project focuses on computational reproducibility and practical bioinformatics learning rather than claiming novel biological discoveries.

## 2. Dataset

| Data type | SRA accession | Technology |
|---|---|---|
| Short reads | SRR8841085 | Illumina MiSeq |
| Long reads | SRR8841084 | Oxford Nanopore MinION |

Illumina reads provide relatively high per-base accuracy, whereas Nanopore reads provide longer sequence information that can help resolve genomic regions during assembly.

Combining these technologies allows long-read assembly to be supported by short-read polishing.

## 3. Read Quality Control and Preprocessing

### FastQC

FastQC was used to examine the quality of the raw Illumina reads, including per-base quality, GC content, sequence length, adapter content and base composition.

The quality profiles showed declining quality toward the 3′ ends, particularly for R2.

**Why this step matters:** Low-quality bases can reduce the reliability of downstream read mapping and assembly polishing. Quality assessment helps determine whether preprocessing is necessary.

### Trimmomatic

Trimmomatic was applied to paired-end Illumina reads using a sliding-window quality threshold and a minimum read length.

Parameters used:

- Sliding window: 4 bases
- Quality threshold: Q20
- Minimum read length: 50 bp

The original dataset contained 8,330,624 read pairs. After trimming, 7,066,880 paired reads were retained, corresponding to approximately 84.9% paired-read retention.

**Why this step matters:** Trimming reduces low-quality terminal regions while retaining usable sequence data. The retention rate should be considered alongside the resulting quality profiles to avoid unnecessarily discarding useful reads.

## 4. Nanopore Read Assessment

The Nanopore dataset contained 213,449 reads and approximately 322 Mb of sequence.

| Metric | Value |
|---|---:|
| Mean read length | 1,508.7 bp |
| Median read length | 1,141 bp |
| Maximum read length | 21,254 bp |
| Read N50 | 2,022 bp |

Approximately 92.1% of reads had a mean per-read quality of at least Q10 under the calculation used in this project.

An aggressive Q20 filter was not applied because it would have discarded essentially all reads under that calculation.

**Important distinction:** Read N50 describes the distribution of individual read lengths. Assembly N50 describes contig lengths in the assembled genome. These metrics are not interchangeable.

## 5. Long-Read Assembly with Canu

Canu was used to assemble the Nanopore reads into draft genomic contigs.

**Why Canu?** Canu is designed for long-read assembly and includes processing stages for correcting and trimming reads before constructing contigs.

The draft assembly contained:

- 65 contigs
- Total length: 10,468,810 bp
- N50: 380,316 bp
- N90: 89,936 bp
- GC content: 70.58%

The genome size and high GC content were broadly consistent with the expected characteristics of the MNU76 genome.

## 6. Illumina Read Mapping

BWA-MEM2 was used to map the Illumina reads against the Canu draft assembly. SAMtools was used to process the resulting alignments.

**Why map reads back to the assembly?** Mapping provides evidence about how well the reads support the draft genome. The alignments also provide the input required for Illumina-supported polishing.

Mapping statistics can help assess read alignment and coverage, although high mapping rates alone do not prove that an assembly is correct.

## 7. Assembly Polishing with Pilon

Pilon was used with the Illumina alignments to identify assembly differences supported by short-read evidence.

These differences can include single-nucleotide substitutions and small insertions or deletions.

**Why polish?** Long-read assemblies can contain nucleotide-level errors. Illumina reads provide relatively accurate evidence that can help correct these differences.

The polished assembly contained:

- 65 contigs
- Total length: 10,532,169 bp
- N50: 382,560 bp
- N90: 90,365 bp
- GC content: 70.78%

The N50 increased modestly, but polishing should primarily be understood as an effort to improve sequence accuracy rather than to reduce the number of contigs. Polishing does not guarantee that every error or repetitive region has been resolved.

## 8. Assembly Assessment with QUAST

QUAST was used to compare the Canu draft assembly with the Pilon-polished assembly.

The principal metrics included:

- **Total length:** Sum of the lengths of assembled contigs.
- **Number of contigs:** Number of separate assembled sequences.
- **Largest contig:** Length of the longest contig.
- **N50:** Length threshold at which contigs of that length or longer account for at least half the total assembly length.
- **N90:** Corresponding threshold for 90% of the total assembly length.
- **GC content:** Proportion of guanine and cytosine bases.

The polished assembly had a slightly larger total length and a modestly higher N50. Both assemblies contained 65 contigs.

Assembly statistics should be interpreted together. No single metric establishes assembly correctness or completeness.

## 9. Genome Annotation with Bakta

Bakta was used to predict coding sequences and other genomic features from the polished assembly.

The analysis predicted:

- 9,777 coding-sequence (CDS) features
- 84 tRNAs
- 27 rRNAs
- 39 ncRNAs
- 1 tmRNA
- Coding density: 84.9%

Approximately 3,124 predicted CDS features were classified as hypothetical proteins.

**Why annotate?** Assembly produces genomic sequences, but annotation assigns predicted biological features to those sequences. This makes the genome more interpretable for downstream biological analysis.

A predicted CDS is not automatically an experimentally confirmed gene, and a hypothetical protein does not have a reliably established function based on the available annotation evidence.

## 10. Biosynthetic Gene Cluster Prediction with antiSMASH

antiSMASH 8.0.4 was used to identify genomic regions associated with secondary-metabolite biosynthesis.

The run identified 42 predicted secondary-metabolite regions, including categories associated with PKS, NRPS, terpenes, RiPPs, siderophores and ectoine.

**Why antiSMASH?** Biosynthetic gene clusters contain groups of genes that may collectively contribute to the production of secondary metabolites. Genome mining can help identify candidate pathways for further investigation.

**Interpretation limit:** A predicted region does not prove that a compound is produced, identify its activity with certainty, or establish that it is a new antibiotic. Experimental validation would be required to confirm metabolite production and biological function.

## 11. CheckM Assessment and Limitation

CheckM was used to attempt a marker-gene-based assessment of the polished genome.

Marker identification and alignment completed, but the final pplacer step was terminated because of computational resource limitations.

Consequently, this project does not report a final CheckM completeness or contamination value. Values from the original publication are not presented as results generated by this reanalysis.

## 12. Limitations

- The assembly generated in this reanalysis was less contiguous than the previously reported assembly.
- CheckM did not produce a final completeness or contamination estimate.
- Different annotation tools and databases can produce different numbers of predicted coding sequences.
- Assembly statistics alone do not establish correctness or completeness.
- antiSMASH predictions require experimental validation before conclusions can be drawn about metabolite production or activity.

## 13. Summary

The workflow demonstrates an end-to-end approach to microbial genome analysis:

**Public sequencing data → Quality control → Read preprocessing → Long-read assembly → Short-read mapping → Assembly polishing → Quality assessment → Genome annotation → Biosynthetic gene cluster prediction**

The main learning outcome was understanding the complementary roles of Illumina and Nanopore sequencing and the reasoning behind each computational stage.

This repository documents an independent computational reanalysis of previously reported data and does not claim the original study's biological discoveries as novel.
