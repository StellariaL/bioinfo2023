1. columns of iris

   ```R
   sapply(iris, class)
   ```
   
   output:
   
   ```R
   Sepal.Length  Sepal.Width Petal.Length  Petal.Width      Species 
   "numeric"    "numeric"    "numeric"    "numeric"     "factor" 
   ```
   
2. mean and sd of Sepal length

   ```R
   mean=aggregate(Sepal.Length~Species,iris,mean)
   sd=aggregate(Sepal.Length~Species,iris,sd)
   result=merge(mean,sd,by='Species',suffixes=c('.mean','.sd'))
   write.csv(result,"result.csv")
   ```
   
   output:
   
   ```R
        Species Sepal.Length.mean Sepal.Length.sd
   1     setosa             5.006       0.3524897
   2 versicolor             5.936       0.5161711
   3  virginica             6.588       0.6358796
   ```
   
3. one way ANOVA of sepal width
   
   ```R
   summary(aov(Sepal.Width~Species, data=iris))
   ```
   
   output:
   
   ```R
                Df Sum Sq Mean Sq F value Pr(>F)    
   Species       2  11.35   5.672   49.16 <2e-16 ***
   Residuals   147  16.96   0.115                   
   ---
   Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1
   ```
   
   p值很小，说明3个物种的sepal width之间存在显著差异。
