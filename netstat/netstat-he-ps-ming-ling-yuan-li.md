netstat命令：用来打印Linux中网络系统的状态信息，可让你得知整个Linux系统的网络情况。

这里就拿IPV4的举例，/proc/net/目录下就有当前tcp udp 的连接状态和 基本信息，netstat就是打开这个目录下的tcp udp然

后解析出来，就是看主机是大小端，然后16进制转为10进制 就哦了















ps命令：用于报告当前系统的进程状态。







是根据proc/PID/stat status cmdline 三个文件来解析，我解析的是 进程名 ID 运行总时间 进程路径 高峰内存（peak） 线程数



stat：进程名  ID 启动时间 线程数

status 有关内存的

cmdline：进程路径，没有就是进程名。

这里主要说一下运行时间，我是声明一个指针数组char \*msg\[50\]用” “切割读取到的字符串，运行（TIME）就等于

atoi\(msg\[13\]\)+atoi\(msg\[14\]\),计时间隔导致的下一个 SIGALRM 发送进程的时延+启动时间。

