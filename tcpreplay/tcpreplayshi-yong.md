\[root@datatest ~\]\# tcpreplay -h

tcpreplay \(tcpreplay\) - Replay network traffic stored in pcap files

Usage:  tcpreplay \[ -&lt;flag&gt; \[&lt;val&gt;\] \| --&lt;name&gt;\[{=\| }&lt;val&gt;\] \]... &lt;pcap\_file\(s\)&gt;



   -q, --quiet                Quiet mode

安静模式



　　-a

精确的时间（使用高速 cpu 发包）

　　

　　-d

输出调试信息（0-5，默认 0）





   -T, --timer=str            Select packet timing mode: select, ioport, gtod, nano

       --maxsleep=num         Sleep for no more then X milliseconds between packets

   -v, --verbose              Print decoded packets via tcpdump to STDOUT

可选参数，每发送一个报文都以 tcpdump 风格打印对应信息







   -A, --decode=str           Arguments passed to tcpdump decoder

可选参数，在使用 tcpdump 风格打印输出信息时，同时再调用 tcpdump 中的参数





   -K, --preload-pcap         Preloads packets into RAM before sending



　　-c

双网卡回放报文必选参数，后跟文件名





   -C, --cachefile=str        Split traffic via a tcpprep cache file

把报文存在内部缓存中



　　-N 

获得网络接口和出口





   -2, --dualfile             Replay two files at a time from a network tap

   -i, --intf1=str            Client to server/RX/primary traffic output interface

双网卡回放报文必选参数，指定从接口



　　-I, --intf2=str Server to client/TX/secondary traffic output interface

　　　   --listnics List available network interfaces and exit



双网卡回放报文必选参数，指定主接口





　　-l, --loop=num Loop through the capture file X times

　　    --loopdelay-ms=num Delay between loops in milliseconds 

　　　　 --pktlen Override the snaplen and use the actual packet len

可选参数，指定循环次数



　　-S

制定包长度









   -L, --limit=num            Limit the number of packets to send

       --duration=num         Limit the number of seconds to send

限制发包数量





　　





   -x, --multiplier=str       Modify replay speed to a given multiple

   -p, --pps=str              Replay packets at a given packets/sec

可选参数，指定每秒发送报文的个数。制定该参数，其他速率相关的参数被忽略，

打印信息不会有速率和每秒发送报文的统计





   -M, --mbps=str             Replay packets at a given Mbps

以 Mbps（兆字节每秒）发送报文



　　-m

可选参数，指定一个倍数值，比默认发送速度快多少倍发送报文





   -t, --topspeed             Replay packets as fast as possible

以最快的速度回放报文







   -o, --oneatatime           Replay one packet at a time for each user input

       --pps-multi=num        Number of packets to send for each time interval

       --unique-ip            Modify IP addresses each loop iteration to generate unique flows

       --unique-ip-loops=str  Number of times to loop before assigning new unique ip

       --no-flow-stats        Suppress printing and tracking flow count, rates and expirations

       --flow-expiry=num      Number of inactive seconds before a flow is considered expired

一次回放一个报文





   -P, --pid                  Print the PID of tcpreplay at startup

       --stats=num            Print statistics every X seconds, or every loop if '0'

可选参数，表示在输出信息中打印 PID 信息，用于单用户和单账户模式下暂停和重

启程序





   -V, --version              Print version information

现实版本号





   -h, --less-help            Display less usage information and exit

   -H, --help                 display extended usage information and exit

   -!, --more-help            extended usage information passed thru pager

       --save-opts\[=arg\]      save the option state to a config file

       --load-opts=str        load options from a config file



Options are specified by doubled hyphens and their name or by a single

hyphen and the flag character.

tcpreplay is a tool for replaying network traffic from files saved with

tcpdump or other tools which write pcap\(3\) files.



Please send bug reports to:  &lt;tcpreplay-users@lists.sourceforge.net&gt;

\[root@datatest ~\]\#

