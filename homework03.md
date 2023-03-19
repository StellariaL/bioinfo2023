1. blast on website
   
   define query and search set:
   
   ![alt text][blastp 1]
   
   only display top 10 alignments; set e-value threshold at 0.5:
   
   ![alt text][blastp 2]
   
   results:
   
   ![alt text][blastp 3]
   
   * p-value: the probability that a random search in the dataset returns a sequence with alignment score no less than the present sequence
   * e-value: the expectation of how many such sequences would occur in a random search.
   
2. blast shuffled sequences
   
   script:
   
   ```bash
   #!/bin/bash
   
   # input sequence
   file=hw_sequences
   s="MSTRSVSSSSYRRMFGGPGTASRPSSSRSYVTTSTRTYSLGSALRPSTSRSLYASSPGGVYATRSSAVRL"
   echo '>original' > ${file}
   echo $s >> ${file}
   
   si=0
   L=${#s}
   ei=$(($L-1))
   
   #shuffle and store
   for i in {1..10}
   do
        shuffled=""
        for c in `seq $si $ei | shuf`;
        do
           shuffled=$shuffled${s:$c:1}
        done
        echo '>shuffled'$i >> ${file}
        echo $shuffled >> ${file}
   done
   
   #blast
   blastp -query ${file} -subject ${file} -out hw_result
   ```
   
   results: (for full file, see hw_result)
   
   ```bash
   Query= original

   Length=70
                                                                         Score     E
   Sequences producing significant alignments:                          (Bits)  Value

     original                                                            123     2e-43
     shuffled1                                                           13.5    3.3
     shuffled4                                                           12.3    9.2

   ```
   
   Here the search only returns 3 hits because the e-value threshold is set at 10.0 (default). Only shuffled1 and shuffled4 meet this criteria.
   
   ```bash
   > original
   Length=70

    Score = 123 bits (308),  Expect = 2e-43, Method: Compositional matrix adjust.
    Identities = 70/70 (100%), Positives = 70/70 (100%), Gaps = 0/70 (0%)

   Query  1   MSTRSVSSSSYRRMFGGPGTASRPSSSRSYVTTSTRTYSLGSALRPSTSRSLYASSPGGV  60
              MSTRSVSSSSYRRMFGGPGTASRPSSSRSYVTTSTRTYSLGSALRPSTSRSLYASSPGGV
   Sbjct  1   MSTRSVSSSSYRRMFGGPGTASRPSSSRSYVTTSTRTYSLGSALRPSTSRSLYASSPGGV  60

   Query  61  YATRSSAVRL  70
              YATRSSAVRL
   Sbjct  61  YATRSSAVRL  70
   ```
   
   This is the original sequence with 100% match.
   
   ```bash
   > shuffled1
   Length=70

    Score = 13.5 bits (23),  Expect = 3.3, Method: Compositional matrix adjust.
    Identities = 12/22 (55%), Positives = 13/22 (59%), Gaps = 1/22 (5%)

   Query  18  PGTASRPSSSRSYVTTSTRTYS  39
              PG A R SSS S+V     TYS
   Sbjct  4   PG-AYRRSSSTSFVYYGMGTYS  24
   ```
   
   This is the alignment with shuffled1. 'Score' is the bit score calculated from the raw score of the alignment. 'Expect' is the e-value for this sequence. In this 22 aa alignment, there are 12 perfect matches, 1 positive (Y and F, both are aromatic amino acids), and 1 gap. 
   
   Only an aligned segment is returned, because blast searches for local alignment. It starts with clusters of matching 'words' (3 consecutive aa), extends in both directions, and returns the extension with highest alignment score. Here, P18-S39 on the query sequence is matched to P4-S24 on shuffled1.
   
   ```bash
   > shuffled4
   Length=70

    Score = 12.3 bits (20),  Expect = 9.2, Method: Compositional matrix adjust.
    Identities = 8/21 (38%), Positives = 11/21 (52%), Gaps = 4/21 (19%)

   Query  14  MFGGPGTASRPSSSRSYVTTS  34
              +F GP     P+S+    TTS
   Sbjct  52  LFPGPA----PTSASRLYTTS  68
   ```
   
   Same as above.
   
   ```bash
   Lambda      K        H        a         alpha
      0.307    0.116    0.309    0.792     4.96
   Gapped
   Lambda      K        H        a         alpha    sigma
      0.267   0.0410    0.140     1.90     42.6     43.6

   Effective search space used: 27500
   ```
   
3. Speeding up in blast.

   Apart from dynamic programming, blast also uses a **'seeding-and-extending'** approach to increase search speed.
   
   Blast breaks subject sequences into short 'words' consisting of several consecutive bases/aas, and records the location of each word in subject sequences in a hash table. For every query, blast breaks it down into words and search for each word in the hash table, creating a series of word hits for every subject sequence. It then identifies hit clusters, and extends these clusters in both directions using dynamic programming to find a final sequence aligned to the query sequence.
   
   This increases search speed because the seeding process allows the algorithm to jump to areas where it is more likely to find a good alignment. Organizing subject sequences into a hash table of words also circumvents repeatedly reading the subject sequences.
   
4. Symmetry of PAM matrices

   The asymmetric matrix is the mutation probability matrix. Entry $M_{ij}=\lambda\frac{A_{ij}}{N_j}$, where $A_{ij}$ is the count of amino acid j mutating into amino acid i (one sequence contains j, while the other sequence contains i at the corresponding place) , and $N_j$ is the count of amino acid j. The mutations are symmetric, so $A_{ij}=A_{ji}$, but since $N_j$ presumably differs from $N_i$, the matrix M is not symmetric.
   
   The symmetric matrix is the $PAM_n$ matrix. $PAM_n(i,j)=log\frac{M^n_{ij}}{f(i)}$, where $f(i)$ is the occurance frequency of amino acid i. This normalizes $M_{ij}$ again according to the abundance of i, so the relationship between i and j is symmetric again, producing a symmetric matrix.
   
   In sequence alignment, the symmertic PAM matrices are used as the scoring matrix, mainly because the entries in the matrices are log odds, and can be directly added along the sequence. The asymmetric mutation probability matrix is used to generate PAM matrices for different evolutionary distances. Using data from sequences of 99% similarity, a mutation probability matrix is calculated. For similarity of (100-n)%, the mutation probability would be multiplied for n times.
   
   In other words, entries in the PAM matrices are additive, while the mutation probability matrix is multipliable. 
   
   [blastp 1]: https://github.com/StellariaL/bioinfo2023/blob/main/blastp%201.png
   [blastp 2]: https://github.com/StellariaL/bioinfo2023/blob/main/blastp%202.png
   [blastp 3]: https://github.com/StellariaL/bioinfo2023/blob/main/blastp%203.png
