# 如何在Linux下配置matlab编译环境

这个命题从何而来呢？小编最近用到GISTIC（用于寻找癌症中somatic CNV，官网：<http://portals.broadinstitute.org/cgi-bin/cancer/publications/pub_paper.cgi?mode=view&paper_id=216&p=t>)，下载好之后运行发现系统少了东西，报错信息如下：

![mark](http://p87h8v5o2.bkt.clouddn.com/blog/180713/57JG0E9ke6.png?imageslim)

于是就在网上搜帖子，生信菜鸟团的一篇文章给了我很大启发，大致内容是说__很多高大上的单位（如BROAD）会以matlab代码的形式发布他们开发的程序，我用的GISTIC恰好就是BROAD研究所开发的__。因此，想要运行这样的程序，就需要在Linux系统里安装matlab编译环境。

## MATLAB简介

首先介绍一下什么是MATLAB，简单来说，是一门编程语言，主要用于算法开发，数据可视化以及数值计算等。MATLAB将[数值分析](https://baike.baidu.com/item/%E6%95%B0%E5%80%BC%E5%88%86%E6%9E%90)、[矩阵计算](https://baike.baidu.com/item/%E7%9F%A9%E9%98%B5%E8%AE%A1%E7%AE%97/8089413)、科学数据可视化以及非[线性](https://baike.baidu.com/item/%E7%BA%BF%E6%80%A7)动态系统的[建模](https://baike.baidu.com/item/%E5%BB%BA%E6%A8%A1/814831)和仿真等诸多强大功能集成在一个易于使用的视窗环境中，为科学研究、工程设计以及必须进行有效数值计算的众多科学领域提供了一种全面的解决方案。生信小伙伴大多使用R,Python，Perl等，对MATLAB的使用频率不高，其实Python的科学计算包中有很多基于MATLAB。 

## 下载MATLAB

下载地址为<https://www.mathworks.com/products/compiler/matlab-runtime.html>，选择一个Linux 64位版本即可。

![mark](http://p87h8v5o2.bkt.clouddn.com/blog/180713/C80e44LG86.png?imageslim)

## 解压并安装MATLAB

重要的事情要先说，这一步很简单，但一定要在安装过程中记住环境变量LD_LIBRARY_PATH 和XAPPLRESDIR的值。

下载完成之后，用unzip命令解压即可，如图：

![mark](http://p87h8v5o2.bkt.clouddn.com/blog/180713/BlemgfKaAC.png?imageslim)

直接运行install就可以（我是在自己的小电脑上跑的，所以用的是root账户，建议大家也用root安装，另外，我用的是深度系统，是带图像界面的，所以可以按照指令一步步下去，很方便）。install后就会出现如下图的按照界面：

![mark](http://p87h8v5o2.bkt.clouddn.com/blog/180713/BaGab8815I.png?imageslim)

单击下一步，一路走下去，用默认安装路径就可以，我这里是/usr/local/MATLAB/MATLAB_Compiler_Runtime, 点击安装。

这里要开始敲黑板了。待安装进度条走完，会跳出如下界面：

![mark](http://p87h8v5o2.bkt.clouddn.com/blog/180713/iDB5lH2a45.png?imageslim)

记录了我们刚开始强调的环境变量的值，建议大家截图或者拍照记录。

之后单击下一步就完成了安装。

## 修改系统环境变量

首先我们使用export命令查看系统全部环境变量，你会发现并没有LD_LIBRARY_PATH 和XAPPLRESDIR所对应的设置，因此需要增加二者的值。这一步对有Linux使用经验的人不算难，百度搜索linux添加环境变量，教程就会出来很多，你可以选择用export命令临时创建环境变量或者选择修改/etc/profile文件。

### 直接使用export命令

直接在命令行终端用export命令修改环境变量，该操作只能临时更改环境变量的值，退出shell后修改随之失效。针对本例，我们需要在终端输入如下命令：

```shell
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/MATLAB/MATLAB_Compiler_Runtime/v83/runtime/glnxa64:/usr/local/MATLAB/MATLAB_Compiler_Runtime/v83/bin/glnxa64:/usr/local/MATLAB/MATLAB_Compiler_Runtime/v83/sys/os/glnxa64
export XAPPLRESDIR=$XAPPLRESDIR:/usr/local/MATLAB/MATLAB_Compiler_Runtime/v83/X11/app-defaults
```

完成之后就会立即生效，此时就可直接执行文章开头要使用的GISTIC了。

### 修改/etc/profile文件

用`vi /etc/profile`打开文件，在末尾添加如下代码（变量值请使用自己之前的记录）：

```shell
export LD_LIBRARY_PATH=/usr/local/MATLAB/MATLAB_Compiler_Runtime/v83/runtime/glnxa64:/usr/local/MATLAB/MATLAB_Compiler_Runtime/v83/bin/glnxa64:/usr/local/MATLAB/MATLAB_Compiler_Runtime/v83/sys/os/glnxa64
export XAPPLRESDIR=/usr/local/MATLAB/MATLAB_Compiler_Runtime/v83/X11/app-defaults
```

添加完成后保存并退出编辑器，然后`source /etc/profile`使修改生效。

完成以上工作后，大家就可以放心大胆地使用matlab编写的软件了。

