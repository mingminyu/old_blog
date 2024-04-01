# 进程管理

<show-structure depth="3"/>

## 1. 端口与服务

### 1.1 查看端口所占用的服务

Linux 中可以通过 `lsfo` 命令查看占用端口的服务:

```Bash
# 查看关口号
lsfo -i :8080

# 查看监听端口的服务
lsfo -nP |grep LISTEN |grep 8080  

# 查看监听的端口服务
lsof -nP -iTCP -sTCP:LISTEN
```

### 1.2 查看进程监听的端口号

当我们想要查看对应 PID 服务所监听的端口，可以使用如下命令

```Bash
lsof -nP -p <PID> | grep LISTEN
lsof -nP | grep LISTEN | grep <PID>
```


### 1.3 查看进程号

Linux 中使用 `ps` 来查看具体进程的信息:

```Bash
ps -ef | grep airflow
```







