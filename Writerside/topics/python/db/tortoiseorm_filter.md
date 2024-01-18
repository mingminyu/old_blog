# 数据筛选

<show-structure depth="2"/>


## 1. 按时间范围筛选

以下是一个 ChatInfo 的数据模型，我们想要根据时间字段 `ts` 的约束条件去筛选数据。

```Python
from tortoise import fields 
from tortoise.models import Model


class ChatInfo(Model):
    title = fields.CharField(max_length=255)
    ts = fields.DatetimeField()
```

### 1.1 筛选近 30 天数据

TortoiseORM 每个字段都会有一些衍生字段可用于筛选，这一点用过 DjangoORM 的同学肯定很熟悉，想要筛选近 30 天的数据，可以借助于 `ts__gte` 字段来进行筛选，`__gte` 表示大于等于，具体代码如下：

```Python
from datetime import datetime, timedelta


dt_p30 = datetime.now() - timedelta(days=30)
chat_infos_p30 = await ChatInfo.filter(ts__gte=dt_p30).all()
```

> 这里代码只是用于示例，`await` 方法需要再 `async` 函数中才能调用。
{style="warning"}

此外，除了有 `__gte` 字段，还有其他字段可用使用：
- `__lte`: 小于等于
- `__gt`: 大于
- `__lt`: 小于
- `__exact`: 等于某个具体日期
- `__range`: 筛选时间窗口


### 1.2 筛选指定时间窗口

如果我们想要筛选某个时间窗口，这里我们以 2024/1/1 ~ 2024/1/10 作为筛选的时间窗口，那么就可以使用 `__range` 字段，示例代码如下：

```Python
import datetime

dt1 = datetime.date(2024, 1, 1)
dt2 = datetime.date(2024, 1, 10)
chat_infos_period = await ChatInfo.filter(ts__range=(dt1, dt2)).all()
```


