# 函数

<show-structure depth="2"/>

## 1. JSON 函数

### 1.1 json_extract

经常使用 Hive 的同学可能知道，对于 JSON 字段的文本我们可以使用 `get_json_object` 函数进行提取。在 Presto 中对应的函数为`json_extract`，两者的使用方式几乎完全一样。

<tabs>
<tab title="示例SQL">
<code-block lang="sql">
<![CDATA[
SELECT 
    JSON_EXTRACT(dialog, "$.sentences") AS json_text
FROM mydb.test
]]>
</code-block>
</tab>
<tab title="样例数据">
<code-block lang="json">
<![CDATA[
// dialog 字段的样本数据
{   
    "date": "2023-01-01",
    "sentences": [
        {"index": 1, "role": "user", "text": "你好"},
        {"index": 2, "role": "robot", "text": "你好啊"},
    ],
    "extra": {"model": "gpt4", "isVip": "true"}
}
]]>
</code-block>
</tab>
<tab title="输出结果">
<code-block lang="python">
<![CDATA[
[
    {"index": 1, "role": "user", "text": "你好"},
    {"index": 2, "role": "robot", "text": "你好啊"},
]
]]>
</code-block>
</tab>
</tabs>

如果你想好只提取 `sentences` 列表中第一个元素的值，那么可以使用 `JSON_EXTRACT(dialog, "$.sentences[0]")` 这样的语句，`[0]` 替换成 `[1]` 的话则表示提取第二个元素值。

如果想要提取所使用的模型，则可以使用 `JSON_EXTRACT(dialog, "$.extra.model")` 来进行提取。



### 1.2 json_extract_scalar

### 1.3 json_parse

### 1.4 json_format

### 1.5 json_size

### 1.6 json_array_contains

### 1.7 json_array_get

### 1.8 json_array_length

### 1.8 is_json_scalar




