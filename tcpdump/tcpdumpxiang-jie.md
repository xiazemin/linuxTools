用简单的话来定义tcpdump，就是：dump the traffic on a network，根据使用者的定义对网络上的数据包进行截获的包分析工具。 tcpdump可以将网络中传送的数据包的“头”完全截获下来提供分析。它支持针对网络层、协议、主机、网络或端口的过滤，并提供and、or、not等逻辑语句来帮助你去掉无用的信息。



 



实用命令实例

默认启动



tcpdump

普通情况下，直接启动tcpdump将监视第一个网络接口上所有流过的数据包。



 



监视指定网络接口的数据包



tcpdump -i eth1

如果不指定网卡，默认tcpdump只会监视第一个网络接口，一般是eth0，下面的例子都没有指定网络接口。　



 



监视指定主机的数据包



打印所有进入或离开sundown的数据包.



tcpdump host sundown

也可以指定ip,例如截获所有210.27.48.1 的主机收到的和发出的所有的数据包



tcpdump host 210.27.48.1 

打印helios 与 hot 或者与 ace 之间通信的数据包



tcpdump host helios and \\( hot or ace \\)

截获主机210.27.48.1 和主机210.27.48.2 或210.27.48.3的通信



tcpdump host 210.27.48.1 and \ \(210.27.48.2 or 210.27.48.3 \\) 

打印ace与任何其他主机之间通信的IP 数据包, 但不包括与helios之间的数据包.



tcpdump ip host ace and not helios

如果想要获取主机210.27.48.1除了和主机210.27.48.2之外所有主机通信的ip包，使用命令：



tcpdump ip host 210.27.48.1 and ! 210.27.48.2

截获主机hostname发送的所有数据



tcpdump -i eth0 src host hostname

监视所有送到主机hostname的数据包



tcpdump -i eth0 dst host hostname

 



监视指定主机和端口的数据包



如果想要获取主机210.27.48.1接收或发出的telnet包，使用如下命令



tcpdump tcp port 23 and host 210.27.48.1

对本机的udp 123 端口进行监视 123 为ntp的服务端口



tcpdump udp port 123 

 



监视指定网络的数据包



打印本地主机与Berkeley网络上的主机之间的所有通信数据包\(nt: ucb-ether, 此处可理解为'Berkeley网络'的网络地址,此表达式最原始的含义可表达为: 打印网络地址为ucb-ether的所有数据包\)



tcpdump net ucb-ether

打印所有通过网关snup的ftp数据包\(注意, 表达式被单引号括起来了, 这可以防止shell对其中的括号进行错误解析\)



tcpdump 'gateway snup and \(port ftp or ftp-data\)'

打印所有源地址或目标地址是本地主机的IP数据包



\(如果本地网络通过网关连到了另一网络, 则另一网络并不能算作本地网络.\(nt: 此句翻译曲折,需补充\).localnet 实际使用时要真正替换成本地网络的名字\)



tcpdump ip and not net localnet

 



监视指定协议的数据包



打印TCP会话中的的开始和结束数据包, 并且数据包的源或目的不是本地网络上的主机.\(nt: localnet, 实际使用时要真正替换成本地网络的名字\)\)



tcpdump 'tcp\[tcpflags\] & \(tcp-syn\|tcp-fin\) != 0 and not src and dst net localnet'

打印所有源或目的端口是80, 网络层协议为IPv4, 并且含有数据,而不是SYN,FIN以及ACK-only等不含数据的数据包.\(ipv6的版本的表达式可做练习\)



tcpdump 'tcp port 80 and \(\(\(ip\[2:2\] - \(\(ip\[0\]&0xf\)&lt;&lt;2\)\) - \(\(tcp\[12\]&0xf0\)&gt;&gt;2\)\) != 0\)'

\(nt: 可理解为, ip\[2:2\]表示整个ip数据包的长度, \(ip\[0\]&0xf\)&lt;&lt;2\)表示ip数据包包头的长度\(ip\[0\]&0xf代表包中的IHL域, 而此域的单位为32bit, 要换算



成字节数需要乘以4,　即左移2.　\(tcp\[12\]&0xf0\)&gt;&gt;4 表示tcp头的长度, 此域的单位也是32bit,　换算成比特数为 \(\(tcp\[12\]&0xf0\) &gt;&gt; 4\)　&lt;&lt;　２,　

即 \(\(tcp\[12\]&0xf0\)&gt;&gt;2\).　\(\(ip\[2:2\] - \(\(ip\[0\]&0xf\)&lt;&lt;2\)\) - \(\(tcp\[12\]&0xf0\)&gt;&gt;2\)\) != 0　表示: 整个ip数据包的长度减去ip头的长度,再减去

tcp头的长度不为0, 这就意味着, ip数据包中确实是有数据.对于ipv6版本只需考虑ipv6头中的'Payload Length' 与 'tcp头的长度'的差值, 并且其中表达方式'ip\[\]'需换成'ip6\[\]'.\)



打印长度超过576字节, 并且网关地址是snup的IP数据包



tcpdump 'gateway snup and ip\[2:2\] &gt; 576'

打印所有IP层广播或多播的数据包， 但不是物理以太网层的广播或多播数据报



tcpdump 'ether\[0\] & 1 = 0 and ip\[16\] &gt;= 224'

打印除'echo request'或者'echo reply'类型以外的ICMP数据包\( 比如,需要打印所有非ping 程序产生的数据包时可用到此表达式 .

\(nt: 'echo reuqest' 与 'echo reply' 这两种类型的ICMP数据包通常由ping程序产生\)\)



tcpdump 'icmp\[icmptype\] != icmp-echo and icmp\[icmptype\] != icmp-echoreply'

 

