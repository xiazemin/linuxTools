tcpreplay是一种pcap包的重放工具, 它可以将用ethreal, wireshark工具抓下来的包原样或经过任意修改后重放回去. 它允许你对报文做任意的修改\(主要是指对2层, 3层, 4层报文头\), 指定重放报文的速度等, 这样tcpreplay就可以用来复现抓包的情景以定位bug, 以极快的速度重放从而实现压力测试。

tcpreplay本身包含了几个辅助工具（tcpprep、tcprewrite、tcpreplay和tcpbridge）, 用于准备发包的cache, 重写报文等。这也是 Tcpreplay 的第一个字母大写T的原因 。

\* tcpprep - 简单的说就是划分哪些包是client的, 哪些是server的, 一会发包的时候client的包从一个网卡发, server的包可能从另一个网卡发。

即区分pcap数据包的流向，即区分出客户端和服务器。　　\\* tcprewrite - 简单的说就是修改2层, 3层, 4层报文头部。

即改写pcap数据包的2-4层的头部信息，即MAC地址、IP地址和PORT等。　　\\* tcpreplay - 真正发包, 可以选择主、从网卡, 发包速度等。

即回放pcap文件中的数据包。

\* tcpreplay-edit-更写pcap数据并回放，将tcprewrite和tcpreplat一条命令实现。

\\* tcpbridge - bridge two network segments with the power of tcprewrite。

以下是官网的原文。

The Tcpreplay suite includes the following tools:

tcpprep - multi-pass pcap file pre-processor which determines packets as client or server and creates cache files used by tcpreplay and tcprewrite

tcprewrite - pcap file editor which rewrites TCP/IP and Layer 2 packet headers

tcpreplay - replays pcap files at arbitrary speeds onto the network

tcpliveplay - Replays network traffic stored in a pcap file on live networks using new TCP connections

tcpreplay-edit - replays & edits pcap files at arbitrary speeds onto the network

tcpbridge - bridge two network segments with the power of tcprewrite

tcpcapinfo - raw pcap file decoder and debugger

在CentOS里安装tcpreplay，总的来说，有如下两种安装方法。



 



　　第一种安装方法：（推荐）



　　直接yum -y install tcpreplay即可。

