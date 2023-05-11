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
     
# 230413
## RNA-seq

1. Expression data
   
   Measured quantity: (mostly) mature mRNA
   
   Methods:
   1. RT-qPCR: for individual genes
   2. array: fluorescence from probe-target hybridization
   3. sequencing: absolute, comparable between samples
      
      poly(A)+ RNA-seq (default): mRNA & polyA-lncRNA *oligo-dT capture/priming*
      
         no strand information by default, needs dUTP
      
      small RNA-seq: miRNA, piRNA, etc. *needs size selection*
      
         add adaptors before RT, retain strand information by default
      
      total RNA-seq: *needs rRNA removal*
      
      scRNA-seq: **how to capture micro quantities of RNA?**
      
         SMATR-seq: add TSO to primer -> can add adaptors w/o enzymatic ligation
                    Tn5 enzyme ->

2. data analysis
   
   - preprocessing
   
   * internal control: stably expressed housekeeping genes
   * external control: spike-in - add known amount of artificial RNA, but cannot guarantee consistancy between samples
   * normalization for array:
        
        median normalization: all data minus median -> normalized median = 0
        
        quantile normalization: rank genes, calculate mean for each rank, assign back
        
   * normalization for RNA-seq: sequence depth, gene length, differential expression
        
        CPM/RPM: (for small RNA) reads/total reads \*10^6
        
        RPKM: (for poly-A/total RNA) reads/(total reads \* gene length) \*10^6
        
        FPKM: (for paired end) RPKM/2
        
        TPM: (for paired end) RPKM/total RPKM \*10^6
        
   * normalization for DE analysis:

        edgeR: adjust counts by TMM (trimmed mean of M-values)
        
        DEseq2: RLE (relative log expression) - calculate geometric mean for each genes, divide by geometric mean and normalization factor (a form of median) 
          
   
   - statistics
   
   * descriptive statistics
     
     relatedness between (high-dimensional) samples: Euclidean distance
     
     between (high-dimensional) variables: Pearson's correlation
     
     Clustering algorithms:
       
       k-means: needs *a priori* k
       
        start with random assignment to k groups, calculate centroid for each group, re-assign to nearest centroid, iterate till stable.
        
        assessment: minimize in-group distance, maximize between-group distance
       
       hierarchical: complete tree
       
        agglomerative clustering: bottom-up, start with small groups of 2, gradually merge
        
        divisive: top-down
        
    - gene function analysis
    
      GO (gene ontology): functional annotation of genes - cellular component, molecular function (binding/catalytic relationships between molecules), biological process
      
        GO annotations are tree-like.
        
      KEGG: pathway annotation
      
      Enrichment analysis:
      
        a. hypergeometric test: use hypergeometric distribution to describe number of a GO term in gene list. small p indicates enrichment.
        
        b. Fisher-exact test
        
        c. chi-square test
        
        d. GSEA (gene set enrichment analysis): more sensitive than above testing methods.
      
## RNA regulation

**An example: alternative splicing detecion**

psi-score: quantify how often an exon is included in the final transcript

# 230511
## ChIP-seq
Interactions! The basis of networks.

'Captured/targeted' sequencing: enrich and sequence selected genomic regions

data produced: peaks of reads on the genome

1. peak calling: bin in sliding windows. count reads, compare with random distribution and retain only significant peaks. merge and smoothen into peak.

  in pair-end sequencing, the steric hinderence of the TF/protein would cause + and - end reads to form two distinct peaks. This can facilitate true peak identification.
  
  Control (Input) sample: sequencing w/o IP, in order to correct artifacts: some sequences may be inherently more easily amplified due to sequence/chromatin states/etc.
  
  for features prone to form long peaks/blocks: need bigger bins or adaptive binning.
  
  use heatmaps aligned by peak centers to visualize general peak shape.

\* computational methods for DNA-protein interaction detection:
* evolutional comparison for conserved sequences
* enriched motifs

somehow not so reliable as wet experiments.

2. summarizing motif/consensus sequence

   Position Frequency Matrix: frequency of each base at each position
   
   Position Weight Matrix: PFM corrected by random sequence
   
   calculate p, E and q value.
   
3. constructing networks
