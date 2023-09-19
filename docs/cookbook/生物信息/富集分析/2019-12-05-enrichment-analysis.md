---
title: 基因功能富集原理学习笔记
urlname: 2019-12-05-enrichment-analysis
author: 章鱼猫先生
date: "2019-12-05 09:06:11"
updated: "2023-07-25 02:48:02"
---

基因功能富集分析中的基因功能指的是众多代表一定的基因功能特征和生物过程的基因功能集(gene set)。由这些基因功能集构成的常用基因功能数据库有 GO, 生物学通路, 包含生化反应、代谢或信号通路的 KEGG, Reactome, Biocarta 等, 整合数据库, 如 MsigDB 等。

基因功能富集分析的方法基于数据来源和算法大致可以分为 4 大类: ORA, FCS, PT, NT 的方法。ORA（过代表分析方法）是最早出现的一类基因功能富集方法, 它针对的数据是一组感兴趣的基因(基因列表), 其目的是在这组基因中发现有明显统计学上富集的基因功能集。其基本步骤包括先将给定的基因列表与待测功能集做交集, 找出其中共同的基因并进行计数(统计值), 最后利用统计检验的方式来评估观察的计数值是否显著高于随机, 即待测功能集在基因列表中是否显著富集。

目前有许多工具及数据库提供 ORA 的使用, 包括 DAVID, GOstat, GenMAPP 等。其中 DAVID 提供的基因功能集数据库最为全面, 不仅包含大量不同物种的基因功能注释信息, 也涵盖了主流的生物通路注释库如 GO 条目和 KEGG 通路, 而且还提供了基因名称转换功能, 及良好的结果展示界面。

ORA 方法中最为广泛使用的是 Fisher 精确检验, 即利用 2×2 的列联表，根据超几何分布来检验基因列表中的基因在待测功能集中是否显著富集。

Fisher 精确检验是进行统计分析时经常碰到的一种分析方法，它基于超几何分布，作用于离散变量，用于检测两种分类方法的结果是否独立。

# **超几何分布**

超几何分布是统计学上一种离散概率分布。它描述了由有限个物件中抽出 n 个物件，成功抽出指定种类的物件的个数（不归还）。

例如在有 N 个样本，其中 m 个是不及格的。超几何分布描述了在该 N 个样本中抽出 n 个，其中 k 个是不合格的的机率：
![](https://shub-1251708715.cos.ap-guangzhou.myqcloud.com/elog-cookbook-img/Fixh0BtmkTX3nlHgKV3_C-MqKTUv.svg)

上式可如此理解：

- ![](https://shub-1251708715.cos.ap-guangzhou.myqcloud.com/elog-cookbook-img/FggaFQ7YVyMX-k13j8zG7SNG72Fh.svg)  表示所有在 N 个样本中抽出 n 个，而抽出的结果不一样的数目。![](https://shub-1251708715.cos.ap-guangzhou.myqcloud.com/elog-cookbook-img/FkNIh2xj3hNQ5oNrIpEbxO6z2tXC.svg) 表示在 m 个样本中，抽出 k 个的方法数目。剩下来的样本都是及格的，而及格的样本有 N-m 个，剩下的抽法便有  ![](https://shub-1251708715.cos.ap-guangzhou.myqcloud.com/elog-cookbook-img/Fk_dHEbwssUmqR94UNL0JsANO1PH.svg) 种。
- 若 n=1，超几何分布还原为伯努利分布。
- 若 N 接近 ∞，超几何分布可视为二项分布(二项式分布，是有放回的抽样)。

# **Fisher 精确检验**

回到 Fisher 精确检验，Fisher 精确检验是基于超几何分布计算的，它分为两种，分别是单边检验（等同于超几何检验）和双边检验。Fisher 检验需要回答的问题是，对数据进行两种分类，这两种分类是否独立？即在第一种分类条件下分为某一类的数据是否更倾向于在第二种分类中归于某类。举例说明（例子来源于 wiki）：

> 我们有 24 位测试对象，根据其性别和是否爱学习，将其分为四类，分类结果如下：

|          | 男性 | 女性 | 总和 |
| -------- | ---- | ---- | ---- |
| 爱学习   | 1    | 9    | 10   |
| 不爱学习 | 11   | 3    | 14   |
| 总和     | 12   | 12   | 24   |

> 现在我们的问题是，是否女性更喜欢学习？从数据直观上来看，男性和女性都是 12 人，但是爱好学习的女性是男性的 9 倍，似乎女性的确更爱好学习，但是我们如何定量的去描述这件事呢？可以看出，我们要解决的是一个假设检验问题，我们将零假设设定为：**是否爱好学习和性别无关。**那么，在零假设下>，根据超几何分布我们观察到表格中数据的概率是：

|          | 男性 | 女性 | 总和        |
| -------- | ---- | ---- | ----------- |
| 爱学习   | a    | b    | a+b         |
| 不爱学习 | c    | d    | c+d         |
| 总和     | a+c  | b+d  | a+b+c+d(=n) |

> 参考：[费雪正确概率检定 - 维基百科](https://zh.wikipedia.org/zh-hans/%E8%B2%BB%E9%9B%AA%E6%AD%A3%E7%A2%BA%E6%A6%82%E7%8E%87%E6%AA%A2%E5%AE%9A)

在这里我们做一个简单的概念转换即可知道软件是如何做 GO 富集分析的：

1. N 为 GO 注释数据库中的总基因数；
2. m 为数据库中属于某个 GO 子类的基因数；
3. n 为我们得到的需要进行 GO 富集分析的基因的总数目；
4. k 为 n 中属于 m 的数目。

因此我们就可以计算基因集 n 是否在 m 类中富集的概率。但是知道这个概率后并不能直接用来作为富集分析的结果，必须要对其进行一个评估，因为我们必须要考虑到随机情况，如果随机从 N 中抽取 n 个基因，其中 k 个在 m 中的概率很高的话，那我们富集得到的通路意义就是极小的。这时候我们引入 p 值对富集分析的概率结果进行分析。

# **p-value 检验**

未完，待续...

# **参考资料**

- 井底蛙蛙呱呱呱，《[浅探富集分析中的超几何分布](https://www.jianshu.com/p/13f46bebebd4)》，简书
- 路口不会转弯，《[GO，KEGG，GSEA 富集分析笔记](https://www.jianshu.com/p/a0bad4119e79)》，简书
- z54572，《[Fisher 精确检验的通俗理解](https://blog.csdn.net/z54572/article/details/61199246)》，CSDN 博客
- 戴思达，《[fisher 精确检验(fisher’s exat test)和超几何分布](https://www.cnblogs.com/sddai/p/6279572.html)》，博客园
- en.wikipedia，《[Fisher's exact test](https://en.wikipedia.org/wiki/Fisher's_exact_test)》，维基百科
- zh.wikipedia，《[组合数学](https://zh.wikipedia.org/wiki/%E7%BB%84%E5%90%88%E6%95%B0%E5%AD%A6)》，维基百科
- 铁汉 1990，《[每日一生信--富集分析（超几何分布）（Fisher'sExactTest）](http://blog.sina.com.cn/s/blog_670445240101m4z3.html)》，新浪博客
- 徐洲更，《[转录组入门(8): 富集分析](https://www.jianshu.com/p/5c8c6a380939)》，简书
- 师师，《[Fisher 精确检验和超几何分布什么关系？](https://www.zhihu.com/question/28637406)》，知乎
- Joey 周琦，《[Fisher's exact test( 费希尔精确检验)](https://www.cnblogs.com/Dzhouqi/p/3440575.html)》，博客园
