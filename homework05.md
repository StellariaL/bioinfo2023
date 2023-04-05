### genome annotations

1. 人类基因组大小及组成

   目前的人类参考基因组GRCh38.p13有3,096,649,726个碱基对，是人类的单倍体基因组，包含X、Y染色体及线粒体基因。

   GRCh38中还有约151 Mbp的未知序列，2022年发布的T2T-CHM13填补了这些缺陷，是一个更加完整的assembly。T2T-CHM13包含3,054,815,472 bp的核DNA和16,569 bp的线粒体DNA，不包括Y染色体。

   人类基因组的内容主要可以分为4类：
   * 编码基因：编码蛋白质的基因
   * 非编码基因：只编码RNA，不翻译为蛋白的基因
   * 转录调控序列
   * 其他结构性序列

2. 非编码RNA

   根据Gencode 2023年4月发布的第43版人类基因组注释，人类基因组的转录本总数为252913，其中有89411条编码了蛋白，其余都是非编码的。

   非编码RNA的主要类型：
   * tRNA: 作为氨基酸载体参与翻译
   * rRNA：作为核糖体的组分参与翻译
   * snRNA：mRNA剪接
   * snoRNA：对上述RNA进行化学修饰
   * miRNA：识别特定mRNA，介导其降解，抑制翻译
   * siRNA：类似，但主要抑制病毒及转座子
   * piRNA：类似，但主要在生殖细胞中抑制转座子
   * lncRNA：长度>200 nt的ncRNA，功能多样，同转录调控、表观修饰、染色质三维结构、翻译调控、miRNA活性调控有关

   参考文献：Hombach S, Kretz M. Non-coding RNAs: Classification, Biology and Functioning. Adv Exp Med Biol. 2016;937:3-17. doi:10.1007/978-3-319-42059-2_1


### bedtools and samtools

1. 文件是单端测序的。

   ```bash
   samtools flagstat COAD.ACTB.bam
   ```
   
   输出为

   ```bash
   185650 + 0 in total (QC-passed reads + QC-failed reads)
   4923 + 0 secondary
   0 + 0 supplementary
   0 + 0 duplicates
   185650 + 0 mapped (100.00% : N/A)
   0 + 0 paired in sequencing
   0 + 0 read1
   0 + 0 read2
   0 + 0 properly paired (N/A : N/A)
   0 + 0 with itself and mate mapped
   0 + 0 singletons (N/A : N/A)
   0 + 0 with mate mapped to a different chr
   0 + 0 with mate mapped to a different chr (mapQ>=5)
   ```
   
   可以看到所有的reads都没有配对。

2. secondary alignment指同一条read比对到了基因组上的多个位置，其中只有一个会被算法标记为primary alignment，而其余都会被记为secondary alignment。primary alignment的选择因算法而异，可能是打分最高或最长的alignment。
   
   secondary alignment对应的BAM flag是256（0x100），可通过-f筛选flag中包含256的reads。
   
   ```bash
   samtools view -f 256 COAD.ACTB.bam | wc -l
   ```
   
   输出为
   
   ```bash
   4923
   ```

3. ACTB的intron
   * 从注释中提取intron

   ```bash
   #!/bin/bash
   
   #读取文件
   echo -n "File path:"
   read file
   
   #防止格式出错
   while true
   do
    if ( [ ${file##*.} == "gtf" ] || [ ${file##*.} == "gff" ] )
    then
     break
    else
    echo -n "not gtf/gff, input again:"
    read file
    fi
   done
   
   # 将gene和exon分别以bed格式输出
   cat $file | awk -v OFS="\t" '$3=="gene" {print $1,$4-1,$5,".",".",$7}' > gene.bed
   cat $file | awk -v OFS="\t" '$3=="exon" {print $1,$4-1,$5,".",".",$7}' > exon.bed

   # gene中不是exon的部分就是intron
   bedtools subtract -a gene.bed -b exon.bed > intron.bed

   exit 0
   ```

   * 提取转录组中map到intron上的reads

   ```bash
   # 对bam进行sort和index
   samtools sort COAD.ACTB.bam > COAD.ACTB_sorted.bam
   samtools index COAD.ACTB_sorted.bam
   # bam和bed取交集，按其在bam中对应的行输出
   bedtools intersect -a COAD.ACTB_sorted.bam -b intron.bed -wa > intersect.bam
   # 转换为fastq格式
   samtools bam2fq intersect.bam > intersect.fastq
   ```

4. 计算reads的genome coverage
   
   ```bash
   # 考虑到是真核生物，加入了-split指令，避免在剪接的转录本中多算intron区段
   bedtools genomecov -ibam COAD.ACTB_sorted.bam -bg -split > coverage.bedgraph
   ```
