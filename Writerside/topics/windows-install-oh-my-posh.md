# 安装 Oh-My-Posh

<show-structure depth="3"/>

## 1. 安装 Scoop


## 2. 安装

```Bash
scoop install https://github.com/JanDeDobbeleer/oh-my-posh/releases/latest/download/oh-my-posh.json
```

安装完成后需要将 oh-my-posh 添加到环境变量中，建议添加到:

```Bash
$env:Path += ";C:\Users\yumingmin\AppData\Local\Programs\oh-my-posh\bin"
```

此时我们打开 Windows Terminal 依旧是无法直接进入 oh-my-posh 的，需要在 C:\Users\yumingmin\Documents\WindowsPowerShell\Microsoft.PowerShell_profile.ps1 文件中添加如下内容:

```Shell
oh-my-posh init pwsh --config "C:\Users\yumingmin\AppData\Local\Programs\oh-my-posh\themes\M365Princess.omp.json" | Invoke-Expression
```

重开打开 Windows Terminal 程序，就可以看到正常进入 Oh-My-Posh 了。


> 这里选择 M365Princess 主题，我个人还是比较喜欢这个主题的。
> 
{style="note"}


## 3. 更新

如果想要更新 oh-my-posh 的话，需要使用 `scoop update` 命令来更新:

```Bash
scoop update oh-my-posh
```


<seealso>
<category ref="ref_docs">
    <a href="https://ohmyposh.dev/docs/installation/windows">Windows 安装 Oh-My-Posh</a>
</category>
<category ref="ref_github">
</category>
<category ref="ref_issues">
</category>
<category ref="ref_hf">
</category>
<category ref="ref_ms">
</category>
</seealso>

