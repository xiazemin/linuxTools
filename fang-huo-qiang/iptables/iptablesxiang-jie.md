基本概念：



1.防火墙工作在主机边缘:对于进出本网络或者本主机的数据报文，根据事先设定好的检查规则对其检查，对形迹可疑的报文一律按照事先定义好的处理机制做出相应处理



对linux而言tcp/ip协议栈是在内核当中，意味着报文的处理是在内核中处理的，也就是说防火墙必须在工作在内核中，防火墙必须在内核中完成tcp/ip报文所流进的位置，用规则去检查，才真正能工作起来。



iptables用来衡量tcp/ip报文的属性：源ip、目标ip、源端口、目标端口；



tcp标志位:   syn、syn+ack、ack、 fin、urg、psh、rst ；



2.应用网关



众多代理服务器都是应用网关，比如squid（使用acl限制应用层）varish这一类代理服务等。



3，入侵检测系统（IDS）：



·网络入侵检测系统  NIDS



·主机入侵检测系统  HIDS



对于IDS常用的检测服务有：snort等



4.入侵防御系统（IPS），比如蜜罐



部署一套入侵检测系统是非常麻烦的，因为必须检测网络任意一个位置



对于IPS常用的检测服务有： tripwire 等



iptables基本概念



对linux来说，是能够实现主机防火墙的功能组件，如果部署在网络边缘，那么既可以扮演网络防火墙的角色，而且是纯软件的



网络数据走向：



请求报文à网关à路由à应用程序（等待用户请求）à内核处理à路由à发送报文



iptables规则功能



表:



filter主要和主机自身有关，主要负责防火墙功能 过滤本机流入流出的数据包是默认使用的表;



input   :负责过滤所有目标地址是本机地址的数据包，就是过滤进入主机的数据包;



forward  :负责转发流经主机但不进入本机的数据包，和NAT关系很大;



output   :负责处理源地址的数据包，就是对本机发出的数据包;



NAT表：



负责网络地址转换，即来源于目的IP地址和端口的转换，一般用于共享上网或特殊端口的转换服务



snat    :地址转换



dnat    :标地址转换



pnat    :标端口转换



mangle 表：



将报文拆开来并修改报文标志位，最后封装起来



5个检查点（内置链）



·PREROUTING



·INPUT



·FORWORD



·OUTPUT



·POSTROUTING    



多条链整合起来叫做表，比如，在input这个链，既有magle的规则也可能有fileter的规则。因此在编写规则的时候应该先指定表，再指定链



netfilter主要工作在tcp/ip协议栈上的，主要集中在tcp报文首部和udp报文首部



规则的属性定义：



1.网络层协议



主要集中在ip协议报文上



2.传输层协议属性：



主要集中在



tcp



udp



icmp  icmp其并不是真正意义传输层的，而是工作在网络层和传输层之间的一种特殊的协议



3.ip报文的属性：



IP报文的属性为: 源地址.目标地址



4.iptables规则匹配



iptables如何查看表和链



大写字母选项：可以实现某种功能，比如添加删除清空规则链；



小写字母选项：用来匹配及其他；



-L ：list 列表



    -n :数字格式显示ip和端口；



    --line-numbers:显示行号；



    -x ： 显示精确值，不要做单位换算；



 



-t :  指定表



     -t{fillter\|nat\|mangle\|raw}



-v ： 显示详细信息 -v -vvv -vvvv ..可以显示更详细的信息



 



5.其他子命令：



管理链：



-F ：清空链



清空nat表中的input链，格式如下：



\#iptables-t nat -F INPUT



\#清空fllter表所有链：



\#iptables-F



-P : 设定默认策略，为指定链设置默认策略，格式如下：



\#设置fllter表input链的默认规则为丢弃



iptables-t fllter -P INPUT DROP



-N ： 新建一条自定义链（内置链不能删除，如果太多，可以自定义链）



\#自定义连只能被调用才可以发挥作用



iptables-N fillter\_web



-X : 删除自定义空链，如果链内有规则，则无法删除



-Z ：计算器清零



iptables-Z



-E ：重命名自定义链



 



iptables管理规则：



-A   ：append附加规则，将新增的规则添加到链的尾部



-I\[n\] ：插入为第n条规则



-D   : 删除第n条规则



-R\[n\] : 替换第N条



表和链的对应关系：



fillter ：INPUT FORWORD OUTPUT



nat : PREROUTING POSTROUTING  OUTPUT



使用-t指定表来查看指定表内的规则：



\#iptables-t nat -L -n



raw : prerouting output



iptables-t raw -L -n



mangle: prerouting input forword output postrouting



iptables-t mangle -L -n



\#查看规则



\[root@test3~\]\# iptables -L -n

Chain INPUT \(policy ACCEPT\)

target     prot opt source              destination        



Chain FORWARD \(policy ACCEPT\)

target     prot optsource              destination        



Chain OUTPUT \(policy ACCEPT\)

target     prot optsource              destination  



通过以上可以观察到，每一个链都有默认策略：policy ACCEPT



