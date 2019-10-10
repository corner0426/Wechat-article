

# 谈一谈基于TCGA拷贝数变异的关联分析——ParseCNV流程搭建

虽然现在拷贝数变异（copy number variant)已不再是时髦的话题，但当小编真的要开始分析这样的数据时，却苦于没有顺手的流程。刚开始入门的时候查了不少资料，大部分都集中在前端call CNV的方向，后期数据转换和统计分析却鲜有涉及。之前做SNP关联分析的经验告诉我PLINK似乎可以，但仔细阅读并RUN一遍之后才发现入坑了，主要是PLINK仅仅提供整个群体水平的检测，不能给出具体哪个CNV显著与表型相关。在经验主义的带领下走了弯路之后，决定回归文章，从论文中寻找靠谱的分析软件，于是就有了与ParseCNV的一场邂逅。

文章发表在《Nucleic Acids Research》上面，不仅详细介绍了自己的算法还讨论了其他功能相似软件的优缺点，强烈推荐有兴趣的读者阅读原文<https://academic.oup.com/nar/article/41/5/e64/2414513>.

下面我们采用懒人的做法，直接用实际数据操练起来。

## 理解TCGA拷贝数变异文件格式和数据意义

在构建流程之前先看看我们手上有怎样的数据是迈出第一步的关键。 TCGA网站__Copy Number Variation Analysis Pipeline__页面对拷贝数变异的文件格式和内容有详细的介绍<https://docs.gdc.cancer.gov/Data/Bioinformatics_Pipelines/CNV_Pipeline/>， 简单地，文件分6列，第5列表示CNV区域设计的探针数（TCGA CNV数据均来自Affymetrix SNP 6.0 array 芯片），第6列表示segment mean value，可以转换成拷贝数，是下游分析的关键。示例文件截图如下：

