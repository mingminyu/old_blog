# 合并、关联、拼接和对比

<show-structure depth="2"/>

## 1. 拼接(concat) {collapsible="true" default-state="expanded"}


## 2. 合并(merge) {collapsible="true" default-state="expanded"}


## 3. 关联(join) {collapsible="true" default-state="expanded"}

### 3.1 join


### 3.2 combine_first

`combine_first` 是 pandas 中的一个函数，它可以将两个 DataFrame 对象按照索引进行合并，用一个对象中的非空值填充另一个对象中的空值。这个函数非常适合处理两个数据集有部分重叠和缺失的情况，可以实现数据的补全和更新。

```Python
df.combine_first(df_other)
```

`df_other` 是另一个 DataFrame 对象，用于和调用函数的对象进行合并。函数的返回值是一个新的 DataFrame 对象，它的行索引和列索引是两个对象的并集，它的值是按照以下规则确定的：
- 如果 `df` 中的值非空，则保留该值
- 如果 `df` 中的值为空，则使用 `df_other` 对象中的值填充

下面我们来看下示例

<tabs>
<tab title="combine_first">
<code-block lang="python">
<![CDATA[
import numpy as np
import pandas as pd

df1 = pd.DataFrame({'A': [1, np.nan, 3], 'B': [4, 5, np.nan]})
df2 = pd.DataFrame({'B': [np.nan, 6, 7], 'C': [np.nan, 2, np.nan], })

print(df1.combine_first(df2))
]]>
</code-block>
</tab>
<tab title="输出">
<code-block lang="python">
<![CDATA[
A    B    C
1.0  4.0  NaN
NaN  5.0  2.0
3.0  7.0  NaN
]]>
</code-block>
</tab>
</tabs>

总结一下，如果 `df` 和 `df_other` 中存在相同的行索引和列索引，那么会使用 `df_other` 来填充 `df` 的非空值；如果不存在同样的行索引和列索引，那么 `combine_first` 的效果相当于按行拼接的 `concat` 操作。


## 4. 对比(compare) {collapsible="true" default-state="expanded"}




<seealso>
<category ref="ref_docs">
    <a href="https://mp.weixin.qq.com/s/as1wGsJ9iGeP6a64cs-PmA">combine_first 全知道</a>
    <a href="https://pandas.pydata.org/docs/user_guide/merging.html">Merge, join, concatenate and compare</a>
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