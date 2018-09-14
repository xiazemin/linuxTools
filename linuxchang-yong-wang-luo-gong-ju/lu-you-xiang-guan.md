nc

nc\(netcat\)被称为网络工具的瑞士军刀，其非常轻巧但功能强大！常常作为网络应用的Debug分析器，可以根据需要创建各种不同类型的网络连接。官方描述的功能包括:

simple TCP proxies

shell-script based HTTP clients and servers

network daemon testing

a SOCKS or HTTP ProxyCommand for ssh\(1\)

and much, much more

总之非常强大，能够实现简单的聊天工具、模拟ssh登录远程主机、远程传输文件等。一个经典的用法是端口扫描。比如我要扫描192.168.56.2主机1~100端口，探测哪些端口开放的（黑客攻击必备）：

fgp@controller:~$ nc -zv 192.168.56.2 1-100 \|& grep 'succeeded!'

Connection to 192.168.56.2 22 port \[tcp/ssh\] succeeded!

Connection to 192.168.56.2 80 port \[tcp/http\] succeeded!

从结果中发现，该主机打开了22和80端口。

tcpdump

tcpdump\(dump traffic on a network\)是一个强大的命令行抓包工具，千万不要被它的名称误导以为只能抓取tcp包，它能抓任何协议的包。它能够实现Wireshark一样的功能，并且更加灵活自由！比如需要抓取目标主机是192.168.56.1，通过端口22的传输数据包：

sudo tcpdump -n -i eth1 'dst host 192.168.56.1 && port 22'

输出为：

tcpdump: verbose output suppressed, use -v or -vv for full protocol decode

listening on eth1, link-type EN10MB \(Ethernet\), capture size 65535 bytes

23:57:39.507490 IP 192.168.56.2.22 &gt; 192.168.56.1.54558: Flags \[P.\], seq 3010719012:3010719120, ack 1116715283, win 354, options \[nop,nop,TS val 1049052 ecr 187891473\], length 108

23:57:39.507607 IP 192.168.56.2.22 &gt; 192.168.56.1.54558: Flags \[P.\], seq 108:144, ack 1, win 354, options \[nop,nop,TS val 1049052 ecr 187891473\], length 36

23:57:39.507784 IP 192.168.56.2.22 &gt; 192.168.56.1.54558: Flags \[P.\], seq 144:252, ack 1, win 354, options \[nop,nop,TS val 1049052 ecr 187891476\], length 108

抓取HTTP包:

sudo tcpdump  -XvvennSs 0 -i eth0 tcp\[20:2\]=0x4745 or tcp\[20:2\]=0x4854

其中0x4745为"GET"前两个字母"GE",0x4854为"HTTP"前两个字母"HT"。

指定-A以ACII码输出数据包，使用-c指定抓取包的个数。

route

route命令用于查看和修改路由表：



查看路由表:



fgp@controller:~$ sudo route -n

Kernel IP routing table

Destination     Gateway         Genmask         Flags Metric Ref    Use Iface

0.0.0.0         192.168.1.1     0.0.0.0         UG    100    0        0 brqcb225471-1f

172.17.0.0      0.0.0.0         255.255.0.0     U     0      0        0 docker0

192.168.1.0     0.0.0.0         255.255.255.0   U     0      0        0 brqcb225471-1f

192.168.56.0    0.0.0.0         255.255.255.0   U     0      0        0 eth1

增加/删除路由分别为add/del子命令,比如删除默认路由：



sudo route del default

增加默认路由，网关为192.168.1.1，网卡为brqcb225471-1f：



sudo route add default gw 192.168.1.1 dev brqcb225471-1f

ip

ip命令可以说是无比强大了，它完全可以替换ifconfig、netstat、route、arp等命令，比如查看网卡eth1 IP地址：



\[\] 内的内容意思是：可写可不写

如果是{}，那就必须要在{}内给出的选择里选一个。

fgp@controller:~$ sudo ip addr ls  dev eth1

3: eth1: &lt;BROADCAST,MULTICAST,UP,LOWER\_UP&gt; mtu 1500 qdisc pfifo\_fast state UP group default qlen 1000

    link/ether 08:00:27:9a:d5:d1 brd ff:ff:ff:ff:ff:ff

    inet 192.168.56.2/24 brd 192.168.56.255 scope global eth1

       valid\_lft forever preferred\_lft forever

    inet6 fe80::a00:27ff:fe9a:d5d1/64 scope link

       valid\_lft forever preferred\_lft forever

