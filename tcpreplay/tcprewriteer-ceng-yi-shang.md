简单地说, tcprewrite 就是改写 pcap 包里的报文头部, 包括 2 层, 3 层, 4 层，即 MAC 地址、IP 地址和PORT 等。tcpreplay 只保证能把包送出去, 至于包真正能到达的地址, 我认为还是根据原来的包的 IP 和 mac，



如果是在同一网段，可能需要把 mac 地址改成测试设备的 mac，如果需要经过网关, 就得将 IP 地址改为测试设备的 IP 以及端口号 。



 



 



　　tcpreplay只保证能把包送出去, 至于包真正能到达的地址, 我认为还是根据原来的包的IP和mac. 如果是在同一网段, 可能需要把mac地址改成测试设备的mac, 如果需要经过网关, 就得将IP地址改为测试设备的IP以及端口号。

　　tcprewrite的基本格式是\(请注意命令中是没有换行符的, 只为了阅读的方便添加了换行符\): 详请可以用tcprewrite ?命令查询详情。



\[root@datatest ~\]\#   tcprewrite --enet-smac=host\_src\_mac,client\_src\_mac   \

              --enet-dmac=host\_dst\_mac, client\_dst\_mac \

              --endpoints=host\_dst\_ip:client\_dst\_ip    \

              --portmap=old\_port1:new\_port1,old\_port2, new\_port2 \

\|             -i input.pcap -c input.cach -o out.pcap

　　解释一下：



　　该命令的输入参数是input.pcap和input.cach文件, 结果将另存为out.pcap文件。



　　该命令将所有input.pcap包里的主机包\(由input.cach文件指定哪些包是主机包, 哪些包是客户端包\)的源mac地址, 目的mac地址。



　　 目的IP地址分别改为 :host\_src\_mac,host\_dst\_mac和host\_dst\_ip。



　　客户端包源mac地址, 目的mac地址。



　　目的IP地址分别改为 :client\_src\_mac, client\_dst\_mac和client\_dst\_ip。



　　将端口号由old\_port1改为 new\_port1, 将端口号由old\_port2改为new\_port2。





 



 









 



 



 



 









　　举个小例子



\[root@datatest ~\]\# tcprewrite --enet-smac=11:22:22:22:22:22,22:22:22:22:22:22  \

--enet-dmac=FF:FF:FF:FF:FF:FF   \

--endpoints=192.168.0.1:192.168.0.11    \

--portmap=5070:5061,9060:5060    \

-i success.pcap -o out.pcap -c success.cach

　　该命令将修改后的包的



　　主机端的二层, 三层, 四层头分别为: 11:22:22:22:22:22,192.168.0.1,5061,



　　客户端包的二层, 三层, 四层头分别为: 22:22:22:22:22:22,192.168.0.11, 5060。





 



 



 



\[root@datatest ~\]\#  tcprewrite --seed=6 -i success.pcap -o out.pcap -c success.cach

　　该命令将根据数字 6 按特征算法生成一个新的 server\_ip 和 client\_ip。



 



 



 



 



\[root@datatest ~\]\#  tcprewrite --pmap=192.168.0.0/16:10.77.0.0/16,172.16.0.0/12:10.1.0.0/24    \

-i success.pcap -o out.pcap -c success.cach

　　该命令将把主机端和客户端 192.168.0.0/16、172.16.0.0/12 网段的 ip 地址分别改为 10.77.0.0/16、10.1.0.0/24。





 



 



\[root@datatest ~\]\#   tcprewrite --srcipmap=192.168.0.0/16:10.77.0.0/16   \

-i success.pcap -o out.pcap -c success.cach

　　该命令将把主机端和客户端的源 ip 为 192.168.0.0/16 网段改为 10.77.0.0/16 网段



 



 



 



\[root@datatest ~\]\#   tcprewrite --dstipmap=172.16.0.0/12:10.1.0.0/24

-i success.pcap -o out.pcap -c success.cach

　　该命令将把主机端和客户端的目的 ip 为 192.168.0.0/16 网段改为 10.77.0.0/16 网段。



 



 



 



 　　修改2层头



　　　　1\) 修改MAC地址

　　　　如果不指定cache文件, 将把所有包的源mac地址和目的mac地址都改写成00:44:66:FC:29:AF和00:55:22:AF:C6:37:



\[root@datatest ~\]\#  tcprewrite --enet-dmac=00:55:22:AF:C6:37 --enet-smac=00:44:66:FC:29:AF --infile=input.pcap --outfile=output.pcap

 



　　指定cache文件后, 将server包的目的/源mac地址改写成00:44:66:FC:29:AF/00:66:AA:D1:32:C2,



