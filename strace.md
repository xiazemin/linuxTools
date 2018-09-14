什么是strace?

strace是一个非常简单的工具，它可以跟踪系统调用的执行。最简单的方式，它可以从头到尾跟踪binary的执行，然后以一行文本输出系统调用的名字，参数和返回值。



其实它可以做的更多：



可以对特定的系统调用或者几组系统调用进行过滤

可以通过统计特定系统调用的调用次数、耗费的时间、成功和失败的次数来配置\(profile\)系统调用的使用I

跟踪发送给进程的信号量

可以通过pid附着\(attach\)到任何运行的进程

如果你使用的是其它Unix系统，它类似于"truss"。其它更复杂的是Sun的Dtrace.



怎么使用它

1\) 找出程序在startup的时候读取的哪个config文件？

有没有尝过解决为什么某些程序不读去你认为它应该读取的config文件的问题？



\[cpp\] view plain copy

$ strace php 2&gt;&1 \| grep php.ini  

open\("/usr/local/bin/php.ini", O\_RDONLY\) = -1 ENOENT \(No such file or directory\)  

open\("/usr/local/lib/php.ini", O\_RDONLY\) = 4  

lstat64\("/usr/local/lib/php.ini", {st\_mode=S\_IFLNK\|0777, st\_size=27, ...}\) = 0  

readlink\("/usr/local/lib/php.ini", "/usr/local/Zend/etc/php.ini", 4096\) = 27  

lstat64\("/usr/local/Zend/etc/php.ini", {st\_mode=S\_IFREG\|0664, st\_size=40971, ...}\) = 0  

可以看出这个版本的PHP从/usr/local/lib/php.init读取config文件（但是先尝试/usr/locl/bin\)

如果只关心特定的系统调用，有更精致的方法



\[cpp\] view plain copy

$ strace -e open php 2&gt;&1 \| grep php.ini  

open\("/usr/local/bin/php.ini", O\_RDONLY\) = -1 ENOENT \(No such file or directory\)  

open\("/usr/local/lib/php.ini", O\_RDONLY\) = 4  

相同的方法适用于很多其它类似的问题。比如说，安装了不同版本的library，不确定实际上加载了哪一个版本。

2\) 为什么这个程序没有打开我的文件？

是否曾经碰到过一个程序拒绝读取它没有权限的文件，但是你发誓原因是它没有真正找到那个文件？对程序跟踪open,access调用，注意失败的情况



\[cpp\] view plain copy

$ strace -e open,access 2&gt;&1 \| grep your-filename  

3\) 某个进程现在在做什么?

某个进程突然占用了很多CPU? 或者某个进程看起来像hanging了？



找到对应的pid，然后



\[cpp\] view plain copy

root@dev:~\# strace -p 15427  

Process 15427 attached - interrupt to quit  

futex\(0x402f4900, FUTEX\_WAIT, 2, NULL   

Process 15427 detached  

嗯，这个例子里面，它在调用futex\(\)的时候挂起了。



"strace -p"非常有用，它减少了很多猜测工作，也不需要重新启动应用。



4\) 是谁偷走了时间？

