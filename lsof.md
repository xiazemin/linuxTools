命令作用

lsof\(list open files\) 是一个列出当前系统打开文件的工具，在linux环境下，任何事物都以文件的形式存在，通过文件不仅仅可以访问常规数据，还可以访问网络连接和硬件。



输出各列信息的意义

COMMAND：进程的名称

PID：进程标识符

USER：进程所有者

FD：文件描述符，应用程序通过文件描述符识别该文件。如cwd、txt等

TYPE：文件类型，如DIR、REG等

DEVICE：指定磁盘的名称

SIZE：文件的大小

NODE：索引节点（文件在磁盘上的标识）

NAME：打开文件的确切名称

其中FD 列中的文件描述符cwd 值表示应用程序的当前工作目录，这是该应用程序启动的目录，除非它本身对这个目录进行更改。 

txt 类型的文件是程序代码，如应用程序二进制文件本身或共享库，如上列表中显示的 /sbin/init 程序。其次数值表示应用 

程序的文件描述符，这是打开该文件时返回的一个整数。如上的最后一行文件/dev/initctl，其文件描述符为 10。u 表示该 

文件被打开并处于读取/写入模式，而不是只读 ® 或只写 \(w\) 模式。同时还有大写 的W 表示该应用程序具有对整个文件的写 

锁。该文件描述符用于确保每次只能打开一个应用程序实例。初始打开每个应用程序时，都具有三个文件描述符，从 0 到 2， 

分别表示标准输入、输出和错误流。所以大多数应用程序所打开的文件的 FD 都是从 3 开始。 

与 FD 列相比，Type 列则比较直观。文件和目录分别称为 REG 和 DIR。而CHR 和 BLK，分别表示字符和块设备； 

或者 UNIX、FIFO 和 IPv4，分别表示 UNIX 域套接字、先进先出 \(FIFO\) 队列和网际协议 \(IP\) 套接字。



常用参数列表

lsof filename 显示打开指定文件的所有进程 

lsof -a 表示两个参数都必须满足时才显示结果 

lsof -c string 显示COMMAND列中包含指定字符的进程所有打开的文件 

lsof -u username 显示所属user进程打开的文件 

lsof -g gid 显示归属gid的进程情况 

lsof +d /DIR/ 显示目录下被进程打开的文件 

lsof +D /DIR/ 同上，但是会搜索目录下的所有目录，时间相对较长 

lsof -d FD 显示指定文件描述符的进程 

lsof -n 不将IP转换为hostname，缺省是不加上-n参数 

lsof -i 用以显示符合条件的进程情况 

lsof -i\[46\] \[protocol\]\[@hostname\|hostaddr\]\[:service\|port\] 

46 –&gt; IPv4 or IPv6 

protocol –&gt; TCP or UDP 

hostname –&gt; Internet host name 

hostaddr –&gt; IPv4地址 

service –&gt; /etc/service中的 service name \(可以不只一个\) 

port –&gt; 端口号 \(可以不只一个\)



常见使用

查找占用端口的线程 lsof -i :8080

