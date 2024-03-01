# Python 风格规范🔥

<show-structure depth="3"/>

## 1. 分号

不要在行尾加分号，也不要用分号将两条命令放在同一行。

## 2. 行长度 {collapsible="true" default-state="expanded"}

每行不超过 80 个字符，但是下面这些情况除外：
- 长的导入模块语句
- 注释里的 URL，路径以及其他的一些长标记
- 不便于换行，不包含空格的模块级字符串常量，比如 url 或者路径
- Pylint 禁用注释（例如：`# pylint: disable=invalid-name`）

除非是在 `with` 语句需要三个以上的上下文管理器的情况下，否则不要使用反斜杠连接行。

Python 会将**圆括号、中括号和花括号**中的行隐式的连接起来 , 你可以利用这个特点。如果需要，你可以在表达式外围增加一对额外的圆括号。

```Python
foo_bar(self, width, height, color='黑', design=None, x='foo',
        emphasis=None, highlight=0)

if (width == 0 and height == 0 and
    color == '红' and emphasis == '加粗'):

 (bridge_questions.clarification_on
  .average_airspeed_of.unladen_swallow) = '美国的还是欧洲的?'

 with (
     very_long_first_expression_function() as spam,
     very_long_second_expression_function() as beans,
     third_thing() as eggs,
 ):
   place_order(eggs, beans, spam, beans)
```










<seealso>
<category ref="ref_docs">
    <a href="https://zh-google-styleguide.readthedocs.io/en/latest/google-python-styleguide/python_style_rules">Python 风格规范</a>
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