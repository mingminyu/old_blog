# 常规设置

<show-structure depth="3"/>


## 1. 自动换行显示(Soft-Wrap)

- https://blog.csdn.net/qq_29856169/article/details/116568703


## 2. 自动添加换行(Hard Wrap)

在使用 WriterSide 写 Markdown 笔记内容时发现，当文本超过一定字数时就被截断后自动换到下一行了，但实际上一段内容我们可能还没有写完，我们并不希望 IDE 帮我们自动换行。简而言之，就是我们希望编辑器在编写 Markdown 内容提供 Soft-Wrap，而不是 Hard Wrap。


通过 <shortcut key="$PyCharmOpenSetting"/> 打开PyCharm 设置选项，找到 **Editor** >> **General** >> **Code Style** >> **Markdown** 路径，在 **Wrapping and Braces** 标签项中修改 **Hard Wrap at:** 修改数值为 **1000**。


修改完后就再次编辑 Markdown 内容时，只会换行显示，而不是截断内容后生成新的一行。