　　　　　　　　　　将client的目的/源mac地址改成:00:55:22:AF:C6:37/00:22:55:AC:DE:AC, 注意是server地址在前.



\[root@datatest ~\]\#  tcprewrite --enet-dmac=00:44:66:FC:29:AF,00:55:22:AF:C6:37 --enet-smac=00:66:AA:D1:32:C2,00:22:55:AC:DE:AC --cachefile=input.cache --infile=input.pcap --outfile=output.pcap

 









　　2\) 修改802.1q VLAN

　　经常客户的抓包带有VLAN头域, 这些包如果不去掉VLAN头是没有办法在自己的交换机上replay的, tcprewrite提了去掉或添加VLAN的方法:

　　　　去掉vlan很简单:



\[root@datatest ~\]\#  tcprewrite --enet-vlan=del --infile=input.pcap --outfile=output.pcap

　　



　　添加vlan也很简单, 下面的命令将VLAN tag设成40, CFI设成1, VLAN priority设成4.



\[root@datatest ~\]\#  tcprewrite --enet-vlan=add --enet-vlan-tag=40 --enet-vlan-cfi=1 --enet-vlan-pri=4 --infile=input.pcap --outfile=output.pcap

 



 



　　3\) 修改二层协议名:

　　好像是将Ethernet协议头转成Cisco HDLC或其它二层协议? 这部分没有真正用过, 需要的

人自行参考tcpreplay官方网站: http://tcpreplay.synfin.net/wiki/manual



 



 



 



   修改3层头

　　从版本3.4.2开始, tcprewrite开始支持ipv6协议， tcpreplay升级蛮快。 tcprewrite修改IP地址后会自动帮你计算校验和, 这点还是蛮周到的。



　　命令行传入IPv6地址的时候要使用方括号, 例如: \[2001::dead:beef\]或\[2001::/16\]





　　1\) 修改目的IP

　　根据cache文件里的标识, 将server的IP改为10.10.1.1, client的IP改为10.10.1.2:



\[root@datatest ~\]\#  tcprewrite --endpoints=10.10.1.1:10.10.1.2 --cachefile=input.cache --infile=input.pcap --outfile=output.pcap --skipbroadcast

 





　　2）修改IP地址的网络部分

　　　　注: 2\)和3\)没有验证过。



　　　　众所周知, IP地址同网络部分和主机部分组成, 下面的命令可以将子网地址为10.0.0.0/8或192.168.0.0/16的IP改成子网为172.16.0.0/12:



\[root@datatest ~\]\#  tcprewrite --pnat=10.0.0.0/8:172.16.0.0/12,192.168.0.0/16:172.16.0.0/12 --infile=input.pcap --outfile=output.pcap --skipbroadcast

 



　　下面的命令是基于client包或server包修改子网地址:



\[root@datatest ~\]\#   tcprewrite --pnat=10.0.0.0/8:192.168.0.0/24 --pnat=10.0.0.0/8:192.168.1.0/24 --cachefile=input.cache --infile=input.pcap --outfile=output.pcap --skipbroadcast

 



 



　　3\) 修改IP头的其它部分:

　　



　　修改IPv4头的TOS为50



\[root@datatest ~\]\#  tcprewrite --tos=50 --infile=input.pcap --outfile=output.pcap

 



　　将IPv6头Traffic Class值改为33



\[root@datatest ~\]\#  tcprewrite --tclass=33 --infile=input.pcap --outfile=output.pcap

 



 



　　修改Flow Label field:



\[root@datatest ~\]\#  tcprewrite --flowlabel=67234 --infile=input.pcap --outfile=output.pcap

 



 



 



 修改4层头

　　和修改IP头一样, 修改4层头的时候tcpwrite会自动计算校验和, 这个就不需要担心了.



 



　　1\) 修改端口号

　　　　将80端口号改为8080, 22改为8022:



\[root@datatest ~\]\#  tcprewrite --portmap=80:8080,22:8022 --infile=input.pcap --outfile=output.pcap

 



　　2\) 强制计算传输层校验和:

　　有些应用可能不计算传输层的校验和, 可以让tcpwrite强制计算一下:



\[root@datatest ~\]\#  tcprewrite --fixcsum --infile=input.pcap --outfile=output.pcap

 



 



　　修改5-7层数据

　　tcpwrite对5-7层的修改非常有限, 顶多也就是抓包没有抓全, 中间的应用层数据丢了。tcpwrite将没有抓到的数据补成全0, 或者修改tcp/udp的长度字节, 或者将该包丢弃. 有需要的直接参考官方资料吧。









