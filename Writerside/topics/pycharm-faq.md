# 常见问题

<show-structure depth="3"/>


## 1. Python 控制台输出乱码

在 2022.2 版本的 PyCharm 中会出现 IPython 控制台输出中文时出现乱码的情况，导致这个问题出现的原因是缺少 `PYTHONIOENCODING` 环境变量导致，我们需要给 PyCharm 正确设置好该环境变量。

**解决方法**: 打开 <ui-path>文件 | 设置 | 构建、执行、部署 | 控制台 | Python 控制台</ui-path>，添加环境变量 `PYTHONIOENCODING='UTF8'` ，重启 PyCharm 后再次 打开 IPython 控制台就可以正常输出中文。


## 2. PyCharm 无法打开

之前 Mac 上 PyCharm 都是正常用的，但是突然某一天就打不开了，通过 ToolBox 或者直接点击应用都不行，解决办法如下:

1. 在终端中进入 /Users/yumingmin/Library/Application Support/JetBrains/PyCharm2023.3 目录下
2. 删除该目录下 pycharm.vmoptions 文件

之后应该就可以重新打开 PyCharm 应用了。