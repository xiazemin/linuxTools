1. 列出所有端口 \(包括监听和未监听的\)

  列出所有端口 netstat -a



复制代码

\# netstat -a \| more

 Active Internet connections \(servers and established\)

 Proto Recv-Q Send-Q Local Address           Foreign Address         State

 tcp        0      0 localhost:30037         \*:\*                     LISTEN

 udp        0      0 \*:bootpc                \*:\*

 

Active UNIX domain sockets \(servers and established\)

 Proto RefCnt Flags       Type       State         I-Node   Path

 unix  2      \[ ACC \]     STREAM     LISTENING     6135     /tmp/.X11-unix/X0

 unix  2      \[ ACC \]     STREAM     LISTENING     5140     /var/run/acpid.socket

复制代码

  列出所有 tcp 端口 netstat -at



复制代码

\# netstat -at

 Active Internet connections \(servers and established\)

 Proto Recv-Q Send-Q Local Address           Foreign Address         State

 tcp        0      0 localhost:30037         \*:\*                     LISTEN

 tcp        0      0 localhost:ipp           \*:\*                     LISTEN

 tcp        0      0 \*:smtp                  \*:\*                     LISTEN

 tcp6       0      0 localhost:ipp           \[::\]:\*                  LISTEN

复制代码

  列出所有 udp 端口 netstat -au



\# netstat -au

 Active Internet connections \(servers and established\)

 Proto Recv-Q Send-Q Local Address           Foreign Address         State

 udp        0      0 \*:bootpc                \*:\*

 udp        0      0 \*:49119                 \*:\*

 udp        0      0 \*:mdns                  \*:\*

 

2. 列出所有处于监听状态的 Sockets

  只显示监听端口 netstat -l



\# netstat -l

 Active Internet connections \(only servers\)

 Proto Recv-Q Send-Q Local Address           Foreign Address         State

 tcp        0      0 localhost:ipp           \*:\*                     LISTEN

 tcp6       0      0 localhost:ipp           \[::\]:\*                  LISTEN

 udp        0      0 \*:49119                 \*:\*

  只列出所有监听 tcp 端口 netstat -lt



\# netstat -lt

 Active Internet connections \(only servers\)

 Proto Recv-Q Send-Q Local Address           Foreign Address         State

 tcp        0      0 localhost:30037         \*:\*                     LISTEN

 tcp        0      0 \*:smtp                  \*:\*                     LISTEN

 tcp6       0      0 localhost:ipp           \[::\]:\*                  LISTEN

  只列出所有监听 udp 端口 netstat -lu



\# netstat -lu

 Active Internet connections \(only servers\)

 Proto Recv-Q Send-Q Local Address           Foreign Address         State

 udp        0      0 \*:49119                 \*:\*

 udp        0      0 \*:mdns                  \*:\*

  只列出所有监听 UNIX 端口 netstat -lx



复制代码

\# netstat -lx

 Active UNIX domain sockets \(only servers\)

 Proto RefCnt Flags       Type       State         I-Node   Path

 unix  2      \[ ACC \]     STREAM     LISTENING     6294     private/maildrop

 unix  2      \[ ACC \]     STREAM     LISTENING     6203     public/cleanup

 unix  2      \[ ACC \]     STREAM     LISTENING     6302     private/ifmail

 unix  2      \[ ACC \]     STREAM     LISTENING     6306     private/bsmtp

复制代码





3. 显示每个协议的统计信息

  显示所有端口的统计信息 netstat -s



复制代码

\# netstat -s

 Ip:

 11150 total packets received

 1 with invalid addresses

 0 forwarded

 0 incoming packets discarded

 11149 incoming packets delivered

 11635 requests sent out

 Icmp:

 0 ICMP messages received

 0 input ICMP message failed.

 Tcp:

 582 active connections openings

 2 failed connection attempts

 25 connection resets received

 Udp:

 1183 packets received

 4 packets to unknown port received.

 .....

复制代码

  显示 TCP 或 UDP 端口的统计信息 netstat -st 或 -su



\# netstat -st 

\# netstat -su

 

4. 在 netstat 输出中显示 PID 和进程名称 netstat -p

netstat -p 可以与其它开关一起使用，就可以添加 “PID/进程名称” 到 netstat 输出中，这样 debugging 的时候可以很方便的发现特定端口运行的程序。



\# netstat -pt

 Active Internet connections \(w/o servers\)

 Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name

 tcp        1      0 ramesh-laptop.loc:47212 192.168.185.75:www        CLOSE\_WAIT  2109/firefox

 tcp        0      0 ramesh-laptop.loc:52750 lax:www ESTABLISHED 2109/firefox

5. 在 netstat 输出中不显示主机，端口和用户名 \(host, port or user\)

当你不想让主机，端口和用户名显示，使用 netstat -n。将会使用数字代替那些名称。



