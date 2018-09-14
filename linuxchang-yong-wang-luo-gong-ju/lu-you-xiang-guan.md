

nc

nc\(netcat\)被称为网络工具的瑞士军刀，其非常轻巧但功能强大！常常作为网络应用的Debug分析器，可以根据需要创建各种不同类型的网络连接。官方描述的功能包括:



simple TCP proxies

shell-script based HTTP clients and servers

network daemon testing

a SOCKS or HTTP ProxyCommand for ssh\(1\)

and much, much more

总之非常强大，能够实现简单的聊天工具、模拟ssh登录远程主机、远程传输文件等。一个经典的用法是端口扫描。比如我要扫描192.168.56.2主机1~100端口，探测哪些端口开放的（黑客攻击必备）：



fgp@controller:~$ nc -zv 192.168.56.2 1-100 \|& grep 'succeeded!'

Connection to 192.168.56.2 22 port \[tcp/ssh\] succeeded!

Connection to 192.168.56.2 80 port \[tcp/http\] succeeded!

从结果中发现，该主机打开了22和80端口。



tcpdump

tcpdump\(dump traffic on a network\)是一个强大的命令行抓包工具，千万不要被它的名称误导以为只能抓取tcp包，它能抓任何协议的包。它能够实现Wireshark一样的功能，并且更加灵活自由！比如需要抓取目标主机是192.168.56.1，通过端口22的传输数据包：



sudo tcpdump -n -i eth1 'dst host 192.168.56.1 && port 22'

输出为：



tcpdump: verbose output suppressed, use -v or -vv for full protocol decode

listening on eth1, link-type EN10MB \(Ethernet\), capture size 65535 bytes

23:57:39.507490 IP 192.168.56.2.22 &gt; 192.168.56.1.54558: Flags \[P.\], seq 3010719012:3010719120, ack 1116715283, win 354, options \[nop,nop,TS val 1049052 ecr 187891473\], length 108

23:57:39.507607 IP 192.168.56.2.22 &gt; 192.168.56.1.54558: Flags \[P.\], seq 108:144, ack 1, win 354, options \[nop,nop,TS val 1049052 ecr 187891473\], length 36

23:57:39.507784 IP 192.168.56.2.22 &gt; 192.168.56.1.54558: Flags \[P.\], seq 144:252, ack 1, win 354, options \[nop,nop,TS val 1049052 ecr 187891476\], length 108

抓取HTTP包:



sudo tcpdump  -XvvennSs 0 -i eth0 tcp\[20:2\]=0x4745 or tcp\[20:2\]=0x4854

其中0x4745为"GET"前两个字母"GE",0x4854为"HTTP"前两个字母"HT"。



指定-A以ACII码输出数据包，使用-c指定抓取包的个数。