通常只需要修改fllter表的默认策略即可，由此如果有报文请求来访问本机的某个服务，那么则会经过input链，因此进来的报文都是需要做过滤的，那么出去的报文则不需要过滤，在有些特定的场所下也需要做过滤



所以写规则的时候必须放将规则写在正确链上，意义非常重大



规则和默认策略都有2个计数器，通过-v选项可以观察规则的匹配情况



\#iptables -t nat -L -n -v



\[root@sshgw~\]\# iptables -L -n -v



ChainINPUT \(policy ACCEPT 7 packets, 975 bytes\)



pkts bytestarget     prot opt in     out    source              destination        



   0    0 ACCEPT     all  --  lo     \*      0.0.0.0/0           0.0.0.0/0          



   0    0 DROP       all  -- eth2   \*       101.61.0.0/10        0.0.0.0/0          



   0    0 DROP       all  -- eth2   \*       127.0.0.0/8          0.0.0.0/0          



   0    0 DROP       all  -- eth2   \*       162.254.0.0/16       0.0.0.0/0          



   0    0 DROP       all  -- eth2   \*       192.0.0.0/24         0.0.0.0/0          



   0    0 DROP       all  -- eth2   \*       192.0.2.0/24         0.0.0.0/0          



   0    0 DROP       all  -- eth2   \*       197.18.0.0/15        0.0.0.0/0          



   0    0 DROP       all  --  eth2  \*       197.51.100.0/24      0.0.0.0/0          



   0    0 DROP       all  -- eth2   \*       203.0.111.0/24       0.0.0.0/0          



   0    0 DROP       all  -- eth2   \*       224.0.0.0/4          0.0.0.0/0          



   0    0 DROP       all --  eth2   \*      240.0.0.0/4         0.0.0.0/0          



776 37056 REFRESH\_TEMP  all --  \*      \*      0.0.0.0/0           0.0.0.0/0          



编写规则语法：



iptables \[-t 表\] 大写选项子命令 \[规则号\] 链名 匹配标准 -j 目标（规则）



目标：



DROP   :   丢弃



REJECT :   拒绝



ACCEPT :   接受



RETURN ：  返回主链继续匹配



REDIRECT:  端口重定向



MASQUERADE :地址伪装



DNAT :    目标地址转换



SNAT ：源地址转换

MARK ：打标签



LOG  



自定义链



匹配标准



iptables的匹配标准大致分为两类：



1.通用匹配



-s \| --src \| --source \[!\] IP/NETWORK



-d ------------------------



-i :指定数据报文流入接口  input prerouting forward



-o :指定数据报文流出接口  output postrouting forward



-p :明确说明只放行哪种协议的报文匹配规则



以当前主机为例：



凡是来自于某个ip段的网络访问本机



\[root@test3xtables-1.4.7\]\# iptables -A INPUT -s 10.0.10.0/24 -d 10.0.10.0/24 -j ACCEPT

\[root@test3 xtables-1.4.7\]\# iptables -L -n -v



ChainINPUT \(policy ACCEPT 10 packets, 1029 bytes\)



pkts bytestarget    prot opt  in    out      source                destination



22  1660    ACCEPT     all  --  \*      \*       10.0.10.0/24                10.0.10.0/24 



Chain FORWARD \(policy ACCEPT 0 packets, 0 bytes\)

pkts bytes target     prot opt in    out       source                   destination 



Chain OUTPUT \(policy ACCEPT 16 packets, 1536 bytes\)

pkts bytes target     prot opt in    out       source                  destination    



pkts     被本机报文所匹配的个数



bytes   报文所有大小记起来之和



opt     额外的选项，--表示没有



target   处理机制



prot     放行哪种协议



source  源地址



destination  目标地址



 



对于严谨的规则，一般默认规则都是拒绝未知，允许已知



如下所示：



只放行信任IP地址段，其他全部禁止



iptables-P INPUT DORP



iptables-A INPUT -s   10.0.10.0/24   -d  10.0.10.0/24 -j ACCEPT



iptables-P OUTPUT DORP



iptables-A OUTPUT -d   10.0.10.0/24  -s    10.0.10.0/24-j ACCEPT



保存规则



\[root@test3~\]\# /etc/init.d/iptables save



iptables:Saving firewall rules to /etc/sysconfig/iptables:\[  OK  \]



保存规则至其他文件



\[root@test3~\]\# iptables-save &gt; /tmp/iptables  



加载iptables文件规则



\[root@test3~\]\# iptables-resotre &lt; /tmp/iptables  



1.2.规则的替换



首先来查看规则



\[root@test3 ~\]\# iptables -L -n --line-number



ChainINPUT \(policy ACCEPT\)



num  target    prot opt source              destination        



1    ACCEPT    all  --  10.0.10.0/24         10.0.10.0/24        



 



ChainFORWARD \(policy DROP\)



num  target    prot opt source              destination        



 



ChainOUTPUT \(policy ACCEPT\)



num  target    prot opt source              destination



替换规则：将规则1替换为 eth0只能够通过某个网段进来



