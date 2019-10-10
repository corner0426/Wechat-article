# 文献精读——精神分裂症中环状RNA表达谱分析（上）

环状RNA的研究依旧如火如荼，但在精神分裂症（schizophrenia ，SCZ）中的研究还是比较匮乏的。今年2月，发表在《[Neuropsychopharmacology](https://www.ncbi.nlm.nih.gov/pubmed/?term=30786269#) 》上的一篇文章首次系统的分析了精神分裂症中环状RNA的表达图谱。这篇文章几乎全是生信分析，且从投稿到接受仅仅只用了三个月的时间，由此可见其行文的严谨性。那么，究竟是什么打动了编辑和reviewers? 让我们用两期的推送仔细品读一下该文章。



## workflow



首先用一张图描绘一下文章的分析流程。如下图，可以看到作者采用了两种建库方式进行测序，分别得到total RNA来源的circRNA reads和线性酶消化后富集的circRNA reads。这两种建库方式各有优势，相辅相成，帮助作者完成图中的各项分析任务。

![](http://ww1.sinaimg.cn/large/005SiqoKly1g1vi4f4zywj30su0jpq3s.jpg)



## 病例对照对比分析发现精神分裂症中环状RNA表达整体下降

在total RNA建库数据集中，通过对17个病例和18个健康对照的大脑DLPFC脑区进行转录组差异分析鉴定得到574个差异环状RNA，其中68%的差异环状RNA表达下调（如图A,B）。在RNase-enriched文库中分析得到了类似的结果，72%（818）个环状RNA表达差异下调（如图C,D）。此外，深入探究发现多数环状RNA仅在对照组中表达，而病例组中不存在。以上结果说明精神分裂症中环状RNA整体表达呈现下降趋势。

![](http://ww1.sinaimg.cn/large/005SiqoKly1g1vik6pumwj30p00hu7d6.jpg)



## DLPFC脑区转录出的环状RNA特点

基于高通量测序得到的环状RNA在分析的时候一般需要对鉴定到的环状RNA的基本特征进行描绘。如下图，通过对比已知数据库，发现超多半数（59%）的环状RNA为novel；对其进行基因组区域注释发现85%的环状RNA来源于蛋白编码区；分析环状RNA的宿主基因发现大多数基因只转录出一个环状RNA转录本。

![](http://ww1.sinaimg.cn/large/005SiqoKly1g1vjh0dbyqj30k20nk0x6.jpg)



## 环状RNA的PCR验证

环状RNA的验证分为两个等级，差异表达验证和成环性验证。对于差异表达验证，作者选取8个在total-RNA文库以及RNase-enriched文库都具有差异表达的环状RNA作为候选分子，设计反向引物。如下图，PCR结果支持差异表达。

![](http://ww1.sinaimg.cn/large/005SiqoKly1g1vk1yrdavj30de087762.jpg)

验证成环性的方法是使用线性RNA酶处理，对比处理前后分子的表达水平是否存在显著差异。如下图，作者按照表达水平均匀选择15个候选环状RNA，采用线性酶处理后发现大部分环状RNA表达稳定。

![](http://ww1.sinaimg.cn/large/005SiqoKly1g1vk54wooqj30hc088mxi.jpg)

## 差异表达宿主基因的功能性富集分析

虽然此步属于常规分析系列，但作者在文章中对结果的解读非常详细，值得借鉴。通过GO分析，差异表达的环状RNA的宿主基因富集于neurogenesis, dendritic morphogenesis, dendritic development, neuron differentiation, and brain development通路中，考虑到精神分裂症的青少年发病倾向以及主要病理特征是影响神经connectivity，这些分子通路的发现具有临床意义。作者还利用`TOPPFun`数据库对宿主基因在已知疾病基因集合中的富集情况进行研究，发现认知和行为障碍类型疾病，包括自闭症，智力迟钝和智力低下，与精神分裂症相关环状RNA的宿主基因显著富集。

![](http://ww1.sinaimg.cn/large/005SiqoKly1g1vku6q0avj30xh0a6gmr.jpg)



故事讲到这里已经具有很强的可信度了。本期带来的虽然以基本分析为主，但简单的分析解释了深刻的生物学意义。下期我们一起详细解读剩下的高级分析部分，看看文章如何升华的。