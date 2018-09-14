简介 



包过滤是在数据包通过网络接口时进行选择性的运行通过或者阻塞。Pf（4）检查包时使用的标准是基于的3层（IPV4或者IPV6）和4层（TCP, UDP, ICMP, ICMPv6）包头。最常用的标准是源和目的地址，源和目的端口，以及协议。 



过滤规则集指定了数据包必须匹配的标准和规则集作用后的结果，在规则集匹配时通过或者阻塞。规则集由开始到结束顺序执行。除非数据包匹配的规则包含 quick关键字，否则数据包在最终执行动作前会通过所有的规则检验。最后匹配的规则具有决定性，决定了数据包最终的执行结果。存在一条潜在的规则是如果 数据包和规则集中的所有规则都不匹配，则它会被通过。 



规则语法 



一般而言，最简单的过滤规则语法是这样的： 



action direction \[log\] \[quick\] on interface \[af\] \[proto protocol\] \ 

from src\_addr \[port src\_port\] to dst\_addr \[port dst\_port\] \ 

\[tcp\_flags\] \[state\] 



action 

数据包匹配规则时执行的动作，放行或者阻塞。放行动作把数据包传递给核心进行进一步出来，阻塞动作根据block-policy 选项指定的方法进行处理。默认的动作可以修改为阻塞丢弃或者阻塞返回。 

direction 

数据包传递的方向，进或者出 

log 

