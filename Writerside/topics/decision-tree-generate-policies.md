# 利用决策树生成应用策略

<show-structure depth="2"/>

> 本篇文章摘录自[《风控策略的自动化生成-利用决策树分分钟生成上千条策略》](https://mp.weixin.qq.com/s/fErTOjdVm28FL5zl_aUBbQ)，仅作为学习使用。

![决策树判断用户是否购买产品](%myimgs%/dt_example_1.png?raw=true)

风控策略同学在挖掘有效的风控规则的时候，时时需要基于业务经营思考将哪几个特征进行组合，这会导致在思考特征组合的时候浪费大量的时间，那么我们有没有什么方法替代人工的分析，直接得出一些策略组合，我们更多的去关注生成的策略是否合理，是否就轻松很多了呢。我们将要介绍的决策树模型，它非常便于理解和可视化，具备自上而下的分支结构以及阈值拆分，可以实现自动化的挖掘大批量的策略组合。

## 1. 简介

在众多的算法中，决策树整体分类准确率不高，但是**部分叶子节点**的准确率却可以很高，因此我们可以提取决策树的叶子规则，并筛选准确率比较高的叶子节点，作为风控策略挖掘手段，并进行策略推荐，替代人工或者辅助人工，大大提高策略发现的效率于效果。

![数据转换为策略](%myimgs%/dt_gen_policies.png?raw=true)

![具体生成策略规则](%myimgs%/dt_example_2.png?raw=true)

从生成的策略规则中可以看到，我们是可以找到 Bad Rate 比较低的客群，如果也低于整体的基准线，那么我们就可以考虑将此规则划分的客群进行授信或提额。

Bad Rate
: 以风控授信场景来说，Good 通常表示好用户，Bad 则表示坏用户，Bad Rate 则表示当前客群中坏用户占比，作为授信场景当然希望应用策略所圈定的客群的 bad rate 越低越好。


## 2. 数据集

数据从真实场景和实际应用出发，利用个人的基本身份信息、个人的住房公积金缴存和贷款等数据信息，来建立准确的风险控制模型，来预测用户是否会逾期还款。一共提供了 **40000** 带标签训练集样本，数据仅有一张表，一共有 19 个基本特征，且均不包含任何缺失值。

| 字段         | 描述                  |
|------------|---------------------|
| **label**  | 是否逾期，1 表示逾期，0 表示未逾期 |
| id         | 主键                  |
| XINGBIE    | 性别                  |
| CSNY       | 出生年月                |
| HYZK       | 婚姻状况                |
| ZHIYE      | 职业                  |
| ZHICHEN    | 职称                  |
| ZHIWU      | 职务                  |
| XUELI      | 学历                  |
| DWJJLX     | 单位经济类型              |
| DWSSHY     | 单位所属行业              |
| GRJCJS     | 个人缴存基数              |
| GRZHZT     | 个人账户状态              |
| GRZHYE     | 个人账户余额              |
| GRZHSNJZYE | 个人账户上年结转余额          |
| GRZHDNGJYE | 个人账户当年归集余额          |
| GRYICE     | 个人月缴存额              |
| DWYICE     | 单位月缴存额              |
| DKFFE      | 贷款发放额               |
| DKYE       | 贷款余额                |
| DKLL       | 贷款利率                |

## 3. 建立决策树

### 3.1 加载数据集

```Python
import pandas as pd
import numpy  as np
pd.set_option('display.max_columns', None)
path  = './train.csv'
df_train = pd.read_csv(path).fillna(-1)  # 缺失值填充 -1

X = train.loc[:, 'XINGBIE':'DKLL']
Y = train['label']
```
{collapsible="true" default-state="collapsed" collapsed-title=""}

### 3.2 训练决策树

sklearn 提供了 `DecisionTreeClassifier` 和 `DecisionTreeRegressor`，前者用于分类任务，后者用于回归任务。

> Scikit-Learn 中提供的决策树，只实现了预剪枝，没有实现后剪枝。
{style="note"}


```Python
from sklearn.tree import DecisionTreeClassifier

clf = DecisionTreeClassifier(
     max_depth=3,
     min_samples_leaf=50
     )
clf = clf.fit(X, Y)
```
{collapsible="true" default-state="collapsed" collapsed-title=""}

## 4. 决策树的可视化





<seealso>
<category ref="ref_docs">
    <a href="https://mp.weixin.qq.com/s/fErTOjdVm28FL5zl_aUBbQ">利用决策树分分钟生成上千条策略</a>
</category>
<category ref="ref_github">
</category>
<category ref="ref_issues">
</category>
<category ref="ref_hf">
</category>
<category ref="ref_ms">
</category>
</seealso>