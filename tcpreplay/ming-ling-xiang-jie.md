P:&lt;list&gt; - Must be one of the listed packets where the list corresponds to the packet number in the capture file. Ex: -xP:1-5,9,15 would only send packets 1 through 5, 9 and 15. 根据参数后的参数值（报文编号）发送指定的报文。可以在 ethereal 中确认报 文的编号，然后把需要的报文发送。可以用于排除 ARP 报文。 F:"&lt;filter&gt;" - BPF filter. See the tcpdump\(8\) man page for syntax. 未知，以后补充。 -X &lt;match&gt; Send all the packets except those specified 可选参数，就是-x 参数的取反，参数内容也是一样。 -v Verbose 可选参数， 显示 trpprep 生成 cache 文件的处理过程， 就是一些信息的即时打印。 -V Version

显示版本号。 Tcpprep 使用小结 再构造 cache 文件的过程中我用的比较多的选项参数就-v、-P、-xB、-xP，一般 都是 client 和 server 的模式，其它两种模式没有实验过，暂时还不知道怎么使 用，bridge 模式我使用过一次，结果发现报文是从一个网卡送出。 对于 tcp 和 udp 协议都做了测试，是可以支持的，icmp 还没有成功。对于网络 上的 BT 报文，只要你有 pcap 文件，也是可以构造 cache 文件来模拟完全真实的 BT 流量。 目前的使用就是这么多，感觉还是很有用的，tcpreplay 的参数有一部分是和 tcpprep 重复，下面的帮助文件说明就不详细说明了，但是特殊有好用 的参数 会使用蓝色字体标记出来给予重视。 存在的不足是还没有学会在 nat 模式下重放 报文，现在所有的报文重放都是在透明模式下完成的。 



Tcpreplay 帮助文件说明 Usage: tcpreplay \[args\] &lt;file\(s\)&gt; 





-A "&lt;args&gt;" Pass arguments to tcpdump decoder \(use w/ -v\) 可选参数， 在使用 tcpdump 风格打印输出信息时， 同时再调用 tcpdump 中的参数， 默认已经带有“-n,-l”，所以一般看到的都是 ip 地址，而没 有主机名的打印， 注意这个是在 tcpreplay 使用了-v 参数时，才能使用，不带-v 不会报错，但是 没有实际意义。格式：-vA “nnt”表示以 tcpdump 风格输出报文信息，并且不 打印时间戳、主机名、端口服务名称。注意不要使用-c 参数来指定打印的数据 报文的个数，这样发送 出去的报文也会变少。



 -b Bridge two broadcast domains in sniffer mode 可选参数，没有用过 



-c &lt;cachefile&gt; Split traffic via cache file 双网卡回放报文必选参数，后面紧跟 cache 文件名，该文件为 tcpprep 根据对应 的 pcap 文件构造出来。 



-C &lt;CIDR1,CIDR2,...&gt; Split traffic by matching src IP 可选参数，



 -D Data dump mode \(set this BEFORE -w and -W\)可选参数，把应用层的数据，使用 dump mode 写入到指定文件中去，和-w、-W 参数一起使用。 



-e &lt;ip1:ip2&gt; Specify IP endpoint rewriting 可选参数，指定端点的 ip，即把发送报文的和接收的报文的 ip 都修改称对应的 参数值中指定的 ip，但是这样发送的出的报文不会区分 client 和 server，还没 有发现使用的地方。 



-f &lt;configfile&gt; Specify configuration file 可选参数，指定配置文件，目前不会使用。



 -F Fix IP, TCP, UDP and ICMP checksums 可选参数，在发送报文时，自动纠正错误的校验和。对测试 DUT 的校验和检验还 是有用的。



 -h Help 显示帮助文件。 



-i &lt;nic&gt; Primary interface to send traffic out of 双网卡回放报文必选参数，指定主接口。



 -I &lt;mac&gt; Rewrite dest MAC on primary interface 可选参数，重写主网卡发送出报文的目的 MAC 地址。 



