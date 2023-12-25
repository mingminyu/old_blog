# Python 字符串

<show-structure for="chapter" depth="2"/>

## 1. f-string
{collapsible="true" default-state="collapsed"}

### 日期格式化

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


<seealso>
<category ref="ref_docs">
    <a href="https://mathspp.com/blog/twitter-threads/datetime-objects-and-f-strings">Datetime objects and f-strings</a>
</category>
</seealso>