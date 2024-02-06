# 核心函数

<show-structure depth="2"/>


## 5. 描述性统计

### 5.1 数据表摘要


### 5.2 最大值/最小值的行索引

在分析表数据时，有时间我们需要找到最大值或者最小值所在行，从而对该行数据做进一步的处理。Pandas 中提供了 `idxmax` 和 `idxmin` 函数定位最大值和最小的行索引。

```Python
import numpy as np
import pandas as pd


df = pd.DataFrame(
        {
            "width": [1, 1, 3, 5, np.nan], 
            "height": [10, 8, 10, np.nan, 5], 
        }, 
    )

# `width` 最小值所在行索引
print(df["width"].idxmin())

# `height` 最大值所在行数据
print(df.loc[df["height"].idxmax()]) 
```

> Pandas Series 也提供了 `argmax` 和 `argmin` 函数，达到的效果与 `idxmax` 和 `idmin` 是一样的。
{style="note"}


> 当然，如果我们想要返回最大值所有的所在行数据的话，使用 `idxmax` 和 `idxmin` 并不能满足，得需要 `rank` 这一类的函数，或者找到最大值（或者最小值）后使用 `df.loc[df["height"] == max_h]` 来筛选出数据。如果想找出次大值这种操作，就只能依赖于排序的功能了。
{style="warning"}







## 6. 函数应用

### 6.1 在每个元素上应用函数

#### map

#### applymap

#### case_when

数分小伙伴们都知道，SQL中 的 `case when` 语句非常好用，尤其在加工变量的时候，可以按照指定的条件的进行赋值，并且结合其他嵌套用法还可以实现非常强大的功能。

同样作为数据分析常用工具之一，Pandas 中却没有像 case when 这样的语句，一直以来收到很多朋友吐槽，这样一个常用的功能竟然没有？一般通过使用 `np.where`、`where`、`mask`、`map`、`apply` 等其他方式来实现 case when 的效果。好消息是，最近 Pandas2.2.0 稳定版本发布了，其中一个新功能就是增加了 `case_when` 方法，可以说这个一直被大家诟病的方法终于补齐了。

`case_when` 属于 Series 对象的方法，DataFrame 对象无法使用。如果判断条件为真(True) 则替换数据，反之保持原值不变，这有点类似于升级版的 `where/mask`。`case_when` 只有一个参数 `caselist`，是一个元组构成的列表，元组内包含判断条件和想要替换的值。具体形式如下：

```Python
caselist = [
  (condition1, replacement1),
  (condition2, replacement2),
  ...
  ]
```

我们先来看一个简单的案例：

```Python
import pandas as pd

df = pd.DataFrame(
    dict(
        engligh=[70, 90, 80, 85, 65, 92], 
        math=[90, 84, 69, 73, 98, 83],
        physic=[84, 58, 74, 93, 87, 86]
        )
    )
df['score'] = df.sum(axis=1)
df['rank'] = df["score"].case_when(
    caselist=[
        ((df["score"] > 220) & (df["score"] <= 230), '5'),
        ((df["score"] > 230) & (df["score"] <= 240), '4'),
        ((df["score"] > 240) & (df["score"] <= 250), '3'),
        ((df["score"] > 250) & (df["score"] <= 260), '2'),
        ((df["score"] > 260) & (df["score"] <= 270), '1'),
    ]
)
```
{collapsible="true" default-state="expanded" collapsed-title="case_when 案例一"}

这个案例并不能很好地反应出 `case_when` 的优势，实际上在这个示例中我们使用 `qcut` 或者 `cut` 会更方便，使用 `map` 实现也很不算麻烦。

如果我们想对不同区间地元素做差异化地运算，那么 `qcut` 或者 `cut` 就完全不可用了，常规的情况下只能使用 `map` 来转换，我们看一下 `case_when` 函数中怎么操作。

```Python
# 使用 between 稍微简化下代码
df["score_new"] = df["score"].case_when(
    caselist=[
        (df["score"].between(220, 230), lambda x: x + 5),
        (df["score"].between(230, 240), lambda x: x + 4),
        (df["score"].between(240, 250), lambda x: x + 3),
        (df["score"].between(250, 260), lambda x: x + 2),
        (df["score"].between(260, 270), lambda x: x + 1),
    ]
)
```
{collapsible="true" default-state="expanded" collapsed-title="case_when 案例二"}

我们也可以在条件表达式中使用 `lambda` 表达式，这样写的好处更多用在函数的封装上。

```Python
df["score_new"] = df["score"].case_when(
    caselist=[
        (lambda s: (s > 220) & (s <= 230), '5'),
        (lambda s: (s > 230) & (s <= 240), '4'),
        (lambda s: (s > 240) & (s <= 250), '3'),
        (lambda s: (s > 250) & (s <= 260), '2'),
        (lambda s: (s > 260) & (s <= 270), '1'),
    ]
)
```
{collapsible="true" default-state="expanded" collapsed-title="case_when 案例三"}


之所以说 `case_when` 很有必要，是因为它的条件和替换值可以是非固定值，我们可以结合 `lambda` 表达式，做到更复杂的操作，但是代码却依旧简洁。

`case_when` 只实现区域内的变量加工，其输出结果也可以与其他函数方法结合，产生更多强大的功能。比如可以将以上全部变量加工过程通过链式的方式更优雅的实现，结合 `assign` 的使用一行代码可完成全部。

```Python
import pandas as pd

df = pd.DataFrame(
    dict(
        english=[70, 90, 80, 85, 65, 92], 
        math=[90, 84, 69, 73, 98, 83],
        physic=[84, 58, 74, 93, 87, 86]
        )
    )

(
    df.assign(score=lambda x: x.sum(axis=1))
      .assign(rank=lambda x: x.score.case_when(
          caselist=[
              ((x.score > 220) & (x.score <= 230), '5'),
              ((x.score > 230) & (x.score <= 240), '4'),
              ((x.score > 240) & (x.score <= 250), '3'),
              ((x.score > 250) & (x.score <= 260), '2'),
              ((x.score > 260) & (x.score <= 270), '1')]))
      .assign(score_new=lambda x: x.score.case_when(
          caselist=[
              (df.english <= 70, lambda x: x + 5),
              ((df.english > 70) & (df.english <= 80), lambda x: x + 3),
              ((df.english > 80) & (df.english <= 90), lambda x: x + 2),
              (df.english > 90, lambda x: x + 1)])
            )
)
```
{collapsible="true" default-state="expanded" collapsed-title="case_when 案例四"}




<seealso>
<category ref="ref_docs">
    <a href="https://mp.weixin.qq.com/s/KTpOU-Fd2kxRT3KZ3Yh8TQ">pandas终于有case_when方法了</a>
    <a href="https://pandas.pydata.org/docs/user_guide/basics.html">Essential basic functionality</a>
    <a href="https://pandas.pydata.org/docs/reference/api/pandas.Series.case_when.html">case_when API</a>
</category>
<category ref="ref_github"></category>
<category ref="ref_issues"></category>
<category ref="ref_hf"></category>
<category ref="ref_ms"></category>
</seealso>