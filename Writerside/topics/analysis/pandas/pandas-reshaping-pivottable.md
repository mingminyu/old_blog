# 重塑与透视表

<show-structure depth="2"/>

## 3. melt

数据分析中经常会遇到这样一个问题，DataFrame 是某一个字段的枚举值进行统计的，我们希望将部分行字段转换为列字段，方便后续进一步对数据进行透视。

<tabs>
<tab title="代码">
<code-block lang="python">
<![CDATA[
import pandas as pd

df_cheese = pd.DataFrame(
        {
            "first": ["John", "Mary"],
            "last": ["Doe", "Bo"],
            "height": [5.5, 6.0],
            "weight": [130, 150],
        }
    )
df_cheese.melt(id_vars=["first", "last"])
]]>
</code-block>
</tab>
<tab title="输出">
<code-block lang="python">
<![CDATA[
first last variable  value
John  Doe   height    5.5
Mary   Bo   height    6.0
John  Doe   weight  130.0
Mary   Bo   weight  150.0
]]>
</code-block>
</tab>
</tabs>

`melt` 函数还有两个常用参数 `var_name` 和 `ignore_index`，后者大家很熟悉了，这里不再赘述，如果指定了 `var_name` 的话，则 `variable` 字段会变重命名为 `var_name` 的值。




<seealso>
<category ref="ref_docs">
    <a href="https://mp.weixin.qq.com/s/--A8VhIo0mlwnIDXDQEmxw">melt函数巧妙解决多列数据判别问题</a>
    <a href="https://pandas.pydata.org/docs/user_guide/reshaping.html">Reshaping and pivot tables</a>
</category>
<category ref="ref_github"></category>
<category ref="ref_issues"></category>
<category ref="ref_hf"></category>
<category ref="ref_ms"></category>
</seealso>