同样可以加速输出，因为不用进行比对查询。



\# netstat -an

如果只是不想让这三个名称中的一个被显示，使用以下命令



\# netsat -a --numeric-ports

\# netsat -a --numeric-hosts

\# netsat -a --numeric-users

 

6. 持续输出 netstat 信息

netstat 将每隔一秒输出网络信息。



复制代码

\# netstat -c

 Active Internet connections \(w/o servers\)

 Proto Recv-Q Send-Q Local Address           Foreign Address         State

 tcp        0      0 ramesh-laptop.loc:36130 101-101-181-225.ama:www ESTABLISHED

 tcp        1      1 ramesh-laptop.loc:52564 101.11.169.230:www      CLOSING

 tcp        0      0 ramesh-laptop.loc:43758 server-101-101-43-2:www ESTABLISHED

 tcp        1      1 ramesh-laptop.loc:42367 101.101.34.101:www      CLOSING

 ^C

复制代码

 

7. 显示系统不支持的地址族 \(Address Families\)

netstat --verbose

在输出的末尾，会有如下的信息



netstat: no support for \`AF IPX' on this system.

netstat: no support for \`AF AX25' on this system.

netstat: no support for \`AF X25' on this system.

netstat: no support for \`AF NETROM' on this system.

 

8. 显示核心路由信息 netstat -r

\# netstat -r

 Kernel IP routing table

 Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface

 192.168.1.0     \*               255.255.255.0   U         0 0          0 eth2

 link-local      \*               255.255.0.0     U         0 0          0 eth2

 default         192.168.1.1     0.0.0.0         UG        0 0          0 eth2

注意： 使用 netstat -rn 显示数字格式，不查询主机名称。



 

9. 找出程序运行的端口

并不是所有的进程都能找到，没有权限的会不显示，使用 root 权限查看所有的信息。



\# netstat -ap \| grep ssh

 tcp        1      0 dev-db:ssh           101.174.100.22:39213        CLOSE\_WAIT  -

 tcp        1      0 dev-db:ssh           101.174.100.22:57643        CLOSE\_WAIT  -

  找出运行在指定端口的进程



\# netstat -an \| grep ':80'

 

10. 显示网络接口列表

\# netstat -i

 Kernel Interface table

 Iface   MTU Met   RX-OK RX-ERR RX-DRP RX-OVR    TX-OK TX-ERR TX-DRP TX-OVR Flg

 eth0       1500 0         0      0      0 0             0      0      0      0 BMU

 eth2       1500 0     26196      0      0 0         26883      6      0      0 BMRU

 lo        16436 0         4      0      0 0             4      0      0      0 LRU

显示详细信息，像是 ifconfig 使用 netstat -ie:



复制代码

\# netstat -ie

 Kernel Interface table

 eth0      Link encap:Ethernet  HWaddr 00:10:40:11:11:11

 UP BROADCAST MULTICAST  MTU:1500  Metric:1

 RX packets:0 errors:0 dropped:0 overruns:0 frame:0

 TX packets:0 errors:0 dropped:0 overruns:0 carrier:0

 collisions:0 txqueuelen:1000

 RX bytes:0 \(0.0 B\)  TX bytes:0 \(0.0 B\)

 Memory:f6ae0000-f6b00000

复制代码

 

11. IP和TCP分析

  查看连接某服务端口最多的的IP地址



复制代码

wss8848@ubuntu:~$ netstat -nat \| grep "192.168.1.15:22" \|awk '{print $5}'\|awk -F: '{print $1}'\|sort\|uniq -c\|sort -nr\|head -20

18 221.136.168.36

3 154.74.45.242

2 78.173.31.236

2 62.183.207.98

2 192.168.1.14

2 182.48.111.215

2 124.193.219.34

2 119.145.41.2

2 114.255.41.30

1 75.102.11.99

复制代码

  TCP各种状态列表



复制代码

wss8848@ubuntu:~$ netstat -nat \|awk '{print $6}'

established\)

Foreign

LISTEN

TIME\_WAIT

ESTABLISHED

TIME\_WAIT

SYN\_SENT

复制代码

  先把状态全都取出来,然后使用uniq -c统计，之后再进行排序。

复制代码

wss8848@ubuntu:~$ netstat -nat \|awk '{print $6}'\|sort\|uniq -c

143 ESTABLISHED

1 FIN\_WAIT1

1 Foreign

1 LAST\_ACK

36 LISTEN

6 SYN\_SENT

113 TIME\_WAIT

1 established\)

复制代码

  最后的命令如下:

netstat -nat \|awk '{print $6}'\|sort\|uniq -c\|sort -rn

分析access.log获得访问前10位的ip地址

awk '{print $1}' access.log \|sort\|uniq -c\|sort -nr\|head -10

