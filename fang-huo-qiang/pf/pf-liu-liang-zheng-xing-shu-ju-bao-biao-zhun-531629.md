流量整形是将数据包标准化避免最终的数据包出现非法的目的。流量整形指引同时也会重组数据包碎片，保护某些操作系统免受某些攻击，丢弃某些带有非法联合标记的TCP数据包。流量整形指引的简单形式是： 



scrub in all 



这会对所有接口上到来的数据包进行流量整形。 



一个不在接口上进行流量整形的原因是要透过PF使用NFS。一些非openbsd的平台发送（和等待）奇怪的数据包，对设置不分片位的数据包进行分片。这 会被流量整形（正确的）拒绝。这个问题可以通过设置no-df选项解决。另一个原因是某些多用户的游戏在进行流量整形通过PF时存在连接问题。除了这些极 其罕见的案例，对所有的数据包进行流量整形时强烈推荐的设置。 



流量整形指引语法相对过滤语法是非常简单的，它可以非常容易的选择特定的数据包进行整形而不对没指定的数据包起作用。 



更多的关于数据整形的原理和概念可以在这篇论文中找到： 

Network Intrusion Detection: Evasion, Traffic Normalization, and End-to-End 

Protocol Semantics. 



选项 



流量整形有下面的选项: 



no-df 

在IP数据包头中清除不分片位的设置。某些操作系统已知产生设置不分片位的分片数据包。尤其是对于NFS。流量整形（scrub）会丢弃这类数据包除非设 置了no-df选项。某些操作系统产生带有不分片位和用0填写IP包头中分类域，推荐使用no-df和random-id 联合使用解决。 

random-id 

用随机值替换某些操作系统输出数据包中使用的可预测IP分类域的值这个选项仅适用于在选择的数据包重组后不进行分片的输入数据包。 

min-ttl num 

增加IP包头中的最小存活时间（TTL）。 

max-mss num 

增加在TCP数据包头中最大分段值（MSS）。 

fragment reassemble 

在传递数据包到过滤引擎前缓冲收到的数据包碎片，重组它们成完整的数据包。优点是过滤规则仅处理完整的数据包，忽略碎片。缺点是需要内存缓冲数据包碎片。这是没有设置分片选项时的默认行为。这也是能和NAT一起工作的唯一分片选项。 

fragment crop 

导致重复的碎片被丢弃，重叠的被裁剪。与碎片重组不同，碎片不会被缓冲，而是一到达就直接传递。 

fragment drop-ovl 

跟 fragment crop 相似，除了所有重复和重叠的碎片和其他更多的通信碎片一样被丢弃。 

reassemble tcp 

TCP连接状态标准化。当使用了 scrub reassemble tcp时，方向（进/出）不用说明，会执行下面的标准化过程： 

+ 连接双方都不允许减少它们的IP TTL值。这样做是为了防止攻击者发送数据包到防火墙，影响防火墙保持的连接状态，使数据包在到达目的主机前就过期。所有数据包的TTL都为了这个连接加到了最大值。 

+ 用随机数字调整TCP数据包头中的 RFC1323 时间戳。这可以阻止窃听者推断主机在线的时间和猜测NAT网关后面有多少主机。 



实例: 



scrub in on fxp0 all fragment reassemble min-ttl 15 max-mss 1400 

scrub in on fxp0 all no-df 

scrub on fxp0 all reassemble tcp 

