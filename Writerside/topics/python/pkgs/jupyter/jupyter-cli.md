# Jupyter 命令行工具

<show-structure depth="2"/>

## 1. 后台运行服务

通常启动 Jupyter 服务需要将其挂载在后台运行，在 Linux 中使用 `nohup` 命令可以将服务在后台挂起：

```Bash
nohup jupyter notebook --ip='*' --no-browser > jupyter.log 2>&1 &
```

如果并不需要日志做分析，我们可以将输出重定向到 /dev/null，避免产生大文件。

```Bash
nohup jupyter notebook --ip='*' --no-browser > /dev/null 2>&1 &
```

## 2. 命令行参数


Jupyter Notebook 和 Jupyter Lab 通常是同一端口上的不同应用，只是通过不同的 URL 进行访问，这两个应用有有一些公用参数，当然各自也有自己独有的参数，下面我们做一个说明。

### 2.1 子命令

| 子命令        | 类型       | 描述                                                           |
|------------|----------|--------------------------------------------------------------|
| build      | Lab      |                                                              |
| clean      | Lab      | 清理 JupyterLab 应用                                             |
| path       | Lab      | 打印 JupyterLab 应用的配置路径                                        |
| paths      | Lab      | 打印 JupyterLab 应用的配置路径                                        |
| workspace  | Lab      | 列举、导入或导出 JupyterLab workspace，其还有 export/ import/ list 三个子命令 |
| workspaces | Lab      | 列举、导入或导出 JupyterLab workspace，其还有 export/ import/ list 三个子命令 |
| licenses   | Lab      |                                                              |
| list       | Notebook | 列举当前正在运行的 notebook server                                    |
| stop       | Notebook | 停止当前正在运行的 notebook server                                    |
| password   | Notebook | 设置登录密码                                                       |

上面除了 `jupyter notebook password` 用得略多外，其他子命令几乎不会使用，这里也不再继续介绍。

### 2.2 lab/ notebook 参数

以下是 Notebook/Lab 常用的参数，其中类型字段如果为空时，则表示 jupyter-notebook 和 jupyter-lab 都适用。

| 参数                       | 类型       | 描述                                        |
|--------------------------|----------|-------------------------------------------|
| -y                       |          | 对于所有问题默认是 yes                             |
| --notebook-dir           |          | 启动的文件夹路径                                  |
| --generate-config        |          | 生成配置文件                                    |
| --allow-root             |          | 运行 root 用户开启服务                            |
| --no-browser             |          | 关闭启动时默认在浏览器打开主页的行为                        |
| --ip                     |          | 服务监听的 IP 地址                               |
| --port                   |          | 服务监控的端口                                   |
| --auto-reload            |          | 当文件发生变化时，自动重载 Jupyter 服务                  |
| --pylab                  |          | 给所有 notebook 启用 `%pylab` 或者 `%matplotlib` |
| --log-level              |          | 设置日志等级                                    |
| --config                 |          | 配置文件                                      |
| --debug                  |          |                                           |
| --show-config-json       |          |                                           |
| --show-config            |          |                                           |
| --port-retries           |          | 当默认端口不可用时，尝试其他端口的最大次数                     |
| --keyfile                |          | 私钥的 keyfile 路径，通常用于 SSL/TLS               |
| --certfile               |          | 设置 SSL/TLS 的证书文件                          |
| --client-ca              |          |                                           |
| --browser                |          | 指定启动时打开服务浏览器                              |
| --no-script              |          | 已经废弃                                      |
| --script                 |          | 已经废弃                                      |
| --splice-source          | Lab      |                                           |
| --collaborative          | Lab      | 是否开启协作模式                                  |
| --expose-app-in-browser  | Lab      |                                           |
| --extensions-in-dev-mode | Lab      |                                           |
| --app-dir                | Lab      | 启动 JupyterLab 的 App 目录                    |
| --core-mode              | Lab      | 以 Core 模式开启服务                             |
| --dev-mode               | Lab      | 以 Dev 模式开启服务                              |
| --watch                  | Lab      | 以监控模式开启服务                                 |
| --no-mathjax             | Notebook | 关闭 MathJAX                                |
| --sock                   | Notebook | 监听的 UNIX socket                           |
| --sock-mode              | Notebook | 创建 UNIX socket 的权限                        |
| --transport              | Notebook | TCP / IPC                                 |
| --gateway                | Notebook | 网关的 URL                                   |

