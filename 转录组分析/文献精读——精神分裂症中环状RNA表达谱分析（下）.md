# 文献精读——精神分裂症中环状RNA表达谱分析（下）

上一期的推送中我们简要介绍了一篇发表在《[Neuropsychopharmacology](https://www.ncbi.nlm.nih.gov/pubmed/?term=30786269#) 》上的精神分裂症环状RNA研究，文章通过对来自DLPFC脑区的转录组测序数据进行分析，揭示了精神分裂症相关环状RNA的表达图谱。接着上期的内容，我们来继续探讨文章后半部分更为高级的生信分析内容。



## 环状RNA的表达变化是否与宿主基因相关联？

为了回答这个问题，研究人员通过Total RNA建库的数据将环状RNA表达相对变化与其宿主基因表达的相对变化进行比较。他们发现尽管只有半数的差异环状RNA的宿主基因体现出相同的变化趋势（如下图A)，但环状RNA表达下调与对应宿主基因的表达差异具有微弱的相关性（P=0.022，如下图A）。但是这种相关性在全部环状RNA集合中便不再存在了（如下图B）。说明环状RNA在精神分裂症中的表达差异与其宿主基因具有一定微弱的关系。

![](http://ww1.sinaimg.cn/large/005SiqoKly1g1yygdj9qwj30ci0o5juh.jpg)



作者进一步研究表达差异的基因是否转录出表达差异的环状RNA，或者表达差异的环状RNA是否来自与表达差异的基因。统计发现798个差异的线性转录本中，仅有37个可以转录出环状RNA，其中仅有1个是差异表达的。

## 你们想要的网络图——ceRNA分析

环状RNA分析的文章怎么能缺少circRNA-miRNA-mRNA的互作网络图呢？套路大家懂的，从差异的环状RNA中筛选miRNA靶点，然后再用miRNA去寻找差异的mRNA。该步分析的核心在于靶点预测软件的选区和结合位点阈值的设定，该文章选用`TargetScan`，阈值设置为`context score cutoff < -0.4`。通过筛选，574个差异环状RNA靶向241个miRNA，进而连接306个差异基因（如图A）。为了进一步减少网络中的分子数目，作者限定差异表达的环状RNA必须排在前45位，重复同样的分析得到99个miRNA和224个差异表达mRNA（如图B）。由于精神分裂症中已有miRNA的研究报道，作者在网络中将这些miRNA及其连接的分子挑选出来组成了新的网络（如图C）。

![](http://ww1.sinaimg.cn/large/005SiqoKly1g1yyin84xzj30re0pc7ca.jpg)



## 来点不一样的——环状RNA可变剪切分析

考虑到大脑DLPFC的环状RNA表达的丰富程度，研究人员假设这些环状RNA的形成可能会伴随着不同程度的可变剪切存在。在技术上，`CIRCexplorer2`程序扩展后具有基因组组装能力，借助`Cufflinks`以及注释信息，能够帮助科研人员鉴定环状RNA的可变剪切事件。

利用RNase-enriched的数据集，研究人员在全部样本中发现约有35%~76%的back-splicing位点存在可变剪切现象，细分后发现精分患者组的可变剪切比例稍低于正常对照（如下图A）。为了更加精确地度量可变剪切事件，作者分别采用四种计算方式：Percent Splicesite Usage (PSU) for alternative 5′/3′ splice site selection, Percent Spliced In (PSI) for cassette exon selection and Percent Intron Retention (PIR) for intron retention 对四种不同的可变剪切事件进行度量。如下图B-E，精神分裂症患者中环状RNA可变剪切水平明显低于对照组，且这种趋势并不是由于测序量差异导致的偏差（如下图F）。

![](http://ww1.sinaimg.cn/large/005SiqoKly1g1yymst1xrj30gj0jidi9.jpg)

## 一点总结

这篇文章在2019年开年凭借几乎纯生物信息分析发表在老牌杂志《[Neuropsychopharmacology](https://www.ncbi.nlm.nih.gov/pubmed/?term=30786269#) 》上自有它的可贵之处。细读下来，全文虽然大部分分析内容比较常规，但行文逻辑以及结果阐述都非常清晰明了。作者在差异分析中不是急于挑选一个候选分子进行下游的生物学验证，而是将观察到的病理组的整体表达下调这一趋势深入研究下去，并且分析了较为复杂的可变剪切事件。作为精神分裂症领域第一篇环状RNA研究的文章，作者在曝谱的同时将故事讲得如此生动还是值得我们学习的。