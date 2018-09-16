tcpdump 与wireshark



Wireshark\(以前是ethereal\)是Windows下非常简单易用的抓包工具。但在Linux下很难找到一个好用的图形化抓包工具。

还好有Tcpdump。我们可以用Tcpdump + Wireshark 的完美组合实现：在 Linux 里抓包，然后在Windows 里分析包。



tcpdump tcp -i eth1 -t -s 0 -c 100 and dst port ! 22 and src net 192.168.1.0/24 -w ./target.cap

\(1\)tcp: ip icmp arp rarp 和 tcp、udp、icmp这些选项等都要放到第一个参数的位置，用来过滤数据报的类型

\(2\)-i eth1 : 只抓经过接口eth1的包

\(3\)-t : 不显示时间戳

\(4\)-s 0 : 抓取数据包时默认抓取长度为68字节。加上-S 0 后可以抓到完整的数据包

\(5\)-c 100 : 只抓取100个数据包

\(6\)dst port ! 22 : 不抓取目标端口是22的数据包

\(7\)src net 192.168.1.0/24 : 数据包的源网络地址为192.168.1.0/24

\(8\)-w ./target.cap : 保存成cap文件，方便用ethereal\(即wireshark\)分析



 



使用tcpdump抓取HTTP包



tcpdump  -XvvennSs 0 -i eth0 tcp\[20:2\]=0x4745 or tcp\[20:2\]=0x4854

0x4745 为"GET"前两个字母"GE",0x4854 为"HTTP"前两个字母"HT"。



 



tcpdump 对截获的数据并没有进行彻底解码，数据包内的大部分内容是使用十六进制的形式直接打印输出的。显然这不利于分析网络故障，通常的解决办法是先使用带-w参数的tcpdump 截获数据并保存到文件中，然后再使用其他程序\(如Wireshark\)进行解码分析。当然也应该定义过滤规则，以避免捕获的数据包填满整个硬盘。



 



输出信息含义

首先我们注意一下，基本上tcpdump总的的输出格式为：系统时间 来源主机.端口 &gt; 目标主机.端口 数据包参数



tcpdump 的输出格式与协议有关.以下简要描述了大部分常用的格式及相关例子.



链路层头

对于FDDI网络, '-e' 使tcpdump打印出指定数据包的'frame control' 域, 源和目的地址, 以及包的长度.\(frame control域

控制对包中其他域的解析\). 一般的包\(比如那些IP datagrams\)都是带有'async'\(异步标志\)的数据包，并且有取值0到7的优先级;

比如 'async4'就代表此包为异步数据包，并且优先级别为4. 通常认为,这些包们会内含一个 LLC包\(逻辑链路控制包\); 这时,如果此包

不是一个ISO datagram或所谓的SNAP包，其LLC头部将会被打印\(nt:应该是指此包内含的 LLC包的包头\).



对于Token Ring网络\(令牌环网络\), '-e' 使tcpdump打印出指定数据包的'frame control'和'access control'域, 以及源和目的地址,

外加包的长度. 与FDDI网络类似, 此数据包通常内含LLC数据包. 不管 是否有'-e'选项.对于此网络上的'source-routed'类型数据包\(nt:

意译为:源地址被追踪的数据包,具体含义未知,需补充\), 其包的源路由信息总会被打印.





对于802.11网络\(WLAN,即wireless local area network\), '-e' 使tcpdump打印出指定数据包的'frame control域,

包头中包含的所有地址, 以及包的长度.与FDDI网络类似, 此数据包通常内含LLC数据包.



\(注意: 以下的描述会假设你熟悉SLIP压缩算法 \(nt:SLIP为Serial Line Internet Protocol.\), 这个算法可以在

RFC-1144中找到相关的蛛丝马迹.\)



对于SLIP网络\(nt:SLIP links, 可理解为一个网络, 即通过串行线路建立的连接, 而一个简单的连接也可看成一个网络\),

数据包的'direction indicator'\('方向指示标志'\)\("I"表示入, "O"表示出\), 类型以及压缩信息将会被打印. 包类型会被首先打印.



类型分为ip, utcp以及ctcp\(nt:未知, 需补充\). 对于ip包,连接信息将不被打印\(nt:SLIP连接上,ip包的连接信息可能无用或没有定义.

reconfirm\).对于TCP数据包, 连接标识紧接着类型表示被打印. 如果此包被压缩, 其被编码过的头部将被打印.

此时对于特殊的压缩包,会如下显示:

\*S+n 或者 \*SA+n, 其中n代表包的\(顺序号或\(顺序号和应答号\)\)增加或减少的数目\(nt \| rt:S,SA拗口, 需再译\).

对于非特殊的压缩包,0个或更多的'改变'将会被打印.'改变'被打印时格式如下:

'标志'+/-/=n 包数据的长度 压缩的头部长度.

其中'标志'可以取以下值:

U\(代表紧急指针\), W\(指缓冲窗口\), A\(应答\), S\(序列号\), I\(包ID\),而增量表达'=n'表示被赋予新的值, +/-表示增加或减少.



比如, 以下显示了对一个外发压缩TCP数据包的打印, 这个数据包隐含一个连接标识\(connection identifier\); 应答号增加了6,

顺序号增加了49, 包ID号增加了6; 包数据长度为3字节\(octect\), 压缩头部为6字节.\(nt:如此看来这应该不是一个特殊的压缩数据包\).



ARP/RARP 数据包



tcpdump对Arp/rarp包的输出信息中会包含请求类型及该请求对应的参数. 显示格式简洁明了. 以下是从主机rtsg到主机csam的'rlogin'

\(远程登录\)过程开始阶段的数据包样例:

arp who-has csam tell rtsg

arp reply csam is-at CSAM

第一行表示:rtsg发送了一个arp数据包\(nt:向全网段发送,arp数据包）以询问csam的以太网地址

Csam（nt:可从下文看出来, 是Csam）以她自己的以太网地址做了回应\(在这个例子中, 以太网地址以大写的名字标识, 而internet

地址\(即ip地址\)以全部的小写名字标识\).



如果使用tcpdump -n, 可以清晰看到以太网以及ip地址而不是名字标识:

arp who-has 128.3.254.6 tell 128.3.254.68

arp reply 128.3.254.6 is-at 02:07:01:00:01:c4



如果我们使用tcpdump -e, 则可以清晰的看到第一个数据包是全网广播的, 而第二个数据包是点对点的:

RTSG Broadcast 0806 64: arp who-has csam tell rtsg

CSAM RTSG 0806 64: arp reply csam is-at CSAM

第一个数据包表明:以arp包的源以太地址是RTSG, 目标地址是全以太网段, type域的值为16进制0806\(表示ETHER\_ARP\(nt:arp包的类型标识\)\),

包的总长度为64字节.

