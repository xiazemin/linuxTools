简介 

网络地址转换是映射整个网络（或者多个网络）到单个IP地址的方法。当你的ISP分配给你的IP地址数目少于你要连上互联网的计算机数目时，NAT是必需的。NAT在RFC1631中描述。 



NAT允许使用RFC1918中定义的保留地址族。典型情况下，你的内部网络可以下面的地址族： 

10.0.0.0/8 \(10.0.0.0 - 10.255.255.255\) 

172.16.0.0/12 \(172.16.0.0 - 172.31.255.255\) 

192.168.0.0/16 \(192.168.0.0 - 192.168.255.255\) 



担任NAT任务的openbsd系统必须至少要2块网卡，一块连接到因特网，另一块连接内部网络。NAT会转换内部网络的所有请求，使它们看起来象是来自进行NAT工作的openbsd主机系统。 



NAT如何工作 



当内部网络的一个客户端连接因特网上的主机时，它发送目的是那台主机的IP数据包。这些数据包包含了达到连接目的的所有信息。NAT的作用和这些信息相关。 



\* 源 IP 地址 \(例如, 192.168.1.35\) 

\* 源 TCP 或者 UDP 端口 \(例如, 2132\) 

当数据包通过NAT网关时，它们会被修改，使得数据包看起来象是从NAT网关自己发出的。NAT网关会在它的状态表中记录这个改变，以便：a）将返回的数据包反向转换和b）确保返回的数据包能通过防火墙而不会被阻塞。例如，会发生下面的改变： 



\* 源 IP: 被网关的外部地址所替换\(例如， 24.5.0.5\) 

\* 源端口:被随机选择的网关没有在用的端口替换 \(例如， 53136\) 



内部主机和因特网上的主机都不会意识到发生了这个转变步骤。对于内部主机，NAT系统仅仅是个因特网网关，对于因特网上的主机，数据包看起来直接来自NAT系统，它完全不会意识到内部工作站的存在。 



当因特网网上的主机回应内部主机的数据包时，它会使用NAT网关机器的外部地址和转换后的端口。然后NAT网关会查询状态表来确定返回的数据包是否匹配某 个已经建立的连接。基于IP地址和端口的联合找到唯一匹配的记录告诉PF这个返回的数据包属于内部主机192.168.1.35。然后PF会进行和出去的 数据包相反的转换过程，将返回的数据包传递给内部主机。 



ICMP数据包的转换也是类似的，只是不进行源端口的修改。 



NAT和包过滤 



注意：转换后的数据包仍然会通过过滤引擎，根据定义的过滤规则进行阻塞或者通过。唯一的例外是如果nat规则中使用了pass关键字，会使得经过nat的数据包直接通过过滤引擎。 



还要注意由于转换是在过滤之前进行，过滤引擎所看到的是上面“nat如何工作”中所说的经过转换后的ip地址和端口的数据包。 



IP 转发 



由于NAT经常在路由器和网关上使用，因此IP转发是需要的，使得数据包可以在openbsd机器的不同网络接口间传递。IP转发可以通过sysctl（3）命令打开： 



\# sysctl -w net.inet.ip.forwarding=1 

\# sysctl -w net.inet6.ip6.forwarding=1 \(if using IPv6\) 



要使这个变化永久生效，可以增加如下行到/etc/sysctl.conf文件中： 



net.inet.ip.forwarding=1 

net.inet6.ip6.forwarding=1 



这些行是本来就存在的，但默认安装中被\#前缀注释掉了。删除\#，保存文件，IP转发在机器重启后就会发生作用。 



配置NAT 



一般的NAT规则格式在pf.conf文件中是这个样子： 



nat \[pass\] on interface \[af\] from src\_addr \[port src\_port\] to \ 

dst\_addr \[port dst\_port\] -&gt; ext\_addr \[pool\_type\] \[static-port\] 



nat 

开始NAT规则的关键字。 

pass 

使得转换后的数据包完全绕过过滤规则。 

