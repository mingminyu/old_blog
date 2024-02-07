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


## 2. 文件备份

在工作和生活中，我们常常面临文件转移的需求，特别是对于那些珍贵的资料，备份显得尤为重要。传统的文件复制方式既耗时又费力，网盘也不是绝对安全，而 `rsync` 命令则为我们提供了高效的解决方案。

### 2.1 rsync 简介

`rsync` 是一款快速、通用的远程（和本地）文件复制工具，它支持多种协议，可以适应不同的网络环境。`rsync` 具有高效、灵活和安全的特点。

### 2.2 rsync 语法

```Bash
rsync [OPTION...] SRC... DEST
rsync [OPTION...] [USER@]HOST:SRC... DEST
rsync [OPTION...] SRC... [USER@]HOST:DEST
```
{collapsible="true" default-state="collapsed" collapsed-title="rsync 语法"}

| 常用选项                  | 说明                                               |
|-----------------------|--------------------------------------------------|
| `-a`                  | 归档模式，等同于 `-rlptgoD`，表示以递归方式并保持文件所有属性             |
| `-r`                  | 递归模式处理子目录                                        |
| `-b`                  | 为 DEST 中已存在的文件创建备份                               |
| `--backup-dir=DIR`    | 指定备份目录                                           |
| `--suffix=SUFFIX`     | 指定备份文件名前缀                                        |
| `-u`                  | 跳过 DEST 中较新的文件                                   |
| `--append`            | 追加数据到较短的文件，会在增量同步时计算文件大小并直接追加新的数据到文件，节省 IO 校验的过程 | 
| `-l`                  | 将符号链接复制为符号链接                                     |    
| `-p`                  | 保持权限                                             |
| `-o`                  | 保留所有者（仅限超级用户）                                    |      
| `-g`                  | 保留组                                              |
| `-D`                  | 保留设备文件（仅限超级用户）                                   |
| `-t`                  | 保留修改时间                                           |                            
| `-e`                  | 指定要使用的远程 shell                                   |                              
| `--delete`            | 从 DEST 中删除 SRC 没有的文件                             |                                 
| `-z`                  | 在传输过程中压缩文件数据                                     |                                    
| `--exclude=PATTERN`   | 排除与 PATTERN 匹配的文件                                | 
| `--include=PATTERN`   | 包含与 PATTERN 匹配的文件                                |
| `--exclude-from=FILE` | 从文件中读取排除文件的匹配规则                                  |
| `--include-from=FILE` | 从文件中读取所需要包含的匹配规则                                 |
| `-P`                  | 等同于 `--partial --progress`                       |
| `--partial`           | 保留部分传输的文件，即中断后可断点续传。默认会删除未完成的文件重新传输              |
| `--progress`          | 显示传输过程中的进度                                       |


### 2.2 增量备份

可以不定期将手机中的文件备份到电脑，或者将电脑中的文件备份到移动硬盘。使用 `rsync` 进行增量备份，只复制有变化的文件，可以提高备份效率。要执行 `rsync` 命令，手机用户可以使用 AidLux，Windows用户可以使用 WSL（Windows Subsystem for Linux），而 Linux 和 MAC 用户，可以直接使用系统终端。

#### I) AidLux 同步文件至 PC

例如，通过 AidLux 将手机相册的照片同步到电脑中，可执行类似如下命令：

```Bash
rsync -e 'ssh -p 9022' -ruv root@ip:/sdcard/DCIM/Camera /mnt/d/my/Pictures
```

AidLux 的默认 ssh 端口是 9022，非 ssh 默认端口 22，因此使用 `-e` 选项指定 ssh 端口 `ssh -p 9022`。

可以将上述命令放在循环中执行，并添加断点续传相关选项，从而降低拿走手机可能造成的网络中断带来的影响：

```Bash
while true; do
    rsync --append -P -e 'ssh -p 9022' -ruv root@ip:/sdcard/DCIM/Camera /mnt/d/my/Pictures
    if [ $? == 0 ]; then
        exit
    fi
done
```

循环中通过 `$?==0` 来判断传输是成功完成还是中断，以控制是否再次启动 `rsync` 继续同步。

#### II) 工作目录备份至移动硬盘

如果是 Windows 用户，需要在 WSL 中挂载移动硬盘。方法是创建装载的目录（如：`sudo mkdir /mnt/e`，将 `e` 替换为要使用的驱动器盘符），然后使用文件系统互操作插件 `drvfs`，通过以下命令完成挂载：

```Bash
sudo mount -t drvfs E: /mnt/e
```

然后就可以执行备份操作，可执行类似如下命令，将 work 目录备份到移动硬盘：

```Bash
rsync -ru /mnt/d/work /mnt/e
```

最后使用以下命令卸载移动硬盘：

```Bash
sudo umount /mnt/e
```

## 3. 查看文件内容

Linux 中有多个命令可以用于查看文件的内容，比如 `head`、`tail`、`cat`、`more` 等，这几种用法相对也比较简单。

### 3.1 筛选内容

通常 Linux 中使用 `cat` 命令查看文件内容时可以结合 `grep` 命令进行内容筛选，来定位出我们比较关注的文本。

```Bash
cat run.log | grep error
```

但有时候会出现使用 `cat` 可以正常查看文件内容，但是结合 `grep` 之后会报错 “**Binary file (standard input) matches**”。 这个时候可以使用 `strings` 命令来查看文件，并结合 `grep` 命令筛选想要的文本内容:

```Bash
strings -f run.log  | grep error
```






<seealso>
<category ref="ref_docs">
    <a href="https://mp.weixin.qq.com/s/ePCTzboNC5Jmn6UgniViQg">rsync:轻松实现文件的增量备份</a>
    <a href="https://mp.weixin.qq.com/s/nPKLuRyQ1lUCL1NRebPmBw">使用awk去除文件中的重复行</a>
</category>
<category ref="ref_github"></category>
<category ref="ref_issues"></category>
<category ref="ref_hf"></category>
<category ref="ref_ms"></category>
</seealso>