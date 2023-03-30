# 230223
## Introduction
### Basics of bioinformatics
* Theoretic basics

  The coming age of bioinformatics: information exceeds interpretation tools. 
 
  How do we make use of the accumulating biological data? 
 
  * information: data produced by biological assays *(e.g. sequences from NGS)*
  * model: representation of structures underlying the data *(e.g. decision trees, neural networks)*
  
    *Models can be causal/interpretable or non-causal/uninterpretable.
  * algorithm: procedures to arrange and process data *(e.g. sorting, dynamic programming)*

* Practical basics

  List of needed skills:
  - [ ] Linux
  - [ ] R
  - [ ] *(optional)* Python
 
  Useful tools:
  * Docker *(virtual machine platform?)*
  * Github/git *(documentation & version control)*
  * Markdown *(language for easy typesetting)*

### Course information
* Theory + practice
* 9 homework projects + 1 10-min presentation
* Time & effort input

# 230302
## Introduction (continued)
### Hidden Markov Model
Predict a hidden sequence of causes (state path) from its result sequence (sequence).

Parameters: 
* Transition probability: *how the previous term in the state path determines the next*
* Emission probability: *how terms in the state path determine the sequence*

**e.g. predict exons, spliceing sites and introns**

Given a state path, the probability of getting the observed sequence can be calculated.

The state path with maximum probability can be found using a **dynamic programming** algorithm.

**3 goals in HMM**
1. Optimal prediction
2. Probability estimation
3. Learning

e.g. Gene prediction problem: given a sequence, predict its features *(exon/intron, modifications, etc.)*

### Nearest Neighbour Model
Predict optimal state from sequence

e.g. predict 3D structures of biomolecules from their sequence.

Given a 3D structure, its free energy can be calculated when all parameters are specified.

The structure with minimal free energy can be found using **dynamic programming**.

## Linux


# 230323
## NGS
### Principles of DNA sequencing
* gel-based systems (Sanger sequencing)
* capillary sequencing (Sanger sequencing)
* Illumina NGS
  
  1. library preparation
     
     fragmentation - end repair & A-tailing - ligate P5 and P7 adaptor - PCR
     
  2. sequencing

     fragmentation - adaptors - bridge PCR - sequencing with fluorescent nt (sequencing by synthesis) **readlength: 150-300 bp**
     
     paired-end reads: better for mapping reads.
     
     adaptor: flow-cell binding site + index + primer binding site
     
 * 3rd generation (single molecule real-time sequencing)
   
   nanopore: fast but low accuracy on repeating nts.
   
### Applications of NGS
* DNA-seq
* RNA-seq
* epigenetics
* interactions

### data analysis
1. quality control: FastQC
2. read mapping: Bowtie
3. mutation identification: detect SNPs, INDELs, CNVs and SVs.
4. visualization

**Normalization is very important!**

### Bowtie
Different from BLAST: allow mismatches, multiple alignments, and must be faster.

BWT algorithm: save space and time.

* BW transformation: data compression method.
  
  rotate - sort from forward - output last -> can achieve no-loss compression
  
  side product: Last-First mapping


# 230330
## NGS (continued)

Paired-end sequencing: can provide information about fragment length.

Mapping: allowed mismatches and multiple locations should be tailored according to specific circumstances.

## Genomic features from DNA-seq
1. assemble genomes
   
   c-value paradox: genome size is not correlated with organism complexity.
   
   GC content: 40-50\% in vertebrates, more variation in invertebrates, plants and bacteria (from 20\% to 70\%+ for bacteria). Somehow related with temperature and DNA stability?
   
   GC distribution: **isochores** - regions with high GC. In most cases, isochores are also gene-rich (in mammalian genome).
   
   genes v.s. non-coding sequences: humans have a particularly large number of transcribed, non-coding sequences.
   * types of ncRNAs:
   
     canonical: r, t, sn, sno, srp
     
     small: mi, pi, si
     
     long (lncRNAs): MALAT1, HOTAIR, etc. (>1kb)
     
   * pseudogenes
   * repeats and transposons
     
     LINE (6-8kb, contain ORFs, autonomous), SINE (100-300bp, non-autonomous), LTR (6-11kb, retrovirus-like, with *gag*, *pol* and sometimes *env*), DNA transposons.
     
   * regulatory elements
   * introns
   * UTRs: transcript=UTR+exon (no intron)

2. variations
   
   genomic structural variation: indel
   
   CNV & segmental duplication
     