-j &lt;nic&gt; Secondary interface to send traffic out of 双网卡回放报文必选参数，指定从接口。



 -J &lt;mac&gt; Rewrite dest MAC on secondary interface 可选参数，重写从网卡发送出报文的目的 MAC 地址。 



-k &lt;mac&gt; Rewrite source MAC on primary interface 可选参数，重写主网卡发送报文的源 MAC 地址。 



-K &lt;mac&gt; Rewrite source MAC on secondary interface可选参数，重写从网卡发送报文的源 MAC 地址。



 -l &lt;loop&gt; Specify number of times to loop 可选参数，指定循环的次数，测试过程发现不是那么好用，有待确认。



 -L &lt;limit&gt; Specify the maximum number of packets to send 可选参数，指定最大的发包数量。可以在确认连接的调试时使用。 



-m &lt;multiple&gt; Set replay speed to given multiple 可选参数，指定一个倍数值，就是必默认发送速率要快多少倍的速率发送报文。 加大发送的速率后，对于 DUT 可能意味着有更多的并发连接和连接数，特别是对 于 BT 报文的重放， 因为连接的超时是固定的， 如果速率增大的话， 留在 session 表中的连接数量增大，还可以通过修改连接的超时时间来达到该目的。



 -M Disable sending martian IP packets 可选参数，表示不发送“火星”的 ip 报文，man 文件中的定义是 0/8、172/8、 255/8。 -



n Not nosy mode \(not promisc in sniff/bridge mode\) 可选参数，在使用-S 参数，不对混杂模式进行侦听。没有测试过。



 -N &lt;CIDR1:CIDR2,...&gt; Rewrite IP's via pseudo-NAT 可选参数，通过伪造的 NAT，重写 IP 地址。这个参数应该有很重要的应用，目 前没有测试使用。



 -O One output mode 可选参数，没有测试使用 



-p &lt;packetrate&gt; Set replay speed to given rate \(packets/sec\) 可选参数， 指定每秒发送报文的个数， 指定该参数， 其它速率相关的参数被忽略， 最后的打印信息不会有速率和每秒发送报文的统计。



 -P Print PID 可选参数，表示在输出信息中打印 PID 的信息，用于单用户或单帐户模式下暂停 和重启程序。



-r &lt;rate&gt; Set replay speed to given rate \(Mbps\) 可选参数，指定发送的速率。目前-m/-r/-p 这 3 个参数的相互关系还需要确认。



 -R Set replay speed to as fast as possible 可选参数，让网卡极限速度发数据包。 



-s &lt;seed&gt; Randomize src/dst IP addresses w/ given seed 可选参数，



 -S &lt;snaplen&gt; Sniff interface\(s\) and set the snaplen length 可选参数，



 -t &lt;mtu&gt; Override MTU \(defaults to 1500\) 可选参数，指定 MTU，标准的 10/100M 网卡的默认值是 1500。 



-T Truncate packets &gt; MTU so they can be sent 可选参数，截去报文中 MTU 大于标准值的部分再发送出去，默认是不发送，skip 掉。目前还有疑问，为什么会产生 MTU 大于 1500 字节的包，在 BT 报文中，这种 包比较常见。 -u pad\|trunc Pad/Truncate packets which are larger than the snaplen 可选参数， 后面的参数值二选一， snaplen 是指保留数据包的长度， 这里的 trunc 参数值和 MTU 没有任何关系，不要混淆。 -v Verbose: print packet decodes for each packet sent 可选参数，没发送一个报文都以 tcpdump 的风格打印出对应的信息。 -V Version 查看版本号。 -w &lt;file&gt; Write \(primary\) packets or data to file 可选参数，将主网卡发送的报文写入一个文件中，参数后紧跟文件名。



