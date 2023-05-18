1. ChIP-seq的control(input)
   
   ChIP-seq通过reads的富集判断目标蛋白与DNA的互作，但一些其他原因，如染色质开放程度、特定的序列特征等，也会导致reads的富集，因此需要通过input排除此类因素。

2. 参数解释

   * findPeaks
     
     findPeaks的基本调用方法为findPeaks <experiment dir> -style <xxx> -o <output> -i <input dir>

     experiment dir是ChIP-seq实验组样本的文件路径，是事先通过makeTagDirectory指令从alignment文件中整理出的中间文件。

     -style指定了findPeaks界定peak的策略，需要根据ChIP-seq的目标蛋白选择合适的策略，最常用的是factor和histone，也有super（用于寻找super enhancer）、groseq（用于GRO-seq数据分析，检测某一时刻正在转录的基因），等等。

     -style factor主要用于在基因组上点状分布的蛋白，例如转录因子。该模式下peak的大小是固定的，由软件根据reads的分布自动计算（也可以添加参数-size <#>手动设置）。该模式还会对初步得到的peak进行筛选，默认两个peak中心之间的距离不得小于peak大小的2倍（可添加参数-minDist <#>手动设置），且要求peak处的reads（tags）密度必须大于peak周围10kb区域内reads密度的4倍（可添加参数-L <#>手动设置）。这些筛选条件会舍弃过密的peak，有利于后续对consensus sequence的分析。

     -style histone主要用于在基因组上成区域分布的蛋白，例如组蛋白修饰。该模式下peak的大小是可变的，软件以-size为大小初步计算peak位置（默认值500bp），然后将距离小于-minDist的peak融合为region（默认值1000bp）。该模式下软件也不会根据peak和周围区域reads密度进行筛选。

     <output>是结果的输出路径，不提供则默认输出到屏幕。

     <input dir>是input（空白对照）样本的存储路径，不提供亦可。

     除上文提到的根据peak周围区域reads密度的筛选之外，软件还会对初步计算出的peak进行两次筛选。软件会根据泊松分布计算每个peak reads数的p值，要求必须小于0.0001（可添加参数-P <#>手动设置）；如果提供了input，软件会要求peak的reads密度经归一化后不得小于input相应位置reads密度的4倍（可添加参数-F <#>手动设置）。

   * findMotifGenome.pl

     findMotifGenome.pl的基本调用方法为findMotifsGenome.pl <peak/BED file> <genome> <output directory> -size # [options]

     peak/BED file是peak文件的路径，作为输入。

     genome是参考基因组名称，如人的hg19、小鼠的mm10等。

     output directory是结果的输出路径

      -size指定了软件用多长的序列寻找motif，默认200，可通过-size <#>手动设置。建议对转录因子设置为50（若寻找临近的共富集motif可增加至200），组蛋白设置为500-1000。若搜索范围是可变的，可设为-size given，软件会在每个peak的全长内寻找motif。

     一些重要的其他options包括：

     -len <#,#,...>指定了motif长度，提供多个值时软件会分别寻找对应长度的motif，默认8,10,12。

     -bg <background peak file>指定了作为背景的序列，软件在初步计算出motif后会计算样本中每个motif相对于背景的丰度，据此检验motif的显著性。不指定-bg时软件会随机选择2×peak数或50000个区域作为背景，并保证其GC含量的分布与样本一致。

     -S <#>指定了输出的motif数量，默认25。

     -mis <#>指定了软件在寻找motif过程中允许的碱基错配数，值越高则越敏感，越低则越保守，默认2。

3. 实际分析

   命令：

   ```bash
       # 生成中间文件
       makeTagDirectory myinput/ip ip.chrom_part.bam
       makeTagDirectory myinput/input input.chrom_part.bam
       
       # 计算peaks
       # 只保留Fold Change>8 && p<0.00000001
       findPeaks myinput/ip/ -style factor -o myoutput/chrom_part.peak -i myinput/input/ -F 8 -P 0.00000001
       # 使用默认参数
       findPeaks myinput/ip/ -style factor -o myoutput/chrom_part.morepeak -i myinput/input/
       
       # 计算motifs
       # findMotifsGenome.pl myoutput/chrom_part.morepeak sacCer2 myoutput/chrom_part.moremotif.output -len 8
   ```

   motif结果截图：
   
   ![alt text][moremotif]
                                          
[moremotif]: https://github.com/StellariaL/bioinfo2023/blob/main/moremotif.png


