Netstat 命令用于显示各种网络相关信息，如网络连接，路由表，接口状态 \(Interface Statistics\)，masquerade 连接，多播成员 \(Multicast Memberships\) 等等。



输出信息含义

执行netstat后，其输出结果为



复制代码

Active Internet connections \(w/o servers\)

Proto Recv-Q Send-Q Local Address Foreign Address State

tcp 0 2 210.34.6.89:telnet 210.34.6.96:2873 ESTABLISHED

tcp 296 0 210.34.6.89:1165 210.34.6.84:netbios-ssn ESTABLISHED

tcp 0 0 localhost.localdom:9001 localhost.localdom:1162 ESTABLISHED

tcp 0 0 localhost.localdom:1162 localhost.localdom:9001 ESTABLISHED

tcp 0 80 210.34.6.89:1161 210.34.6.10:netbios-ssn CLOSE



Active UNIX domain sockets \(w/o servers\)

Proto RefCnt Flags Type State I-Node Path

unix 1 \[ \] STREAM CONNECTED 16178 @000000dd

unix 1 \[ \] STREAM CONNECTED 16176 @000000dc

unix 9 \[ \] DGRAM 5292 /dev/log

unix 1 \[ \] STREAM CONNECTED 16182 @000000df

复制代码



从整体上看，netstat的输出结果可以分为两个部分：



一个是Active Internet connections，称为有源TCP连接，其中"Recv-Q"和"Send-Q"指%0A的是接收队列和发送队列。这些数字一般都应该是0。如果不是则表示软件包正在队列中堆积。这种情况只能在非常少的情况见到。



另一个是Active UNIX domain sockets，称为有源Unix域套接口\(和网络套接字一样，但是只能用于本机通信，性能可以提高一倍\)。

Proto显示连接使用的协议,RefCnt表示连接到本套接口上的进程号,Types显示套接口的类型,State显示套接口当前的状态,Path表示连接到套接口的其它进程使用的路径名。



常见参数

-a \(all\)显示所有选项，默认不显示LISTEN相关

-t \(tcp\)仅显示tcp相关选项

-u \(udp\)仅显示udp相关选项

-n 拒绝显示别名，能显示数字的全部转化成数字。

-l 仅列出有在 Listen \(监听\) 的服務状态



-p 显示建立相关链接的程序名

-r 显示路由信息，路由表

-e 显示扩展信息，例如uid等

-s 按各个协议进行统计

-c 每隔一个固定时间，执行该netstat命令。



提示：LISTEN和LISTENING的状态只有用-a或者-l才能看到