你可以重新编译app，打开profiling，以获取精确的信息。但是通常利用strace附着\(attach）一个进程以快速地看一下当前时间花费在哪里非常有用。可以看下是否90%的CPU用在真正的工作，或者用在其它方面了。



\[cpp\] view plain copy

root@dev:~\# strace -c -p 11084  

Process 11084 attached - interrupt to quit  

Process 11084 detached  

% time     seconds  usecs/call     calls    errors syscall  

------ ----------- ----------- --------- --------- ----------------  

 94.59    0.001014          48        21           select  

  2.89    0.000031           1        21           getppid  

  2.52    0.000027           1        21           time  

------ ----------- ----------- --------- --------- ----------------  

100.00    0.001072                    63           total  

root@dev:~\#   



在执行strace -c -p命令以后，等到你关注的时间到了后，按ctrl-c退出，strace会列出如上的profiling数据。



在这个例子中，程序花了绝大部分时间在等待select\(\)。它在每一个slect\(\)调用这件调用getpid\(\)和time\(\)，这是一种典型的事件循环。



你也可以运行"start to finish"，这里是"ls"



\[cpp\] view plain copy

root@dev:~\# strace -c &gt;/dev/null ls  

% time     seconds  usecs/call     calls    errors syscall  

------ ----------- ----------- --------- --------- ----------------  

 23.62    0.000205         103         2           getdents64  

 18.78    0.000163          15        11         1 open  

 15.09    0.000131          19         7           read  

 12.79    0.000111           7        16           old\_mmap  

  7.03    0.000061           6        11           close  

  4.84    0.000042          11         4           munmap  

  4.84    0.000042          11         4           mmap2  

  4.03    0.000035           6         6         6 access  

  3.80    0.000033           3        11           fstat64  

  1.38    0.000012           3         4           brk  

  0.92    0.000008           3         3         3 ioctl  

  0.69    0.000006           6         1           uname  

  0.58    0.000005           5         1           set\_thread\_area  

  0.35    0.000003           3         1           write  

  0.35    0.000003           3         1           rt\_sigaction  

  0.35    0.000003           3         1           fcntl64  

  0.23    0.000002           2         1           getrlimit  

  0.23    0.000002           2         1           set\_tid\_address  

  0.12    0.000001           1         1           rt\_sigprocmask  

------ ----------- ----------- --------- --------- ----------------  

100.00    0.000868                    87        10 total  

正如你的预期，它耗费了大部分时间在两次调用来读取目录条目上（因为运行于一个小的目录上，所有只有两次）

5\) 为什么 无法连接到服务器?

调试进程无法连接到远端服务器有时候是件非常头痛的事。DNS会失败，connect会挂起，server有可能返回一些意料之外的数据。可以使用tcpdump来分析这些情况，它是一个非常棒的工作。但是有时候你strace可以给你更简单，耿直借的角度，因为strace只返回你的进程相关的系统调用产生的数据。如果你要从100个连接到统一个数据服务器的运行进程里面找出一个连接所做的事情，用strace就比tcpdump简单得多。



下面是跟踪"nc"连接到www.news.com 80端口的例子



\[cpp\] view plain copy

$ strace -e poll,select,connect,recvfrom,sendto nc www.news.com 80  

sendto\(3, "\\24\\0\\0\\0\\26\\0\\1\\3\\255\\373NH\\0\\0\\0\\0\\0\\0\\0\\0", 20, 0, {sa\_family=AF\_NETLINK, pid=0, groups=00000000}, 12\) = 20  

connect\(3, {sa\_family=AF\_FILE, path="/var/run/nscd/socket"}, 110\) = -1 ENOENT \(No such file or directory\)  

connect\(3, {sa\_family=AF\_FILE, path="/var/run/nscd/socket"}, 110\) = -1 ENOENT \(No such file or directory\)  

connect\(3, {sa\_family=AF\_INET, sin\_port=htons\(53\), sin\_addr=inet\_addr\("62.30.112.39"\)}, 28\) = 0  

poll\(\[{fd=3, events=POLLOUT, revents=POLLOUT}\], 1, 0\) = 1  

sendto\(3, "\\213\\321\\1\\0\\0\\1\\0\\0\\0\\0\\0\\0\\3www\\4news\\3com\\0\\0\\34\\0\\1", 30, MSG\_NOSIGNAL, NULL, 0\) = 30  

poll\(\[{fd=3, events=POLLIN, revents=POLLIN}\], 1, 5000\) = 1  

recvfrom\(3, "\\213\\321\\201\\200\\0\\1\\0\\1\\0\\1\\0\\0\\3www\\4news\\3com\\0\\0\\34\\0\\1\\300\\f"..., 1024, 0, {sa\_family=AF\_INET, sin\_port=htons\(53\), sin\_addr=inet\_addr\("62.30.112.39"\)}, \[16\]\) = 153  

connect\(3, {sa\_family=AF\_INET, sin\_port=htons\(53\), sin\_addr=inet\_addr\("62.30.112.39"\)}, 28\) = 0  

poll\(\[{fd=3, events=POLLOUT, revents=POLLOUT}\], 1, 0\) = 1  

sendto\(3, "k\\374\\1\\0\\0\\1\\0\\0\\0\\0\\0\\0\\3www\\4news\\3com\\0\\0\\1\\0\\1", 30, MSG\_NOSIGNAL, NULL, 0\) = 30  

