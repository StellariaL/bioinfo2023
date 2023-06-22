### 使用python实现Viterbi算法

* Problem: 给定一段DNA序列，预测5'-splicing site位置
* Model: HMM
* Goal: optimal prediction

代码：

```python
   import sys
   import numpy as np

   # 将碱基对应为4个不同的emission state
   code={'A':0,'C':1,'G':2,'T':3}
   # 将exon, 5'-splicing site, intron对应为3个不同的hidden state
   states={0:'E',1:'5',2:'I'}

   def viterbi(seq,start,end,emat,tmat):
       # paths用于记录上一步的最优路径
       paths=[]
       # scores用于记录得分
       # 分数计算只和上一步有关，因此只需反复更新一个一维数组，不必一直保留前面的得分
       scores=start
       l=len(seq)
       # iteration
       for i in range(1,l+1):
           c=code[seq[i-1]]
           # 计算从上一个state到达该处所有路径对应的分数
           s=np.tile(scores,(3,1))+tmat+np.transpose(np.tile(emat[c,...],(3,1)))
           # 选择最优路径，更新分数
           scores=np.max(s,axis=1)
           # 记录最优路径
           paths.append(np.argmax(s,axis=1))
       # traceback
       result=np.zeros(l,dtype=int)
       # 从得分最高的终止处开始
       result[-1]=np.argmax(scores+end)
       # 根据paths的记录依次前推
       for i in range(2,l+1):
           result[l-i]=paths[l-i+1][result[l-i+1]]
       return result

   # 所有概率均转化为对数相加，避免数值过小产生计算误差
   # 定义序列起始和终止处的概率
   start=np.array([0,-np.inf,-np.inf])
   end=np.array([-np.inf,-np.inf,-1])
   # emission probability matrix
   # 行对应emission state，列对应hidden state
   emat=np.log10(np.array([[0.25,0.05,0.4],[0.25,0,0.1],[0.25,0.95,0.1],[0.25,0,0.4]]))
   # transition probability matrix
   # 行对应下一个state，列对应上一个
   tmat=np.log10(np.array([[0.9,0,0],[0.1,0,0],[0,1,0.9]]))
   # observed sequence
   seq=''
   result=viterbi(seq,start,end,emat,tmat)
   # 按对应关系输出预测结果
   prediction=''
   for r in result:
       prediction+=states[r]
   print('Prediction:')
   print(seq)
   print(prediction)
```

测试输出：

```
   输入：seq='CTTCATGTGAAAGCAGACGTAAGTCA' （课件示例）
   输出：
   Prediction:
   CTTCATGTGAAAGCAGACGTAAGTCA
   EEEEEEEEEEEEEEEEEE5IIIIIII
```
