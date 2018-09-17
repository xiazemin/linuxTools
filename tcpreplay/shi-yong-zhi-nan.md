1. 什么是tcpreplay



引用一段tcpreplay官方网站\(http://tcpreplay.synfin.net/trac/\)的话来解释什么是

tcpreplay\[1\]:



\#摘自tcpreplay官方网站\(http://tcpreplay.synfin.net/trac/\):

\|Tcpreplay is a suite of BSD licensed tools written by Aaron Turner for UNIX

\|\(and Win32 under Cygwin\) operating systems which gives you the ability to use

\|previously captured traffic in libpcap format to test a variety of network

\|devices. It allows you to classify traffic as client or server, rewrite Layer

\|2, 3 and 4 headers and finally replay the traffic back onto the network and

\|through other devices such as switches, routers, firewalls, NIDS and IPS's.

\|Tcpreplay supports both single and dual NIC modes for testing both sniffing

\|and inline devices. 



简单的说, tcpreplay是一种pcap包的重放工具, 它可以将用ethreal, wireshark工具抓

下来的包原样或经过任意修改后重放回去. 它允许你对报文做任意的修改\(主要是指对2层

, 3层, 4层报文头\), 指定重放报文的速度等, 这样tcpreplay就可以用来复现抓包的情景

以定位bug, 以极快的速度重放从而实现压力测试.



tcpreplay本身包含了几个辅助工具, 用于准备发包的cache, 重写报文等:



    \* tcpprep - 简单的说就是划分哪些包是client的, 哪些是server的, 一会发包的

    时候client的包从一个网卡发, server的包可能从另一个网卡发.

    \* tcprewrite - 简单的说就是修改2层, 3层, 4层报文头部. 

    \* tcpreplay - 真正发包, 可以选择主、从网卡, 发包速度等.

    \* tcpbridge - bridge two network segments with the power of tcprewrite 



2. 安装指南



tcpreplay官方提供的下载地址为: http://tcpreplay.synfin.net/trac/wiki/Download,

由于tcpreplay依赖libpcap库,

所以安装tcpreplay之前必须先安装libpcap\(在windows下为winpcap\), 否则

./configure的时候你会得到提示说libpcap库没有安装.



linux下的依赖库libpcap由tcpdump工程组开发, 好像也是个开源工程, 可以到

http://www.tcpdump.org/下载到, 可以用源码安装, 貌似比较简单. 



windows\(包括cygwin\) 下的依赖库winpcap则必须到 winpcap的官方网站上去下载:

http://www.winpcap.org/install/default.htm. winpcap是libpcap在windows上的移植,

这个貌似不是开源的, 所以你只能得到一个静态库和编程接口"WpdPack\_4\_1\_2.zip",

解压缩后可以得到文件夹"WpdPack", 将该文件夹拷贝到cygwin的根目录,

即可完成winpcap的安装, 在 "./configure"

的时候选上参数--with-libpcap=/wpdpack\(我自己试验过,

貌似没有这个参数也可以成功, 不过还是建议加上这个参数\)\[2\]:



\#winpcap的安装过程:

\|$ unzip WpdPack\_4\_1\_2.zip

\|$ cp -r WpdPack/ / \(安装tcpreplay的依赖winpcap, 即把WpdPack拷贝到根目录下.\)



\#tcpreplay的安装过程:

\|$ ./autogen.sh \(Subversion checkouts only\) 

\|$ ./configure --with-libpcap=/wpdpack

\|$ make 

\|\# make test \(Note: tcprewrite tests are currently broken on Cygwin/Win32\) 

\|\# make install 



3. 使用指南



到此为止, 你机器上的tcpreplay已经可以使用了, 具体怎么用, 网上的攻略也很多, 但

最权威的使用指南当然是官网的Online Manual:

http://tcpreplay.synfin.net/trac/wiki/manual, 前面简单地介绍了tcpreplay内嵌的

几个工具的用途, 下面给出一个我用过的一个例子, 仅供更好的理解Online Manual:



以前的版本貌似可以用一个命令把pcap包直接经过修改发出去, 但是3.0以后的

tcpreplay不支持这样了, 发包前先得用tcpgrep建立一个cache, 再用tcprewite修改包的

信息, 最后用tcpreplay发出去:



建立cache文件的作用解释，主要是加速报文的发送，cache文件中存放着pcap文件中每个

帧的编号和时间戳等信息，以达到tcpreplay回放时可以更加快速的发送报文的目的。



3.1 tcpprep\(pcap pre-processor\)



tcpprep工具会生成一个cache文件, 里面保存着哪些包将从主网口发出去, 哪些包将从从

网口发出去. 举个例子, 如果你用wireshark抓了一个pcap文件, 里面可能既有A地址发给

B地址的包, 也有B地址发给A址的包, 用tcpprep工具可以指定从A到B的包从主网卡发出, 

从B到A的包从次网卡发出.



3.1.1 根据报文源IP确定client/server报文



\#tcpprep的用法举例, 根据源IP:

\|$ tcpprep -c 172.22.64.2/24 -i mgcp.pcap -o mgcp.cach



上面的命令指定所有源IP为172.22.64.2/24的包, 都将从主网卡发出, 其它的从次网卡发

出. 输入文件是mgcp.pcap, 输出文件为mgcp.cach.



3.1.2 使用自动模式确定client/server报文



\#tcpprep的用法举例, 自动模式:

\|$ tcpprep -a client -i mgcp.pcap -o mgcp.cach



上面的命令采用自动/client模式指定分包模式. 自动模式这里按我的理解解释一下:

tcpprep在自动模式下认为有下面行为的IP为client端: 1.发TCP SYN包的一方, 2.发DNS 

包的一方, 3. 收入到 ICMP-Port Unreachable的一方. 认为有下面行为的一方为server 

端: 1.发TCP Syn/Ack的一方, 2.发DNS应答的一方, 3.发ICMP-Port Unreachable的一方.

而被认定为server的那一方发的那些包, 将从主网卡发出, 被认定为client的包则从次网

卡发出. 而自动/client模式将所有没有认出的包都归为client, 同理自动/server模式将

没有认出的包都归为server.这种模式貌似不如按IP地址分类的方式好用.



tcpprep还有很多其它指定发送方向的方式, 详情请阅Online Manual或man手册.



3.2 tcprewrite



简单地说, tcprewrite就是改写pcap包里的报文头部, 包括2层, 3层, 4层, 5-7层. 从3.0

版本以后, 所有改写pcap报文头的操作都从tcpreplay中移到了tcprewrite里了.



使用tcprewrite对packet进行修改可以有两个方法, 一种方法是一次修改一项, 生成一个

文件, 再拿这个文件作为输入文件..., 直到完成最后的修改, 如:



tcprewrite --option1=xxx -c input.cach -i input.pcap -o 1.pcap

tcprewrite --option2=xxx -c input.cach -i 1.pcap -o 2.pcap

...

tcprewrite --optionN=xxx -c input.cach -i N-1.pcap -o N.pcap



还有一种方法是一古脑的把所有的选项放到一个命令里完成, 如:



tcprewrite --option1=xxx --option2=xxx ... --optionN=xxx -i input.pcap -c input.cach -o out.pcap



两种方法都是可行的, 也各有利弊. 第一方法明了, 但是复杂, 第二种方法简单但不容易

理解. 我的建议是先用第一种方法做试验, 容易调试, 等修改成功了以后再把所有的选项

合到一起, 在实际使用的时候用第二种方法. 下面我先给出一个例子, 然后再逐个分析一

下用tcprewrite是如何修改二层, 三层, 四层, 5-7层头的, 以便理解tcprewrite的工作原

理.



3.2.1 tcprewrite的一个例子



tcpreplay只保证能把包送出去, 至于包真正能到达的地址, 我认为还是根据原来的包的

IP和mac. 如果是在同一网段, 可能需要把mac地址改成测试设备的mac, 如果需要经过网

关, 就得将IP地址改为测试设备的IP以及端口号.



tcprewrite的基本格式是\(请注意命令中是没有换行符的, 只为了阅读的方便添加了换行

符\): 详请可以用tcprewrite ?命令查询详情.



\#tcprewrite的格式:

\|$ tcprewrite --enet-smac=host\_src\_mac,client\_src\_mac   \

\|              --enet-dmac=host\_dst\_mac, client\_dst\_mac \

\|              --endpoints=host\_dst\_ip:client\_dst\_ip    \

\|              --portmap=old\_port1:new\_port1,old\_port2, new\_port2 \

\|              -i input.pcap -c input.cach -o out.pcap



解释一下, 该命令的输入参数是input.pcap和input.cach文件, 结果将另存为out.pcap文

件. 该命令将所有input.pcap包里的主机包\(由input.cach文件指定哪些包是主机包, 哪些

包是客户端包\)的源mac地址, 目的mac地址, 目的IP地址分别改为 :host\_src\_mac,

host\_dst\_mac和host\_dst\_ip, 客户端包源mac地址, 目的mac地址, 目的IP地址分别改为

:client\_src\_mac, client\_dst\_mac和client\_dst\_ip, 将端口号由old\_port1改为

new\_port1, 将端口号由old\_port2改为new\_port2.



\#tcprewrite的用法举例:

\|$ tcprewrite --enet-smac=11:22:22:22:22:22,22:22:22:22:22:22   \

\|             --enet-dmac=11:11: 11:11:11:11,22:11:11:11:11:11  \

\|             --endpoints=192.168.0.1:192.168.0.11              \

\|             --portmap=5070:5061,9060:5060                     \

\|             -i success.pcap -o out.pcap -c success.cach



该命令将修改后的包, 主机包的二层, 三层, 四层头分别为: 11:22:22:22:22:22,

192.168.0.1, 5061, 客户端包的二层, 三层, 四层头分别为: 22:22:22:22:22:22,

192.168.0.11, 5060.



3.2.2 修改2层头



1\) 修改MAC地址



