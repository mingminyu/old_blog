# 大模型工具

<show-structure depth="2"></show-structure>

## 1. ChatGPT-Next-Web


## 2. ChatBox


## 3. AtomGPT

[AtomGPT](https://github.com/mingminyu/AtomGPT) 是我自己尝试使用 NiceGUI 框架仿 ChatGPT 的 WEB UI，整个框架集成了前端 AtomUI 和 FastAPI 后端接口，值得注意的是，该框架暂未提供 LLM NLU 引擎，如果是大模型的应用接口需要单独起一块服务。

> AtomUI 是我自己基于 NiceGUI 开发的整合前端 UI 框架。

通过如下命令来启动 WEB UI:

```Bash
python webui.py
```

> 项目后续还会提供了 Windows 和 Mac 系统的桌面版安装版本，待开发。

## 4. LibreChat

[LibreChat](https://docs.librechat.ai/index.html) 是一款非常不错的开源工具，其界面几乎完全仿照 ChatGPT 界面，此外它还集成了多个其他大模型，例如 Google Gemini。美中不足的是，该工具不支持本地大模型的加入，期待后续添加。

> 按照官方教程安装时，可能会遇到 Sharp 库无法正常安装，需要根据终端中给出的[下载地址](https://github.com/lovell/sharp-libvips/releases/download/v8.14.5/libvips-8.14.5-darwin-arm64v8.tar.br)，用浏览器下载好后，将文件放在 `~/.npm/_libvips` 下，再使用 `npm ci` 命令安装就可以了。
> 
{style="note"}


## 5. LM Studio

[LM Studio](https://lmstudio.ai)

> 没有听过 Mac 非 M 系列的版本
> 
{style="warning"}



<seealso>
<category ref="ref_github">
    <a href="https://github.com/mingminyu/AtomGPT">AtomGPT</a>
</category>
</seealso>