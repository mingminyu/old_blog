# Jupyter 插件推荐

<show-structure depth="2"/>

Jupyter 有很多非常优秀的第三方插件，学习并充分利用好这些插件能节省我们大量的代码量书写，让我们更关注数据本身的表现以及核心逻辑的实现。

本篇文章就 Jupyter 中非常好用的插件做一个介绍，以下常用的插件列表。

| 插件           | 描述         |
|--------------|------------|
| AutoProfiler | 数据可视化+探索分析 |
| PivotTableJS | 数据可视化+探索分析 |
| PyGWalker    | 数据可视化+探索分析 |
| QGrid        | 数据可视化      |
| ITables      | 数据可视化      |
| nbextensions | 插件集合       |


## 1. AutoProfiler

AutoProfiler 是一个开源的 DataFrame 分析工具，它专为 Jupyter Notebook 设计。当我们在 Jupyter Notebook 中更改或创建 DataFrame 时，AutoProfiler 都会自动读取当前内存中 DataFrame 并进行分析，而无需手动编写代码或调用其他分析工具，为我们提供关于**数据列分布、样本数据以及统计信息**。

### 1.1 安装

AutoProfiler 安装很简单，直接使用 `pip` 进行安装即可：

```Bash
pip install -U digautoprofiler
```

### 1.2 使用

AutoProfiler 是作为插件形式在 Jupyter Notebook 进行使用的，我们并不需要在代码中显式地进行调用。

```Python
import pandas as pd


df_housing = pd.read_csv("./df_housing_sample.csv")
# 将df_housing数据帧中的"date"列转换为日期格式
df_housing["date"] = pd.to_datetime(df_housing["date"], format="%Y%m%d")
expensive_rents = df_housing[df_housing["price"] > 5000]
```
{ignore-vars="true"}


我们只需要点击侧边栏插件，就可以直接对 DataFrame 进行可视化预览

<video src="https://user-images.githubusercontent.com/13400543/199877605-ba50f9c8-87e5-46c9-8207-1c6496bb3b18.mov"/>

### 1.3 界面预览

