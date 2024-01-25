# FlashText 教程

<show-structure depth="2"/>

文本搜索与替换是文本处理中常见的任务，无论是在文本分析、数据清洗还是信息提取方面，都需要有效的工具来处理文本数据。FlashText 是一个非常强大的文本搜索和替换库，它提供了高效的方式来查找大量文本中的关键词并进行替换。

> 这篇微信公众号[文章](https://mp.weixin.qq.com/s/owv5FlZpeFx8lAxyl3KSdA)内容已经过时了，不知道是摘录的，很多内容都是有问题的。
> 
> 首先 FlashText 并不能通过建立关键词的**正则匹配规则**来进行搜索，其次也没有使用 `whole_word` 参数来设定全词和部分匹配的功能。
> 
{style="warning"}

## 1. 特性

FlashText 具备以下特点：
- **高性能**：可快速处理大规模文本数据，适用于大数据分析和处理任务
- **简单易用**：简洁的 API，使用户能够轻松地执行文本搜索和替换操作，无需复杂的正则表达式
- **多关键词匹配**：支持同时匹配多个关键词，可以一次性查找多个关键词的出现

## 2. 安装

安装非常简单，直接使用 pip 来安装 flashtext：

```Bash
pip install flashtext
```

## 3. 基础使用

### 3.1 关键词搜索与替换

`add_keyword` 方法可以添加搜索关键词以及对应的替换词，`replace_keywords` 方法则可以在指定文本中去搜索到这些关键词进行替换。

```Python
from flashtext import KeywordProcessor

# 添加搜索关键词以及替词
keyword_processor = KeywordProcessor()
keyword_processor.add_keyword("Python", "Python3")
keyword_processor.add_keyword("flashtext", "text search")

# 在文本中搜索并替换
text = "Python is a popular programming language. flashtext is a fast text search library."
result = keyword_processor.replace_keywords(text)
print(result)
```

> 使用 `add_keywords_from_dict` 和 `add_keywords_from_list` 方法可以一次性添加多个关键词和替换词

### 3.2 提取替换的关键词

使用 `extract_keywords` 来获取所替换的关键词。


### 3.3 移除关键词

使用 `remove_keyword`、`remove_keywords_from_dict` 以及 `remove_keywords_from_list` 来获搜索关键词。


<seealso>
<category ref="ref_docs">
    <a href="https://mp.weixin.qq.com/s/owv5FlZpeFx8lAxyl3KSdA">flashtext 一个超酷的 Python 库</a>
    <a href="https://flashtext.readthedocs.io/en/latest">FlashText 文档</a>
</category>
<category ref="ref_github">
    <a href="https://github.com/vi3k6i5/flashtext">FlashText Github</a>
</category>
<category ref="ref_issues"></category>
<category ref="ref_hf"></category>
<category ref="ref_ms"></category>
</seealso>