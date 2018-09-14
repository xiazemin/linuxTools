PF提供了许多方法来进行规则集的简化。一些好的例子是使用宏和列表。另外，规则集的语言或者语法也提供了一些使规则集简化的捷径。首要的规则是，规则集越简单，就越容易理解和维护。 



使用宏 



宏是非常有用的，因为它提供了硬编码地址，端口号，接口名称等的可选替代。在一个规则集中，服务器的IP地址改变了？没问题，仅仅更新一下宏，不需要弄乱你花费了大量时间和精力建立的规则集。 



通常的惯例是在PF规则集中定义每个网络接口的宏。如果网卡需要被使用不同驱动的卡取代，例如，用intel代替3com，可以更新宏，过滤规则集会和以 前功能一样。另一个优点是，如果在多台机器上安装同样的规则集，某些机器会有不同的网卡，使用宏定义网卡可以使的安装的规则集进行最少的修改。使用宏来定 义规则集中经常改变的信息，例如端口号，IP地址，接口名称等等，建议多多实践！ 



\# define macros for each network interface 

IntIF = "dc0" 

ExtIF = "fxp0" 

DmzIF = "fxp1" 



另一个惯例是使用宏来定义IP地址和网络，这可以大大减轻在IP地址改变时对规则集的维护。 



\# define our networks 

IntNet = "192.168.0.0/24" 

ExtAdd = "24.65.13.4" 

DmzNet = "10.0.0.0/24" 



如果内部地址扩展了或者改到了一个不同的IP段，可以更新宏为： 



IntNet = "{ 192.168.0.0/24, 192.168.1.0/24 }" 



当这个规则集重新载入时，任何东西都跟以前一样。 



使用列表 



来看一下一个规则集中比较好的例子使得RFC1918定义的内部地址不会传送到因特网上，如果发生传送的事情，可能导致问题。 



block in quick on tl0 inet from 127.0.0.0/8 to any 

block in quick on tl0 inet from 192.168.0.0/16 to any 

block in quick on tl0 inet from 172.16.0.0/12 to any 

block in quick on tl0 inet from 10.0.0.0/8 to any 

block out quick on tl0 inet from any to 127.0.0.0/8 

block out quick on tl0 inet from any to 192.168.0.0/16 

block out quick on tl0 inet from any to 172.16.0.0/12 

block out quick on tl0 inet from any to 10.0.0.0/8 



看看下面更简单的例子： 



block in quick on tl0 inet from { 127.0.0.0/8, 192.168.0.0/16, \ 

172.16.0.0/12, 10.0.0.0/8 } to any 

block out quick on tl0 inet from any to { 127.0.0.0/8, \ 

192.168.0.0/16, 172.16.0.0/12, 10.0.0.0/8 } 



这个规则集从8行减少到2行。如果联合使用宏，还会变得更好： 



NoRouteIPs = "{ 127.0.0.0/8, 192.168.0.0/16, 172.16.0.0/12, \ 

10.0.0.0/8 }" 

ExtIF = "tl0" 

block in quick on $ExtIF from $NoRouteIPs to any 

block out quick on $ExtIF from any to $NoRouteIPs 



注意虽然宏和列表简化了pf.conf文件，但是实际是这些行会被pfctl（8）扩展成多行，因此，上面的例子实际扩展成下面的规则： 



block in quick on tl0 inet from 127.0.0.0/8 to any 

block in quick on tl0 inet from 192.168.0.0/16 to any 

block in quick on tl0 inet from 172.16.0.0/12 to any 

block in quick on tl0 inet from 10.0.0.0/8 to any 

block out quick on tl0 inet from any to 10.0.0.0/8 

block out quick on tl0 inet from any to 172.16.0.0/12 

block out quick on tl0 inet from any to 192.168.0.0/16 

block out quick on tl0 inet from any to 127.0.0.0/8 



可以看到，PF扩展仅仅是简化了编写和维护pf.conf文件，实际并不简化pf\(4\)对于规则的处理过程。 



宏不仅仅用来定义地址和端口，它们在PF的规则文件中到处都可以用： 



pre = "pass in quick on ep0 inet proto tcp from " 

post = "to any port { 80, 6667 } keep state" 



\# David‘s classroom 

$pre 21.14.24.80 $post 



\# Nick‘s home 

$pre 24.2.74.79 $post 

$pre 24.2.74.178 $post 



扩展后: 



pass in quick on ep0 inet proto tcp from 21.14.24.80 to any \ 

port = 80 keep state 

pass in quick on ep0 inet proto tcp from 21.14.24.80 to any \ 

port = 6667 keep state 

pass in quick on ep0 inet proto tcp from 24.2.74.79 to any \ 

port = 80 keep state 

pass in quick on ep0 inet proto tcp from 24.2.74.79 to any \ 

port = 6667 keep state 

pass in quick on ep0 inet proto tcp from 24.2.74.178 to any \ 

port = 80 keep state 

pass in quick on ep0 inet proto tcp from 24.2.74.178 to any \ 

port = 6667 keep state 



PF 语法 



PF的语法相当灵活，因此，允许编写非常灵活的规则集。PF能够自动插入某些关键字因此它们不必在规则中明确写出，关键字的顺序也是随意的，因此不需要记忆严格的语法限制。 



减少关键字 



要定义全部拒绝的策略，使用下面2条规则： 



block in all 

block out all 



这可以简化为: 



block all 



如果没有指定方向，PF会认为规则适用于数据包传递的进、出2个方向。 



同样的， "from any to any" 和 "all" 子句可以在规则中省略，例如 



block in on rl0 all 

pass in quick log on rl0 proto tcp from any to any port 22 keep state 



可以简化为: 



block in on rl0 

pass in quick log on rl0 proto tcp to port 22 keep state 



第一条规则阻塞rl0上从任意到任意的进入数据包，第二条规则允许rl0上端口22的TCP流量通过。 



Return 简化 



用于阻塞数据包，回应TCP RST或者ICMP不可到达的规则集可以这么写： 



block in all 

block return-rst in proto tcp all 

block return-icmp in proto udp all 

block out all 

block return-rst out proto tcp all 

block return-icmp out proto udp all 



可以简化为：: 



block return 



当PF看到return关键字，PF可以智能回复合适应答，或者完全不回复，取决于要阻塞的数据包使用的协议。W 



关键字顺序 



在大多数情况下，关键字的顺序是非常灵活的。例如，规则可以这么写： 



pass in log quick on rl0 proto tcp to port 22 \ 

flags S/SA keep state queue ssh label ssh 



也可以这么写： 



pass in quick log on rl0 proto tcp to port 22 \ 

queue ssh keep state label ssh flags S/SA 



其他类似的顺序也能够正常工作。

