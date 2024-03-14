# Python 字符串

<show-structure for="chapter" depth="2"/>

## 1. f-string
{collapsible="true" default-state="collapsed"}

### 1.1 日期格式化

通常我们打印一个特定格式的日期字符串时，会采用如下做法:

```Python
from datetime import datetime

now = datetime.now()
now_str = now.strftime("%Y-%m-%d")
print(now_str)  # 2023-12-20
```
{ignore-vars="true"}

这里为什么不直接使用 `datetime.now().strftime("%Y-%m-%d")` 这样的形式去表达，是因为我们后续很有可能使用到 `now` 变量来计算时间差，而格式化的日期字符串可能仅仅是为了打印执行到当前代码行的时间戳。
{ignore-vars="true"}

接下来，我们就学习一下如何通过 Python F-String 来简洁地打印日期:

```Python
from datetime import datetime

now = datetime.now()
print(f"{now:%Y-%m-%d}")  # 2023-12-20

# Today is the 20-12-2023
print(f"{now:Today is the %d-%m-%Y}")

# Today is a fine Wednesday of December
print(f"Today is a fine {now:%A} of {now:%B}")

# 20/12 is day number 354 of the year 2023
print(f"{now:%d/%m} is day number {now:%j} of the year {now:%Y}")

# Wed Dec 20 11:44:41 2023
print(f"{now:%c}")
```
{ignore-vars="true"}

可以看到，我们并不需要实例化一个变量来保存日期格式化后的字符串，整个代码就会变得简洁很多。

## 2. 判断字符串是否为数字

Python 中字符串的 `isdigit` 和 `isnumeric` 方法可以判断该字符串是否可以转换为数字。
- `isdigit`: 检查字符串是否只包含数字字符，返回一个布尔值。
- `isnumeric`: 检查字符串是否只包含数字字符，但是可以识别其他数字字符，例如分数等，返回一个布尔值。

<tabs>
<tab title="isdigit">
<code-block lang="python">
<![CDATA[
print("1234".isdigit())   # True
print("123.4".isdigit())  # False
]]>
</code-block>
</tab>
<tab title="isnumeric">
<code-block lang="python">
<![CDATA[
print("1234".isnumeric())   # True
print("123.4".isnumeric())  # False
]]>
</code-block>
</tab>
</tabs>


需要注意的是，这两个方法并不会识别小数或者负数，如果想要识别字符串是否是小数或负数，那么可以用 `try...except...` 来判断。

```Python
def is_number(text: str) -> bool:
    try:
        float(text)
        return True
    except ValueError:
        return False
        

print(is_number("123.1"))   # True
print(is_number("123.1b"))  # False
```



<seealso>
<category ref="ref_docs">
    <a href="https://mathspp.com/blog/twitter-threads/datetime-objects-and-f-strings">Datetime objects and f-strings</a>
</category>
</seealso>