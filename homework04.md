1. bowtie优化速度及内存的原理

   1.1 速度：
   
   * 利用BWT压缩后F列和L列中相同字母排列顺序相同的特性，同时使用checkpointing，每隔m行记录一次L列中ACGT累计数目。检索到一个match后，可以快速找到该碱基在F列中的位置，然后据此在L列上找到参考序列的下一个碱基，同query比较。
   * 使用suffix array sample，每隔n行记录一次L列在原序列上的位置，可在检索到合法的alignment后快速推知该位点在参考序列上的位置。

   1.2 内存：
   
   * BWT压缩后F列和L列均与参考序列本身长度相同，不需要额外的空间。
   * suffix array sample可在N/n的空间内记录L列各行对应的位置，无需使用同参考序列长度相同的空间存储retrace所需的信息；checkpointing也避免了额外占用空间存储F列。
 
2. mapping with bowtie

   ```bash
   bowtie -v 2 -m 10 --best --strata BowtieIndex/YeastGenome -f THA2.fa -S THA2.sam
   cat THA2.sam | grep -v '^@' | awk '{print $3}' | sort | uniq -c
   ```

Output:

   ```bash
   # reads processed: 1250
   # reads with at least one reported alignment: 1158 (92.64%)
   # reads that failed to align: 77 (6.16%)
   # reads with alignments suppressed due to -m: 15 (1.20%)
   Reported 1158 alignments to 1 output stream(s)

     92 *
     18 chrI
     51 chrII
     15 chrIII
    194 chrIV
     25 chrIX
     12 chrmt
     33 chrV
     17 chrVI
    125 chrVII
     68 chrVIII
     71 chrX
     56 chrXI
    169 chrXII
     67 chrXIII
     58 chrXIV
    101 chrXV
     78 chrXVI
   ```

3. sam format

   3.1 CIGAR string

      CIGAR string记录了query序列比对到参考序列的情况，包括匹配、插入/删除、间隔等比对状态及对应的碱基数。
   
   3.2 soft clip的含义

      soft clip表示query序列两端无法比对到参考序列上的片段，但依然作为query的一部分参与比对，并被显示在SEQ一列中。在CIGAR string中用S\*表示，其中\*是soft clip的碱基个数。

   3.3 mapping quality

      MAPQ是sam格式的第5列，表示该条比对的质量。MAPQ的官方定义是$-10log_{10}P(比对位置错误)$，分值越高，该条比对的可信度越高。实际比对中，不同软件会采用不同方式计算MAPQ，如bowtie2会在计算时综合考虑该条alignment的得分（考虑错配数目、错配碱基测序质量、gap数量及长度）和该query其他alignment的得分。

      可以通过MAPQ过滤掉不可信或有多个相近alignment的比对。

   3.4 反推参考序列

      可以，具体方法是查询每条比对的MD tag，其中的数字表示参考序列同query相同，A/T/G/C表示错配时参考序列在该处的碱基，^后面的碱基表示参考序列中没有的碱基（query相对于参考序列有插入）。

4. 使用bwa

   代码：
   
   ```bash
   # 通过share把解压好的bwa文件夹拷贝到container中
   # 安装bwa
   make
   # 获取基因组序列
   wget --timestamping 'ftp://hgdownload.cse.ucsc.edu/goldenPath/sacCer3/chromosomes/*'
   gunzip chr*.fa.gz
   touch SacCer.fa
   cat chr*.fa >> SacCer.fa
   # 建立索引
   cd bwa
   ./bwa index /home/test/mapping/SacCer.fa
   # 由于THA2.fa中的reads长度在30nt左右，根据bwa内置算法的特点，选择适用于<100bp单端测序的aln + samse命令
   # aln命令对每个read检索best hit，记录suffix array coordinates (SA)
   ./bwa aln /home/test/mapping/SacCer.fa /home/test/mapping/THA2.fa > /home/test/mapping/THA2_bwa_aln.sai
   # samse命令根据.sai文件、参考序列和测序结果生成sam文件
   ./bwa samse /home/test/mapping/SacCer.fa /home/test/mapping/THA2_bwa_aln.sai /home/test/mapping
   /THA2.fa > /home/test/mapping/THA2_bwa_aln.sam
   # 检查比对结果
   cd ..
   cat THA2_bwa_aln.sam | grep -v '^@' | awk '{print $3}' | sort | uniq -c
   ```

   结果：
   
   ```bash
     24 *
     17 chrI
     54 chrII
     17 chrIII
    202 chrIV
     26 chrIX
     18 chrM
     38 chrV
     18 chrVI
    129 chrVII
     70 chrVIII
     77 chrX
     60 chrXI
    178 chrXII
     72 chrXIII
     59 chrXIV
    108 chrXV
     83 chrXVI
   ```
   
   同bowtie的结果比较：
   
   ```bash
     92 *
     18 chrI
     51 chrII
     15 chrIII
    194 chrIV
     25 chrIX
     12 chrmt
     33 chrV
     17 chrVI
    125 chrVII
     68 chrVIII
     71 chrX
     56 chrXI
    169 chrXII
     67 chrXIII
     58 chrXIV
    101 chrXV
     78 chrXVI
   ```
5. Genome browser
   
   用IGV查看THA2.sam经排序后生成的bam文件。
   
   ![alt text][genome browser]
   
   截图为基因HSP150，该基因位于+链，只有1个外显子。可以看到THA2中有若干reads map到了HSP150两侧。
   
   bam轨道中的灰色矩形指示map到此处的reads，其上的线条指示了reads和参考序列有差异的碱基；Coverage轨道中的灰色矩形高度表示该处的read count，其上的线条则指示该处同参考序列有差异的reads占比。
   
   例如，第一条线对应的参考序列为T（红色），而此处的4条reads中有1条测出了C（蓝色），因此bam轨道中有一个矩形上有1条蓝色短线，Coverage轨道中则有一条3/4红色，1/4蓝色的线条。
   
   [genome browser]: https://github.com/StellariaL/bioinfo2023/blob/main/genome%20browser.png