-W &lt;file&gt; Write secondary packets or data to file 可选参数，将从网卡发送的报文写入一个文件中，参数后紧跟文件名。 -x &lt;match&gt; Only send the packets specified 可选参数，发送匹配参数值的报文，这里各个参数具体的含义和 tcpprep 中的一 样， S:&lt;CIDR1&gt;,... - Src IP must match specified CIDR\(s\) 在 CIDR 模式下必须匹配源 IP，格式：-xS:100.1.1.0/24,10.10.10.0/26。多个 用逗号隔开，参数个数没有试过，3 个没有问题。 D:&lt;CIDR1&gt;,... - Dst IP must match specified CIDR\(s\) 在 CIDR 模式下必须匹配目的 IP，格式同上。 B:&lt;CIDR1&gt;,... - Both src and dst addresses must match 必须同时匹配源和目的 IP，格式同上。 E:&lt;CIDR1&gt;,... - Either src or dst address must match 匹配源或目的 IP，格式同上。 P:&lt;list&gt; - Must be one of the listed packets where the list corresponds to the packet number in the capture file. Ex: -xP:1-5,9,15 would only send packets 1 through 5, 9 and 15. 根据参数后的参数值（报文编号）发送指定的报文。可以在 ethereal 中确认报 文的编号，然后把需要的报文发送。可以用于排除 ARP 报文。 F:"&lt;filter&gt;" - BPF filter. See the tcpdump\(8\) man page for syntax. 未知，以后补充。 -X &lt;match&gt; Send all the packets except those specified 可选参数，-x 的参数内容取反。参数内容一样。 -1 Send one packet per key press

可选参数，参数内容就是阿拉伯数字 1，这个参数对于确定连接的建立，相当好 用，根据按回车键发送报文，可以将报文一个一个发送，来判断连接的状态。也 可以用于故障定位。 -2 &lt;datafile&gt; Layer 2 data 可选参数，在 2 层加入数据。 -4 &lt;PORT1:PORT2,...&gt; Rewrite port numbers 可选参数，重写端口号，对于测试特殊端口的应用比较实用。 &lt;file1&gt; &lt;file2&gt; ... File list to replay 可选参数，没有实验过。 



配置实例 

1、 重放在客户端 ftp 连接的报文 

a、 在客户端使用 ethereal 抓包，存为 ftp.pcap 文件



 b、 将 ftp.pcap 文件进行 tcpprep 操作，制作 cache 文件。 



\[root@A ~\]\# tcpprep -an client -i ftp.pcap -o ftp.cache –v 



c、 将 DUT 设备的两个接口和 PC 的两个接口使用网线连接，使用 tcpreplay 重 放报文。注意防火墙的配置为网桥（透明）模式。 



\[root@A ~\]\# tcpreplay -c ftp.cache -i eth0 -j eth1 ftp.pcap -R –v 



-R 参数表示全速发送，-v 显示打印信息。 



2、 重放在客户端 BT 连接的报文 

a、 在实验室 BT 下载一些台湾的娱乐节目和热门的大片，使用 ethereal 抓包， 存为 bt.pcap 文件。注意 pcap 文件大小的控制，对 pc 的内存要求比较高，我保 存了一个 600 多 M 的 pcap 文件用了 40 多分钟，大家有需要可以直接从实验室 copy。 



b、 将 bt.pcap 文件进行 tcpprep 操作，制作 cache 文件。



 \[root@A ~\]\# tcpprep -an client -i bt.pcap -o bt.cache -C "100M BT Packet" –v



制作 cache 文件，在 cache 文件中写入“100M BT Packet”的注释。 



c、 使用 tcpreplay 重放报文。 



\[root@A ~\]\# tcpreplay -c bt.cache -i eth0 -j eth1 bt.pcap -v –R 



3、 重放 tftp 服务器上抓到的报文 

a、 在 tftp 服务器上使用 ethereal 抓包，存为 tftp.pcap 文件。 



b、 将 pcap 文件进行 tcpprep 的操作，制作 cache 文件。 



\[root@A ~\]\# tcpprep -an server -i tftp.pcap -o tftp.cache –v 



