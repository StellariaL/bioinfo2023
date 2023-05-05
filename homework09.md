### Differential expression

1. multiple test correction
  
   进行假设检验时一般使用的判断标准是p值，含义是null hypothesis为真时，得到更极端结果的概率。但是当使用同一组数据同时检验多个假设时，在null hypothesis下也有可能随机地出现某个p<0.05的结果，因此需要使用更严格的判断标准来找出真正具有显著意义的结果。

   FDR(False Discovery Rate)或q值的含义是，在给定的检验标准下，被接受的假设中假阳性的比例。q值是多重假设检验中与p值相对应的量。

2. DESeq2和edgeR的归一化原理
   
   * DESeq2: 采用RLE进行归一

     对每个基因，首先计算所有样本raw counts的几何均数，然后用raw counts除以几何均数，每个样本中该数值的中位数即为样本的标准化因子。对每个样本，raw counts除以标准化因子即为归一后的表达水平。

     $$
     log(size \quad factor)=median(\frac{log(raw \quad counts \quad of \quad gene \quad i)}{\frac{1}{n}\sum log(raw \quad counts)})
     $$
     
     $$
     normalized \quad expression=\frac{raw \quad counts}{size \quad factor}
     $$

     该方法认为多数基因的表达水平在样本之间应当是相同的，只是由于样本间采样、测量等随机因素产生了差异。用基因的raw counts除以几何均数，数值大小反映了该基因在该样本中偏离整体表达水平的幅度；因此，该数值的中位数就反映了整个样本偏离整体表达水平的情况。而由于真正差异性表达的基因只占很少的一部分，这种整体的偏离应当是其他干扰因素造成的，因此作为标准化因子除去。完成归一化后，样本中的多数基因表达水平都会被还原到靠近几何均数的数值，而那些依然偏离均数的基因则应当是真正的差异表达基因。

   * edgeR：采用TMM进行归一

     TMM首先选择一个样本作为参考，然后在每个样本中选取一组基因作为代表性基因集，根据代表性基因集在样本和参考之间的差异计算每个样本的标准化因子。

     参考样本的选择：对每个样本计算CPM并排序，选取75%分位数，计算平均值，75%分位数最接近平均值的样本即为参考样本。

     代表性基因集的选择：选择的目的是找出一组在该样本和参考样本中表达量适中且差别不大的基因。首先，计算每个基因在样本和参考样本中的log fold change:

     $$
     log \quad fold \quad change=log_2 \frac{CPM \quad in \quad ref}{CPM \quad in \quad sample}
     $$
     
     log fold change越极端，基因表达量差别越大。

     接下来计算每个基因在样本和参考样本中的平均表达量，用的是几何均值的对数：

     $$
     average \quad level= \frac{log_2 (CPM \quad in \quad ref)+log_2 (CPM \quad in \quad sample)}{2}
     $$

     对两个值分别排序，去除log fold change前30\%和后30\%的基因、average level前5\%和后5\%的基因，剩余的基因即为代表性基因集。

     代表性基因集的log fold change按raw counts加权求平均：

     $$
     weighted \quad average= \frac{ \sum read \quad counts \cdot log \quad fold \quad change}{total \quad reads}
     $$

     将weighted average进一步中心化可得每个样本的标准化因子：

     $$
     log_2 scaling \quad factor=weighted \quad average- \frac{ \sum weighted \quad average}{n}
     $$
     
     raw counts除以标准化因子即可。