\[root@test3~\]\# iptables -R  INPUT 1 -s 10.0.10.0/24-d 10.0.10.62 -i eth0 -j ACCEPT



\[root@test3~\]\# iptables -L -n --line-number



ChainINPUT \(policy ACCEPT\)



num  target    prot opt source              destination        



1    ACCEPT    all  --  10.0.10.0/24         10.0.10.62    



2.扩展匹配



\#所有的扩展匹配表示要使用-m来指定扩展的名称来引用，而每个扩展模块一般都会有自己特有的专用选项，在这些选项中，有些是必备的：



 



2.1隐含扩展



如下所示：



\#端口之间必须是连续的



-p tcp--sport\|--dport 21-80



\#取反，非21-80的端口



-p tcp--sport\|--dport !21-80



\#检测报文中的标志位



--tcp-flagsSYN,ACK,RST,FIN, SYN



ALL                   \#表示为所有标志位



NONE                    \#表示没有任何一个标志位



\#--tcp-flags ALL NONE   \#表示所有标志位都检测，但是其中多有都为0



\#--tcp-flage ALL SYN,FIN \#表示SYN,FIN都为1（即握手又断开）



\#生成环境下tcp-flags 用的非常多，意义非常重要



例：放行本机对web的访问



\[root@test3~\]\# iptables -A INPUT -d 10.0.10.62  -ptcp --dport 80 -j ACCEPT



\[root@test3~\]\# iptables -L -n



ChainINPUT \(policy DROP\)



target     prot opt source               destination        



ACCEPT     all --  10.0.10.0/24         10.0.10.62          



ACCEPT     tcp --  0.0.0.0/0            10.0.10.62          tcp dpt:80



放行出去的报文，源端口为80



\[root@test3~\]\# iptables -A OUTPUT -s 10.0.10.62 -p tcp --sport 80 -j ACCEPT



查看匹配规则



\[root@test3 ~\]\# iptables -L -n --line-number



ChainINPUT \(policy DROP\)



num  target    prot opt source              destination        



1    ACCEPT    all  --  10.0.10.0/24         10.0.10.62          



2    ACCEPT    tcp  --  0.0.0.0/0            10.0.10.62          tcp dpt:80



 



ChainFORWARD \(policy DROP\)



num  target    prot opt source              destination        



 



ChainOUTPUT \(policy DROP\)



num  target    prot opt source              destination        



1    ACCEPT    all  --  10.0.10.0/24         10.0.10.0/24        



2    ACCEPT    tcp  --  10.0.10.62           0.0.0.0/0           tcp spt:80



考虑要点：



（1）规则为放行出去的响应报文



（2）考虑源IP地址为本机，目标为访问的时候拆开报文才可以获知，而写规则的时候是面向所有主机，所以这里不用写



（3）源端口：80 ，因为用户访问的时候一定会访问其80端口，无可非议的



（4）目标端口：请求到来的时候事先无法断定对方的端口是多少，所以不用写



 



2.2协议匹配



通常对协议做匹配则使用 -p 参数 来指定协议即可



匹配UDP：UDP只有端口的匹配，没有任何可用扩展，格式如下



-p udp--sport \| --dport



匹配ICMP格式如下



-picmp --icmp-\[number\]



icmp常见类型：请求为8（echo-request），响应为0\(echo-reply\)



例：默认规则input output 都为DROP,使其本机能ping（响应的报文）的报文出去



通过此机器去ping网关10.0.10.1 ， 可结果却提示not permitted，使其能通10.0.10.0/24网段中的所有主机



\[root@test3~\]\#iptables -A OUTPUT -s 10.0.10.62 -d 10.0.10.0/24 -p icmp --icmp-type8 -j ACCEPT



可看到无法响应：0表示响应进来的报文规则，并没有放行自己作为服务端的的角色规则



\[root@test3~\]\# iptables -A INPUT -s 10.0.10.0/24 -d 10.0.10.62 -p icmp --icmp-type0 -j ACCEPT



\#ping 10.0.10.x



允许类型为0（响应报文）出去



\[root@test3~\]\# iptables -A OUTPUT -s 10.0.10.62 -d  10.0.10.0/24 -picmp --icmp-type 0 -j ACCEPT



例2：本机DNS服务器，要为本地客户端做递归查询；iptables的input output默认为drop 本机地址是10.0.10.62



\[root@test3~\]\# iptables -A INPUT -d 10.0.10.62 -p udp --dprot 53 -j ACCEPT



\[root@test3~\]\# iptables -A OUTPUT -S 10.0.10.62 -p udp --sprot 53 -j ACCEPT



客户端请求可以进来，响应也可以出去，但是自己作为客户端请求别人是没有办法出去的，所以：



\[root@test3~\]\# iptables -A OUTPUT -s 10.0.10.62 -p udp --dport 53 -j ACCEPT



\[root@test3~\]\# iptables -A INPUT -d 10.0.10.62 -p udp --sprot 53 -j ACCEPT



如果为tcp 则将以上udp改为tcp即可

