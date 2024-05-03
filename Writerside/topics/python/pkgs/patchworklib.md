# Patchworklib 教程

<show-structure depth="2" />

[patchworklib](https://github.com/ponnhide/patchworklib) 是一个开源的 python 库，可以将多个图表的整合为单一图表。这个库受到了 ggplot2 拼接功能的启发，使得用户可以便捷地通过使用 `/` 和 `|` 符号来对齐 matplotlib 图表。

另外，虽然一些基于 matplotlib 的第三方库（如 plotnine 和 seaborn）能够用简洁的 Python 代码生成精美的图形，但它们中的许多图形并不支持作为 matplotlib 的子图来处理，通常需要手动调整其位置，而 patchworklib 库完美解决了这个问题。通过应用 patchworklib，各种 seaborn 或 plotnine 图表都能作为 matplotlib 的子图进行有效处理。

## 1. 安装

可以直接通过 pip 进行安装：

```Bash
pip install patchworklib
```

## 2. 简单用法

使用 patchworklib ，你可以通过 `|` 和 `/` 运算符来快速、自由地排列的 matplotlib 绘图，如下所示。

```Python
import patchworklib as pw
import seaborn as sns 

# 加载 fmri 数据集
fmri = sns.load_dataset("fmri")
ax1 = pw.Brick(figsize=(3,2))
sns.lineplot(
    x="timepoint", y="signal", hue="region", 
    style="event", data=fmri, ax=ax1
)
ax1.legend(bbox_to_anchor=(1.05, 1.0), loc='upper left')
ax1.set_title("ax1")

# 加载 titanic 数据集
titanic = sns.load_dataset("titanic")
ax2 = pw.Brick(figsize=(1,2))
sns.barplot(
    x="sex", y="survived", hue="class", data=titanic, ax=ax2
    )
ax2.move_legend(new_loc='upper left', bbox_to_anchor=(1.05, 1.0))
ax2.set_title("ax2")

# 使用 | 运算符排列子图
ax12 = ax1 | ax2
ax12.savefig()

# 加载 diamonds 数据集
diamonds = sns.load_dataset("diamonds")
ax3 = pw.Brick(figsize=(6,2))
sns.histplot(
    diamonds, x="price", hue="cut", multiple="stack", 
    palette="light:m_r", edgecolor=".3", linewidth=.5, 
    log_scale=True, ax=ax3
    )
ax3.move_legend(new_loc='upper left', bbox_to_anchor=(1.0, 1.0))
ax3.set_title("ax3")

# 使用 / 和 | 运算符排列子图
(ax3 / (ax1 | ax2)).savefig()
```

<seealso>
    <category ref="ref_docs">
        <a href="https://mp.weixin.qq.com/s/NzP_LWSHABYqGOLWLDwH6A">强大的 Python 库: Patchworklib</a>
        <a href="https://mp.weixin.qq.com/s/LmNUmonlGkNLYvGdFQp7hg">有趣的 Python 库: Patchworklib</a>
    </category>
    <category ref="ref_github">
        <a href="https://github.com/ponnhide/patchworklib">Patchworklib</a>
    </category>
</seealso>
