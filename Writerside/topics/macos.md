# MacOS

## 1. 创建中文名称的英文目录

MAC 下用户目录下像“**下载**”，在 Terminal 中显示实际路径为 ~/Downloads，对于写代码人来说，这种方式非常友好，毕竟使用中文作为目录名的话在运行程序时很容易出现问题。

我们创建一个 **Codes.localized** 英文目录（目录会带有 .localized），

```Bash
mkdir ~/Codes.localized
cd ~/Codes.localized && mkdir .localized
cd .localized && touch zh.strings
```

在 **zh.strings** 文件中添加如下内容，注意最后的分号，然后重启 Finder。

```Bash
"Codes" = "代码"; 
```

> 重启 "Finder"，按住键盘上的 **option** 键，然后在访达图标上点击右键，就有一个【重新开启】，点击即可。
> 
{style="note"}





<seealso>
<category ref="ref_docs">
    <a href="https://www.cnblogs.com/shuiche/p/14543081.html">Mac 创建中文名称的英文文件夹</a>
</category>
<category ref="ref_github"></category>
<category ref="ref_issues"></category>
<category ref="ref_hf"></category>
<category ref="ref_ms"></category>
</seealso>