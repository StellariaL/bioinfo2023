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


   > shuffled1
   Length=70

    Score = 13.5 bits (23),  Expect = 3.3, Method: Compositional matrix adjust.
    Identities = 12/22 (55%), Positives = 13/22 (59%), Gaps = 1/22 (5%)

   Query  18  PGTASRPSSSRSYVTTSTRTYS  39
              PG A R SSS S+V     TYS
   Sbjct  4   PG-AYRRSSSTSFVYYGMGTYS  24


   > shuffled4
   Length=70

    Score = 12.3 bits (20),  Expect = 9.2, Method: Compositional matrix adjust.
    Identities = 8/21 (38%), Positives = 11/21 (52%), Gaps = 4/21 (19%)

   Query  14  MFGGPGTASRPSSSRSYVTTS  34
              +F GP     P+S+    TTS
   Sbjct  52  LFPGPA----PTSASRLYTTS  68



   Lambda      K        H        a         alpha
      0.307    0.116    0.309    0.792     4.96
   Gapped
   Lambda      K        H        a         alpha    sigma
      0.267   0.0410    0.140     1.90     42.6     43.6

   Effective search space used: 27500
   ```
   
