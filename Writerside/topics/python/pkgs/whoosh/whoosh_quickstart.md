# 快速入门

<show-structure depth="2"/>

## 1. 简介

[Whoosh](https://whoosh.readthedocs.io/en/latest/intro.html) 是 Python 中一款轻量级的搜索工具，是由 Matt Chaput 创建。它一开始是一个为 Houdini 3D 动画软件包的在线文档提供简单、快速的搜索服务工具，之后便慢慢成为一个成熟的搜索解决工具并已开源。

Whoosh 纯由 Python 编写而成，是一个灵活的，方便的，轻量级的搜索引擎工具，现在同时支持 Python2、3，其优点如下：
- Whoosh 纯由 Python 编写而成，但很快，只需要 Python 环境即可，不需要编译器
- 默认使用 Okapi BM25F 排序算法，也支持其他排序算法
- 相比于其他搜索引擎，Whoosh 会创建更小的 index 文件
- Whoosh 中的 index 文件编码必须是 unicode
- Whoosh 可以储存任意的 Python 对象

相比于 ElasticSearch 或者 Solr 等成熟的搜索引擎工具，Whoosh 显得更轻便，操作更简单，可以考虑在小型的搜索项目中使用。

## 2. Index & Query

对于熟悉 ES 的人来说，搜索的两个重要的方面为 `mapping` 和 `query`，也就是索引的构建以及查询，背后是复杂的索引储存、Query 解析以及排序算法等。如果你有 ES 方面的经验，那么，对于 Whoosh 是十分容易上手的。

Whoosh 的入门使用主要是 `index` 以及 `query`，搜索引擎的强大功能之一在于它能够提供**全文检索**，这依赖于排序算法，比如 BM25，也依赖于我们怎样储存字段。因此，`index` 作为名词时，是指字段的索引，`index` 作为动词时，是指建立字段的索引。而 `query` 会将我们需要查询的语句，通过排序算法，给出合理的搜索结果。

> 想要了解更多关于 Whoosh 的使用，请参阅 Whoosh 官方文档。

## 3. 使用示例

### 3.1 数据集

这里使用 poem.csv 数据集，数据结构如下所示：

| title | dynasty | poet | content  |
|-------|---------|------|----------|
| 静夜思   | 唐代      | 李白   | 床前明月光... |
| 相思    | 唐代      | 王维   | 红豆生南国... |

### 3.2 字段

根据数据集的特征，这里我们创建 4 个字段：title、dynasty、poet、content，创建代码如下：

```Python
from whoosh.index import create_in
from whoosh.fields import Text, ID, Schema
from jieba.analyse import ChineseAnalyzer

# 创建 schema, stored 为 `True` 表示能够被检索
schema = Schema(
    title=TEXT(stored=True, analyzer=ChineseAnalyzer()),
    dynasty=ID(stored=True),
    poet=ID(stored=True),
    content=TEXT(stored=True, analyzer=ChineseAnalyzer())
)
```

其中，`ID` 只能为一个单元值，不能分割为若干个词，常用于文件路径、URL、日期、分类； `TEXT` 字段为文本内容，建立文本的索引并存储，支持词汇搜索；`Analyzer` 选择 Jieba 中文分词器。

### 3.3 创建索引文件

接下来需要创建索引文件，先利用程序先解析 poem.csv 文件，并将它转化为 `index`，写入到 `index_dir` 目录下。

```Python
import pandas as pd
from whoosh.index import create_in

# 解析poem.csv文件
df_poem = pd.read_csv("poem.csv")
texts = df.to_json(orient="records")

# 存储schema信息至indexdir目录
index_dir = 'index_dir'
if not os.path.exists(index_dir):
    os.mkdir(index_dir)

ix = create_in(index_dir, schema)
# 按照schema定义信息，增加需要建立索引的文档
writer = ix.writer()

for tile, dynasty, poet, content  in texts:
    writer.add_document(
        title=title, dynasty=dynasty, poet=poet, content=content
        )
writer.commit()
```

`index` 创建成功后，会生成 `index_dir` 目录，里面含有上述 poem.csv 数据的各个字段的索引文件。

### 3.4 查询

`index` 创建成功后，我们可以进行查询了，比如我们想要查询 `content` 中含有**明月**的诗句，可以输入以下代码：

```Python
# 创建一个检索器
searcher = ix.searcher()

# 检索 content 中出现`明月`的文档
results = searcher.find("content", "明月")
print(f"一共发现 {len(results)} 份文档")
```



<seealso>
<category ref="ref_docs">
    <a href="https://whoosh.readthedocs.io/en/latest/intro.html">Whoosh 文档</a>
    <a href="https://mp.weixin.qq.com/s/VFk9cSox76KNAYFjCwpKnw">Python轻量级搜索工具——Whoosh</a>
</category>
</seealso>
