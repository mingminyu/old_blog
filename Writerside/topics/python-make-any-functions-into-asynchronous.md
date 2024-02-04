# 异步执行任何函数

<show-structure depth="2"/>

Python 协程可以极大地提高代码的运行效率，但经常使用 Python 的同学都知道，协程是在 Python 3.4~3.5 版本中才被引入，很多优秀的库（比如 requests）并不是采用异步的方式编写功能的，这就导致我们在这些库的使用上一般只能采用同步的方式，这无疑是大大地浪费了计算资源。

好在 `asyncio` 模块提供了非常帮的功能，能够帮我们将任意的同步函数转换为异步函数去执行，进而大大提高代码的运行效率，本篇文章我们来介绍下如何将同步函数转换为异步函数执行。

## 1. 常规方式调用

使用 `requests` 来获取网页内容，简单的示例代码如下：

```Python
import requests
from requests import Response

bing_url = "https://www.bing.com"
resp: Response = requests.get(bing_url)
print(resp.json())
```

## 2. 转换为异步函数

`requests.get` 是一个同步执行的过程，并且是 IO 密集型的任务，通过改造成异步函数后将极大地提高运行效率。

```Python
import asyncio
import requests
from asyncio import Task
from requests import Response


async def fetch_status(url: str) -> dict:
    print(f"Fetching status for: {url}")
    resp: Response = await asyncio.to_thread(requests.get, url, None)
    print("Done!")
    return {"status": resp.status_code, "url": url}
    

async def main() -> None:
    apple_task: Task[dict] = asyncio.create_task(fetch_status("https://apple.com"))
    bing_task: Task[dict] = asyncio.create_task(fetch_status("https://bing.com"))
    apple_status: dict = await apple_task
    bing_status: dict = await bing_task
    
    print("Apple status:", apple_status)
    print("Bing status:", bing_status)
    

if __name__ == "__main__":
    asyncio.run(main())
```
{collapsible="true" default-state="collapsed" collapsed-title="转换requests.get为异步执行"}

上面这个示例因为 `requests.get` 本身就比较快，所以效果的提升并不那么直观，那么下面我们改造一下，让每次请求等待 2 秒后返回结果。

```Python
import time
import asyncio
from asyncio import Task


def sim_request(url: str) -> dict:
    """延迟 2 秒返回"""
    time.sleep(2)
    return {"status": True}


async def fetch_status(url: str) -> dict:
    print(f"Fetching status for: {url}")
    resp = await asyncio.to_thread(sim_request, url)
    print("Done!")
    return {"status": resp["status"], "url": url}


async def main() -> list:
    t1 = time.time()
    tasks = []
    for i in range(30):
        sim_url = f"https://myexample.com/{i}"
        task: Task[dict] = asyncio.create_task(fetch_status(sim_url))
        tasks.append(task)

    results = [await task for task in tasks]
    print("Cost time:", time.time() - t1)
    return results


if __name__ == "__main__":
    asyncio.run(main())
```
{collapsible="true" default-state="collapsed"}

原本应该耗时 60 秒的任务，最终耗时 **6** 秒就完成了，是不是很神奇呢😏。
