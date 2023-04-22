1. 归一化基因表达值的几种方法
   
   * CPM/RPM：$CPM=\frac{reads}{total reads/10^6}$

     相当于计算每个基因在全部reads中的占比，由于没有考虑基因长度对reads count的影响，一般用于计算小RNA表达值。

   * RPKM： $RPKM=\frac{reads}{gene length(kb)*total reads/10^6}$

     在RPM的基础上，认为在表达量相同的情况下，测到的reads占比和基因长度成正比，因此额外除以基因长度来进行归一化。

   * FPKM：$FPKM=RPKM/2$

     FPKM针对双端测序结果，每个reads pair被map了2次，因此多除了2。

   * TPM： $TPM=\frac{RPKM}{\sum RPKM}\times 10^6$

     TPM认为每个样本的总表达量都是1，因此额外除以总RPKM。

   * TMM：

     TMM认为多组样本中绝大多数基因的表达水平都是接近的，只有少数基因存在差异性表达，但是不考虑基因长度对reads count的影响。
     
     TMM首先在所有样本中找到一组较为居中的数据作为参考：计算所有样本CPM的Q3，选择Q3最接近于Q3平均值的样本作为参考组。
     
     接下来计算每个样本的修正因子：对于每个基因，M-value=log2(sample CPM/reference CPM)，表征了它在样本和参考组中CPM的差异，差异越大，M-value取值越极端；A=1/2(log2sample CPM + log2reference CPM)，相当于对样本和参考CPM几何均值取log2，表征了它在样本和参考组中的表达水平，表达越高，A越大；选取一组M-value和A都较为居中的基因（即：表达量居中，且样本和参考组中差异不大），作为样本的代表性基因，将它们的M-value取加权平均即为该组样本的修正因子（表达量低的基因容易表现出较大的M-value，而表达量高的基因反之，加权是为了平衡这一点）。
     
     最后，将raw read counts除以2^normalization factor，即可获得归一化的表达量。
     
   * RLE：

     RLE的原理和TMM类似，也是认为绝大多数基因表达水平接近。
     
     计算RLE时，首先将每个基因的read counts除以该基因在多个样本中read counts的几何均值，然后在每个样本中取中位数作为修正因子，最后用read counts除以修正因子。换言之，RLE的修正因子反映了样本整体上偏离平均表达量的情况。
     
2. raw reads counts in different sequencing strategies:

   standard Illumina: E
   Ligation method: D
   dUTPs method: A

3. shape02

   判断sequencing protocol:

   ```bash
   /usr/local/bin/infer_experiment.py -r GTF/Arabidopsis_thaliana.TAIR10.34.bed -i bam/Shape02.bam
   ```

   output:

   ```bash
   Reading reference gene model GTF/Arabidopsis_thaliana.TAIR10.34.bed ... Done
   Loading SAM/BAM file ...  Total 200000 usable reads were sampled


   This is PairEnd Data
   Fraction of reads failed to determine: 0.0315
   Fraction of reads explained by "1++,1--,2+-,2-+": 0.4769
   Fraction of reads explained by "1+-,1-+,2++,2--": 0.4916
   ```

   两种对应方式测得的基因数相近，说明采用的是非链特异的建库方法。

   获得reads count matrix:

   ```bash
   /home/software/subread-2.0.3-source/bin/featureCounts \
   > -s 0 -p -t exon -g gene_id \
   > -a GTF/Arabidopsis_thaliana.TAIR10.34.gtf \
   > -o result/Shape02.featurecounts.exon.txt bam/Shape02.bam
   ```

   输出的Shape02.featurecounts.exon.txt.summary统计了mapping后各类reads的数目。

   ```bash
   Status  bam/Shape02.bam
   Assigned        2559170
   Unassigned_Unmapped     0
   Unassigned_Read_Type    0
   Unassigned_Singleton    0
   Unassigned_MappingQuality       0
   Unassigned_Chimera      0
   Unassigned_FragmentLength       0
   Unassigned_Duplicate    0
   Unassigned_MultiMapping 0
   Unassigned_Secondary    0
   Unassigned_NonSplit     0
   Unassigned_NoFeatures   59487
   Unassigned_Overlapping_Length   0
   Unassigned_Ambiguity    111786
   ```

   找到AT1G09530基因的raw reads count:

   ```bash
   cat Shape02.featurecounts.exon.txt | grep AT1G09530 | awk '{print $1,$7}'
   ```

   output:

   ```bash
   AT1G09530 86
   ```

   这里的86是没有经过任何归一化处理的raw count。

4. z score of logCPM

   解压后将R的工作目录设置为解压后的文件夹。

   ```R
   library(pheatmap)
   # 这个函数用于提取样本所在的文件夹名称，即该样本的癌症种类
   gettype=function(s){
     t=strsplit(s,split='/')[[1]][1]
     return(t)
   }
   # 只读取基因名称和reads count
   classes=c("character",rep("NULL",5),"integer")
   # 批量读取所有文件，并整合到一个数据框rawm中
   path=c("COAD","ESCA","READ")
   samplelist=list.files(path,pattern="*.txt$",full.names=TRUE)
   n=length(samplelist)
   rawm=read.table(samplelist[1],colClasses=classes,skip=2)
   colnames(rawm)=c("GeneId",1)
   for (i in c(2:n)){
     temp=read.table(samplelist[i],colClasses=classes,skip=2)
     colnames(temp)=c("GeneId",i)
     rawm=merge(rawm,temp,by="GeneId",suffixes=NULL)
   }
   rownames(rawm)=rawm$GeneId
   rawm$GeneId=NULL
   # 计算CPM
   CPM.matrix=t(1000000*t(rawm)/colSums(rawm))
   # 计算logCPM
   log10.CPM.matrix=log10(CPM.matrix+1)
   # 计算z score
   z.scores=(log10.CPM.matrix - rowMeans(log10.CPM.matrix))/apply(log10.CPM.matrix,1,sd)

   # 将极端值clip到可接受的范围内
   z.scores[z.scores>2]=2
   z.scores[z.scores <= -2] = -2
   # 去除全都是NaN的行
   z.scores=z.scores[apply(z.scores, 1, function(y) any(!is.na(y))),]
   # 其他的NaN取0
   z.scores[is.na(z.scores)]=0
   # 注释样本的癌症类型
   annotation_col = data.frame(TumorType=factor(sapply(samplelist,gettype)))
   rownames(annotation_col) = colnames(z.scores)
   ann_colors = list(TumorType = c(COAD = "#7570B3", ESCA = "#E7298A", READ = "#66A61E"))
   # 绘制热图，选择只按列聚为两类，聚类方法为默认基于欧氏距离的hierarchical clustering
   pheatmap(z.scores,
            color = colorRampPalette(c("navy", "white", "firebrick3"))(50),
            cutree_col = 2,
            cluster_rows=FALSE, show_rownames=FALSE, show_colnames=FALSE, cluster_cols=TRUE,
            annotation_col = annotation_col,annotation_colors = ann_colors)
   ```

   output:
   
   ![alt text][heatmap]
   
   观察到聚类结果中食道癌样本中有一部分被单独分了一类，结肠癌、直肠癌基本都被分到了另一类，说明后两种癌症的转录组相对较为接近。
   
   
   [heatmap]: https://github.com/StellariaL/bioinfo2023/blob/main/heatmap.png