interface 

进行数据包转换的网络接口。 

af 

地址类型，inet代表Ipv4，inet6代表Ipv6。PF通常能根据源/目标地址自动确定这个参数。 

src\_addr 

被进行转换的IP头中的源（内部）地址。源地址可以指定为： 

+ 单个的Ipv4或者Ipv6地址. 

+ CIDR 网络地址. 

+ 能够在规则集载入时通过DNS解析到的合法的域名，IP地址会替代规则中的域名。 

+ 网络接口名称。网络接口上配置的所有ip地址会替代进规则中。 

+ 带有/掩码（例如/24）的网络接口的名称。每个根据掩码确定的CIDR网络地址都会被替代进规则中。. 

+ 带有（）的网络接口名称。这告诉PF如果网络接口的IP地址改变了，就更新规则集。这个对于使用DHCP或者拨号来获得IP地址的接口特别有用，IP地址改变时不需要重新载入规则集。 

+ 带有如下的修饰词的网络接口名称： 

o :network - 替代CIDR网络地址段 \(例如, 192.168.0.0/24\) 

o :broadcast - 替代网络广播地址\(例如， 192.168.0.255\) 

o :peer - 替代点到点链路上的ip地址。 



另外，：0修饰词可以附加到接口名称或者上面的修饰词后面指示PF在替代时不包括网络接口的其余附加（alias）地址。这些修饰词也可以在接口名称在括号（）内时使用。例如：fxp0:network:0 



+ 表. 

+ 上面的所有项但使用！（非）修饰词 

+ 使用列表的一系列地址. 

+ 关键字 any 代表所有地址 

+ 关键字 all 是 from any to any的缩写。 



src\_port 

4层数据包头中的源端口。端口可以指定为： 

+ 1 到 65535之间的整数 

+ /etc/services中的合法服务名称 

+ 使用列表的一系列端口 

+ 一个范围: 

o != \(不等于\) 

o &lt; \(小于\) 

o &gt; \(大于\) 

o &lt;= \(小于等于\) 

o &gt;= \(大于等于\) 

o &gt;&lt; \(范围\) 

o &lt;&gt; \(反转范围\) 



最后2个是二元操作符（他们需要2个参数），在范围内不包括参数。 



o : \(inclusive range\) 



inclusive range 也是二元操作符但范围内包括参数。 



Port选项在NAT规则中通常不使用，因为目标通常会NAT所有的流量而不过端口是否在使用。 

dst\_addr 

被转换数据包中的目的地址。目的地址类型和源地址相似。 

dst\_port 

4层数据包头中的目的目的端口，目的端口类型和源端口类型相似。 

ext\_addr 

NAT网关上数据包被转换后的外部地址。外部地址可以是： 

+ 单个的Ipv4或者Ipv6地址. 

+ CIDR 网络地址. 

+ 能够在规则集载入时通过DNS解析到的合法的域名，IP地址会替代规则中的域名。 

+ 网络接口名称。网络接口上配置的所有ip地址会替代进规则中。 

+ 带有/掩码（例如/24）的网络接口的名称。每个根据掩码确定的CIDR网络地址都会被替代进规则中。. 

+ 带有（）的网络接口名称。这告诉PF如果网络接口的IP地址改变了，就更新规则集。这个对于使用DHCP或者拨号来获得IP地址的接口特别有用，IP地址改变时不需要重新载入规则集。 

+ 带有如下的修饰词的网络接口名称： 

o :network - 替代CIDR网络地址段 \(例如, 192.168.0.0/24\) 

o :broadcast - 替代网络广播地址\(例如， 192.168.0.255\) 

o :peer - 替代点到点链路上的ip地址。 



另外，：0修饰词可以附加到接口名称或者上面的修饰词后面指示PF在替代时不包括网络接口的其余附加（alias）地址。这些修饰词也可以在接口名称在括号（）内时使用。例如：fxp0:network:0 

+ 使用列表的一系列地址. 

