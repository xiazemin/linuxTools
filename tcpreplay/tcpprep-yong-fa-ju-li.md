tcpprep 是一个在 tcprewrite 和 tcpreplay 之前使用的 pcap 文件的处理程序。使用 tcpprep 的目的就是建立一个 cach 文件，用于分离通信流量中的两方（通常叫做 主要的/次要的 或者 客户端/服务器），为 tcprewrite 和 tcpreplay 处理与发送报文做准备。



　　如果你正打算在两块网卡上使用 tcpreplay 的话，那么 tcpprep 就是用来决定每一个报文（packet）从哪一个接口发出。通过使用这样一个分离的程序来建立一个 cach 文件，tcpreplay 就可以根据这个 cach 文件通过自身的计算来分离流量，高速率的发送报文。cach 文件的作用主要是加速报文的发送，cach 文件中存放着 pcap 文件中每个帧的编号和时间戳等信息，以达到 tcpreplay 回放时可以更加快速的发送报文的目的。 



　　举个例子, 如果你用 wireshark 或者ethereal抓了一个 pcap 文件，里面可能既有 A 地址发给 B 地址的包，也有 B地址发给 A 址的包，用 tcpprep 工具可以指定从 A 到 B 的包从主网卡发出, 从 B 到 A 的包从次网卡发出。 







　　

　　tcpprep 用法举例 

1、根据报文源 IP 确定 client/server 报文



\[root@datatest ~\]\#  tcpprep -c 172.22.64.2/24 -i mgcp.pcap -o mgcp.cach 

　　上面的命令指定所有源 IP 为 172.22.64.2/24 的包, 都将从主网卡发出， 其它的从次网卡发出 。输入文件是mgcp.pcap, 输出文件为mgcp.cach。



 



2、使用自动模式确定 client/server 报文 



 \[root@datatest ~\]\#  tcpprep -a client -i mgcp.pcap -o mgcp.cach

　　上面的命令采用自动/client 模式指定分包模式。



　　自动模式这里按我的理解解释一下：

　　tcpprep 在自动模式下认为有下面行为的 IP 为 client 端：



　　　　1、发 TCP SYN 包的一方



　　　　2、发 DNS 包的一方



　　　　3、 收入到 ICMP-Port Unreachable 的一方；



      认为有下面行为的一方为 server 端：



　　　　1、发 TCPSyn/Ack 的一方



　　　　2、发 DNS 应答的一方



　　　　3、发 ICMP-Port Unreachable 的一方



　　而被认定为 server 的那一方发的那些包, 将从主网卡发出, 被认定为 client 的包则从次网卡发出. 而自动/client 模式将所有没有认出的包都归为 client, 同理自动/server 模式将没有认出的包都归为 server.这种模式貌似不如按 IP 地址分类的方式好用 。

