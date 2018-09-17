\[root@datatest ~\]\# tcpbridge -h

tcpbridge \(tcpbridge\) - Bridge network traffic across two interfaces

Usage:  tcpbridge \[ -&lt;flag&gt; \[&lt;val&gt;\] \| --&lt;name&gt;\[{=\| }&lt;val&gt;\] \]...



   -r, --portmap=str          Rewrite TCP/UDP ports

   -s, --seed=num             Randomize src/dst IPv4/v6 addresses w/ given seed

   -N, --pnat=str             Rewrite IPv4/v6 addresses using pseudo-NAT

   -S, --srcipmap=str         Rewrite source IPv4/v6 addresses using pseudo-NAT

   -D, --dstipmap=str         Rewrite destination IPv4/v6 addresses using pseudo-NAT

   -b, --skipbroadcast        Skip rewriting broadcast/multicast IPv4/v6 addresses

   -C, --fixcsum              Force recalculation of IPv4/TCP/UDP header checksums

   -m, --mtu=num              Override default MTU length \(1500 bytes\)

       --mtu-trunc            Truncate packets larger then specified MTU

   -E, --efcs                 Remove Ethernet checksums \(FCS\) from end of frames

       --ttl=str              Modify the IPv4/v6 TTL/Hop Limit

       --tos=num              Set the IPv4 TOS/DiffServ/ECN byte

       --tclass=num           Set the IPv6 Traffic Class byte

       --flowlabel=num        Set the IPv6 Flow Label

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

   -i, --intf1=str            Primary interface \(listen in uni-directional mode\)

   -I, --intf2=str            Secondary interface \(send in uni-directional mode\)

   -u, --unidir               Send and receive in only one direction

       --listnics             List available network interfaces and exit

   -L, --limit=num            Limit the number of packets to send

   -M, --mac=str              MAC addresses of local NIC's

   -x, --include=str          Include only packets matching rule

   -X, --exclude=str          Exclude any packet matching this rule

   -P, --pid                  Print the PID of tcpbridge at startup

   -v, --verbose              Print decoded packets via tcpdump to STDOUT

   -A, --decode=str           Arguments passed to tcpdump decoder

   -V, --version              Print version information

   -h, --less-help            Display less usage information and exit

   -H, --help                 display extended usage information and exit

   -!, --more-help            extended usage information passed thru pager

       --save-opts\[=arg\]      save the option state to a config file

       --load-opts=str        load options from a config file



Options are specified by doubled hyphens and their name or by a single

hyphen and the flag character.

tcpbridge is a tool for selectively briding network traffic across two

interfaces and optionally modifying the packets in between



Please send bug reports to:  &lt;tcpreplay-users@lists.sourceforge.net&gt;

\[root@datatest ~\]\#

