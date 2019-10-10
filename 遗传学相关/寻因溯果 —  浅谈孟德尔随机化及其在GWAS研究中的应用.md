

# 寻因溯果 — — 浅谈孟德尔随机化及其在GWAS研究中的应用

要聊起今天的话题就得从流行病学讲起。人们通常采用随机对照试验（random control trial, RCT）研究暴露因素X与疾病结局Y之间的直接关联证据，但该方法往往受限于人类医学伦理与诸多的试验设计，再加上近年来广泛出现的大样本GWAS数据，表观遗传学以及各种“组学”数据，使得在因果关系推断的研究中面临诸多挑战。借助孟德尔随机化（Mendelian Randomization, MR）的设计思想，将基因或者表观遗传标记作为待研究暴露因素的工具变量，为解决上述问题提供了有效的途径。

本文以笔记的形式简要介绍孟德尔随机化的理论基础和基于GWAS数据的R包实践。

## MR法研究设计原理

首先，通过回忆高中物理知识大家都知道孟德尔遗传的基本思想遵循`"亲代等位基因随机分配给子代"`，在基因型决定表型的前提假设下，基因型通过表型与疾病建立联系，因此，基因型可以作为工具变量来推断表型与疾病的关联。



![](http://ww1.sinaimg.cn/large/005SiqoKly1g1dqezlk69j30k10l9dgf.jpg)

在该模型中，遗传变异可以直接准确测量，并且不受外界环境等因素影响，属于长期而稳定的暴露因素，因此，MR设计可以最大程度的降低偏倚的作用。

## 两样本MR（Two-sample MR）

单独介绍两样本MR是因为其设计策略是建立在遗传变异-暴露因素和遗传变异-结局变量的关联研究人群来自相同的人群的两个独立样本，如暴露因素的GWAS研究和结局变量的GWAS研究。经过改进的两样本MR方法具有一个显著的优点，即不依赖基因型数据，只需通过现有GWAS结果统计量即可估算暴露因素与结局变量之间的因果关联。2018年发表在《eLife》上面的一篇文章（PMID：29846171）开发了一个数据库和R包，专门针对现有GWAS数据进行两样本MR分析。

## 借助巨人的肩膀——TwoSampleMR

我们以实战的形式介绍软件的使用，具体的理论基础和更多的细节请读者参考原著论文和其他相关资料。

完成整个MR分析可以分为四个步骤：

* 选择基因型数据作为工具变量（如果是SNP，需要进行LD clumping）
* 从GWAS summary中制作工具变量的exposure和outcome文件
* 将工具变量在暴露和结局变量间的效应值进行协同统一，主要是针对两个GWAS研究所用的参考基因型不同
* 进行MR分析，敏感性分析以及绘图等



## 实战代码

```R
#安装
install.packages("devtools")
library(devtools)
install_github("MRCIEU/TwoSampleMR")

#获取工具变量在暴露研究中的数据
bmi_file <- system.file("data/bmi.txt", package="TwoSampleMR")
bmi_exp_dat <- read_exposure_data(bmi_file)

#Clumping
bmi_exp_dat <- clump_data(bmi_exp_dat)

#从结局变量中取出工具变量所对应的效应值
outcome_dat <- read_outcome_data(
    snps = bmi_exp_dat$SNP,
    filename = "gwas_summary.csv",
    sep = ",",
    snp_col = "rsid",
    beta_col = "effect",
    se_col = "SE",
    effect_allele_col = "a1",
    other_allele_col = "a2",
    eaf_col = "a1_freq",
    pval_col = "p-value",
    units_col = "Units",
    gene_col = "Gene",
    samplesize_col = "n"
)
###在R包说明文档中，昨天首先介绍的是直接从数据库已有的GWAS中提取信息，这种方式比较快捷，但需要用户自行注册goole账号，并且数据库中还要有自己感兴趣的GWAS数据###

#Harmonise data
dat <- harmonise_data(
    exposure_dat = bmi_exp_dat, 
    outcome_dat = chd_out_dat
)

#Perform MR
res <- mr(dat)
###默认情况下会执行多种MR算法，并分别给出结果。也支持用户指定算法###
mr_method_list()###获取全部MR算法
mr(dat, method_list=c("mr_egger_regression", "mr_ivw"))###指定两种MR算法

#敏感性分析
mr_heterogeneity(dat)

#绘制散点图
p1 <- mr_scatter_plot(res, dat)
p1[[1]]

```



## 一些需要注意是事

熟悉遗传学研究的朋友都知道基因是具有多效性的（pleiotropy），也就是说一个SNP可能不单单与目标暴露因素有关，也存在同时与其他暴露因素有关系的可能性，在这种情况下，需要进行敏感性分析（sensitivity analysis）来确定非特异SNP的存在对结果造成的影响。另一方面，如果多个SNPs共同作为工具变量，基因多效性带来的偏倚也会存在，可以使用MR-Egger回归分析的方法来评价偏倚大小。



参考文献：

王莉娜,  Zuofeng Z . 孟德尔随机化法在因果推断中的应用[J]. 中华流行病学杂志, 2017, 38(4):547.

Gibran Hemani, Jie Zheng, Kaitlin H Wade, Charles Laurin, Benjamin Elsworth, Stephen Burgess, Jack Bowden, Ryan Langdon, Vanessa Tan, James Yarmolinsky, Hashem A. $*The MR-Base platform supports systematic causal inference across the human phenome.* eLife 2018. 