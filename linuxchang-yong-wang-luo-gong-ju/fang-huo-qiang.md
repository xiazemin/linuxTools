iptables

iptables是强大的包过滤工具，Docker、Neutron都网络配置都离不开iptables。iptables通过一系列规则来实现数据包过滤、处理，能够实现防火墙、NAT等功能。当一个网络数据包进入到主机之前，先经过Netfilter检查，即iptables规则，检查通过则接受（Accept）进入本机资源，否则丢弃该包（Drop）。规则是有顺序的，如果匹配第一个规则，则执行该规则的Action，不会执行后续的规则。iptables的规则有多个表构成，每个表又由链（chain）构成，每个表的功能不一样，本文只涉及两个简单的表，即Filter表和NAT表，望文生义即可了解，Filter表用于包过滤，而NAT表用来进行源地址和目的地址的IP或者端口转换。



1.Filter表

Filter表主要和进入Linux本地的数据包有关，也是默认的表。该表主要由三条链构成：



INPUT：对进入主机的数据包过滤

OUTPUT：对本地发送的数据包过滤

FORWARD：传递数据包到后端计算机，与NAT有点类似。

查看本地的Filter表:



fgp@controller:~$ sudo iptables -n -t filter --list

\[root@portal ~\]\# iptables -n -t filter --list

Chain INPUT \(policy ACCEPT\)

target     prot opt source               destination

 

Chain FORWARD \(policy ACCEPT\)

target     prot opt source               destination

 

Chain OUTPUT \(policy ACCEPT\)

target     prot opt source               destination

其中-n表示不进行域名解析,-t指定使用的表,--list表示列出所有规则。我们发现目前没有定义任何规则。注意链后面的policy为ACCEPT，表示若通过所有的规则都不匹配，则为默认action accept。



接下来将通过几个demo实例演示怎么使用Filter表。



注意：



本文实验使用的是本机虚拟机，其中宿主机地址为192.168.56.1，虚拟机地址为192.168.56.2，在实验中会涉及丢弃192.168.56.1的数据包，如果您连接的是远程云主机，将导致和远程主机断开连接。

以下每个步骤，除非特别说明，下一个步骤执行前，务必清空上一个步骤的规则：

sudo iptables -F

首先看一个简单的例子，把192.168.56.1加入黑名单禁止其访问：



sudo iptables -A INPUT -i eth1 -s 192.168.56.1 -j DROP

例子中-A表示追加规则，INPUT是链名,-i指定网卡，-s指定源IP地址，-j指定action，这里为DROP，即丢弃包。



此时192.168.56.1这个ip不能和主机通信了，ssh会立即掉线，只能通过vnc连接了！



-s不仅能够指定IP地址，还可以指定网络地址，使用-p指定协议类型，比如我们需要丢掉所有来自192.168.56.0/24这个网络地址的ICMP包，即不允许ping：



sudo iptables -A INPUT -s 192.168.56.0/24 -i eth1 -p icmp -j DROP

输出结果：



➜  ~ nc -z 192.168.56.2 22

Connection to 192.168.56.2 port 22 \[tcp/ssh\] succeeded!

➜  ~ ping 192.168.56.2

PING 192.168.56.2 \(192.168.56.2\): 56 data bytes

Request timeout for icmp\_seq 0

Request timeout for icmp\_seq 1

^C

--- 192.168.56.2 ping statistics ---

3 packets transmitted, 0 packets received, 100.0% packet loss

我们发现能够通过nc连接主机，但ping不通。



我们还可以通过--dport指定目标端口，比如不允许192.168.56.1这个主机ssh连接（不允许访问22端口）：



sudo iptables -A INPUT -s 192.168.56.1 -p tcp --dport 22 -i eth1 -j DROP

注意：使用--dport或者--sport必须同时使用-p指定协议类型，否则无效！



以上把192.168.56.1打入了ssh黑名单,此时能够ping通主机，但无法通过ssh连接主机。



Filter表的介绍就到此为止，接下来看NAT表的实例。



2.NAT表

NAT表默认由以下三条链构成：



PREROUTING：在进行路由判断前所要进行的规则\(DNAT/Redirect\)

POSTROUTING: 在进行路由判断之后要进行的规则\(SNAT/MASQUERADE\)

OUTPUT: 与发送的数据包有关

根据需要修改的是源IP地址还是目标IP地址，NAT可以分为两种：



DNAT：需要修改目标地址（IP或者端口），使用场景为从外网来的数据包需要映射到内部的一个私有IP，比如46.64.22.33-&gt;192.168.56.1。显然作用在PREROUTING。

SNAT：需要修改源地址（IP或者端口），使用场景和DNAT相反，内部私有IP需要发数据包出去，必须首先映射成公有IP，比如192.168.56.1-&gt;46.64.22.33。显然作用在POSTROUTING。

首先实现介绍一个简单的demo，端口转发，我们把所有来自2222的tcp请求转发到本机的22端口,显然需要修改目标地址，因此属于DNAT：



sudo iptables -t nat -A PREROUTING -p tcp --dport 2222 -j REDIRECT --to-ports 22

此时在192.168.56.1上使用ssh连接，指定端口为2222：