如果不指定cache文件, 将把所有包的源mac地址和目的mac地址都改写成

00:44:66:FC:29:AF和00:55:22:AF:C6:37:



    $ tcprewrite --enet-dmac=00:55:22:AF:C6:37 --enet-smac=00:44:66:FC:29:AF --infile=input.pcap --outfile=output.pcap



指定cache文件后, 将server包的目的/源mac地址改写成

00:44:66:FC:29:AF/00:66:AA:D1:32:C2, 将client的目的/源mac地址改成:

00:55:22:AF:C6:37/00:22:55:AC:DE:AC, 注意是server地址在前.



    $ tcprewrite --enet-dmac=00:44:66:FC:29:AF,00:55:22:AF:C6:37 --enet-smac=00:66:AA:D1:32:C2,00:22:55:AC:DE:AC --cachefile=input.cache --infile=input.pcap --outfile=output.pcap



2\) 修改802.1q VLAN



经常客户的抓包带有VLAN头域, 这些包如果不去掉VLAN头是没有办法在自己的交换机上

replay的, tcprewrite提了去掉或添加VLAN的方法:



去掉vlan很简单:



    $ tcprewrite --enet-vlan=del --infile=input.pcap --outfile=output.pcap



添加vlan也很简单, 下面的命令将VLAN tag设成40, CFI设成1, VLAN priority设成4. 



    $ tcprewrite --enet-vlan=add --enet-vlan-tag=40 --enet-vlan-cfi=1 --enet-vlan-pri=4 --infile=input.pcap --outfile=output.pcap



