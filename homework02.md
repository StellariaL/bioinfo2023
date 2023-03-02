1. find CDS on Chr XI, sort according to end coords, print last 10
```bash
cat 1.gtf | awk '$1=="XI" && $3=="CDS" {print $1,$3,$4,$5,$10}' | sort -n -k 5 | tail
```
output:
```
XI CDS 83075 83547 "YKL190W";
XI CDS 84704 85900 "YKL189W";
XI CDS 86228 88786 "YKL188C";
XI CDS 89287 91536 "YKL187C";
XI CDS 9094 11226 "YKL220C";
XI CDS 92747 93298 "YKL186C";
XI CDS 94499 96262 "YKL185W";
XI CDS 96757 98154 "YKL184W";
XI CDS 98398 98607 "YKL183C-A";
XI CDS 98721 99638 "YKL183W";
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
cat 1.gtf | awk '$1="XV" && $3=="gene" {print $10,$5-$4+1}' | sort -n -k 2 | tail -5
```
output:
```
"YDR457W"; 9807
"YHR099W"; 11235
"YKR054C"; 12279
"Q0045"; 12884
"YLR106C"; 14733
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
