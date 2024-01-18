# Envoy 教程

<show-structure depth="2"/>

## 1. 简介

Envoy 是由 Kenneth Reitz 创建的，他也是著名的 `requests` 库的作者。Kenneth Reitz 以其对用户友好的 API 设计而闻名，Envoy 也不例外。它让运行外部命令变得简单快捷，只需要几行代码就可以完成。

Envoy 的核心是 `run` 函数，它可以执行任何你通常在命令行中运行的命令，此时会创建一个子进程来执行这个命令，然后返回一个响应对象，其中包含了命令的输出、错误信息和退出码等。

## 2. 安装

Envoy 的安装非常简单，直接使用 pip 安装即可：

```Bash
pip install envoy
```

## 3. 执行命令

### 3.1 执行简单命令

假设你想要查看当前目录下的文件列表，在终端的命令行中，我们可能会使用 `ls`（Unix 系统）或 `dir`（Windows 系统）命令。在 Python 中，你可以使用 Envoy 来做同样的事情：

```Python
import envoy

# 执行命令
r = envoy.run('ls')

# 打印命令的输出
print(r.std_out)

# 打印错误（如果有的话）
if r.std_err:
    print("错误信息：", r.std_err)

# 打印退出码
print("退出码：", r.status_code)
```

我们使用 `envoy.run` 函数执行了 `ls` 命令，`run` 函数返回了一个响应对象，我们可以从中获取**命令的输出、错误信息和退出码**。

### 3.2 管道命令

Envoy 也支持管道命令，这让它在处理更复杂的命令时显得非常有用。例如，你可能想要查找包含特定文本的文件。在命令行中，你可能会使用 `grep` 命令，并将其与其他命令结合使用。在 Python 中，你可以这样做：

```Python
import envoy

# 使用管道命令搜索包含'example'的行
r = envoy.run('cat test.txt | grep example')

# 打印匹配的行
print(r.std_out)
```

我们使用了 `cat` 命令来显示 test.txt 文件的内容，并通过管道将其传递给 `grep` 命令来搜索包含单词 "example" 的行。Envoy 让这个过程变得异常简单，只需要一行代码。

## 4. 总结

Envoy 库提供了一个简化的接口来运行外部命令，它是 `subprocess` 模块的一个优雅的封装。无论是想要快速执行一个简单的命令，还是需要构建一个复杂的管道命令，Envoy 都能帮助我们以 Pythonic 的方式轻松实现。

> 值得注意的是，Envoy 并没有特别详细的使用文档，一些高级的使用方法还需要自己看源码去探索。


<seealso>
<category ref="ref_github">
    <a href="https://github.com/not-kennethreitz/envoy">Envoy主页</a>
</category>
</seealso>