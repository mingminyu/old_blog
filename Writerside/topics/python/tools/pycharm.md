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
UX394X3HLT-eyJsaWNlbnNlSWQiOiJVWDM5NFgzSExUIiwibGljZW5zZWVOYW1lIjoiSG9uZ2lrIFVuaXZlcnNpdHntmY3snbXrjIDtlZnqtZAiLCJsaWNlbnNlZVR5cGUiOiJDTEFTU1JPT00iLCJhc3NpZ25lZU5hbWUiOiLkvJfliJvkupEg5bel5L2c5a6kIiwiYXNzaWduZWVFbWFpbCI6ImhhbmF6YXdhbWl0b0BnbWFpbC5jb20iLCJsaWNlbnNlUmVzdHJpY3Rpb24iOiJGb3IgZWR1Y2F0aW9uYWwgdXNlIG9ubHkiLCJjaGVja0NvbmN1cnJlbnRVc2UiOmZhbHNlLCJwcm9kdWN0cyI6W3siY29kZSI6IkdPIiwicGFpZFVwVG8iOiIyMDI0LTEyLTEzIiwiZXh0ZW5kZWQiOmZhbHNlfSx7ImNvZGUiOiJSUzAiLCJwYWlkVXBUbyI6IjIwMjQtMTItMTMiLCJleHRlbmRlZCI6ZmFsc2V9LHsiY29kZSI6IkRNIiwicGFpZFVwVG8iOiIyMDI0LTEyLTEzIiwiZXh0ZW5kZWQiOmZhbHNlfSx7ImNvZGUiOiJDTCIsInBhaWRVcFRvIjoiMjAyNC0xMi0xMyIsImV4dGVuZGVkIjpmYWxzZX0seyJjb2RlIjoiUlNVIiwicGFpZFVwVG8iOiIyMDI0LTEyLTEzIiwiZXh0ZW5kZWQiOmZhbHNlfSx7ImNvZGUiOiJSU0MiLCJwYWlkVXBUbyI6IjIwMjQtMTItMTMiLCJleHRlbmRlZCI6dHJ1ZX0seyJjb2RlIjoiUEMiLCJwYWlkVXBUbyI6IjIwMjQtMTItMTMiLCJleHRlbmRlZCI6ZmFsc2V9LHsiY29kZSI6IkRTIiwicGFpZFVwVG8iOiIyMDI0LTEyLTEzIiwiZXh0ZW5kZWQiOmZhbHNlfSx7ImNvZGUiOiJSRCIsInBhaWRVcFRvIjoiMjAyNC0xMi0xMyIsImV4dGVuZGVkIjpmYWxzZX0seyJjb2RlIjoiUkMiLCJwYWlkVXBUbyI6IjIwMjQtMTItMTMiLCJleHRlbmRlZCI6ZmFsc2V9LHsiY29kZSI6IlJTRiIsInBhaWRVcFRvIjoiMjAyNC0xMi0xMyIsImV4dGVuZGVkIjp0cnVlfSx7ImNvZGUiOiJSTSIsInBhaWRVcFRvIjoiMjAyNC0xMi0xMyIsImV4dGVuZGVkIjpmYWxzZX0seyJjb2RlIjoiSUkiLCJwYWlkVXBUbyI6IjIwMjQtMTItMTMiLCJleHRlbmRlZCI6ZmFsc2V9LHsiY29kZSI6IkRQTiIsInBhaWRVcFRvIjoiMjAyNC0xMi0xMyIsImV4dGVuZGVkIjpmYWxzZX0seyJjb2RlIjoiREIiLCJwYWlkVXBUbyI6IjIwMjQtMTItMTMiLCJleHRlbmRlZCI6ZmFsc2V9LHsiY29kZSI6IkRDIiwicGFpZFVwVG8iOiIyMDI0LTEyLTEzIiwiZXh0ZW5kZWQiOmZhbHNlfSx7ImNvZGUiOiJQUyIsInBhaWRVcFRvIjoiMjAyNC0xMi0xMyIsImV4dGVuZGVkIjpmYWxzZX0seyJjb2RlIjoiUlNWIiwicGFpZFVwVG8iOiIyMDI0LTEyLTEzIiwiZXh0ZW5kZWQiOnRydWV9LHsiY29kZSI6IldTIiwicGFpZFVwVG8iOiIyMDI0LTEyLTEzIiwiZXh0ZW5kZWQiOmZhbHNlfSx7ImNvZGUiOiJQU0kiLCJwYWlkVXBUbyI6IjIwMjQtMTItMTMiLCJleHRlbmRlZCI6dHJ1ZX0seyJjb2RlIjoiUENXTVAiLCJwYWlkVXBUbyI6IjIwMjQtMTItMTMiLCJleHRlbmRlZCI6dHJ1ZX0seyJjb2RlIjoiUlMiLCJwYWlkVXBUbyI6IjIwMjQtMTItMTMiLCJleHRlbmRlZCI6dHJ1ZX0seyJjb2RlIjoiRFAiLCJwYWlkVXBUbyI6IjIwMjQtMTItMTMiLCJleHRlbmRlZCI6dHJ1ZX0seyJjb2RlIjoiUERCIiwicGFpZFVwVG8iOiIyMDI0LTEyLTEzIiwiZXh0ZW5kZWQiOnRydWV9XSwibWV0YWRhdGEiOiIwMTIwMjMxMjI4TFBBQTAwNDAwOCIsImhhc2giOiI1MjY4NzM4Ny8yNTMxODE0MTotMTUwNDg0MzkwMSIsImdyYWNlUGVyaW9kRGF5cyI6NywiYXV0b1Byb2xvbmdhdGVkIjpmYWxzZSwiaXNBdXRvUHJvbG9uZ2F0ZWQiOmZhbHNlLCJ0cmlhbCI6ZmFsc2UsImFpQWxsb3dlZCI6dHJ1ZX0=-YqDHrEIEaf/x1JqIdTI64AYA1IpRoYiqoZ/1YDnfpEqSFNJIC4er7K1hjUm9tFslnY2XoNRs04JSUG8CgNkTgIKA4xLyxGBufJYyHv26UKQmyf1nb1pM9XATb3pWSQ3h6o8/4x3jacVk3zbAuXt3uV6eEj2bCZvhGpATFmK1JVsSor+XgPr5aYePCtyymiyPOq33ghW5onzSI5LsQR5motHvLgWmjf0Mkutys3SmWt13YVcIe5yCzhlTNCZw++CAuRh2Fx/JXZhRt+kUqW2yLbkIo3kNEg2I31H8qya+RDJ09Qz7DsDkrIgODqX4Wbd2fy1C7Q1CcjjksvjhswnpNA==-MIIETDCCAjSgAwIBAgIBDzANBgkqhkiG9w0BAQsFADAYMRYwFAYDVQQDDA1KZXRQcm9maWxlIENBMB4XDTIyMTAxMDE2MDU0NFoXDTI0MTAxMTE2MDU0NFowHzEdMBsGA1UEAwwUcHJvZDJ5LWZyb20tMjAyMjEwMTAwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQC/W3uCpU5M2y48rUR/3fFR6y4xj1nOm3rIuGp2brELVGzdgK2BezjnDXpAxVDw5657hBkAUMoyByiDs2MgmVi9IcqdAwpk988/Daaajq9xuU1of59jH9eQ9c3BmsEtdA4boN3VpenYKATwmpKYkJKVc07ZKoXL6kSyZuF7Jq7HoQZcclChbF75QJPGbri3cw9vDk/e46kuzfwpGftvl6+vKibpInO6Dv0ocwImDbOutyZC7E+BwpEm1TJZW4XovMBegHhWC04cJvpH1u98xoR94ichw0jKhdppywARe43rGU96163RckIuFmFDQKZV9SMUrwpQFu4Z2D5yTNqnlLRfAgMBAAGjgZkwgZYwCQYDVR0TBAIwADAdBgNVHQ4EFgQU5FZqQ4gnVc+inIeZF+o3ID+VhcEwSAYDVR0jBEEwP4AUo562SGdCEjZBvW3gubSgUouX8bOhHKQaMBgxFjAUBgNVBAMMDUpldFByb2ZpbGUgQ0GCCQDSbLGDsoN54TATBgNVHSUEDDAKBggrBgEFBQcDATALBgNVHQ8EBAMCBaAwDQYJKoZIhvcNAQELBQADggIBANLG1anEKid4W87vQkqWaQTkRtFKJ2GFtBeMhvLhIyM6Cg3FdQnMZr0qr9mlV0w289pf/+M14J7S7SgsfwxMJvFbw9gZlwHvhBl24N349GuthshGO9P9eKmNPgyTJzTtw6FedXrrHV99nC7spaY84e+DqfHGYOzMJDrg8xHDYLLHk5Q2z5TlrztXMbtLhjPKrc2+ZajFFshgE5eowfkutSYxeX8uA5czFNT1ZxmDwX1KIelbqhh6XkMQFJui8v8Eo396/sN3RAQSfvBd7Syhch2vlaMP4FAB11AlMKO2x/1hoKiHBU3oU3OKRTfoUTfy1uH3T+t03k1Qkr0dqgHLxiv6QU5WrarR9tx/dapqbsSmrYapmJ7S5+ghc4FTWxXJB1cjJRh3X+gwJIHjOVW+5ZVqXTG2s2Jwi2daDt6XYeigxgL2SlQpeL5kvXNCcuSJurJVcRZFYUkzVv85XfDauqGxYqaehPcK2TzmcXOUWPfxQxLJd2TrqSiO+mseqqkNTb3ZDiYS/ZqdQoGYIUwJqXo+EDgqlmuWUhkWwCkyo4rtTZeAj+nP00v3n8JmXtO30Fip+lxpfsVR3tO1hk4Vi2kmVjXyRkW2G7D7WAVt+91ahFoSeRWlKyb4KcvGvwUaa43fWLem2hyI4di2pZdr3fcYJ3xvL5ejL3m14bKsfoOv
```

## 4. 常见问题

### 4.1 IPython 控制台输出乱码

在 2022.2 版本的 PyCharm 中会出现 IPython 控制台输出中文时出现乱码的情况，导致这个问题出现的原因是缺少 `PYTHONIOENCODING` 环境变量导致，我们需要给 PyCharm 正确设置好该环境变量。

**解决方法**: 打开 <ui-path>文件 | 设置 | 构建、执行、部署 | 控制台 | Python 控制台</ui-path>，添加环境变量 `PYTHONIOENCODING='UTF8'` ，重启 PyCharm 后再次 打开 IPython 控制台就可以正常输出中文。
