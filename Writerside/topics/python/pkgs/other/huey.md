# Huey 教程

<show-structure depth="2"/>

Huey 是一个由 Python 编写的小型多线程任务队列库，它支持任务调度以及执行。Huey 支持多种存储后端，如 Redis、SQLite 等，可以帮助你在后台执行耗时任务，或者在指定时间执行定时任务。

## 1. 安装

安装 Huey 非常简单，你可以使用 pip 命令直接安装：

```Bash
pip install huey
```

如果你打算使用 Redis 作为后端，你还需要安装 Redis 并启动服务。对于 SQLite 后端，Huey 会自动管理，无需额外安装。

## 2. 示例

### 2.1 异步发送邮件

假设你正在开发一个网站，需要在用户注册后发送一封欢迎邮件。我们可以使用 Huey 来异步处理这个任务，避免用户等待。

首先，创建一个 tasks.py 文件，并定义一个发送邮件的任务：

```Python
from huey import RedisHuey

# 实例化 Huey 对象，指定 Redis 作为后端
huey = RedisHuey('my_app')

@huey.task()
def send_welcome_email(user_email: str):
    print(f"Sending welcome email to {user_email}...")
    # 这里是发送邮件的逻辑
    print(f"Welcome email has been sent to {user_email}!")
```

在用户注册的代码中，我们可以这样触发异步任务：

```Python
from tasks import send_welcome_email

def user_signup(request):
    # 用户注册逻辑
    user_email = request.form['email']
    
    # 发送异步任务
    send_welcome_email(user_email)
    
    # 返回响应
    return "Please check your email for a welcome message!"
```


当调用 `send_welcome_email` 函数时，Huey 会将任务放入队列，并在后台工作线程中执行，无需用户等待邮件发送过程。

### 2.2 定时清理过期数据

让我们再来看一个例子，如果你的应用需要定期清理过期的数据，Huey 也可以帮你轻松实现定时任务。 首先，在 tasks.py 文件中定义一个清理数据的任务：

```Python
from huey import crontab

@huey.periodic_task(crontab(minute='0', hour='3'))
def cleanup_expired_data():
    # 获取当前时间
    now = datetime.datetime.now()
    
    # 设置过期时间
    expiry_time = now - datetime.timedelta(days=30)
    
    # 这里是清理过期数据的逻辑
    print(f"Cleaning up data older than {expiry_time}...")
    # 假设这里有一些清理数据的代码
    print("Expired data cleaned up!")
```


在这个例子中，我们使用了 Huey 的 `crontab` 来设置任务的执行时间。`cleanup_expired_data` 任务会在每天凌晨 3 点执行，自动清理超过 30 天的数据。

更多关于 Huey 的使用方法，请参与 [Huey 官方文档](https://huey.readthedocs.io/en/latest)。

<seealso>
<category ref="ref_docs">
    <a href="https://mp.weixin.qq.com/s/lQpatjBfxQqtU0-7Hebg6A">Huey介绍</a>
    <a href="https://huey.readthedocs.io/en/latest/">Huey官方文档</a>
</category>
</seealso>



