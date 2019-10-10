# 再谈RNA数据标准化——quantile normalization

熟悉转录组数据（特别是RNA-Seq）分析的小伙伴肯定对RNA数据标准化流程了如指掌，曾经的我也天真地以为会做了RPKM和TPM就万事大吉了，可是前两天处理一个流程的时候读到了下面一段文字，让我瞬间惶恐万分：

![mark](http://p87h8v5o2.bkt.clouddn.com/blog/181102/BIhe4A4E6h.png?imageslim)

quantile normalization 是什么鬼？为什么跟我做差异分析的时候说的不一样？于是赶紧维基百科一下，这还真是专业的统计术语，主要目的是使数据分布均一。那对于转录组数据而言，quantile normalization到底是一个怎样的操作呢？

## 先说转录组矩阵

大伙很清楚了，转录组数据（这里不仅仅包括测序，芯片也可以）可以看作是一个二维矩阵，矩阵中每一列是表示一个样本，每一行表示一个基因。如下图示例：

![mark](http://p87h8v5o2.bkt.clouddn.com/blog/181102/IKBg9K35j9.png?imageslim)

看到这组数据，大家可能会有以下两个直观的感受：样本s1整体的测序深度可能比其他样本高一个数量级；基因g1的表达水平在这三个样本中相对较高。如果拿着这个数据直接进行统计分析的话，或多或少会产生一些bias。

## quantile normalization做了什么

* 首先对数据按列排序，并记录真实数值的排序索引
* 全部列按照从大到小顺序排列好后以行为单位，计算平均值
* 根据原始数值的索引位置，将行平均值在列内重排

具体示例我就直接照搬维基百科了：

![mark](http://p87h8v5o2.bkt.clouddn.com/blog/181102/ddBEgdJBBl.png?imageslim)

## 有没有代码实现？

这是大家最关心的问题，作为生物专业的人员，不能什么代码都要自己去创造，学会借用才是关键。简单搜了一下，发现R里面有一个包可以用，叫做`preprocessCore`，来自于强大的Bioconductor。这个包的manu大家自行去网上学习吧，这里直接给出实战代码，主要是矩阵格式的处理，当然了，这方面我也是小白，现学现卖，大家凑合着看吧。

```R
library(preprocessCore)
rt = read.table('test_matrix.csv', sep = ',', header = T, check.names = F)
rt = as.matrix(rt)
rownames(rt) = rt[,1]
exp = rt[,2:ncol(rt)]
dimnames = list(rownames(exp), colnames(exp))
data = matrix(as.numeric(as.matrix(exp)), nrow = nrow(exp), dimnames = dimnames)
data_q <- normalize.quantiles(data)
data_q = matrix(as.numeric(as.matrix(data_q)), nrow = nrow(exp), dimnames = dimnames)
write.csv(data_q, file='test_matrix.quantile.csv')
```

## 什么时候用？

最后就是要知道什么时候对转录组数据应用quantile normalization。这是个比较大的命题，估计能写一篇review了，但其实开篇的时候就提到了差异分析的时候基本是不用的，因为已经有了RPKM和TPM。我此处用的目的是研究表达和GWAS关联信号的相关，因此需要对表达数据均一化，其余方面的分析我还没做过，不敢妄言，好在前人有了很好的总结，贴出来以后再看吧。<https://www.biorxiv.org/content/biorxiv/early/2014/12/04/012203.full.pdf>