3\) 修改二层协议名:



好像是将Ethernet协议头转成Cisco HDLC或其它二层协议? 这部分没有真正用过, 需要的

人自行参考\[2\].



3.2.3 修改3层头



从版本3.4.2开始, tcprewrite开始支持ipv6协议\(我写这篇文章的时候版本号还是3.4.1呢

, tcpreplay升级蛮快哦^\)^\). tcprewrite修改IP地址后会自动帮你计算校验和, 这点还是

蛮周到的^\)^, 命令行传入IPv6地址的时候要使用方括号, 例如: \[2001::dead:beef\]或

\[2001::/16\]



1\) 修改目的IP



根据cache文件里的标识, 将server的IP改为10.10.1.1, client的IP改为10.10.1.2:



$ tcprewrite --endpoints=10.10.1.1:10.10.1.2 --cachefile=input.cache --infile=input.pcap --outfile=output.pcap --skipbroadcast 



2\) 修改IP地址的网络部分



注: 2\)和3\)没有验证过



众所周知, IP地址同网络部分和主机部分组成, 下面的命令可以将子网地址为10.0.0.0/8

或192.168.0.0/16的IP改成子网为172.16.0.0/12:



    $ tcprewrite --pnat=10.0.0.0/8:172.16.0.0/12,192.168.0.0/16:172.16.0.0/12 --infile=input.pcap --outfile=output.pcap --skipbroadcast