指定数据包被pflogd\( 进行日志记录。如果规则指定了keep state, modulate state, or synproxy state 选项，则只有建立了连接的状态被日志。要记录所有的日志，使用log-all 

quick 

如果数据包匹配的规则指定了quick关键字，则这条规则被认为时最终的匹配规则，指定的动作会立即执行。 

interface 

数据包通过的网络接口的名称或组。组是接口的名称但没有最后的整数。比如ppp 或 fxp，会使得规则分别匹配任何ppp或者fxp接口上的任意数据包。 

af 

数据包的地址类型，inet代表Ipv4，inet6代表Ipv6。通常PF能够根据源或者目标地址自动确定这个参数。 

protocol 

数据包的4层协议: 

+ tcp 

+ udp 

+ icmp 

+ icmp6 

+ /etc/protocols中的协议名称 

+ 0～255之间的协议号 

+ 使用列表的一系列协议. 

src\_addr, dst\_addr 

IP头中的源/目标地址。地址可以指定为： 

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

src\_port, dst\_port 

4层数据包头中的源/目标端口。端口可以指定为： 

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



tcp\_flags 

指定使用TCP协议时TCP头中必须设定的标记。 标记指定的格式是： flags check/mask. 例如: flags S/SA -这指引PF只检查S和A\(SYN and ACK\)标记，如果SYN标记是“on”则匹配。 



state 

指定状态信息在规则匹配时是否保持。 

+ keep state - 对 TCP, UDP, ICMP起作用 

+ modulate state - 只对 TCP起作用. PF会为匹配规则的数据包产生强壮的初始化序列号。 

+ synproxy state - 代理外来的TCP连接以保护服务器不受TCP SYN FLOODs欺骗。这个选项包含了keep state 和 modulate state 的功能。 



默认拒绝 



按照惯例建立防火墙时推荐执行的是默认拒绝的方法。也就是说先拒绝所有的东西，然后有选择的允许某些特定的流量通过防火墙。这个方法之所以是推荐的是因为它宁可失之过于谨慎（也不放过任何风险），而且使得编写规则集变得简单。 



产生一个默认拒绝的过滤规则，开始2行过滤规则必须是： 



block in all 

block out all 

这会阻塞任何通信方在任何方向上进入任意接口的所有流量。 



通过流量 



流量必须被明确的允许通过防火墙或者被默认拒绝的策略丢弃。这是数据包标准如源/目的端口，源/目的地址和协议开始活动的地方。无论何时数据包在被允许通过防火墙时规则都要设计的尽可能严厉。这是为了保证设计中的流量，也只有设计中的流量可以被允许通过。 



实例: 



\# 允许本地网络192.168.0.0/24流量通过dc0接口进入访问openbsd机器的IP地址 

\#192.168.0.1，同时也允许返回的数据包从dc0接口出去。 

pass in on dc0 from 192.168.0.0/24 to 192.168.0.1 

pass out on dc0 from 192.168.0.1 to 192.168.0.0/24 





\# Pass TCP traffic in on fxp0 to the web server running on the 

\# OpenBSD machine. The interface name, fxp0, is used as the 

\# destination address so that packets will only match this rule if 

\# they‘re destined for the OpenBSD machine. 

pass in on fxp0 proto tcp from any to fxp0 port www 



quick 关键字 



前面已经说过，每个数据包都要按自上至下的顺序按规则进行过滤。默认情况下，数据包被标记为通过，这个可以被任一规则改变，在到达最后一条规则前可以被来 回改变多次，最后的匹配规则是“获胜者”。存在一个例外是：过滤规则中的quick关键字具有取消进一步往下处理的作用，使得规则指定的动作马上执行。看 一下下面的例子： 



错误: 



block in on fxp0 proto tcp from any to any port ssh 

pass in all 



在这样的条件下，block行会被检测，但永远也不会有效果，因为它后面的一行允许所有的流量通过。 



正确: 



block in quick on fxp0 proto tcp from any to any port ssh 

pass in all 



这些规则执行的结果稍有不同，如果block行被匹配，由于quick选项的原因，数据包会被阻塞，而且剩下的规则也会被忽略。 





状态保持 



PF一个非常重要的功能是“状态保持”或者“状态检测”。状态检测指PF跟踪或者处理网络连接状态的能力。通过存贮每个连接的信息到一个状态表中，PF能够快速确定一个通过防火墙的数据包是否属于已经建立的连接。如果是，它会直接通过防火墙而不用再进行规则检验。 



状态保持有许多的优点，包括简单的规则集和优良的数据包处理性能。 

PF is able to match packets moving in either direction 

to state table entries meaning that filter rules which pass returning traffic 

don‘t need to be written. 并且，由于数据包匹配状态连接时不再进行规则集的匹配检测，PF用于处理这些数据包的时间大为减少。 



当一条规则使用了keep state选项，第一个匹配这条规则的数据包在收发双方之间建立了一个状态。现在，不仅发送者到接收者之间的数据包匹配这个状态绕过规则检验，而且接收者回复发送者的数据包也是同样的。例如： 



pass out on fxp0 proto tcp from any to any keep state 



这允许fxp0接口上的任何TCP流量通过，并且允许返回的流量通过防火墙。状态保持是一个非常有用的特性，由于状态查询比使用规则进行数据包检验快的多，因此它可以大幅度提高防火墙的性能。 



状态调整选项和状态保持的功能在除了仅适用于TCP数据包以为完全相同。在使用状态调整时，输入连接的初始化序列号（ISN）是随机的，这对于保护某些选 择ISN存在问题的操作系统的连接初始化非常有用。从openbsd 3.5开始，状态调整选项可以应用于包含非TCP的协议规则。 



对输出的TCP, UDP, ICMP数据包保持状态，并且调整TCP ISN。 



pass out on fxp0 proto { tcp, udp, icmp } from any \ 

to any modulate state 

状态保持的另一个优点是ICMP通信流量可以直接通过防火墙。例如，如果一个TCP连接使用了状态保持，当和这个TCP连接相关的ICMP数据包到来时，它会自动找到合适的状态记录，直接通过防火墙。 



状态记录的范围被state-policy runtime选项总体控制，也能基于单条规则由if-bound, group-bound, 和 floating state选项关键字设定。这些针对单条规则的关键字在使用时具有和state-policy选项同样的意义。例如： 



pass out on fxp0 proto { tcp, udp, icmp } from any \ 

to any modulate state \(if-bound\) 



状态规则指示为了使数据包匹配状态条目，它们必须通过fxp0网络接口传递。 



需要注意的是，nat，binat，rdr规则隐含在连接通过过滤规则集审核的过程中产生匹配连接的状态。 



UDP状态保持 



大家都听说过，“不能为UDP产生状态，因为UDP是无状态的协议”。确实，UDP通信会话没有状态的概念（明确的开始和结束通信），这丝毫不影响PF为 UDP会话产生状态的能力。对于没有开始和结束数据包的协议，PF仅简单追踪匹配的数据部通过的时间。如果到达超时限制，状态被清除，超时的时间值可以在 pf.conf配置文件中设定。 



TCP 标记 



基于标记的TCP包匹配经常被用于过滤试图打开新连接的TCP数据包。TCP标记和他们的意义如下所列： 



\* F : FIN - 结束; 结束会话 

\* S : SYN - 同步; 表示开始会话请求 

\* R : RST - 复位;中断一个连接 

\* P : PUSH - 推送; 数据包立即发送 

\* A : ACK - 应答 

\* U : URG - 紧急 

\* E : ECE - 显式拥塞提醒回应 

\* W : CWR - 拥塞窗口减少 



要使PF在规则检查过程中检查TCP标记，flag关键需按如下语法设置。 



flags check/mask 



mask部分告诉PF仅检查指定的标记，check部分说明在数据包头中哪个标记设置为“on”才算匹配。 



pass in on fxp0 proto tcp from any to any port ssh flags S/SA 



上面的规则通过的带SYN标记的TCP流量仅查看SYN和ACK标记。带有SYN和ECE标记的数据包会匹配上面的规则，而带有SYN和ACK的数据包或者仅带有ACK的数据包不会匹配。 



注意：在前面的openbsd版本中，下面的语法是支持的： 



. . . flags S 



现在，这个不再支持，mask必须被说明。 



标记常常和状态保持规则联合使用来控制创建状态条目： 



pass out on fxp0 proto tcp all flags S/SA keep state 



这条规则允许为所有输出中带SYN和ACK标记的数据包中的仅带有SYN标记的TCP数据包创建状态。 



大家使用标记时必须小心，理解你在做什么和为什么这样做。小心听取大家的建议，因为相当多的建议时不好的。一些人建议创建状态“只有当SYN标记设定而没有其他标记”时，这样的规则如下： 

. . . flags S/FSRPAUEW 糟糕的主意!! 



这个理论是，仅为TCP开始会话时创建状态，会话会以SYN标记开始，而没有其他标记。问题在于一些站点使用ECN标记开始会话，而任何使用ECN连接你的会话都会被那样的规则拒绝。比较好的规则是： 



. . . flags S/SAFR 



这个经过实践是安全的，如果流量进行了整形也没有必要检查FIN和RST标记。整形过程会让PF丢弃带有非法TCP标记的进入数据包（例如SYN和FIN以及SYN和RST）。强烈推荐总是进行流量整形： 



scrub in on fxp0 

. 

. 

. 

pass in on fxp0 proto tcp from any to any port ssh flags S/SA \ 

keep state 



TCP SYN 代理 



通过，当客户端向服务器初始化一个TCP连接时，PF会在二者直接传递握手数据包。然而，PF具有这样的能力，就是代理握手。使用握手代理，PF自己会和 客户端完成握手，初始化和服务器的握手，然后在二者之间传递数据。这样做的优点是在客户端完成握手之前，没有数据包到达服务器。这样就消沉了TCP SYN FLOOD欺骗影响服务器的问题，因为进行欺骗的客户端不会完成握手。 



TCP SYN 代理在规则中使用synproxy state关键字打开。例如： 



pass in on $ext\_if proto tcp from any to $web\_server port www \ 

flags S/SA synproxy state 



这样， web服务器的连接由PF进行TCP代理。 



由于synproxy state工作的方式，它具有keep state 和 modulate state一样的功能。 



如果PF工作在桥模式下，SYN代理不会起作用。 



阻塞欺骗数据包 



地址欺骗是恶意用户为了隐藏他们的真实地址或者假冒网络上的其他节点而在他们传递的数据包中使用虚假的地址。一旦实施了地址欺骗，他们可以隐蔽真实地址实施网络攻击或者获得仅限于某些地址的网络访问服务。 



PF通过antispoof关键字提供一些防止地址欺骗的保护。 



antispoof \[log\] \[quick\] for interface \[af\] 



log 

指定匹配的数据包应该被pflogd（8）进行日志记录 

quick 

如果数据包匹配这条规则，则这是最终的规则，不再进行其他规则集的检查。 

interface 

激活要进行欺骗保护的网络接口。也可以是接口的列表。 

af 

激活进行欺骗保护的地址族，inet代表Ipv4，inet6代表Ipv6。 



实例: 



antispoof for fxp0 inet 



当规则集载入时，任何出现了antispoof关键字的规则都会扩展成2条规则。假定接口fxp0具有ip地址10.0.0.1和子网掩码255.255.255.0（或者/24），上面的规则会扩展成： 



block in on ! fxp0 inet from 10.0.0.0/24 to any 

block in inet from 10.0.0.1 to any 



这些规则实现下面的2个目的： 



\* 阻塞任何不是由fxp0接口进入的10.0.0.0/24网络的流量。由于10.0.0.0/24的网络是在fxp0接口，具有这样的源网络地址的数据包绝不应该从其他接口上出现。 

\* 阻塞任何由10.0.0.1即fxp0接口的IP地址的进入流量。主机绝对不会通过外面的接口给自己发送数据包，因此任何进入的流量源中带有主机自己的IP地址都可以认为是恶意的！ 



注意：antispoof规则扩展出来的过滤规则会阻塞loopback接口上发送到本地地址的数据包。这些数据包应该明确的配置为允许通过。例如： 



pass quick on lo0 all 



antispoof for fxp0 inet 



使用antispoof应该仅限于已经分配了IP地址的网络接口，如果在没有分配IP地址的网络接口上使用，过滤规则会扩展成： 



block drop in on ! fxp0 inet all 

block drop in inet all 



这样的规则会存在阻塞所有接口上进入的所有流量的危险。 



被动操作系统识别 



被动操作系统识别是通过基于远端主机TCP SYN数据包中某些特征进行操作系统被动检测的技术。这些信息可以作为标准在过滤规则中使用。 



PF检测远端操作系统是通过比较TCP SYN数据包中的特征和已知的特征文件对照来确定的，特征文件默认是/etc/pf.os。如果PF起作用，可是使用下面的命令查看当前的特征列表。 



\# pfctl -s osfp 



在规则集中，特征可以指定为OS类型，版本，或者子类型/补丁级别。这些条目在上面的pfctl命令中有列表。要在过滤规则中指定特征，需使用os关键字： 



pass in on $ext\_if any os OpenBSD keep state 

block in on $ext\_if any os "Windows 2000" 

block in on $ext\_if any os "Linux 2.4 ts" 

block in on $ext\_if any os unknown 



指定的操作系统类型unknow允许匹配操作系统未知的数据包。 



注意以下的内容：: 



\*操作系统识别偶尔会出错，因为存在欺骗或者修改过的使得看起来象某个操作系统得数据包。 

\* 某些修改或者打过补丁得操作系统会改变栈得行为，导致它或者和特征文件不符，或者符合了另外得操作系统特征。 

\* OSFP 仅对TCP SYN数据包起作用，它不会对其他协议或者已经建立得连接起作用。 



IP 选项 



默认情况下，PF阻塞带有IP选项得数据包。这可以使得类似nmap得操作系统识别软件工作困难。如果你的应用程序需要通过这样的数据包，例如多播或者IGMP，你可以使用allow-opts关键字。。 



pass in quick on fxp0 all allow-opts 



过滤规则实例 



下面是过滤规则得实例。运行PF的机器充当防火墙，在一个小得内部网络和因特网之间。只列出了过滤规则，queueing, nat, rdr,等等没有在实例中列出。 





ext\_if = "fxp0" 

int\_if = "dc0" 

lan\_net = "192.168.0.0/24" 



\# scrub incoming packets 

scrub in all 



\# setup a default deny policy 

block in all 

block out all 



\# pass traffic on the loopback interface in either direction 

pass quick on lo0 all 



\# activate spoofing protection for the internal interface. 

antispoof quick for $int\_if inet 



\# only allow ssh connections from the local network if it‘s from the 

\# trusted computer, 192.168.0.15. use "block return" so that a TCP RST is 

\# sent to close blocked connections right away. use "quick" so that this 

\# rule is not overridden by the "pass" rules below. 

block return in quick on $int\_if proto tcp from ! 192.168.0.15 \ 

to $int\_if port ssh flags S/SA 



\# pass all traffic to and from the local network 

pass in on $int\_if from $lan\_net to any 

pass out on $int\_if from any to $lan\_net 



\# pass tcp, udp, and icmp out on the external \(Internet\) interface. 

\# keep state on udp and icmp and modulate state on tcp. 

pass out on $ext\_if proto tcp all modulate state flags S/SA 

pass out on $ext\_if proto { udp, icmp } all keep state 



\# allow ssh connections in on the external interface as long as they‘re 

\# NOT destined for the firewall \(i.e., they‘re destined for a machine on 

\# the local network\). log the initial packet so that we can later tell 

\# who is trying to connect. use the tcp syn proxy to proxy the connection. 

pass in log on $ext\_if proto tcp from any to { !$ext\_if, !$int\_if } \ 

port ssh flags S/SA synproxy state 