![示例图](https://raw.githubusercontent.com/cmudig/AutoProfiler/main/.github/screenshots/profiler_sc.png)

接下来，我们对可视化图做一下解释，让大家能更清楚地认知到图所表示的含义，针对 DataFrame 不同的数据类型（category、datetime、int、float）所展示的默认图表是不一样的。

#### 数值类型

对于 `int` 和 `float` 类型的数据列，默认展示的是柱形图，AutoProfiler 会自动对数据进行分箱，不仅会展示出其分布情况，图中还会展示出不同分位数下对应的具体数值。

![int类型分布图](%myimgs%/autoprofiler_example_1.png?raw=true)

![float类型分布图](%myimgs%/autoprofiler_example_5.png?raw=true)

> 点击对应的柱，会默认在 notebook 中插入相关 DataFrame 筛选代码，我们可以进一步查看具体的数据表现。

#### 时间类型

时间类型是指时间戳的数据列格式，AutoProfiler 也会对时间列进行分箱，并以折线图进行展示。我们可以使用 **Ctrl+鼠标坐标+拖拽** 的方式来查看局部信息（点击右上角 clear zoom 恢复原始状态）。

![时间类型分布图](%myimgs%/autoprofiler_example_2.png?raw=true)

> 时间类型的折线图是不能通过点击自动生成筛选代码的，这一点和柱状图是不一样的。
{style="warning"}

#### 类别类型

类别类型是以数据表+色阶图的形式进行展示的，最上面的 `[16]` 表示唯一值的数量，下面则是不同类型值的数量和占比，点击对应的行也会自动插入筛选代码，并且能看到一些数据汇总信息。

![类别类型分布图](%myimgs%/autoprofiler_example_4.png?raw=true)


#### 添加图表

我们还可以添加自定义图表，需要注意的是，Y 轴必须是数值类型，当所选择的 X 轴是 category 类型的话，那么所展示的就是条形图；如果 X 轴是时间类型，则展示为折线图。

![X轴为日期](%myimgs%/autoprofiler_example_6.png?raw=true)

![X轴为类别](%myimgs%/autoprofiler_example_7.png?raw=true)


### 1.4 总结

AutoProfiler 是一款非常优秀且颜值爆表的可视化工具，但是实际上这类 Jupyter 可视化工具有很多，但都会存在一个问题，就是在数据量较大情况下可能表现不佳，有可能导致 Notebook 内核挂掉，所以如果所加载的数据较大时，请谨慎使用。此外，个人觉得如果能通过简单的代码，手动添加想要可视化的 DataFrame，在通用性上会好很多，本质上我们也不需要对每个 DataFrame 都进行可视化。

## 2. PivotTableJS

PivotTableJS 是一个通过 IPython widgets 集成到 Python中 的 JavaScript 库，允许用户直接从 DataFrame 数据创建交互式和灵活的汇总报表。通过 PivotTableJS 可以进行高效、清晰的数据分析和表示，帮助将数据从 Pandas DataFrame 转换为易于观察的交互式数据透视表。


PivotTableJS 提供了以下数据展示形式：
- Table / Table BarChart
- HeatMap / Row HeatMap / Col HeatMap
- Horizontal BarChart / Horizontal Stacked BarChart 
- BarChart / Stacked BarChart 
- Line Chart
- Area Chart
- Scatter Chart
- TSV Export

### 2.1 安装

直接使用 pip 进行安装 pivottablejs

```Bash
pip install pivottablejs
```

### 2.2 使用

`pivot_ui` 函数可以自动从 DataFrame 生成交互式用户界面，使用户可以简单地修改，检查聚合项，并快速轻松地更改数据结构。

```Python
import pandas as pd
from pivottablejs import pivot_ui

df_housing = pd.read_csv("./house_sample.csv")
# 将df_housing数据帧中的"date"列转换为日期格式
df_housing["date"] = pd.to_datetime(df_housing["date"], format="%Y%m%d")
df_housing = df_housing[df_housing.columns[:-1]]

pivot_ui(df_housing)
```
{ignore-vars="true"}

![PivotTableJS](%myimgs%/pivottablejs_example_1.png?raw=true)

此外，`pivot_ui` 函数还提供一些常用参数：
- `rows`: 图表的维度列
- `cols`: 图表的度量列
- `outfile_path`: 导出 HTML 到 Jupyter 服务器的路径
- `url`: 通过此 URL 访问 `outfile_path` 的 HTML 文件


### 2.3 优点与不足

PivotTableJS 的优点与缺点都非常明显，先来说说它的优点：
- 图表种类丰富
- 操作比较简单
- 部分图表可动态交互

当然，PivotTableJS 也存在很多不足： 
- 图表的美观度不够
- 对数据列不会进行自动分箱 
- 最好不要将数值列拖到维度上进行展示，Notebook 可能会卡死 
- 部分图标并不能动态点击
- 必须要在联网的状态下才可用
- 对于文本类字段，一些异常值会导致所有图表都无法显示

## 3. PyGWalker

PyGWalker 是个在 Jupyter Notebook 环境中运行的可视化探索式分析工具，仅一条命令即可生成一个可交互的图形界面，以类似 Tableau/PowerBI 的方式，通过拖拽字段进行数据分析。

过去在 Python 中进行数据可视化分析时，经常需要查询大量的可视化类的代码，并编写胶水代码将其应用在数据集上。PyGWalker 的目标是通过一行代码，将数据集转化为一个可视化分析工具，只需拖拉拽即可生成图表，从而减少数据分析师在数据可视化上的时间成本。

### 3.1 安装

```Bash
pip install pygwalker
```

### 3.2 基础使用

PyGWalker 提供了 Bar/ Line/ Area/ Trail/ Scatter/ Tick/ Rectangle/ Arc/ Text/ Box/ Table 等可视化图表。

此外，PyGWalker 也可以和 Streamlit 结合，做到更好的 Web 可视化，可参考此案例 。
- [PyGWalker + streamlit for Bike sharing dataset](https://pygwalker-in-app-dngxb2r82ho2zqct244v7b.streamlit.app)
- [Earthquake Dashboard](https://earthquake-dashboard-pygwalker.streamlit.app)


```Python
import pandas as pd
import pygwalker as pyg

df_housing = pd.read_csv("./house_sample.csv")
df_housing["date"] = pd.to_datetime(df_housing["date"], format="%Y%m%d")
df_housing = df_housing[df_housing.columns[:-1]]

walker = pyg.walk(df_housing)
```
{ignore-vars="true"}

![PyGWalker](%myimgs%/pygwalker_example_1.png?raw=true)

PyGWalker 提供 `Data` 和 `Visualization` 两个选项，操作上和 Tableau 非常类似，
- `Data`: 以分页形式展示数据
- `Visualization`: 创建可视化图表，并且我们可以建立多个 Sheet 来展示不同的图标。

> PyGWalker 并不能像 Tableau 一样可以建立报表看板。
{style="warning"}

### 3.3 参数解释

| 参数                   | 描述                                                |
|----------------------|---------------------------------------------------|
| dataset              | 数据集                                               |
| gid                  | GraphicWalker 的 div 元素 ID                         |
| env                  | PyGWalker 所使用的环境，默认是 JupyterWidget，其他可选值: Jupyter |
| fieldSpecs           | 指定字段名称                                            |
| hideDataSourceConfig | 隐藏数据导入和导出的按钮                                      |
| themeKey             | 主题类型: vega / g2                                   |
| dark                 | 自动探索系统主题: media / light / dark                    |
| return_html          | 返回 HTML                                           |
| spec                 | 图表配置数据文件                                          |
| use_preview          | 是否启用预览                                            |
| store_chart_data     | 是否保存图表到本地                                         |
| use_kernel_calc      | 是否允许进行数据计算                                        |

## 4. QGrid

除了 PyGWalker 之外，QGrid 也是一个很好的工具，它可以很容易地将 DataFrame 架转换为视觉上直观的交互式数据表。

> 实验证明，无论是 Jupyter Notebook 还是 JupyterLab，QGrid 都无法正常使用，如果要使用的话，建议多关注下官方仓库的更新。
{style="warning"}

### 4.1 安装

对于 Jupyter Notebook 而言，使用如下命令进行安装:

```Bash
pip install qgrid

jupyter nbextension enable --py --sys-prefix qgrid
```


对于 JupyterLab 而言，`qgrid` 的安装需要使用 `jupyter labextension install` 进行安装:

```Bash
jupyter labextension install qgrid2
```


### 4.2 基础使用

通过 `show_grid` 函数来可视化 DataFrame，目前在 JupyterLab 中并没有办法进行使用。

```Python
import qgrid
import pandas as pd

df_housing = pd.read_csv("./house_sample.csv")
df_housing["date"] = pd.to_datetime(df_housing["date"], format="%Y%m%d")
df_housing = df_housing[df_housing.columns[:-1]]
df_housing_qgrid = qgrid.show_grid(df_housing, show_toolbar=True)
df_housing_qgrid
```
{ignore-vars="true"}


### 4.3 界面预览


![QGrid 预览](https://github.com/quantopian/qgrid/blob/master/docs/images/filtering_demo.gif?raw=true)


除了可以基础的预览数据外，QGrid 对于每个字段多可以进行筛选，并且我们可以直接在数据集上添加和删除数据。 


## 5. ITables

ITables 和 QGrid 包对于快速查看数据模式是必要的。然而，如果我们想要进一步理解数据并进行数据转换，它们的特征是不够的。因此，在获得更复杂的见解的情况下，使用 PivotTableJS 和 PyGWalker 可能更合适。

### 5.1 安装

直接使用 pip 就可以安装:

```Bash
pip install itables
```

### 5.2 基础使用

在 Jupyter 中使用 ITables 需要引入 `init_notebook_mode` 和 `show` 函数，

```Python
import pandas as pd
from itables import init_notebook_mode, show

init_notebook_mode(all_interactive=False)

df_housing = pd.read_csv("./house_sample.csv")
df_housing["date"] = pd.to_datetime(df_housing["date"], format="%Y%m%d")
df_housing = df_housing[df_housing.columns[:-1]]
show(df_housing)
```
{ignore-vars="true"}

### 5.3 界面预览

![ITables 界面预览](https://github.com/mwouts/itables/blob/main/docs/df_example.png?raw=true)

ITables 提供了**数据分页预览、搜索、字段排序**的功能，但是没有字段筛选功能。

## 6. nbextensions

nbextensions 是 Jupyter 一款非常著名的插件，它将一系列 js 脚本嵌入到 Jupyter 中，增强 Jupyter 的交互式体验，可以让你的 Jupyter 变得非常强大。

### 6.1 安装


```Bash
pip install jupyter_contrib_nbextensions
jupyter contrib nbextension install --user
jupyter nbextension enable codefolding/main
```

安装完成后有可能在当前 Notebook 上并不能看到 NBExtensions 选项，需要重启你的 Jupyter Notebook，打开 http://localhost:8888/nbextensions 就可以看到 nbextensions 插件的配置页面了。


## Jupytext


## StickyLand




<seealso>
<category ref="ref_docs">
    <a href="https://mp.weixin.qq.com/s/Ff8lqk89zOz44-88Uw4ayQ">一款无代码实时自动分析Pandas DataFrame的工具</a>
    <a href="https://mp.weixin.qq.com/s/KIZOZrpRgpC2ypByJi0KnQ">必看这 4 个Pandas神器</a>
    <a href="https://mp.weixin.qq.com/s/j57blXwEc05ixaZMaBUzbQ">10 个 Python 超实用的机器学习库</a>
    <a href="https://mp.weixin.qq.com/s/m9LX9ymL6_zBElvYc9nDPg">一个有趣且实用的Jupyter插件: StickyLand</a>
</category>
<category ref="ref_github">
    <a href="https://github.com/cmudig/AutoProfiler">AutoProfiler</a>
    <a href="https://github.com/nicolaskruchten/jupyter_pivottablejs">PivotTableJS</a>
    <a href="https://github.com/Kanaries/pygwalker">PyGWalker</a>
    <a href="https://github.com/quantopian/qgrid">QGrid</a>
    <a href="https://github.com/mwouts/itables">ITables</a>
    <a href="https://github.com/mwouts/jupytext">Jupytext</a>
</category>
<category ref="ref_issues"></category>
<category ref="ref_hf"></category>
<category ref="ref_ms"></category>
</seealso>