下面的命令是基于client包或server包修改子网地址:



    $ tcprewrite --pnat=10.0.0.0/8:192.168.0.0/24 --pnat=10.0.0.0/8:192.168.1.0/24 --cachefile=input.cache --infile=input.pcap --outfile=output.pcap --skipbroadcast



3\) 修改IP头的其它部分:



修改IPv4头的TOS为50



    $ tcprewrite --tos=50 --infile=input.pcap --outfile=output.pcap



将IPv6头Traffic Class值改为33



    $ tcprewrite --tclass=33 --infile=input.pcap --outfile=output.pcap





修改Flow Label field:



    $ tcprewrite --flowlabel=67234 --infile=input.pcap --outfile=output.pcap



3.2.4 修改4层头



和修改IP头一样, 修改4层头的时候tcpwrite会自动计算校验和, 这个就不需要担心了.



1\) 修改端口号



将80端口号改为8080, 22改为8022:



    $ tcprewrite --portmap=80:8080,22:8022 --infile=input.pcap --outfile=output.pcap



2\) 强制计算传输层校验和:



有些应用可能不计算传输层的校验和, 可以让tcpwrite强制计算一下:



    $ tcprewrite --fixcsum --infile=input.pcap --outfile=output.pcap



3.2.5 修改5-7层数据



tcpwrite对5-7层的修改非常有限, 顶多也就是抓包没有抓全, 中间的应用层数据丢了.

tcpwrite将没有抓到的数据补成全0, 或者修改tcp/udp的长度字节, 或者将该包丢弃. 有

需要的直接参考官方资料吧.\[2\]



3.3 tcpreplay



在linux下, 用ifconfig命令可以获得接口的名字, 但在cygwin下必须用下面的命令获得

接口的名字:



\#在cygwin下获得接口的名字:

\|$ tcpreplay --listnics



\#结果可能如下所示:

\|$ tcpreplay --listnics

\|Available network interfaces:

\|Alias   Name    Description

\|%0      \Device\NPF\_GenericDialupAdapter

\|        Adapter for generic dialup and VPN capture

\|%1      \Device\NPF\_{6B508B29-B3E3-4D0B-892F-02914AC9A668}

\|    Intel\(R\) 82566DM Gigabit Network Connection \(Microsoft's Packet

\|    Scheduler\) 

\|%2      \Device\NPF\_{CBCE38CA-1FAD-4AEB-89DF-FD2D8EF861FA}

\|    D-Link DFE-530TX PCI Fast Ethernet Adapter \(rev.C\)

\|    \(Microsoft's Packet Scheduler\) 

\|%3      \Device\NPF\_{ABB813FE-3C51-49A3-8146-16CD2C4507C3}

\|    D-Link DFE-530TX PCI Fast Ethernet Adapter \(rev.C\)

\|    \(Microsoft's Packet Scheduler\) 



从中可以看出这台机器有两块网卡, 一块叫做%1, 另一块叫做%2. 下面就可以指定网卡

重发包了:



\#用tcpreplay发包:

\|$ tcpreplay -c mgcp.cach -i %1 -I %2 out.pcap



这条命令是把out.pcap包, 按照mgcp.cach里划分的主机包和客户端包的方式, 将主机包

从网卡%1发出去, 将客户端包从网卡%2发出去.

