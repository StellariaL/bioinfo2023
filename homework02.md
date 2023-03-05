1. find CDS on Chr XI, sort according to end coords, print last 10
```bash
cat 1.gtf | awk '$1=="XI" && $3=="CDS" {print $1,$3,$4,$5,$10}' | sort -n -k 4 | tail
```
output: (the entire line is too long, so I only selected a few columns)
```
XI CDS 631152 632798 "YKR097W";
XI CDS 633029 635179 "YKR098C";
XI CDS 635851 638283 "YKR099W";
XI CDS 638904 639968 "YKR100C";
XI CDS 640540 642501 "YKR101W";
XI CDS 646356 649862 "YKR102W";
XI CDS 653080 656733 "YKR103W";
XI CDS 656836 657753 "YKR104W";
XI CDS 658719 660464 "YKR105C";
XI CDS 661442 663286 "YKR106W";
```

2. list different features on Chr IV, sort according to counts
```bash
cat 1.gtf | awk '$1=="IV" {print $3}' | sort | uniq -c | sort -n -k 1
```
output:
```
    853 start_codon
    853 stop_codon
    886 gene
    886 transcript
    895 CDS
    933 exon
```

3. find 2 longest CDS on - chain and not on Chr IV, print lengths
```bash
cat 1.gtf | awk '$1!="IV" && $3=="CDS" && $7=="-" {print $5-$4+1}' | sort -n | tail -2
```
output:
```
12276
14730
```

4. find 5 longest genes on Chr XV, print gene id and lengths
```bash
cat 1.gtf | awk '$1="XV" && $3=="gene" {name=$10;gsub("\"","",name);gsub("\;","",name);print name,$5-$4+1}' | sort -n -k 2 | tail -5
```
output:
```
YDR457W 9807
YHR099W 11235
YKR054C 12279
Q0045 12884
YLR106C 14733
```

5. count column number of 1.gtf
   awk分列时的默认分隔符是tab和空格，而1.gtf中attribute一列的格式为 attribute name+空格+content ，因此直接用awk划分时会将attribute划分为2n列。
   
   不同基因的attribute数量不同，因此直接使用print NF输出列数时会发现各行的列数有很大差异。
```bash
# 为方便查看，在输出列数后统计了每类列数一共有多少行
grep -v '^#' 1.gtf | awk '{print NF}' | sort -n | uniq -c
```
output:
```
   2116 16
   5010 18
      1 24
   2115 26
   8472 28
   9932 30
   4067 32
  10534 34
```
   而如果用-F规定只用tab分隔，就能得到gtf格式标准的9列了。
```bash
grep -v '^#' 1.gtf | awk -F "\t" '{print NF}' | sort -n | uniq -c
```
output:
```
 42247 9
```
