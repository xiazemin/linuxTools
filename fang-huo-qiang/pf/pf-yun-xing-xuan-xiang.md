运行选项是控制pf操作的选择。这些选项在pf.conf中使用set指定。 



set block-policy 

设定过滤规则中指定的block动作的默认行为。 

+ drop - 数据包悄然丢弃. 

+ return - TCP RST 数据包返回给遭阻塞的TCP数据包，ICMP不可到达数据包返回给其他。 

注意单独的过滤规则可以重写默认的响应。 



set debug 

设定 pf的调试级别。 

+ none - 不显示任何调试信息。 

+ urgent - 为严重错误产生调试信息，这是默认选择。 

+ misc - 为多种错误产生调试信息。（例如，收到标准化/整形的数据包的状态，和产生失败的状态）。. 

+ loud - 为普通条件产生调试信息（例如，收到被动操作系统检测信息状态）。 



set fingerprints file 

设定应该装入的进行操作系统识别的操作系统特征文件来，默认是 /etc/pf.os. 



set limit 

frags - 在内存池中进行数据包重组的最大数目。默认是5000。 

src-nodes - 在内存池中用于追踪源地址（由stick－address 和 source-track选项产生）的最大数目，默认是10000。 

states - 在内存池中用于状态表（过滤规则中的keep state）的最大数目，默认是10000。 



set loginterface int 

设定PF要统计进/出流量和放行/阻塞的数据包的数目的接口卡。统计数目一次只能用于一张卡。注意 match, bad-offset, 等计数器和状态表计数器不管 loginterface是否设置都会被记录。 



set optimization 

为以下的网络环境优化PF： 

+ normal - 适用于绝大多数网络，这是默认项。 

+ high-latency - 高延时网络，例如卫星连接。 

+ aggressive - 自状态表中主动终止连接。这可以大大减少繁忙防火墙的内存需求，但要冒空闲连接被过早断开的风险。 

+ conservative - 特别保守的设置。这可以避免在内存需求过大时断开空闲连接，会稍微增加CPU的利用率。 



set state-policy 

设定 PF在状态保持时的行为。这种行为可以被单条规则所改变。见状态保持章节。 

+ if-bound - 状态绑定到产生它们的接口。如果流量匹配状态表种条目但不是由条目中记录的接口通过，这个匹配会失败。数据包要么匹配一条过滤规则，或者被丢弃/拒绝。 

+ group-bound - 行为基本和if-bound相同，除了数据包允许由同一组接口通过，例如所有的ppp接口等。 

+ floating - 状态可以匹配任何接口上的流量。只要数据包匹配状态表条目，不管是否匹配它通过的接口，都会放行。这是默认的规则。 



set timeout 

interval - 丢弃过期的状态和数据包碎片的秒数。 

frag - 不能重组的碎片过期的秒数。 



例如: 



set timeout interval 10 

set timeout frag 30 

set limit { frags 5000, states 2500 } 

set optimization high-latency 

set block-policy return 

set loginterface dc0 

set fingerprints /etc/pf.os.test 

set state-policy if-bound 