poll\(\[{fd=3, events=POLLIN, revents=POLLIN}\], 1, 5000\) = 1  

recvfrom\(3, "k\\374\\201\\200\\0\\1\\0\\2\\0\\0\\0\\0\\3www\\4news\\3com\\0\\0\\1\\0\\1\\300\\f"..., 1024, 0, {sa\_family=AF\_INET, sin\_port=htons\(53\), sin\_addr=inet\_addr\("62.30.112.39"\)}, \[16\]\) = 106  

connect\(3, {sa\_family=AF\_INET, sin\_port=htons\(53\), sin\_addr=inet\_addr\("62.30.112.39"\)}, 28\) = 0  

poll\(\[{fd=3, events=POLLOUT, revents=POLLOUT}\], 1, 0\) = 1  

sendto\(3, "\\\\\\2\\1\\0\\0\\1\\0\\0\\0\\0\\0\\0\\3www\\4news\\3com\\0\\0\\1\\0\\1", 30, MSG\_NOSIGNAL, NULL, 0\) = 30  

poll\(\[{fd=3, events=POLLIN, revents=POLLIN}\], 1, 5000\) = 1  

recvfrom\(3, "\\\\\\2\\201\\200\\0\\1\\0\\2\\0\\0\\0\\0\\3www\\4news\\3com\\0\\0\\1\\0\\1\\300\\f"..., 1024, 0, {sa\_family=AF\_INET, sin\_port=htons\(53\), sin\_addr=inet\_addr\("62.30.112.39"\)}, \[16\]\) = 106  

connect\(3, {sa\_family=AF\_INET, sin\_port=htons\(80\), sin\_addr=inet\_addr\("216.239.122.102"\)}, 16\) = -1 EINPROGRESS \(Operation now in progress\)  

select\(4, NULL, \[3\], NULL, NULL\)        = 1 \(out \[3\]\)  



发生了什么？



注意到尝试连接/var/run/nscd/socket?这意味着nc首先尝试连接NSCD--the Name Service Cache Daemon--它通常用来基于NIS,YP,LDAP或者类似的目录协议提供域名查询。在这里它失败了。

然后它连接DNS（DNS是port 53,所以"sin\_port=htons\(53\)"\)。然后它调用了"sendto（）“,发送包含www.news.com的DNS 包。然后读回响应。不知为何，它尝试了三次，最后一次有细微的却别，我猜是它www.news.com十一个CNAME（别名），多次请求可能是nc故意的。



最后，它发起一个connect\(\)请求到得到的IP地址，注意到返回值是EINPROGRESS。这意味这connect是非阻塞的，nc希望继续处理，然后它调用slect\(\)，连接建立后，select返回成功。



添加"read","write"到过滤系统调用列表中，连接时输入一个字串，可能会得到如下



Notice the connection attempts to /var/run/nscd/socket? They mean nc first tries to connect to NSCD - the Name Service Cache Daemon - which is usually used in setups that rely on NIS, YP, LDAP or similar directory protocols for name lookups. In this case the connects fails.



It then moves on to DNS \(DNS is port 53, hence the "sin\_port=htons\(53\)" in the following connect. You can see it then does a "sendto\(\)" call, sending a DNS packet that contains www.news.com. It then reads back a packet. For whatever reason it tries three times, the last with a slightly different request. My best guess why in this case is that www.news.com is a CNAME \(an "alias"\), and the multiple requests may just be an artifact of how nc deals with that.



Then in the end, it finally issues a connect\(\) to the IP it found. Notice it returns EINPROGRESS. That means the connect was non-blocking - nc wants to Go on processing. It then calls select\(\), which succeeds when the connection was successful.



Try adding "read" and "write" to the list of syscalls given to strace and enter a string when connected, and you'll get something like this:



\[cpp\] view plain copy

read\(0, "test\\n", 1024\)                 = 5  

write\(3, "test\\n", 5\)                   = 5  

poll\(\[{fd=3, events=POLLIN, revents=POLLIN}, {fd=0, events=POLLIN}\], 2, -1\) = 1  

read\(3, "  

这表示从标准输入读入"test"+换行符，并写到网络连接中，然后调用poll等待响应，读取响应，写回标准输出。

一切看起来都正常工作。