ssh fgp@192.168.56.2 -p 2222

我们能够顺利登录，说明端口转发成功。



另一个例子是使用双网卡linux系统作为路由器，我们有一台服务器controller有两个网卡:



eth0: 192.168.1.102 \# 可以通外网

eth1: 192.168.56.2 \# 不可以通外网，用作网关接口。

另外一台服务器node1只有一个网卡eth1，IP地址为192.168.56.3，不能通外网。我们设置默认路由为controller机器的eth1：



sudo route add default gw 192.168.56.2 dev eth1

此时路由表信息为:



fgp@node1:~$ sudo route -n

Kernel IP routing table

Destination     Gateway         Genmask         Flags Metric Ref    Use Iface

0.0.0.0         192.168.56.2    0.0.0.0         UG    0      0        0 eth1

192.168.56.0    0.0.0.0         255.255.255.0   U     0      0        0 eth1

由路由表可知，node1上的数据包会发送到网关192.168.56.2，即controller节点.



接下来我们要在服务器controller上配置NAT，我们需要实现192.168.56.0/24的IP都转发到eth0，显然是SNAT，修改的源地址为eth0 IP地址192.168.1.102:



sudo iptables -t nat -A POSTROUTING -s 192.168.56.0/24 -o eth0 -j SNAT --to-source 192.168.1.102

其中-t指定nat表，-A 指定链为POSTROUTING，-s 为源ip地址段，-o指定转发网卡，注意-j参数指定action为SNAT，并指定eth0 IP地址\(注意eth0可能配置多个ip地址，因此必须指定--to-source）。



此时在node1机器上检测网络连通性:



fgp@node1:~$ ping baidu.com -c 2

PING baidu.com \(180.149.132.47\) 56\(84\) bytes of data.

64 bytes from 180.149.132.47: icmp\_seq=1 ttl=48 time=7.94 ms

64 bytes from 180.149.132.47: icmp\_seq=2 ttl=48 time=6.32 ms

 

--- baidu.com ping statistics ---

2 packets transmitted, 2 received, 0% packet loss, time 1002ms

rtt min/avg/max/mdev = 6.328/7.137/7.946/0.809 ms

node1能够正常上网。



以上通过使用controller的网卡eth0作为路由实现了node1的上网，但同时有一个问题存在，我们在指定SNAT时必须手动指定IP，如果eth0 IP地址变化了，必须修改iptables规则。显然这样很难维护，我们可以通过MASQUERADE实现动态SNAT，不需要指定IP地址：



sudo iptables -t nat -A POSTROUTING -s 192.168.56.0/24 -o eth0 -j MASQUERADE

其他

使用iptables-save能够导出规则，使用iptables-restore能够从文件中导入规则。



ipset

以上我们通过iptables封IP，如果IP地址非常多，我们就需要加入很多的规则，这些规则需要一一判断，性能会下降（线性的）。ipset能够把多个主机放入一个集合，iptables能够针对这个集合设置规则，既方便操作，又提高了执行效率。注意ipset并不是只能把ip放入集合，还能把网络地址、mac地址、端口等也放入到集合中。



首先我们创建一个ipset：



sudo ipset create blacklist hash:ip

以上创建了一个blacklist集合，集合名称后面为存储类型，除了hash表，还支持bitmap、link等，后面是存储类型，我们指定的是ip，表示我们的集合元素为ip地址。



我们为这个blacklist集合增加一条规则，禁止访问：



sudo iptables -I INPUT -m set --match-set blacklist src -j DROP 

此时只要在blacklist的ip地址就会自动加入黑名单。



我们把192.168.56.1和192.168.56.3加入黑名单中：



sudo ipset add blacklist 192.168.56.3

sudo ipset add blacklist 192.168.56.1

此时ssh连接中断，使用vnc连接查看：



fgp@controller:~/github/int32bit.github.io$ sudo ipset list blacklist

Name: blacklist

Type: hash:ip

Revision: 2

Header: family inet hashsize 1024 maxelem 65536

Size in memory: 176

References: 1

Members:

192.168.56.1

192.168.56.3

把192.168.56.1移除黑名单：



sudo ipset del blacklist 192.168.56.1

我们上面的例子指定的类型为ip，除了ip，还可以是网络段，端口号（支持指定TCP/UDP协议），mac地址，网络接口名称，或者上述各种类型的组合。比如指定 hash:ip,port就是 IP地址和端口号共同作为hash的键。指定类型为net既可以放入ip地址，也可以放入网络地址。



另外ipset还支持timeout参数，可以指定时间，单位为秒，超过这个时间，ipset会自动从集合中移除这个元素，比如封192.168.56.11分钟时间不允许访问



sudo ipset create blacklist hash:net timeout 300

sudo ipset add blacklist 192.168.56.1 timeout 60

以上首先创建了支持timeout的集合，这个集合默认超时时间为300s，接着把192.168.56.1加入到集合中并设置时间为60s。



注意：执行ipset add时指定timeout必须保证创建的集合支持timeout参数，即设置默认的timeout时间.如果不想为集合设置默认timeout时间，而又想支持timeout，可以设置timeout为0，相当于默认不会超时。

