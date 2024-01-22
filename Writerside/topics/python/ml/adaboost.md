# AdaBoost 介绍

<show-structure depth="2"/>

> Writerside 不支持行内数学公式，后续进行补充。
{style="warning"}

## 1. AdaBoost 算法简介

Adaboost（Adaptive Boosting）是一种基于 Boosting 的算法，用于提高一组弱分类器的性能。它通过组合多个简单的模型（通常称为弱学习器）来创建一个强大的总体模型。

Adaboost 算法本身是通过**改变数据分布来**实现的，它根据每次训练集中每个样本的分类是否正确，以及上次的总体分类的准确率，**来确定每个样本的权值**。将修改过权值的**新数据**送给下层分类器进行训练，最后将每次得到的分类器融合起来，作为最后的决策分类器。

## 2. AdaBoost 算法原理

**输入**：训练数据集中每条样本点由输入特征与标签组成，

```tex
T = \{(x_1, y_1), (x_2, y_2), \dots, (x_N, y_N)\}
```

**输出**：最终分类器 



<seealso>
<category ref="ref_docs">
    <a href="https://mp.weixin.qq.com/s/anOPos-Tiq2yIptpxnTcVA"></a>
</category>
</seealso>