查看网卡eth1配置：



fgp@controller:~$ sudo ip link ls eth1

3: eth1: &lt;BROADCAST,MULTICAST,UP,LOWER\_UP&gt; mtu 1500 qdisc pfifo\_fast state UP mode DEFAULT group default qlen 1000

    link/ether 08:00:27:9a:d5:d1 brd ff:ff:ff:ff:ff:ff

查看路由：



fgp@controller:~$ ip route

default via 192.168.1.1 dev brqcb225471-1f

172.17.0.0/16 dev docker0  proto kernel  scope link  src 172.17.0.1

192.168.1.0/24 dev brqcb225471-1f  proto kernel  scope link  src 192.168.1.105

192.168.56.0/24 dev eth1  proto kernel  scope link  src 192.168.56.2

查看arp信息：



fgp@controller:~$ sudo ip neigh

192.168.56.1 dev eth1 lladdr 0a:00:27:00:00:00 REACHABLE

192.168.0.6 dev vxlan-80 lladdr fa:16:3e:e1:30:c8 PERMANENT

172.17.0.2 dev docker0 lladdr 02:42:ac:11:00:02 STALE

192.168.56.3 dev eth1  FAILED

192.168.1.1 dev brqcb225471-1f lladdr 30:fc:68:41:12:c6 STALE

查看网络命名空间:



fgp@controller:~$ sudo ip netns ls

qrouter-24bf83c7-f61d-496b-8115-09f0f3d64d21

qdhcp-9284d7a8-711a-4927-8a10-605b34372768

qdhcp-cb225471-1f85-4771-b24b-a4a7108d93a4

进入某个网络命名空间:



fgp@controller:~$ sudo ip netns exec qrouter-24bf83c7-f61d-496b-8115-09f0f3d64d21 bash

root@controller:~\# ifconfig

qg-0d258e6d-83 Link encap:Ethernet  HWaddr fa:16:3e:93:6f:a3

          inet addr:172.16.1.101  Bcast:172.16.1.255  Mask:255.255.255.0

          inet6 addr: fe80::f816:3eff:fe93:6fa3/64 Scope:Link

          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1

          RX packets:1035 errors:0 dropped:0 overruns:0 frame:0

          TX packets:16 errors:0 dropped:0 overruns:0 carrier:0

          collisions:0 txqueuelen:1000

          RX bytes:102505 \(102.5 KB\)  TX bytes:1200 \(1.2 KB\)

 

brctl

brctl是linux网桥管理工具，可用于查看网桥、创建网桥、把网卡加入网桥等。



查看网桥:



fgp@controller:~$ sudo brctl show

bridge name     bridge id               STP enabled     interfaces

brq9284d7a8-71          8000.12841adee45f       no              tap36daf550-27

                                                        tape729e013-df

                                                        vxlan-80

brqcb225471-1f          8000.080027c9b4f2       no              eth0

                                                        tap0d258e6d-83

                                                        tapb844e7a5-83

docker0         8000.0242e4580b61       no              veth50ed8dd

以上因为部署了openstack neutron以及docker，因此网桥比较复杂。 其他子命令如addbr用于创建网桥、delbr用户删除网桥（删除之前必须处于down状态，使用ip link set br\_name down\)、addif把网卡加到网桥等。



traceroute

ping命令用于探测两个主机间连通性以及响应速度，而traceroute会统计到目标主机的每一跳的网络状态（print the route packets trace to network host），这个命令常常用于判断网络故障，比如本地不通，可使用该命令探测出是哪个路由出问题了。如果网络很卡，该命令可判断哪里是瓶颈：



fgp@controller:~$ sudo traceroute  -I -n int32bit.me

traceroute to int32bit.me \(192.30.252.154\), 30 hops max, 60 byte packets

 1  192.168.1.1  4.610 ms  5.623 ms  5.515 ms

 2  117.100.96.1  5.449 ms  5.395 ms  5.356 ms

 3  124.205.97.48  5.362 ms  5.346 ms  5.331 ms

 4  218.241.165.5  5.322 ms  5.310 ms  5.299 ms

 5  218.241.165.9  5.187 ms  5.138 ms  7.386 ms

 ...

可以看到，从主机到int32bit.me共经过30跳，并统计了每一跳间的响应时间。



另外可以参考tracepath。



mtr

mtr是常用的网络诊断工具\(a network diagnostic tool\)，它把ping和traceroute并入一个程序的网络诊断工具中并实时刷新。



mtr -n int32bit.me

