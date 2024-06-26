# 常见案例

<show-structure depth="3"/>

## 1. 根据标识符提取文件中段落 SQL

我们通常会把跑批任务的所有 SQL 单独写在一个文件中，这些段落 SQL 各自都会包含一个标签，具体示例内容如下:


```SQL
--[sql1]
SELECT * FROM test.student_info;

--[sql2]
CREATE TABLE IF NOT EXISTS test.teacher_info (
    uid INTEGER PRIMARY KEY,
    name TEXT NOT NULL,
    age INTEGER NOT NULL,
    school TEXT NOT NULL
)
;
SELECT * FROM test.teacher_info
;

--[sql3]
SELECT * FROM test.school_info
;

--[sql4]
SELECT * FROM test.school_info2

--[sql5]
```
{collapsed-title="示例SQL" collapsible="true"}

我们希望将标签和 SQL 能被一一对应地提取出来，形式如下，这样在使用脚本执行 SQL 时能有更高的自由度。

```Python
{
    "sql1": "xxx",
    "sql2": "xxx",
    "sql3": "xxx",
    "sql4": "xxx",
    "sql5": "xxx",
}
```

下面我们看下如何定义正则来提取相应的内容:

```Python
import re

pattern = r'--\[(.*?)\]\s*(.*?)(?=--\[|$)'
result = re.findall(pattern, sql, re.DOTALL)
print(result)
```

提取出来的信息如下所示，实际应用中我们还需要对文本做进一步处理。

```Python
[('sql1', 'SELECT * FROM test.student_info;\n\n'),
 ('sql2',
  'CREATE TABLE IF NOT EXISTS test.teacher_info (\n    uid INTEGER PRIMARY KEY,\n    name TEXT NOT NULL,\n    age INTEGER NOT NULL,\n    school TEXT NOT NULL\n)\n;\nSELECT * FROM test.teacher_info\n;\n\n'),
 ('sql3', 'SELECT * FROM test.school_info\n;\n\n'),
 ('sql4', 'SELECT * FROM test.school_info2\n\n'),
 ('sql5', '')]
```

> 使用 PyParsing 也可以对文本内容进行提取，尝试下来提取效果并不好。
>
{style="warning"}


## 2. 移除 SQL 内容中注释

有些时候，我们希望后台日志记录执行的 SQL 并没有什么注释信息，我们依旧可以使用正则来将相关注释移除掉

```Python
import re

content = """
SELECT * FROM test.student_info;
/** abc **/
CREATE TABLE IF NOT EXISTS test.teacher_info (
    uid INTEGER PRIMARY KEY,
    name TEXT NOT NULL,  -- 123
    age INTEGER NOT NULL,
    school TEXT NOT NULL
)
;
SELECT * FROM test.teacher_info
;

--[a]
SELECT * FROM test.teacher_info        

SELECT * FROM test.school_info2 -- 123

--[sql5]
"""

sql = re.sub(r'(/\*.*?\*/|--.*?$)', ' ', content, flags=re.MULTILINE)
```
{collapsible="true"}




