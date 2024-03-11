# Toad 教程

<show-structure depth="3"/>

Toad 是针对工业届建模而开发的工具包，针对风险评分卡的建模有针对性的功能。本教程针对 Toad 的各类主要功能进行介绍，包括：
1. EDA相关功能
2. 如何使用 Toad 高效分箱并进行特征筛选
3. WOE 转化
4. 逐步回归特征筛选
5. 模型检验和评判
6. 标准评分卡转化和输出 
7. 其他功能

其中，Toad 的主要功能极大简化了建模中最重要最费时的流程，即特征筛选和分箱。

## 1. 安装

使用 pip 和 conda 都可以非常方便地安装 Toad:

<tabs>
<tab title="pip">
<code-block lang="bash">
<![CDATA[
pip install toad
]]>
</code-block>
</tab>
<tab title="conda">
<code-block lang="bash">
<![CDATA[
conda install toad -c conda-forge
]]>
</code-block>
</tab>
</tabs>

## 2. 读取数据

示例数据 [train.csv](https://pan.baidu.com/s/1rIaQm0XunlBU_5YXt6CXBw) 为 165 维，包括一列主键 target 和 month 字段，其中包含了若干个离散型变量和连续性变量，且有一定的缺失值。

<tabs>
<tab title="加载数据">
<code-block lang="python">
<![CDATA[
import pandas as pd

df = pd.read_csv('train.csv')
print('Shape: ', df.shape)
print('Month: ', df.month.unique())
]]>
</code-block>
</tab>
<tab title="输出">
<code-block lang="python">
<![CDATA[
Shape: (108940, 167)
Month: ['2019-03' '2019-04' '2019-05' '2019-06' '2019-07']
]]>
</code-block>
</tab>
</tabs>


> 示例数据的下载密码为: fma6

数据包含 2019 年 3-7 月的数据。其中我们将用 3-4 月数据用于训练样本，5-7 月数据作为时间外样本（OOT）。


<tabs>
<tab title="切分数据集">
<code-block lang="python">
<![CDATA[
df_train = df.loc[data.month.isin(['2019-03','2019-04'])]
df_oot = df.loc[~data.month.isin(['2019-03','2019-04'])]

print('Train size:', train.shape, '\nOOT size:', OOT.shape)
]]>
</code-block>
</tab>
<tab title="输出">
<code-block lang="python">
<![CDATA[
Train size: (43576, 167)
OOT size: (65364, 167)
]]>
</code-block>
</tab>
</tabs>

## 3. EDA 分析

在 EDA 部分，Toad 提供了两个常用函数: `toad.detect` 和 `toad.quality`，其中 `toad.detect` 用于获取数据集的统计分析，而 `toad.quality` 则可以直接获取特征的 IV、GINI 以及 Entropy。

### 3.1 toad.detect


`toad.detect` 用于检测数据情况（EDA），输出每列特征的统计性特征和其他信息，主要的信息包括：缺失值、唯一值数量、数值变量的平均值、离散值变量的众数等：

```Python
toad.detect(df_train)
```
通过 EDA 统计结果，我们可以得知:
- 正样本占比 2.2%，即 `target` 的均值是 0.0219479
- 部分特征有缺失值，且缺失值数量不等，我们需要注意下 missing 列(缺失率)，这也是我们后续筛选变量的重要依据
- 数值型变量和离散型变量有若干个，部分离散型变量的唯一值较多

### 3.2 toad.quality

`toad.quality` 输出每个变量的 iv、gini、entropy 以及 unique values，结果以 iv 值排序。其中 `target` 参数为设定标签字段，`iv_only` 设置是否只输出特征 iv 值。

> 使用时需要注意两点:
> 1. 对于数据量大或高维度数据，建议设置 `iv_only=True`
> 2. 需要去掉主键，日期等高 unique values 且不用于建模的特征
{style="warning"}


```Python
# 去掉 ID 列和 month 列
drop_cols = ['APP_ID_C','month']
toad.quality(df_train.drop(drop_cols,axis=1), 'target', iv_only=True)
```

## 4. 特征筛选

### 4.1 基于 IV 的特征筛选

Toad 提供了 `toad.selection.select` 函数用于特征筛选，它根据特征的缺失值占比、iv值、高相关性进行筛选，其可设置参数有:
- `empty`: 若变量缺失率大于阈值则会被删除，默认为 0.9
- `iv`: 若变量的 IV 值小于阈值则会被删除，默认为 0.02
- `corr`: 若两个变量的相关性高于阈值，IV 低的变量则会被删除，默认为 0.7
- `return_drop`: 若设置为 True，则会返回被删除的变量，默认为 False
- `exclude`: 设定不需要筛选的特征列表

```Python
df_train_selected, dropped = toad.selection.select(
        df_train,
        target='target', 
        empty=0.5, 
        iv=0.05, 
        corr=0.7,
        return_drop=True, 
        exclude=['APP_ID_C','month']
        )
print(dropped)
print(df_train_selected.shape)
```

> 因为特征的 IV 值计算并不需要进行参数拟合，所以整个过程是飞常高效的。

### 4.2 基于逐步回归的特征筛选

除了基于 IV 的方法筛选特征之外，Toad 还提供了 `toad.selection.stepwise` 来逐步递归的方式来筛选特征。相较于前面提到的 IV 筛选特征的方法，此方法是需要不断通过裁剪或者添加入模特征列表，基于训练后模型的表现来决定哪些特征被删除或者保留，所以必然时耗会长点。

`stepwise` 方法支持前向、后向以及双向，可用参数如下:
- `estimator`: 用于拟合的模型，支持 ols、lr、lasso、ridge
- `direction`: 逐步回归的方向
  - forward: 前向
  - backward: 后向
  - both: 双向
- `criterion`: 评估标准
  - aic
  - bic
  - ks
  - auc
- `max_iter`: 最大训练次数
- `return_drop`: 是否返回被提出的特征
- `exclude`: 不需要被训练的字段，比如 ID 或者时间字段

> 经过验证 `direction="both"` 的效果最好，`estimator="ols"` 以及 `criterion="aic"` 运行速度快且结果对逻辑回归建模有较好的代表性。
> 
> 需要的注意的是，递归特征的筛选需要将特征先进行 WOE 转换。
> 
{style="note"}

```Python
## 将woe转化后的数据做逐步回归
df_final_train = toad.selection.stepwise(
    df_train_woe,
    target='target',
    estimator='ols', 
    direction='both', 
    criterion='aic',
    exclude=drop_cols
    )
## 将选出的变量应用于 test/OOT 数据
df_final_oot = df_oot_woe[df_final.columns]
## 逐步回归从 31 个变量中选出了 10 个
print(df_final_train.shape)
cols = list(df_final.drop(drop_cols + ['target'], axis=1).columns)
```


### 4.3 变量评估

熟悉建模的同学肯定知道，我们不光要看变量的 IV 表现，有时间也要看挑选后的变量是否稳定，这就用到 PSI 这样的指标来评估变量的稳定性，`toad.metrics.PSI` 就为我们提供了这样的功能。

```Python
toad.metrics.PSI(df_final_train[cols], df_final_oot[cols])
```


## 5. 分箱

### 5.1 自动分箱

Toad 中 `toad.transform.Combiner` 类提供了分箱功能，它支持数值型和离散型变量的分箱，默认使用卡方分箱，分箱步骤如下:
1. 初始化 `Combiner` 类的实例 `c`
2. 训练分箱: 通过调用 `c.fit` 方法训练分箱，参数解释如下
   - `y`: 标签字段
   - `method`: 
     - chi: 卡方
     - dt: 决策树
     - kmean: 聚类
     - quantile: 等频
     - step: 等步长
   - `min_samples`: 每箱至少包含样本量，可以是数字或者占比
   - `n_bins`: 设定特征的分箱数，若无法分出这么多箱，则会分出最多的箱数
   - `empty_separate`: 是否将空箱单独分开
3. 查看分箱节点: 通过 `Combiner` 类的 `.export` 方法到处分箱细节
4. 手动调整分箱: 手动调整分箱结果，并使用 `c.load` 方法加载分箱结果
5. 应用分箱结果: 通过 `c.transform` 方法应用分箱结果，它还有一个 `labels` 参数，用于设置是否将分箱结果转化成箱标签:
   - 当 `labels=False` 时输出 0, 1, 2（散变量根据占比高低排序）
   - 当 `labels=Trues` 时输出 `(-inf, 0]`、`(0,10]`、`(10, inf)`

<tabs>
<tab title="代码">
<code-block lang="python">
<![CDATA[
c = toad.transform.Combiner()
## 使用特征筛选后的数据进行训练：使用稳定的卡方分箱
## 规定每箱至少有 5% 数据, 空值将自动被归到最佳箱。
c.fit(
    df_train_selected.drop(drop_cols, axis=1),
    y='target',
    method='chi',
    min_samples=0.05,
    empty_separate=False
)
## 为了演示，仅展示部分分箱
print('var_d2: ', c.export()['var_d2'])
print('var_d5: ', c.export()['var_d5'])
print('var_d6: ', c.export()['var_d6'])
]]>
</code-block>
</tab>
<tab title="输出结果">
<code-block lang="python">
<![CDATA[
var_d2: [747.0, 782.0, 820.0]
var_d5: [['O', 'nan', 'F'], ['M']]
var_d6: [['PUBLIC LTD COMPANIES', 'NON-RESIDENT INDIAN', 'PRIVATE LTD COMPANIES', 'PARTNERSHIP FIRM', 'nan'], ['RESIDENT INDIAN', 'TRUST', 'TRUST-CLUBS/ASSN/SOC/SEC-25 CO.', 'HINDU UNDIVIDED FAMILY', 'CO-OPERATIVE SOCIETIES', 'LIMITED LIABILITY PARTNERSHIP', 'ASSOCIATION', 'OVERSEAS CITIZEN OF INDIA', 'TRUST-NGO']]
]]>
</code-block>
</tab>
</tabs>

> 注意删去不需要分箱的列，特别是 ID 列和时间列
> 
{style="warning"}

### 5.2 观察分箱结果

`toad.plot` 模块提供了一部分的可视化功能，我们可以通过观察图来手动调整分箱节点。

#### I) toad.plot.bin_plot

`bin_plot` 有两个参数:
- `x`: 需要观察的变量
- `target`: 标签字段

```Python
from toad.plot import bin_plot

# `transform` 方法用于自动将变量进行 woe 话，设置 `labels=True` 则可以更好的可视化
bin_plot(
    c.transform(df_train_selected[[col, 'target']], labels=True), 
    x="var_d2", target='target'
)
bin_plot(
    c.transform(df_train_selected[[col, 'target']], labels=True), 
    x="var_d5", target='target'
)
```

![bin_plot](https://toad.readthedocs.io/en/stable/_images/tutorial_18_2.png)
![bin_plot2](https://toad.readthedocs.io/en/stable/_images/tutorial_21_2.png)

柱形图代表了样本量占比，折线图(红色)则代表了正样本占比。

#### II) toad.plot.badrate_plot

`badrate_plot` 函数用于观察不同时间段下每箱中的正样本比例，它有两个参数:
- `x`: 需要观察的变量
- `target`: 标签字段
- `by`: 需要观察的特征

> 时间列需要预先分好并设成string，不支持 timestamp
>
{style="warning"}

```Python
from toad.plot import badrate_plot

col = 'var_d2'
badrate_plot(
    c.transform(df_train[[col,'target','month']], labels=True), 
    target='target', x='month', by=col
    )
badrate_plot(
    c.transform(df_oot[[col,'target','month']], labels=True), 
    target='target', x='month', by=col
    )
badrate_plot(
    c.transform(df[[col,'target','month']], labels=True), 
    target='target', x='month', by=col
    )
```


![](https://toad.readthedocs.io/en/latest/_images/tutorial_chinese_20_1.png)
![](https://toad.readthedocs.io/en/latest/_images/tutorial_chinese_20_2.png)
![](https://toad.readthedocs.io/en/latest/_images/tutorial_chinese_20_3.png)

> 当随时间变化而分组的正样本占比增大时则为优，代表了变量在更新的时间区分度更强。分组的线之间没有交叉为优，代表分箱稳定。
> 
{style="note"}


### 5.3 调整分箱

通过 `.update` 方法修改分箱，我们可以通过调整后的分箱再观察分箱效果。

```Python
# iv 值较低，假设我们要 'F' 淡出分出一组来提高 iv
# 调整 var_d5 的分组
rule = {'var_d5':[['O', 'nan'], ['F'], ['M']]}

## 调整分箱
c.update(rule)
## 查看手动分箱稳定性
bin_plot(
    c.transform(df_train_selected[['var_d5','target']], labels=True),
    x='var_d5', target='target'
    )
badrate_plot(
    c.transform(df_oot[['var_d5','target','month']], labels=True),
    target='target', x='month', by='var_d5'
    )
```

![](https://toad.readthedocs.io/en/latest/_images/tutorial_chinese_23_2.png)
![](https://toad.readthedocs.io/en/latest/_images/tutorial_chinese_23_3.png)


## 6. WOE 转化

WOE 转化在分箱调整好之后进行，步骤如下：
1. 用调整好的 Combiner 转化数据： `c.transform(dataframe, labels=False)`，注意这只会转化被分箱的变量
2. 通过 `toad.transform.WOETransformer()` 初始化 WOE Transer
3. 通过 `fit_transform` 训练并输出 woe 转化的数据，这会转化所有变量，包括未被分箱 transform 的列，通过 `exclude` 删去不要 WOE 转化的列，特别是 `target` 字段。参数如下:
   - `target`: 目标列数据（非列名）
   - `exclude`: 不需要被 WOE 转化的变量
4. 根据训练好的 WOE Transer 调用 `transform` 方法转化验证集数据


```Python
# 初始化 WOETransformer
transer = toad.transform.WOETransformer()
# combiner.transform() & transer.fit_transform() 转化训练数据，并去掉 target 列
df_train_woe = transer.fit_transform(
    c.transform(df_train_selected),
    df_train_selected['target'], 
    exclude=drop_cols+['target']
    )
df_oot_woe = transer.transform(c.transform(OOT))
print(df_train_woe.head(3))
```


## 7. 训练模型

Toad 对于模型训练做了高度封装，你可以像

```Python
from sklearn.linear_model import LogisticRegression

lr = LogisticRegression()
lr.fit(df_final_train[cols], df_final_train['target'])

```


## 8. 模型评估

`toad.metrics` 模块用于评估模型的性能，目前提供了 KS、AUC、F1 评估指标，此外我们之前提及的 PSI 也是可以评估预测的稳定性。 

### 8.1 KS 和 AUC

<tabs>
<tab title="评估模型性能">
<code-block lang="python">
<![CDATA[
from toad.metrics import KS, AUC
## 预测训练和隔月的OOT
pred_train = lr.predict_proba(df_final_train[cols])[:, 1]
print("Train KS: ", KS(pred_train, df_final_train['target']))
print("Train AUC: ", AUC(pred_train, df_final_train['target']))
pred_oot5 = lr.predict_proba(
    df_final_oot.loc[df_final_oot.month == '2019-05', cols]
    )[:, 1]
pred_oot6 = lr.predict_proba(
    df_final_oot.loc[df_final_oot.month == '2019-06', cols]
    )[:, 1]
pred_oot7 = lr.predict_proba(
    df_final_oot.loc[df_final_oot.month == '2019-07', cols]
    )[:, 1]
]]>
</code-block>
</tab>
<tab title="输出">
<code-block lang="python">
<![CDATA[
Train KS: 0.3707986228750539
Train AUC: 0.75060723924743
5月 KS: 0.3686687175756087
6月 KS: 0.3495273403486497
7月 KS: 0.3796914199845523
]]>
</code-block>
</tab>
</tabs>


此外，PSI 也可以用于验证分数的稳定性。

<tabs>
<tab title="OOT KS表现">
<code-block lang="python">
<![CDATA[
print("May PSI: ", toad.metrics.PSI(pred_train, pred_oot5))
print("June PSI: ", toad.metrics.PSI(pred_train, pred_oot6))
print("July PSI: ", toad.metrics.PSI(pred_train, pred_oot7))
]]>
</code-block>
</tab>
<tab title="输出">
<code-block lang="python">
<![CDATA[
May PSI: 0.12760761722158315
June PSI: 0.1268648506657109
July PSI: 0.1268648506657109
]]>
</code-block>
</tab>
</tabs>


### 8.2 KS_bucket

`KS_bucket` 输出模型预测分箱后的评判信息，包括每组的分数区间样本量、坏账率 、KS等，具体参数如下:
- `bucket`: 分箱的数量
- `method`: 分箱方法，建议使用 `quantile`（等频）或者 `step`（等步长）。


坏账率(Bad Rate)
: - 组之间的坏账率差距越大越好
: - 可以用于观察是否有跳点
: - 可以用与找最佳切点
: - 可以对比


```Python
toad.metrics.KS_bucket(
    pred_train, 
    df_final_train['target'], 
    bucket=10,
    method='quantile'
    )
```

## 9. 建立评分卡

`toad.ScoreCard` 可以将逻辑回归模型转换为标准评分卡，支持传入逻辑回归参数，参数如下:
- `combiner`: Combiner 实例
- `transer`: WOE Transer
- `pdo`: 默认值位 60
- `rate`: 默认值位 2
- `base_odds`: 默认值为 20
- `base_score`: 基础分，默认为 750
- `card`: 支持传入专家评分卡
- `C`: 
- `kwargs`: 支持传入逻辑回归参数(`sklearn.linear_model.LogisticRegression`)


```Python
card = toad.ScoreCard(
    combiner=c,
    transer=transer,
    # class_weight='balanced',
    # C=0.1,
    # base_score=600,
    # base_odds=35 ,
    # pdo=60,
    # rate=2
)
card.fit(df_final_trains[col], df_final_train['target'])
## 直接使用原始数据进行评分
card.predict(df_train)
## 输出标准评分卡
card.export()
```

> 评分卡在 `fit` 时使用 WOE 转换后的数据来计算最终的分数，分数一旦计算完成，便无需 WOE 值，可以直接使用 原始数据进行评分。
> 
{style="warning"}

### 9.1 评分卡的转换逻辑

Toad 采用标准评分卡转换逻辑进行评分转换，详见[评分转换逻辑](https://toad.readthedocs.io/en/latest/reference.html)。


## 10. 其他功能

此外，Toad 还提供了 `toad.tranform.GBDTransformer` 用于 GBDT 编码，通过 LR+GBDT 建立建模，这一操作主要用于创建更强的特征。


```Python
gbdt_transer = toad.transform.GBDTTransformer()
gbdt_transer.fit(
    df_final_train[cols + ['target']], 'target', 
    n_estimators=10, max_depth=2
    )
df_gbdt_vars = gbdt_transer.transform(df_final_train[cols])
print(df_gbdt_vars.shape)  # (43576, 40)
```


<seealso>
<category ref="ref_docs">
    <a href="https://toad.readthedocs.io/en/stable/tutorial.html">Toad 文档</a>
    <a href="https://toad.readthedocs.io/en/latest/tutorial_chinese.html">Toad 中文文档</a>
</category>
<category ref="ref_github">
    <a href="https://github.com/amphibian-dev/toad">Toad</a>
</category>
<category ref="ref_issues">
    <a href="https://github.com/amphibian-dev/toad/issues/31">Toad#31</a>
</category>
<category ref="ref_hf">
</category>
<category ref="ref_ms">
</category>
</seealso>
