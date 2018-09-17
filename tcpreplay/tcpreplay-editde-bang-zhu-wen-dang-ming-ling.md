\[root@datatest ~\]\# tcpreplay-edit -h

tcpreplay \(tcpreplay\) - Replay network traffic stored in pcap files

Usage:  tcpreplay-edit \[ -&lt;flag&gt; \[&lt;val&gt;\] \| --&lt;name&gt;\[{=\| }&lt;val&gt;\] \]... &lt;pcap\_file\(s\)&gt;

-r, --portmap=str          Rewrite TCP/UDP ports

-s, --seed=num             Randomize src/dst IPv4/v6 addresses w/ given seed

-N, --pnat=str             Rewrite IPv4/v6 addresses using pseudo-NAT

-S, --srcipmap=str         Rewrite source IPv4/v6 addresses using pseudo-NAT

-D, --dstipmap=str         Rewrite destination IPv4/v6 addresses using pseudo-NAT

-e, --endpoints=str        Rewrite IP addresses to be between two endpoints

-b, --skipbroadcast        Skip rewriting broadcast/multicast IPv4/v6 addresses

-C, --fixcsum              Force recalculation of IPv4/TCP/UDP header checksums

-m, --mtu=num              Override default MTU length \(1500 bytes\)

```
   --mtu-trunc            Truncate packets larger then specified MTU
```

-E, --efcs                 Remove Ethernet checksums \(FCS\) from end of frames

```
   --ttl=str              Modify the IPv4/v6 TTL/Hop Limit

   --tos=num              Set the IPv4 TOS/DiffServ/ECN byte

   --tclass=num           Set the IPv6 Traffic Class byte

   --flowlabel=num        Set the IPv6 Flow Label
```

-F, --fixlen=str           Pad or truncate packet data to match header length

```
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
```

-q, --quiet                Quiet mode

-T, --timer=str            Select packet timing mode: select, ioport, gtod, nano

```
   --maxsleep=num         Sleep for no more then X milliseconds between packets
```

-v, --verbose              Print decoded packets via tcpdump to STDOUT

-A, --decode=str           Arguments passed to tcpdump decoder

-K, --preload-pcap         Preloads packets into RAM before sending

-c, --cachefile=str        Split traffic via a tcpprep cache file

-2, --dualfile             Replay two files at a time from a network tap

-i, --intf1=str            Client to server/RX/primary traffic output interface

-I, --intf2=str            Server to client/TX/secondary traffic output interface

```
   --listnics             List available network interfaces and exit
```

-l, --loop=num             Loop through the capture file X times

```
   --loopdelay-ms=num     Delay between loops in milliseconds

   --pktlen               Override the snaplen and use the actual packet len
```

-L, --limit=num            Limit the number of packets to send

```
   --duration=num         Limit the number of seconds to send
```

-x, --multiplier=str       Modify replay speed to a given multiple

-p, --pps=str              Replay packets at a given packets/sec

-M, --mbps=str             Replay packets at a given Mbps

-t, --topspeed             Replay packets as fast as possible

-o, --oneatatime           Replay one packet at a time for each user input

```
   --pps-multi=num        Number of packets to send for each time interval

   --unique-ip            Modify IP addresses each loop iteration to generate unique flows

   --unique-ip-loops=str  Number of times to loop before assigning new unique ip
```

-!, --no-flow-stats        Suppress printing and tracking flow count, rates and expirations

-", --flow-expiry=num      Number of inactive seconds before a flow is considered expired

-P, --pid                  Print the PID of tcpreplay at startup

-\#, --stats=num            Print statistics every X seconds, or every loop if '0'

-V, --version              Print version information

-h, --less-help            Display less usage information and exit

-H, --help                 display extended usage information and exit

-!, --more-help            extended usage information passed thru pager

-$, --save-opts\[=arg\]      save the option state to a config file

-%, --load-opts=str        load options from a config file

Options are specified by doubled hyphens and their name or by a single

hyphen and the flag character.

tcpreplay is a tool for replaying network traffic from files saved with

tcpdump or other tools which write pcap\(3\) files.

Please send bug reports to:  &lt;tcpreplay-users@lists.sourceforge.net&gt;

\[root@datatest ~\]\#

　tcpreplay-edit 实时修改包数据并回放，它是将 tcprewrite 和 tcpreplay 用一条命令实现。其好处是修改包数据不会新生成 pcap 文件。如果是需要不断的改写一个包文件并回放建议使用 tcpreplay-edit，如果是需要一次改写一个包文件并多次回放建议使用 tcprewrite 和 tcpreplay 的结合，这样具有更好的回放速率。 





　　tcpreplay-edit小例子



　　编写脚本，不断改写包文件的 IP 地址并回放： 



　　比如，你自己写个test.sh脚本，然后赋予chmod 777 test.sh。执行就是



for i in {1..255}

do

tcpreplay-edit --endpoints=1.1.2.$i:1.1.1.2 -t -i eth2 -I eth1 -c edit.cach edit.pcap

done

