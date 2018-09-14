## ping {#ping}

使用这个命令判断网络的连通性以及网速，偶尔还顺带当做域名解析使用（查看域名的IP）：

ping google.com

默认使用该命令会一直发送ICMP包直到用户手动中止，可以使用-c命令指定发送数据包的个数，使用-W指定最长等待时间,如果有多张网卡，还可以通过-I指定发送包的网卡。

小技巧: 在ping过程中按下ctrl+\|会打印出当前的summary信息，统计当前发送包数量、接收数量、丢包率等。

其他比如-b发送广播，另外注意ping只能使用ipv4，如果需要使用ipv6，可以使用ping6命令。

telnet

telnet协议客户端\(user interface to the TELNET protocol\)，不过其功能并不仅仅限于telnet协议，有时也用来探测端口，比如查看本地端口22是否开放：



fgp@controller:~$ telnet localhost 22

Trying ::1...

Connected to localhost.

Escape character is '^\]'.

SSH-2.0-OpenSSH\_6.6.1p1 Ubuntu-2ubuntu2.6

可见成功连接到localhost的22端口，说明端口已经打开，还输出了banner信息。



ifconfig

ifconfig也是熟悉的网卡配置工具\(configure a network interface\)，我们经常使用它来查看网卡信息（比如IP地址、发送包的个数、接收包的个数、丢包个数等）以及配置网卡（开启关闭网卡、修改网络mtu、修改ip地址等）。



查看网卡ip地址：



fgp@controller:~$ ifconfig eth0

eth0      Link encap:Ethernet  HWaddr 08:00:27:c9:b4:f2

          inet6 addr: fe80::a00:27ff:fec9:b4f2/64 Scope:Link

          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1

          RX packets:27757 errors:0 dropped:0 overruns:0 frame:0

          TX packets:589 errors:0 dropped:0 overruns:0 carrier:0

          collisions:0 txqueuelen:1000

          RX bytes:10519777 \(10.5 MB\)  TX bytes:83959 \(83.9 KB\)

为网卡eth0增加一个新的地址（虚拟网卡）：



fgp@controller:~$ sudo ifconfig eth0:0 10.103.240.2/24

fgp@controller:~$ ifconfig eth0:0

eth0:0    Link encap:Ethernet  HWaddr 08:00:27:c9:b4:f2

          inet addr:10.103.240.2  Bcast:10.103.240.255  Mask:255.255.255.0

          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1

关闭网卡以及开启网卡：



sudo ifconfig eth0 down

sudo ifconfig eth0 up

nslookup & dig

nslookup用于交互式域名解析\(query Internet name servers interactively\)，当然也可以直接传入域名作为Ad-Hoc命令使用，比如查看google.com的ip地址：



fgp@controller:~$ nslookup google.com

Server:         114.114.114.114

Address:        114.114.114.114\#53

 

Non-authoritative answer:

Name:   google.com

Address: 37.61.54.158

查看使用的DNS服务器地址：



fgp@controller:~$ nslookup

&gt; server

Default server: 114.114.114.114

Address: 114.114.114.114\#53

Default server: 8.8.8.8

Address: 8.8.8.8\#53

dig命令也是域名解析工具\(DNS lookup utility\)，不过提供的信息更全面：



fgp@controller:~$ dig google.com

 

; &lt;&lt;&gt;&gt; DiG 9.9.5-3ubuntu0.8-Ubuntu &lt;&lt;&gt;&gt; google.com

;; global options: +cmd

;; Got answer:

;; -&gt;&gt;HEADER&lt;&lt;- opcode: QUERY, status: NOERROR, id: 53828

;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 4, ADDITIONAL: 4

 

;; QUESTION SECTION:

;google.com.                    IN      A

 

;; ANSWER SECTION:

google.com.             2730    IN      A       37.61.54.158

 

;; AUTHORITY SECTION:

google.com.             10204   IN      NS      ns2.google.com.

google.com.             10204   IN      NS      ns4.google.com.

google.com.             10204   IN      NS      ns3.google.com.

google.com.             10204   IN      NS      ns1.google.com.

 

;; ADDITIONAL SECTION:

ns1.google.com.         86392   IN      A       216.239.32.10

ns2.google.com.         80495   IN      A       216.239.34.10

ns3.google.com.         85830   IN      A       216.239.36.10

ns4.google.com.         13759   IN      A       216.239.38.10

 

;; Query time: 17 msec

;; SERVER: 114.114.114.114\#53\(114.114.114.114\)

;; WHEN: Thu May 05 00:11:48 CST 2016

;; MSG SIZE  rcvd: 180

whois

whois用于查看域名所有者的信息\(client for the whois directory service\)，比如注册邮箱、手机号码、域名服务商等：



fgp@controller:~$ whois coolshell.cn

Domain Name: coolshell.cn

ROID: 20090825s10001s91994755-cn

Domain Status: ok

Registrant ID: hc401628324-cn

Registrant: 陈皓

Registrant Contact Email: haoel@hotmail.com

Sponsoring Registrar: 阿里云计算有限公司（万网）

Name Server: f1g1ns1.dnspod.net

Name Server: f1g1ns2.dnspod.net

Registration Time: 2009-08-25 00:40:26

Expiration Time: 2020-08-25 00:40:26

DNSSEC: unsigned

我们发现coolshell.cn这个域名是陈皓在万网购买注册的，注册时间是2009年，注册邮箱是haoel@hotmail.com。