pool\_type 

指定转换后的地址池的类型 

static-port 

告诉PF不要转换TCP和UDP数据包中的源端口 





这条规则最简单的形式如下： 

nat on tl0 from 192.168.1.0/24 to any -&gt; 24.5.0.5 



这条规则是说对从tl0网络接口上到来的所有192.168.1.0/24网络的数据包进行NAT，将源地址转换为24.5.0.5。 



尽管上面的规则是正确的，但却不是推荐的形式。因为维护起来有困难，当内部或者外部网络有变化时都要修改这一行。比较一下下面这条比较容易维护的规则：（tl0时外部，dc0是内部）： 



nat on tl0 from dc0:network to any -&gt; tl0 



优点是相当明显的，你可以任意改变2个接口的IP地址而不用改变这条规则。 



象上面这样在地址转换中使用接口名称时，IP地址在pf.conf文件载入时确定，并不是凭空的。如果你使用DHCP还配置外部地址，这会存在问题。如果 你分配的IP地址改变了，NAT仍然会使用旧的IP地址转换出去的数据包。这会导致对外的连接停止工作。为解决这个问题，应该给接口名称加上括号，告诉 PF自动更新转换地址。 



nat on tl0 from dc0:network to any -&gt; \(tl0\) 



这个方法对IPv4 和 IPv6地址都用效。 



双向映射 \(1:1 映射\) 



双向映射可以通过使用binat规则建立。Binat规则建立一个内部地址和外部地址一对一的映射。这会很有用，比如，使用独立的外部IP地址用内部网络 里的机器提供web服务。从因特网到来连接外部地址的请求被转换到内部地址，同时由（内部）web服务器发起的连接（例如DNS查询）被转换为外部地址。 和NAT规则不过，binat规则中的tcp和udp端口不会被修改。 





例如: 



web\_serv\_int = "192.168.1.100" 

web\_serv\_ext = "24.5.0.6" 



binat on tl0 from $web\_serv\_int to any -&gt; $web\_serv\_ext 



转换规则例外设置 



使用no关键字可以在转换规则中设置例外。例如，如果上面的转换规则修改成这样： 



no nat on tl0 from 192.168.1.10 to any 

nat on tl0 from 192.168.1.0/24 to any -&gt; 24.2.74.79 



则除了192.168.1.10以外，整个192.168.1.0/24网络地址的数据包都会转换为外部地址24.2.74.79。 



注意第一条匹配的规则起了决定作用，如果是匹配有no的规则，数据包不会被转换。No关键字也可以在binat和rdr规则中使用。 



检查 NAT 状态 



要检查活动的NAT转换可以使用pfctl（8）带-s state 选项。这个选项列出所有当前的NAT会话。 

\# pfctl -s state 

fxp0 TCP 192.168.1.35:2132 -&gt; 24.5.0.5:53136 -&gt; 65.42.33.245:22 TIME\_WAIT:TIME\_WAIT 

fxp0 UDP 192.168.1.35:2491 -&gt; 24.5.0.5:60527 -&gt; 24.2.68.33:53 MULTIPLE:SINGLE 



解释 \(对第一行\): 



fxp0 

显示状态绑定的接口。如果状态是浮动的，会出现self字样。 



TCP 

连接使用的协议。 



192.168.1.35:2132 

内部网络中机器的IP地址 \(192.168.1.35\)，源端口（2132）在地址后显示，这个也是被替换的IP头中的地址。 of the machine on the internal network. The 

source port \(2132\) is shown after the address. This is also the address 

that is replaced in the IP header. 



24.5.0.5:53136 

IP 地址 \(24.5.0.5\) 和端口 \(53136\) 是网关上数据包被转换后的地址和端口。 



65.42.33.245:22 

IP 地址 \(65.42.33.245\) 和端口 \(22\) 是内部机器要连接的地址和端口。 



TIME\_WAIT:TIME\_WAIT 

这表明PF认为的目前这个TCP连接的状态。 

