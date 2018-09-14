地址池是提供2个以上的地址供一组用户共享的。地址池可以是rdr规则中的重定向地址；可以是nat规则中的转换地址；也可以是route-to, reply-to, 和 dup-to filter选项中的目的地址。 



有4种使用地址池的方法： 



\* bitmask - 截取被修改地址（nat规则的源地址；rdr规则的目标地址）的最后部分和地址池地址的网络部分组合。例如：如果地址池是192.0.2.1/24，而被 修改地址是10.0.0.50，则结果地址是192.0.2.50。如果地址池是192.0.2.1/25，而被修改地址是10.0.0.130，这结果 地址是192.0.2.2。 

\* random - 从地址池中随机选择地址. 

\* source-hash - 使用源地址 hash 来确定使用地址池中的哪个地址。这个方法保证给定的源地址总是被映射到同一个地址池。Hash算法的种子可以在source-hash关键字后通过16进 制字符或者字符串来指定。默认情况下，pfctl（8）在规则集装入时会随机产生种子。 

\* round-robin - 在地址池中按顺序循环，这是默认方法，也是表中定义的地址池唯一的方法。 



除了round-robin方法，地址池的地址必须表达成CIDR（Classless Inter-Domain Routing ）的网络地址族。round-robin 方法可以接受多个使用列表和表的单独地址。 



sticky-address 选项可以在 random 和round-robin 池类型中使用，保证特定的源地址始终映射到同样的重定向地址。 



NAT 地址池 



地址池在NAT规则中可以被用做转换地址。连接的源地址会被转换成使用指定的方法从地址池中选择的地址。这对于PF负载一个非常大的网络的NAT会非常有用。由于经过NAT的连接对每个地址是有限的，增加附加的转换地址允许NAT网关增大服务的用户数量。 



在这个例子中，2个地址被用来做输出数据包的转换地址。对于每一个输出的连接，PF按照顺序循环使用地址。 





nat on $ext\_if inet from any to any -&gt; { 192.0.2.5, 192.0.2.10 } 



这个方法的一个缺点是成功建立连接的同一个内部地址不会总是转换为同一个外部地址。这会导致冲突，例如：浏览根据用户的ip地址跟踪登录的用户的web站 点。一个可选择的替代方法是使用source-hash 方法，以便每一个内部地址总是被转换为同样的外部地址。，要实现这个方法，地址池必须是CIDR网络地址。 



nat on $ext\_if inet from any to any -&gt; 192.0.2.4/31 source-hash 



这条NAT规则使用地址池192.0.2.4/31 \(192.0.2.4 - 192.0.2.5\)做为输出数据包的转换地址。每一个内部地址会被转换为同样的外部地址，由于source-hash关键字的缘故。 



外来连接负载均衡 



地址池也可以用来进行外来连接负载均衡。例如，外来的web服务器连接可以分配到服务器群。 



web\_servers = "{ 10.0.0.10, 10.0.0.11, 10.0.0.13 }" 



rdr on $ext\_if proto tcp from any to any port 80 -&gt; $web\_servers \ 

round-robin sticky-address 



成功的连接将按照顺序重定向到web服务器，从同一个源到来的连接发送到同一个服务器。这个sticky connection会和指向这个连接的状态一起存在。如果状态过期，sticky 

connection也过期。那个主机的更多连接被重定向到按顺序的下一个web服务器。 



输出流量负载均衡 



地址池可以和route-to过滤选项联合使用，在多路径路由协议（例如BGP4）不可用是负载均衡2个或者多个因特网连接。通过对round-robin地址池使用route-to，输出连接可以平均分配到多个输出路径。 



需要收集的附加的信息是邻近的因特网路由器IP地址。这要加入到route-to选项后来控制输入数据包的目的地址。 



下面的例子通过2条到因特网的连接平衡输出流量： 



lan\_net = "192.168.0.0/24" 

int\_if = "dc0" 

ext\_if1 = "fxp0" 

ext\_if2 = "fxp1" 

ext\_gw1 = "68.146.224.1" 

ext\_gw2 = "142.59.76.1" 



pass in on $int\_if route-to \ 

{ \($ext\_if1 $ext\_gw1\), \($ext\_if2 $ext\_gw2\) } round-robin \ 

from $lan\_net to any keep state 



route-to 选项用来在收到流量的内部接口上指定平衡的流量经过各自的网关到输出的网络接口。注意route-to 选项必须在每个需要均衡的过滤规则上出现。返回的数据包会路由到它们出去时的外部接口（这是由ISP做的），然后正常路由回内部网络。 



要保证带有属于$ext\_if1源地址的数据包总是路由到$ext\_gw1（$ext\_if2 和 $ext\_gw2也是同样的），下面2行必须包括在规则集中： 



pass out on $ext\_if1 route-to \($ext\_if2 $ext\_gw2\) from $ext\_if2 \ 

to any 

pass out on $ext\_if2 route-to \($ext\_if1 $ext\_gw1\) from $ext\_if1 \ 

to any 



最后，NAT也可以使用在输出接口中： 



nat on $ext\_if1 from $lan\_net to any -&gt; \($ext\_if1\) 

nat on $ext\_if2 from $lan\_net to any -&gt; \($ext\_if2\) 



一个完整的输出负载均衡的例子应该是这个样子： 



lan\_net = "192.168.0.0/24" 

int\_if = "dc0" 

ext\_if1 = "fxp0" 

ext\_if2 = "fxp1" 

ext\_gw1 = "68.146.224.1" 

ext\_gw2 = "142.59.76.1" 



\# nat outgoing connections on each internet interface 

nat on $ext\_if1 from $lan\_net to any -&gt; \($ext\_if1\) 

nat on $ext\_if2 from $lan\_net to any -&gt; \($ext\_if2\) 



\# default deny 

block in from any to any 

block out from any to any 



\# pass all outgoing packets on internal interface 

pass out on $int\_if from any to $lan\_net 

\# pass in quick any packets destined for the gateway itself 

pass in quick on $int\_if from $lan\_net to $int\_if 

\# load balance outgoing tcp traffic from internal network. 

pass in on $int\_if route-to \ 

{ \($ext\_if1 $ext\_gw1\), \($ext\_if2 $ext\_gw2\) } round-robin \ 

proto tcp from $lan\_net to any flags S/SA modulate state 

\# load balance outgoing udp and icmp traffic from internal network 

pass in on $int\_if route-to \ 

{ \($ext\_if1 $ext\_gw1\), \($ext\_if2 $ext\_gw2\) } round-robin \ 

proto { udp, icmp } from $lan\_net to any keep state 



\# general "pass out" rules for external interfaces 

pass out on $ext\_if1 proto tcp from any to any flags S/SA modulate state 

pass out on $ext\_if1 proto { udp, icmp } from any to any keep state 

pass out on $ext\_if2 proto tcp from any to any flags S/SA modulate state 

pass out on $ext\_if2 proto { udp, icmp } from any to any keep state 



\# route packets from any IPs on $ext\_if1 to $ext\_gw1 and the same for 

\# $ext\_if2 and $ext\_gw2 

pass out on $ext\_if1 route-to \($ext\_if2 $ext\_gw2\) from $ext\_if2 to any 

pass out on $ext\_if2 route-to \($ext\_if1 $ext\_gw1\) from $ext\_if1 to any 