注意：我在测试的时候犯了一个错误，使用 DUT 的 tftp 升级来做实验，同时穿 过 DUT 重放报文，结果在网卡发送报文的后，DUT 的 mac 地址做了的回应，导致 交互过程没有穿过 DUT，这个问题比较搞笑，上午弄了半天才发现原因，开始还 以为 udp 的连接不能重放。 



c、 使用 tcpreplay 重放报文。 



\[root@A ~\]\# tcpreplay -c tftp.cache -i eth0 -j eth1 tftp.pcap –v 







由于时间问题，这次不能对 man 文件一一做解释，这个说明文档主要是对-h 打印出来的命 令参数作一个说明， 结合几个实际的例子来说明 tcpprep 的使用。 强烈建议大家去官方网站去阅 读他们提供的文档，http://tcpreplay.synfin.net/trac/，我这里有打印的内容，有兴 趣的可以拿去看一下。





tcpprep帮助文件说明

Usage: tcpprep \[-a -n &lt;mode&gt; -N &lt;type&gt; \| -c &lt;cidr&gt; \| -p \| -r &lt;regex&gt;\]

-o &lt;out&gt; -i &lt;in&gt; &lt;args&gt;

-a &lt;st1:city w:st="on"&gt;&lt;st1:place w:st="on"&gt;Split&lt;/st1:place&gt;&lt;/st1:city&gt; traffic in Auto Mode&lt;o:p&gt;&lt;/o:p&gt;

一般情况下都需要该参数，表示按模式自动分离的通讯流量生成 cache 文件，这个参数一 半都和-n 参数一起使用，表示自动分离采取的拓扑模式，来决定采取那种模式分离通讯流量的 双方。

-c CIDR1,CIDR2,...

Split traffic in CIDR Mode

可选参数，表示分离流量时采用 CIDR（无类别域间路由选择）模式。格式：tcpprep

-ac

&lt;st1:chsdate w:st="on" isrocdate="False" islunardate="False" day="30" month="12" year="1899"&gt;10.10.0&lt;/st1:chsdate&gt;.0/24，表示把源地址匹配 10.10.0.0/24 网段的报文全部 由主网卡发送，剩下的报文由从网卡发送出来，这里还有一点需要补充，就是 tcpreplay 在重放 报文时对两个网卡的定义很明确， 一个主网卡 （primary interface） 一个是从网卡 ， （secondary interface） 不同的模式， ， 两块网卡的属性不一样。 该参数不能和-r,-a 一起使用。 &lt;o:p&gt;&lt;/o:p&gt;

-C &lt;comment&gt;

Embed comment in tcpprep cache file

可选参数，表示在 cache 文件中嵌入注释内容，可以用于注释说明 cache 文件的内容，注 意使用时参数位置，不要放在最后，我测试时放在-o 参数值的后面就报错，放到-i 参数之前就 可以。生成 cache 文件后使用-P 可以查看写入的内容。

-h

Help

显示帮助文件。

-i &lt;capfile&gt;

Input capture file to process

生成 cache 文件的必带参数，后面紧跟 pcap 文件名，表示这个 pcap 文件需要处理。

-m &lt;minmask&gt;

Minimum mask length in Auto/Router mode

可选参数，在选用 router 模式时使用，表示最小掩码，默认是 30（2 个有效 ip 地址）。

-M &lt;maxmask&gt;

Maximum mask length in Auto/Router mode

可选参数，在选用 router 模式时使用，表示最大掩码，默认是 8（1600 万个 ip 地址）。

-n &lt;auto mode&gt;

Use specified algorithm in Auto Mode

生成 cache 文件的必带参数，后面紧跟模式名称，可选项有 \(bridge\|router\|client\|server\)，目前&lt;st1:chsdate

w:st="on" isrocdate="False"

islunardate="False" day="30" month="12" year="1899"&gt;2.3.5&lt;/st1:chsdate&gt;

