# Supervisor 教程

<show-structure depth="3"/>

在实际的工作中，往往会有部署持久化进程的需求，比如接口服务进程，又或者是消费者进程等。这类进程通常是作为后台进程持久化运行的。
一般的部署方法是通过 nohup cmd & 命令来部署。但是这种方式有个弊端是在某些情况下无法保证目标进程的稳定性运行，有的时候 nohup 运行的后台任务会因为未知原因中断，从而导致服务或者消费中断，进而影响项目的正常运行。
为了解决上述问题，通过引入 Supervisor 来部署持久化进程，提高系统运行的稳定性。


<seealso>
<category ref="ref_docs">
    <a href="https://mp.weixin.qq.com/s/5kwZbmE3Ouxx1qFCHjVGYw">强大的 Python 库: Supervisor</a>
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

