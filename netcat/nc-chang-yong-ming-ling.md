nc\(NetCat\)，在网络工具中有”瑞士军刀”的美誉，它短小精悍，功能强大，下面分享一些我平时经常用到的功能，更多的功能请google之。



1.基本参数

想要连接到某处: nc \[-options\] hostname port\[s\] \[ports\] …

绑定端口等待连接: nc -l -p port \[-options\] \[hostname\] \[port\]

参数:

-g gateway source-routing hop point\[s\], up to 8

-G num source-routing pointer: 4, 8, 12, …

-h 帮助信息

-i secs 延时的间隔

-l 监听模式，用于入站连接

-n 指定数字的IP地址，不能用hostname

-o file 记录16进制的传输

-p port 本地端口号

-r 任意指定本地及远程端口

-s addr 本地源地址

-u UDP模式

-v 详细输出——用两个

-v可得到更详细的内容

-w secs timeout的时间

-z 将输入输出关掉——用于扫描时，其中端口号可以指定一个或者用lo-hi式的指定范围。



2.简单用法举例

1）端口扫描

\# nc -v -w 2 192.168.2.34 -z 21-24

nc: connect to 192.168.2.34 port 21 \(tcp\) failed: Connection refused

Connection to 192.168.2.34 22 port \[tcp/ssh\] succeeded!

nc: connect to 192.168.2.34 port 23 \(tcp\) failed: Connection refused

nc: connect to 192.168.2.34 port 24 \(tcp\) failed: Connection refused



2\)从192.168.2.33拷贝文件到192.168.2.34

在192.168.2.34上： nc -l 1234 &gt; test.txt

在192.168.2.33上： nc 192.168.2.34 &lt; test.txt



3\)简单聊天工具

在192.168.2.34上： nc -l 1234

在192.168.2.33上： nc 192.168.2.34 1234

这样，双方就可以相互交流了。使用ctrl+C\(或D）退出。



3.用nc命令操作memcached

1）存储数据：printf “set key 0 10 6\r\nresult\r\n” \|nc 192.168.2.34 11211

2）获取数据：printf “get key\r\n” \|nc 192.168.2.34 11211

3）删除数据：printf “delete key\r\n” \|nc 192.168.2.34 11211

4）查看状态：printf “stats\r\n” \|nc 192.168.2.34 11211

5）模拟top命令查看状态：watch “echo stats” \|nc 192.168.2.34 11211

6）清空缓存：printf “flush\_all\r\n” \|nc 192.168.2.34 11211 \(小心操作，清空了缓存就没了）







4. 使用nc来传文件



发送端：   cat a.txt  \|  nc -l  3333



接收端：   nc 192.168.0.3    3333 &gt;   a.txt







或者







发送端:      cat a.txt     \|   nc 192.168.0.3     9999 



接收端:      nc -l 9999 &gt; a.txt

