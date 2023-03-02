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