版本只支持这 4 种模式。模式的选择很关键，例如在客户端使用 ftp 软件下载文件，那么你在客 户端抓到的报文生成的 pcap 文件， 那么就选用 client 模式， 在服务器端抓到的报文生成的 pcap 文件就选用 server 模式。 只有模式选对了， 才能正确的分离流量从正确的接口发出正确的报文。 注意：Server 端的报文由主网卡发送出去，Client 端的报文由从网卡发送出去。怎么确定主从 网卡由 tcpreplay 的命令（-i –j 两个参数）来决定。

-N client\|server

Classify non-IP traffic as client/server

可选参数，表示非 IP 的流量（例如 ARP 报文）从哪个接口送出，因为很多的 tcpprcp 支持 的模式中， 都依赖于 IP 头部中的 IP 地址信息来决定报文是从 client 端还是从 server 端发送出 去。但是并不是所有的报文都是 IPv4 结构的，所以这种情况下，tcpprep 不能确定这些非 IPv4 类型的报文应该从哪个接口发送出去，所以，默认的配置就是从 client 的接口发送出去。如果 你硬要正确的分离出非 IPv4 报文的话，可以使用 MAC address 模式（--mac）。3.0 版本才支持。







离通讯流量，它区分的依据是认为 0-1023 端口都是服务器的 端发出的报文，其它的端口都是客户端发出的报文，具体的端口对应的/etc/services 文件里的 的内容。使用的格式：-p /etc/services，可以根据自己的需要来制作一个文件也可以。

-P &lt;file&gt;

Print comment in tcpprep file

可选参数，查看 cache 文件的内容。

-r &lt;regex&gt;

&lt;st1:place w:st="on"&gt;&lt;st1:city

w:st="on"&gt;Split&lt;/st1:city&gt;&lt;/st1:place&gt; traffic in Regex Mode

可选参数，表示使用 Regex 模式分离通讯流量，有点类似于 CIDR 模式，但是它匹配的是服 务器的源 IP。man 文件提示不能和-a、-c 参数一起使用，但是我使用了也没有报错，格式：-r "\(192\)"或-r "\(192\|172\)\.....\*"，具体应用还有待实验。

-R &lt;ratio&gt;

Specify a ratio to use in Auto Mode

可选参数， 一个比例值， 这个比例值的意义是服务器端发起的连接数和客户端发起的连接数

的比例，这个值大于 2 的话就视为 server 端。这个英文原意我也不是太肯定，大家可以参考一 下原文：

The ratio of server connections to client connections as a server in auto mode.

necessary to

be classified

A system is classified as a server if \[\# server connections\] &gt;= Default is: 2.0

\(\[\# client connections\] \* \[ratio\]\).

-s &lt;file&gt;

Specify service ports in /etc/services format

可选参数，在 man 文件中没有对该参数的解释，估计就是按/etc/services 文件里的格式来 定义服务的端口，没有太多的研究意义。

-x &lt;match&gt;

Only send the packets specified

重要的可选参数，表示按照参数定义的需求来定义发送报文。后面还有具体的参数，因为在 我们的抓包过程中，可能会由于网络环境原因，抓到了许多我们不需要回放的报文，我们就可以 根据这个参数决定我们需要回放哪些报文内容。具体的参数意思如下：

S:&lt;CIDR1&gt;,... - Src IP must match specified CIDR\(s\)

在 CIDR 模式下必须匹配源 IP，格式：-xS:100.1.1.0/24,&lt;st1:chsdate w:st="on" year="1899" month="12" day="30" islunardate="False" isrocdate="False"&gt;10.10.10&lt;/st1:chsdate&gt;.0/26。多个用逗号隔开，参数个数没有试过，3 个没有问题。

D:&lt;CIDR1&gt;,... - Dst IP must match specified CIDR\(s\)

在 CIDR 模式下必须匹配目的 IP，格式同上。

B:&lt;CIDR1&gt;,... - Both src and dst addresses must match

必须同时匹配源和目的 IP，格式同上。

E:&lt;CIDR1&gt;,... - Either src or dst address must match

匹配源或目的 IP，格式同上。

