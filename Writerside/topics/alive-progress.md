# AliveProgress 教程

<show-structure depth="2"/>

[alive-progress](https://github.com/rsalmei/alive-progress) 是一个 Python 库，用于在控制台中创建活跃的进度条。它提供了丰富的配置选项，包括各种样式的进度条、进度块和旋转指示器。

通过 alive-progress 可以轻松地为长时间运行的任务添加一个漂亮的进度条，提高用户体验。

## 1. 安装

直接使用 `pip` 安装 alive-progress：

```Bash
pip install alive-progress
```

## 2. 基础进度条

下面这个例子我们创建了多个任务进度条，当执行进度超过总进度条没有达到总进度条时，或者超过总进度条时都会呈现不同的效果。

```Python
import time
from alive_progress import alive_bar

for total in 5000, 7000, 4000, 0:
    with alive_bar(total) as bar:
        for _ in range(5000):
            time.sleep(.001)
            bar()
```
{collapsible="true" default-state="expanded"}

![运行效果](https://raw.githubusercontent.com/rsalmei/alive-progress/main/img/alive-demo.gif)


> 上述代码在 Jupyter Notebook 或者 IPython 中都看不到动画效果，在终端中执行脚本是可以看到效果的。
{style="warning"}

## 3. 显示信息

结合 logging 日志模块，我们可以在进度条更新过程中显示一些信息。

```Python
import time
import logging
from alive_progress import alive_bar


logging.basicConfig(level=logging.INFO)
logger = logging.getLogger("AliveProgress")

with alive_bar(total) as bar:
    for i in range(1000):
        if i and i % 300 == 0:
            print("found")
        
        if i in (450, 700):
            logger.info("cool")
        
        time.sleep(.01)
        bar()
```
{collapsible="true" default-state="expanded"}

## 4. 自动迭代

通过 `alive_it` 直接封装迭代对象，可以像 TQDM 一样自动更新进度条，不需要手动调用来更新。但与 TQDM 不一样的是，这个过程中我们使用 `print` 打印信息时，并不会打乱进度条的显示样式。

```Python
import time
from alive_progress import alive_it

items = range(100)
for item in alive_it(items):
    print(item)
    time.sleep(.01)
```

此外，我们也可以通过更新进度条显示文本，来让进度信息显示得更完整。

```Python
import time
from alive_progress import alive_it

items = range(100)
bar = alive_it(items)
for item in bar:
    # print(item)
    bar.text(f"iter {item}")
    time.sleep(.1)
```

此外，`alive_it` 函数还有一个 `finalize` 参数，用于设定进度条完成时显示的信息。

```Python
import time
from alive_progress import alive_it

items = range(100)
bar = alive_it(
    items, 
    finalize=lambda x: x.text('Success!'), 
    receipt_text=True
    )

for item in bar:
    bar.text(f"iter {item}")
    time.sleep(.1)
```

除了 `text` 方法外，还有 `title` 方法可设置标题属性。

## 5. 样式

此外，alive_progress 还提供了一些额外的样式，用来定制化进度条。

### 5.1 spinner 图标

进度条加载过程中所显示的 loading 图标，还有以下参数值可选。


| classic    | stars       | twirl       | twirls  | pulse     |
|------------|-------------|-------------|---------|-----------|
| vertical   | waves       | waves2      | waves3  | dots      |
| horizontal | dots_waves  | dots_waves2 | it      | ball_belt |
| balls_belt | triangles   | brackets    | bubbles | circles   |
| squares    | flowers     | elements    | loving  | notes     |
| notes2     | arrow       | arrows      | arrows2 | arrows_in |
| arrows_out | radioactive | boat        | fish    | fish2     |
| fishes     | crab        | alive       | wait    | wait2     |
| wait3      | wait4       |

下面选择 `squares` 来定制化加载动画图标

```Python
import time
from alive_progress import alive_it

items = range(100)
bar = alive_it(items, spinner="squares",)
for item in bar:
    bar.text(f"iter {item}")
    time.sleep(.1)
```

### 5.2 bar 图标

当然，bar 图标也是可以更换的，具体有以下参数值可选。

| classic   | classic2 | smooth | brackets | blocks  |
|-----------|----------|--------|----------|---------|
| solid     | bubbles  | checks | circles  | squares |
| halloween | filling  | notes  | ruler    | ruler2  |
| fish      | scuba    |


示例代码：

```Python
import time
from alive_progress import alive_it

items = range(100)
bar = alive_it(items, bar="ruler")
for item in bar:
    bar.text(f"iter {item}")
    time.sleep(.1)
```


更多使用方法请参考 alive_progress 官方文档。


<seealso>
<category ref="ref_docs">
    <a href="https://mp.weixin.qq.com/s/lgJzipJ7QnvqXzXvXvM0hA">AliveProgress 简介</a>
</category>
<category ref="ref_github">
    <a href="https://github.com/rsalmei/alive-progress">AliveProgress Github</a>
</category>
<category ref="ref_issues">
</category>
<category ref="ref_hf"></category>
<category ref="ref_ms"></category>
</seealso>