# 配置 SSH 密钥

<show-structure depth="3"/>

这里假设我们已经成功安装了 Git 工具，接下里我们通过配置 SSH 密钥，从而可以直接使用 `git` 等命令来提交更改 Github 上。


## 1. 配置 Git 信息

首先我们配置下 Git 的用户名和邮箱。

```Bash
git config --global user.name mingminyu
git config --global user.email yu_mingm623@163.com
```

## 2. 生成 SSH 密钥

如果是 Linux 或者 MacOS 系统的话，执行以下命令生成 SSH 密钥。

```Bash
ssh-keygen -t ed25519 -C "yu_mingm623@163.com" 
```

如果是 Windows 系统的话，则直接可以通过 Git GUI 获取 SSH 秘密钥。


## 3. 添加 SSH 秘钥

在 Github 个人设置中添加好 SSH 密钥，执行如下命令，如果看到如下命令则证明 SSH 已经被成功配置好了。

```Bash
$ ssh -T git@github.com
Hi mingminyu! You've successfully authenticated, but GitHub does not provide shell access.
```