![mark](http://p87h8v5o2.bkt.clouddn.com/blog/180707/B6lDh83jCD.png?imageslim)

分析的难点之一是如何将segment mean value对应到ParseCNV可以识别的变异种类（以CNState value表示）。仔细阅读ParseCNV的说明书后，读者会发现这样的描述：__CNState values: state1,cn=0 state2,cn=1 state5,cn=3 state6,cn=4. These states are also referred to as homozygous deletion, hemizygous deletion, hemizygous duplication, and homozygous duplication respectively in literature__, 即ParseCNV将变异类型作为4种分类变量处理，下面的关键点是如何找到cut-off值，将segment mean value对应到CNState分类中。最开始，各位读者可能会跟小编有同样的思路，就是根据segment mean value的计算公式反推出copy number，然后对应到变异种类即可。但实际操作起来总有顾虑，感觉不是很严谨。小编的做法是直接与ParseCNV的开发者讨论，通过近一周的交流（考虑昼夜时差，每天只能来回一封），我get到完全不同的思路。

软件开发者给我的回复是这样的：__You could plot the histogram distribution of Segment_Mean to see if a value <= a cut-off has high density to define "state1,cn=0/state2,cn=1" or a value >= a cut-iff to define "state5,cn=3/state6,cn=4" if you are interested in differentiating those states. __ 也就是说我们需要在segment mean value的分布中去找高密度分布对应的CNState。于是赶紧贴上我的密度分布图：

![mark](http://p87h8v5o2.bkt.clouddn.com/blog/180707/hlBLDBg54d.png?imageslim)   

发现仅仅有两个peak，也就是只能对应到到state2,cn=1和state5,cn=3上，通过curve fitting analysis判断cut-off分别是-0.2和0.2。为了方便我理解，作者还给我一篇应用型的文章，我也一并附给大家<http://mcr.aacrjournals.org/content/12/4/485.long>.

## 实践步骤1——ParseCNV输入文件制作

在找到segment mean value和CNState对应关系之后，我们就可以用脚本来制作ParseCNV的输入文件了。首先,ParseCNV需要3种输入文件：CNV Calls，FAM 和MAP。CNV Calls文件是核心，包含样本拷贝数信息，基本格式为

```markdown
chr1:156783977-156788016      numsnp=6      length=4,040       state2,cn=1 sample1.txt startsnp=rs16840314 endsnp=rs10489835
```

大家很容易用小脚本从TCGA的Copy Number Segment 文件转换得到，这里小编提供awk命令作为参考：

`awk '{ORS="";print "chr"$2":"$3"-"$4" numsnp="$5" length="$4-$3" ";if($6<-.2){print "state2,cn=1"}else if($6>.2){print "state5,cn=3"}else{print "state3,cn=2"}print " "$1" startsnp="$2"_"$3" endsnp="$2"_"$4" conf=NA\n"}' TCGA.txt | sed '1d' > TCGA.rawcnv` 

值得注意的是，CNV Calls文件需要case，control分开，因此读者需要对两组样本分别执行以上脚本。

FAM文件相对简单，是对样本信息的简单描述（只需case样本即可）。文件包含6列信息，分别是：

```markdown
FamID IndividualID(InCNVCallFile) FatherID MotherID Gender(1=male,2=female) Affected(1=unaffected,2=affected,-9=exclude)
```

制作该文件相对容易，如果你的case样本是没有血缘关系的独立个体，可以从case.rawcnv文件中用以下命令提取：

`awk '{print $5}' Cases.rawcnv | uniq | sed 's/^/0\t/' | sed 's/$/\t0\t0\t0\t2/' > Cases.fam `

MAP文件本身是用于存储CNV区间的SNP信息的，对于TCGA3级数据来说，我们无法获得同一个芯片上的SNP信息，因此ParseCNV支持自己自动定义SNP，我们所要做的就是提供一个空文本文件即可。



## 实践步骤2——ParseCNV关联分析

ParseCNV的manu很长并且不易阅读，是由于其算法复杂功能强大所致的。对于初级用户来说，我们暂时只用先了解其主要功能即可。我们的目的很明确，就是找到case/control之间频率差别较大的CNV，那么。用默认参数即可解决我们的问题，唯一需要注意的是TCGA CNV数据采用的基因组坐标基于hg19，而ParseCNV默认使用hg18，需要用--build参数修改。

`perl ParseCNV.pl Cases.rawcnv Controls.rawcnv Cases.fam ChrSnp0Pos.map --build hg19`

## 实践步骤3——ParseCNV结果解读

ParseCNV的输出结果简直可以用饕餮盛宴来形容，有多达12个文件生成，如下图：

![mark](http://p87h8v5o2.bkt.clouddn.com/blog/180710/lGl8fa4fEl.png?imageslim)

其中最主要的输出文件为__CNVR_ALL_ReviewedCNVRs.txt__,几乎包含所有你想要的统计信息。由于文件内容多达50多列，小编选取其中相对常用的13列进行展示，如下图：

![mark](http://p87h8v5o2.bkt.clouddn.com/blog/180710/DKBIFhImEH.png?imageslim)



每一列header对应的解释如下：

| Column       | Description                                                  |
| ------------ | ------------------------------------------------------------ |
| CNVR         | CNV region of greatest significance and overlap coordinates. |
| DelTwoTailed | Two-tailed Fisher’s exact *P* -value based on the contingency table Cases Del/Cases Diploid/Controls Del/Controls Diploid as listed separately. |
| DupTwoTailed | Two-tailed Fisher’s exact *P* -value based on the contingency table Cases Dup/Cases Diploid/Controls Dup/Controls Diploid as listed separately. |
| ORDel        | The odds ratio for deletion.                                 |
| ORDup        | The odds ratio for duplication.                              |
| Cases Del    | The number of cases with a deletion detected in this region by PennCNV. |
| Control Del  | The number of control subjects with a deletion detected in this region by PennCNV. |
| Cases Dup    | The number of cases with a duplication detected in this region by PennCNV. |
| Control Dup  | The number of control subjects with a duplication detected in this region by PennCNV. |
| CNVType      | Deletion or duplication CNVR significant in combined report. |
| Cytoband     | Cytoband genomic landmark designations.                      |
| Gene         | The closest proximal gene based on UCSC Genes, which includes both RefSeq Genes and Hypothetical Gene transcripts. |
| Distance     | The distance from the CNVR to the closest proximal gene annotated. If the value is 0, the CNVR resides directly on the gene. |

