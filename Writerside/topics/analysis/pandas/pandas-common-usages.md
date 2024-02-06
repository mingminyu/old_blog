# 常用操作

<show-structure depth="2"/>

## 1. 使用 truncate 截断数据

在 Pandas 中，`truncate` 函数用于截断 DataFrame 或 Series 中的数据，它可以按指定位置或指定值进行截断。

`truncate` 函数有两个常用参数：
- `before`: 截断前要保留的第一个索引值（包括该值）
- `after`: 截断后要保留的最后一个索引值（包括该值）



<tabs>
<tab title="代码">
<code-block lang="python">
<![CDATA[
import pandas as pd

data = {'A': [1, 2, 3, 4, 5], 'B': [10, 20, 30, 40, 50]}  
df = pd.DataFrame(data)

# 截断 DataFrame，保留 2 到 4 行
df_truncated = df.truncate(before=1, after=3)  
print(df_truncated)
]]>
</code-block>
</tab>
<tab title="输出">
<code-block lang="python">
<![CDATA[
A   B
2  20
3  30
4  40
]]>
</code-block>
</tab>
</tabs>

## 2. 使用 take 选择数据

`take` 函数是 Pandas 库中的一个函数，用于从数组或 Series 中选择元素。该函数接受两个参数：
- `indexer`: ：用于指定需要选择元素的索引 
- `axis`: 沿着哪个轴进行选择，0 表示按行选择，1 表示按列选择。

<tabs>
<tab title="代码">
<code-block lang="python">
<![CDATA[
import pandas as pd  

data = {'A': [1, 2, 3], 'B': [4, 5, 6], 'C': [7, 8, 9]}  
df = pd.DataFrame(data)
rows = df.take([0, 2])  
print(rows)
]]>
</code-block>
</tab>
<tab title="输出">
<code-block lang="python">
<![CDATA[
A  B  C
1  4  7
3  6  9
]]>
</code-block>
</tab>
</tabs>

## 3. 使用 add_prefix(add_suffix) 给列名添加前缀(后缀)

`add_prefix` 函数是 Pandas 库中的一个函数，用于给 DataFrame 或 Series 中的列添加前缀。该函数接受两个参数:
- `prefix`: 需要添加的前缀字符串
- `axis`: 指定要沿着哪个轴添加前缀。0 表示按行添加，1 表示按列添加

<tabs>
<tab title="代码">
<code-block lang="python">
<![CDATA[
import pandas as pd

data = {'A': [1, 2, 3], 'B': [4, 5, 6], 'C': [7, 8, 9]}  
df = pd.DataFrame(data)
print(df.add_prefix('Col'))
]]>
</code-block>
</tab>
<tab title="输出">
<code-block lang="python">
<![CDATA[
ColA  ColB  ColC
1     4     7
2     5     8
3     6     9
]]>
</code-block>
</tab>
</tabs>

> `add_suffix` 是给列名添加后缀，使用方式与 `add_prefix` 几乎一致。
{style="note"}

## 4. 使用 to_markdown 输出展示

`to_markdown` 函数是 Pandas 库中的一个函数，用于将 DataFrame 对象转换为 Markdown 格式的字符串。

> Markdown是一种轻量级的标记语言，常用于编写文档或笔记。

使用`to_markdown` 函数可以将 DataFrame 中的数据转换为 Markdown 格式的表格，方便在文档或笔记中引用和展示，主要有以下常用参数：
- `index`: 是否将索引列包含在输出的 Markdown 字符串中
- `columns`: 是否将列标签包含在输出的 Markdown 字符串中
- `header`: 是否将列名作为 Markdown 表头的第一行
- `index_names`: 是否将索引名称包含在输出的 Markdown 字符串中
- `col_space`: 设置 Markdown 表格中每一列的宽度
- `table_id`: 为输出的 Markdown 表格添加一个 ID 属性
- `encoding`: 指定输出的 Markdown 字符串的编码方式


<tabs>
<tab title="代码">
<code-block lang="python">
<![CDATA[
import pandas as pd

data = {
    'Name': ['Alice', 'Bob', 'Charlie'],  
    'Age': [25, 30, 35],  
    'Gender': ['F', 'M', 'M']
}  
df = pd.DataFrame(data)
markdown_str = df.to_markdown()
print(markdown_str)
]]>
</code-block>
</tab>
<tab title="输出">
<code-block lang="python">
<![CDATA[
|    | Name    |   Age | Gender   |
|---:|:--------|------:|:---------|
|  0 | Alice   |    25 | F        |
|  1 | Bob     |    30 | M        |
|  2 | Charlie |    35 | M        |
]]>
</code-block>
</tab>
</tabs>

## 5. 使用 ffill(bfill) 向前(后)填充

`ffill` 函数是 Pandas 库中的一个函数，用于进行**前向填充**（forward fill）操作。该函数会根据相邻行的值来填充缺失值，从第一行开始向前填充，直到最后一行。这种填充方式通常被称为"前向填充"或"向前填充"。

> Pandas 中还有一个后向填充函数 `bfill`，原理和使用方向上与 `ffill` 几乎一致。

使用 `ffill` 函数时，需要将待填充的 DataFrame 或 Series 对象作为参数传入。该函数会返回一个新的 DataFrame 或 Series 对象，其中缺失值被填充为相邻行中最近的非缺失值。

<tabs>
<tab title="代码">
<code-block lang="python">
<![CDATA[
import numpy as np  
import pandas as pd  

df = pd.DataFrame(
        {'A': [1, 2, np.nan, 4], 'B': [5, np.nan, np.nan, 8]}
)
df_filled = df.ffill()   
print(df_filled)
]]>
</code-block>
</tab>
<tab title="输出">
<code-block lang="python">
<![CDATA[
A    B
1.0  5.0
2.0  5.0
2.0  5.0
4.0  8.0
]]>
</code-block>
</tab>
</tabs>

## 6. 使用 set_option 将宽表转为原始记录表

`set_option` 是 Pandas 库中的一个函数，用于设置 Pandas 的全局选项。通过这个函数，你可以调整 Pandas 库在处理数据时的各种行为和表现（如显示行数、列数等）。

Pandas 提供了大量的全局选项，用于控制其行为。这些选项可以帮助你调整 Pandas 的性能、内存使用、输出格式以及许多其他方面。`pd.set_option` 函数使你能够轻松地设置这些选项，从而定制 Pandas 的运行方式。

```Python
import pandas as pd  
  
pd.set_option('display.max_columns', 50)
pd.set_option('display.max_rows', 50)
```








<seealso>
<category ref="ref_docs">
    <a href="https://mp.weixin.qq.com/s/CXpgWkxarO6TQAEBHYHQ_A">Pandas 骚操作</a>
</category>
<category ref="ref_github"></category>
<category ref="ref_issues"></category>
<category ref="ref_hf"></category>
<category ref="ref_ms"></category>
</seealso>