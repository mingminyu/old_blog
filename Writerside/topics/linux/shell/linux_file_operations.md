# 文件操作

<show-structure depth="2"/>


## 1. 查找文件

Linux 使用 `find` 命令查找文件，可以结合管道或者其他 shell 命令做一些更复杂的操作。


### 1.1 使用正则表达式查找

`-name` 参数除了可以指定完整的文件名，也可以接收一个正则表达式来匹配文件名，比如我们想要查找当前文件夹下所有 wav 文件，可以执行以下命令：

```Bash
find ./ -name '*.wav'
```

### 1.2 根据文件的文件状态查找

在一些误操作的情况下，可能把一些文件添加到某个文件夹下，最简单的操作就是找出近期添加的文件进行删除，`find` 命令提供了一些参数用来定位文件的时间状态：

| 参数        | 描述             |
|-----------|----------------|
| -amin -20 | 20 分钟内访问过的文件   |
| -amin +20 | 20 分钟前访问过的文件   |
| -atime -1 | 1 天内访问过的文件     |
| -atime +1 | 1 天前访问过的文件     |
| -mmin -20 | 20 分钟内修改过的文件   |
| -mtime -1 | 1 天内修改过的文件     |
| -cmin -20 | 20 分钟内状态改变过的文件 |
| -ctime -1 | 1 天内状态改变过的文件   |


示例: 查找 20 分钟内访问过的文件

```Bash
find ./ -name '*.wav' -mmin -20 -ls
```

### 1.3 查找出文件后进行批量删除

使用 `find` 命令找出对应文件后，结合 `-exec rm {} \;` 命令，可以对找出的文件进行批量删除。

示例: 查找 20 分钟内访问过的文件并删除

```Bash
find ./ -name '*.wav' -mmin -20 -ls -exec rm {} \;
```


<seealso>
<category ref="ref_docs">
    <a href="https://blog.csdn.net/weixin_43922901/article/details/106186331">Linux 删除指定时间的文件</a>
</category>
<category ref="ref_github"></category>
<category ref="ref_issues"></category>
<category ref="ref_hf"></category>
<category ref="ref_ms"></category>
</seealso>
