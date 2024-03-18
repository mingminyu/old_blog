# PyCharm 教程

<show-structure depth="2"/>

如果你使用的技术是 Python 生态圈的话，那么一定对 PyCharm 不陌生，可以说它是最好的一款 Python 编码的 IDE。本篇文章，我们介绍一下 PyCharm 的使用，熟悉这些操作可以让整个编码效率大大提升。

## 1. 快捷键

### 1.1 代码编辑器

| 功能             | Mac                                            | Windows                              |
|----------------|------------------------------------------------|--------------------------------------|
| 格式化代码          | <shortcut>COMMAND+OPTION+L</shortcut>          | <shortcut>Control+Alt+L</shortcut>   |
| 优化导入库          | <shortcut>COMMAND+OPTION+O</shortcut>          | <shortcut>Control+Alt+O</shortcut>   |
| 多行合并一行         | <shortcut>Control+SHIFT+J</shortcut>           | <shortcut>Control+SHIFT+J</shortcut> |
| 编辑多行           | <shortcut>Control+SHIFT+J</shortcut>           | <shortcut>Control+SHIFT+J</shortcut> |
| 注释/取消注释        | <shortcut>SHIFT+OPTION+COMMAND+鼠标左键</shortcut> | <shortcut>Control+/</shortcut>       |
| 在上方插入空行        | <shortcut>OPTION+COMMAND+Enter</shortcut>      |                                      |
| 在下插入空行         | <shortcut>SHIFT+Enter</shortcut>               |                                      |
|                | <shortcut>COMMAND+Enter</shortcut>             |                                      |
| 上移动代码行         | <shortcut>SHIFT+OPTION+↑</shortcut>            |                                      |
| 下移动代码行         | <shortcut>SHIFT+OPTION+↓</shortcut>            |                                      |
| 复制选中的代码        | <shortcut>COMMAND+D</shortcut>                 |                                      |
| 展开代码块          | <shortcut>COMMAND+加号</shortcut>                |                                      |
| 折叠代码块          | <shortcut>COMMAND+减号</shortcut>                |                                      |
| 大小写转换          | <shortcut>SHIFT+COMMAND+U</shortcut>           |                                      |
| 插入最近的单词        | <shortcut>OPTION+/</shortcut>                  |                                      |
| 查找             | <shortcut>COMMAND+F</shortcut>                 |                                      |
| 替换             | <shortcut>COMMAND+R</shortcut>                 |                                      |
| 逐步选中代码块        | <shortcut>COMMAND+W</shortcut>                 |                                      |
| 逐步取消选中代码块      | <shortcut>SHIFT+COMMAND+W</shortcut>           |                                      |
| 从当前位置选中到代码块的开始 | <shortcut>SHIFT+COMMAND+[</shortcut>           |                                      |
| 从当前位置选中到代码块的结束 | <shortcut>SHIFT+COMMAND+]</shortcut>           |                                      |
| 代码块自动缩进        | <shortcut>OPTION+COMMAND+I</shortcut>          |                                      |
| 历史复制粘贴记录       | <shortcut>SHIFT+COMMAND+V</shortcut>           |                                      |
| 删除代码代码行        | <shortcut>COMMAND+Y</shortcut>                 |                                      |
| 向前删除代码         | <shortcut>OPTION+DELETE</shortcut>             |                                      |
| 向后删除代码         | <shortcut>FN+OPTION+DELETE</shortcut>          |                                      |
| 从当前位置选中行首代码    | <shortcut>SHIFT+COMMAND+←</shortcut>           |                                      |
| 从当前位置选中行尾代码    | <shortcut>SHIFT+COMMAND+→</shortcut>           |                                      |


## 2. 推荐插件

| 插件                    | 描述               |
|-----------------------|------------------|
| .ignore               | 生成 Git ignore 文件 |
| Atom Material Icons   | 界面图标美化           |
| Batch Scripts Support |                  |
| CMD Support           | 执行 cmd 脚本文件      |
| Gitee                 | Gitee 插件         |
| Mypy                  | 代码语法错误提示         |
| Regexp Tester         | 正则测试器            |
| TONGYI Lingma         | 通义千问灵码           |
| Prettier              |                  |
| CSV Editor            | CSV 文件编辑器        |
| TOML                  | 高亮 TOML 文件语法     |


## 3. 激活专业版

访问[网站](https://www.ajihuo.com/pycharm/4197.html) 获取 PyCharm 激活码，注意需要关注公众通过验证码才可以获取激活码，通常一个激活码也可以激活 IDEA、WebStorm、PhpStorm、GoLand 其他产品。如果你有 edu 教育邮箱的话，也可以直接去 JetBrains 官网申请免费资格，可以开庭 JetBrains 全家桶。

以下激活码截至 2024/10/13 过期，有需要的自己使用:

```Bash
```


Mac: https://mp.weixin.qq.com/s/7GY9_APwwXVIe-tcQhRlfw
Windows: https://mp.weixin.qq.com/s/vgTko0lyNkCyqWTiF3Ueww

## 4. 常见问题

### 4.1 IPython 控制台输出乱码

在 2022.2 版本的 PyCharm 中会出现 IPython 控制台输出中文时出现乱码的情况，导致这个问题出现的原因是缺少 `PYTHONIOENCODING` 环境变量导致，我们需要给 PyCharm 正确设置好该环境变量。

**解决方法**: 打开 <ui-path>文件 | 设置 | 构建、执行、部署 | 控制台 | Python 控制台</ui-path>，添加环境变量 `PYTHONIOENCODING='UTF8'` ，重启 PyCharm 后再次 打开 IPython 控制台就可以正常输出中文。


### 4.2 PyCharm 无法打开

之前 Mac 上 PyCharm 都是正常用的，但是突然某一天就打不开了，通过 ToolBox 或者直接点击应用都不行，解决办法如下:

1. 在终端中进入 /Users/yumingmin/Library/Application Support/JetBrains/PyCharm2023.3 目录下
2. 删除该目录下 pycharm.vmoptions 文件

之后应该就可以重新打开 PyCharm 应用了。

