### Linux

#### ps

Linux中的ps命令是**Process Status**的缩写。ps命令用来列出系统中当前运行的那些进程。ps命令列出的是当前那些进程的快照，

就是执行ps命令的那个时刻的那些进程，**如果想要动态的显示进程信息，就可以使用top命令**。

常用命令： ps -ef 显示所有进程 ps -aux 显示所有包含其他使用者的进程



```java
ps工具标识进程的5种状态码:
D 不可中断 uninterruptible sleep (usually IO)
R 运行 runnable (on run queue)
S 中断 sleeping
T 停止 traced or stopped
Z 僵死 a defunct (”zombie”) process
```

#### kill 进程

```java

kill -STOP [pid]
发送SIGSTOP (17,19,23)停止一个进程，而并不消灭这个进程。
kill -CONT [pid]
发送SIGCONT (19,18,25)重新开始一个停止的进程。
kill -KILL [pid]
发送SIGKILL (9)强迫进程立即停止，并且不实施清理操作。
kill -9 -1
终止你拥有的全部进程。
```

把所有进程显示出来，并输出到ps001.txt文件

命令：

　　ps -aux > ps001.txt

### 如何排查用户态CPU使用率高

1. 通过`top`命令找到CPU使用率最高的进程
2. 通过`top -Hp 进程号`命令找到相应进程下消耗最多的线程号
3. 通过`printf "%x\n" `输出该线程号的16进制
4. 通过`jstack 进程号 | grep 16进制线程号 -A 10`命令找到该线程下CPU消耗最多的线程方法堆栈 