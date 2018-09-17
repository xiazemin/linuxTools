\[root@datatest ~\]\# tcprewrite -h

tcprewrite \(tcprewrite\) - Rewrite the packets in a pcap file.

Usage:  tcprewrite \[ -&lt;flag&gt; \[&lt;val&gt;\] \| --&lt;name&gt;\[{=\| }&lt;val&gt;\] \]...



   -r, --portmap=str          Rewrite TCP/UDP ports

重写 tcp/udp 端口





   -s, --seed=num             Randomize src/dst IPv4/v6 addresses w/ given seed

随机改写 IP 地址





   -N, --pnat=str             Rewrite IPv4/v6 addresses using pseudo-NAT

通过伪 NAT 重写 ip 地址





   -S, --srcipmap=str         Rewrite source IPv4/v6 addresses using pseudo-NAT







   -D, --dstipmap=str         Rewrite destination IPv4/v6 addresses using pseudo-NAT

   -e, --endpoints=str        Rewrite IP addresses to be between two endpoints



在最后 2 个点之间重写 ip 地址





   -b, --skipbroadcast        Skip rewriting broadcast/multicast IPv4/v6 addresses

不重写广播/多播 IP 地址





   -C, --fixcsum              Force recalculation of IPv4/TCP/UDP header checksums

强制重新计算 TP/RCP/UDP 校验和







   -m, --mtu=num              Override default MTU length \(1500 bytes\)

       --mtu-trunc            Truncate packets larger then specified MTU

指定 MTU







   -E, --efcs                 Remove Ethernet checksums \(FCS\) from end of frames

       --ttl=str              Modify the IPv4/v6 TTL/Hop Limit

       --tos=num              Set the IPv4 TOS/DiffServ/ECN byte

       --tclass=num           Set the IPv6 Traffic Class byte

       --flowlabel=num        Set the IPv6 Flow Label

删除以太网最后一帧的校验和（删除最后 2 个字节）









   -F, --fixlen=str           Pad or truncate packet data to match header length

       --fuzz-seed=num        Fuzz 1 in X packets.  Edit bytes, length, or emulate packet drop

       --fuzz-factor=num      Set the Fuzz 1 in X packet ratio \(default 1 in 8 packets\)

       --skipl2broadcast      Skip rewriting broadcast/multicast Layer 2 addresses

       --dlt=str              Override output DLT encapsulation

       --enet-dmac=str        Override destination ethernet MAC addresses

       --enet-smac=str        Override source ethernet MAC addresses

       --enet-subsmac=str     Substitute MAC addresses

       --enet-mac-seed=num    Randomize MAC addresses

       --enet-mac-seed-keep-bytes=num Randomize MAC addresses

       --enet-vlan=str        Specify ethernet 802.1q VLAN tag mode

       --enet-vlan-tag=num    Specify the new ethernet 802.1q VLAN tag value

       --enet-vlan-cfi=num    Specify the ethernet 802.1q VLAN CFI value

       --enet-vlan-pri=num    Specify the ethernet 802.1q VLAN priority

       --hdlc-control=num     Specify HDLC control value

       --hdlc-address=num     Specify HDLC address

       --user-dlt=num         Set output file DLT type

       --user-dlink=str       Rewrite Data-Link layer with user specified data



填充或截取包的数据以匹配包头长度







   -i, --infile=str           Input pcap file to be processed

处理输出 pcap 文件







   -o, --outfile=str          Output pcap file

输出 pcap 文件







   -c, --cachefile=str        Split traffic via tcpprep cache file

使用 tcpprep cach 文件分离流量











   -v, --verbose              Print decoded packets via tcpdump to STDOUT

tcpdump 风格打印对应信息









   -A, --decode=str           Arguments passed to tcpdump decoder

       --skip-soft-errors     Skip writing packets with soft errors

可选参数，在使用 tcpdump 风格输出信息时，同时再调用 tcpdump 中的参数











   -V, --version              Print version information

显示版本号











   -h, --less-help            Display less usage information and exit

   -H, --help                 display extended usage information and exit

   -!, --more-help            extended usage information passed thru pager

       --save-opts\[=arg\]      save the option state to a config file

       --load-opts=str        load options from a config file



Options are specified by doubled hyphens and their name or by a single

hyphen and the flag character.



Please send bug reports to:  &lt;tcpreplay-users@lists.sourceforge.net&gt;

\[root@datatest ~\]\#