3. 光照前后的差异表达基因：

   DESeq2脚本：
   
   ```R
   library(DESeq2)

   raw.counts=read.table("count_exon.txt", sep='\t', header = T,row.names = 1)
   ud.raw.counts=raw.counts[,c("UD1_1","UD1_2","UD1_3","UD0_1","UD0_2","UD0_3")]
   ud.filtered.counts=ud.raw.counts[rowMeans(ud.raw.counts)>5,]
   cond=factor(c(rep("Ctrl",3),rep("Exp",3)),levels = c("Ctrl","Exp"))
   
   colData=data.frame(row.names = colnames(ud.filtered.counts),conditions=cond)
   dds=DESeqDataSetFromMatrix(ud.filtered.counts,colData,design=~conditions)
   dds=DESeq(dds)
   res=results(dds)
   
   # 这里直接进行了q<0.05和abs(log2FC)>1的过滤
   diff.table=subset(res,padj<0.05 & abs(log2FoldChange)>1)
   write.table(diff.table,"uvr8.light.vs.dark.txt",sep='\t',row.names=T,quote=F)
   ```
   
   edgeR脚本：
   
   ```R
   library(edgeR)

   raw.counts=read.table("count_exon.txt", sep='\t', header = T,row.names = 1)
   ud.raw.counts=raw.counts[,c("UD1_1","UD1_2","UD1_3","UD0_1","UD0_2","UD0_3")]
   ud.filtered.counts=ud.raw.counts[rowMeans(ud.raw.counts)>5,]
   cond=factor(c(rep("Ctrl",3),rep("Exp",3)),levels = c("Ctrl","Exp"))

   design=model.matrix(~cond)
   y=DGEList(counts=ud.filtered.counts)
   y=calcNormFactors(y,method="TMM")
   y=estimateDisp(y,design=design)
   fit=glmFit(y,design=design)
   lrt=glmLRT(fit,coef=2)
   diff.table=topTags(lrt,n=nrow(y))$table
   
   # 这里也直接进行了过滤
   diff.table.filtered=diff.table[abs(diff.table$logFC)>1 & diff.table$FDR<0.05,]
   write.table(diff.table.filtered, file = 'edger.uvr8.light.vs.dark.txt', sep = "\t", quote = F, row.names = T, col.names = T)
   ```
   
4. venn图展示两个package结果的区别

   脚本：
   
   ```R
   library(ggplot2)
   library(ggvenn)
   jpeg("DE_venn.jpg",height=1024,width=1536)
   
   deseq.table=read.table("uvr8.light.vs.dark.txt",sep='\t')
   edger.table=read.table("edger.uvr8.light.vs.dark.txt",sep='\t')
   compare=list('deseq'=row.names(deseq.table),'edger'=row.names(edger.table))
   p=ggvenn(compare,fill_alpha = .7,fill_color=c("#74BF6C","#FAE035"),set_name_size = 30,
            text_size=25)
   p
   dev.off()
   ```
   
   结果：
   
    ![alt text][venn]

5. edgeR变化最大的20个基因logCPM的Z-score

   脚本：
   
   ```R
   library(pheatmap)
   jpeg("top20heatmap.jpg",height=384,width=256)
   
   # 直接接着venn图的脚本运行了，没有重新读入数据
   edger.table$zscore=(edger.table$logCPM-mean(edger.table$logCPM))/sd(edger.table$logCPM)
   edger.table=edger.table[order(edger.table$logFC),]
   topgenes=edger.table[c(1:10,49:58),]
   z.score=topgenes[c('zscore')]
   p=pheatmap(z.score,cellwidth = 40,cluster_cols=F,color = colorRampPalette(c("navy", "white", "firebrick3"))(50))
   p
   dev.off()
   ```
   
   结果：
   
   ![alt text][top20heatmap]
   
### GO/KEGG

1. GO analysis
   
   结果：
   
   ![alt text][GOanalysis]
   
   可以看到显著富集的terms包括类黄酮合成、对光照刺激的响应、叶绿体重定位等。
   
2. Fold Enrichment, p-value和FDR
   
   Fold Enrichment=$\frac{proportion \quad in \quad sample}{proportion \quad in \quad database}=\frac{counts \quad in \quad sample}{sample \quad size} \cdot \frac{database \quad size}{counts \quad in \quad database}$
   
   相当于是计算了某个GO term在样本中的占比比整个数据库中高了多少倍。
   
   p-value相当于计算随机抽样时产生当前富集结果的概率。数据库中基因总数为N，该GO term下的基因数目为K，样本包含n个基因，其中k个属于该GO term。如果完全随机地从数据库中抽取n个基因，其中属于该GO term的基因数X应服从超几何分布H(N,n,K)，即P(X=k)=$\frac{C_K^k C_{N-K}{n-k}}{C_N^n}$，这一概率即为raw p-value。
   
   FDR
   
[venn]: https://github.com/StellariaL/bioinfo2023/blob/main/DE_venn.jpg
[top20heatmap]: https://github.com/StellariaL/bioinfo2023/blob/main/top20heatmap.jpg
