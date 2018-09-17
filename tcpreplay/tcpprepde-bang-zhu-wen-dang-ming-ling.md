\[root@datatest ~\]\# tcpprep -h

tcpprep \(tcpprep\) - Create a tcpreplay cache cache file from a pcap file.

Usage:  tcpprep \[ -&lt;flag&gt; \[&lt;val&gt;\] \| --&lt;name&gt;\[{=\| }&lt;val&gt;\] \]...



   -a, --auto=str             Auto-split mode

一般情况下都需要的参数，表示按模式自动分离通信流量生成 cach 文件 



   -c, --cidr=str             CIDR-split mode

可选参数， 表示分离流量时采用 CIDR（无类别域间路由选择，是一个在 internet

上创建附加地址的方法，这些地址提供给服务提供商，再由服务提供商分配给客户。

CIDR 将路由集中起来，使一个 IP 地址代表主要骨干提供商提供的几千个 IP 地址，

从而减轻 internet 路由器的负担） 



   -r, --regex=str            Regex-split mode

可选参数，表示使用 regex 模式分离通信流量，有点类似于 CIDR 模式，但是它匹

配的是服务器源 IP 



   -p, --port                 Port-split mode

可选参数，基于目的端口来分离通信流量，它区分的依据是任何 0-1023 端口都是

服务器的端发出的报文，其他的端口都是客户端发出的报文 



   -e, --mac=str              Source MAC split mode

表示基于服务器源 MAC 地址分离通信流量 



       --reverse              Matches to be client instead of server



   -C, --comment=str          Embedded cache file comment

可选参数，表示在 cach 文件中嵌入注释内容，用于注释说明 cach 文件的内容。使用位置不要放在最后，生成 cach 后可以使用-P 查看写入的内容。





       --no-arg-comment       Do not embed any cache file comment

   -x, --include=str          Include only packets matching rule

重要可选参数，表示按照参数定义的需求来定义发送报文



   -X, --exclude=str          Exclude any packet matching this rule

可选参数，是-x 的取反



   -o, --cachefile=str        Output cache file



   生成 cach 文件必带参数，后跟 cach 后缀的文件名，表示这个输出的 cach 文

件以这个文件名命名

-i, --pcap=str             Input pcap file to process

  生成 cach 文件必带参数，后跟 pcap 文件名，表示这个 pcap 文件需要处理

-P, --print-comment=str    Print embedded comment in the specified cache file

可选参数，表示查看 cach 文件的内容



   -I, --print-info=str       Print basic info from the specified cache file

表示打印 cach 文件的基本信息





   -S, --print-stats=str      Print statistical information about the specified cache file

表示打印 cach 文件的统计信



   -s, --services=str         Load services file for server ports

  表示从服务器端口下载服务文件

-N, --nonip                Send non-IP traffic out server interface

表示从服务器端口发送无 IP 流量





   -R, --ratio=str            Ratio of client to server packets



　　可选参数，一个比例值。服务器端发起的连接数和客户端发起的连接数的比例，值

大于 2 就视为服务器端

-m, --minmask=num          Minimum network mask length in auto mode

可选参数，在选用 router 模式时使用，表示最小掩码，掩码默认是 30





   -M, --maxmask=num          Maximum network mask length in auto mode

可选参数，在选用 router 模式时使用，表示最大掩码，默认是 8





   -v, --verbose              Print decoded packets via tcpdump to STDOUT

可选参数，显示 tcpprep 生成 cach 文件的处理过程。信息的随时大于





   -A, --decode=str           Arguments passed to tcpdump decoder

可选参数，在实验 tcpdump 风格打印输出信息时，同时再调用 tcpdump 中的参数





   -V, --version              Print version information

   -h, --less-help            Display less usage information and exit

   -H, --help                 display extended usage information and exit

   -!, --more-help            extended usage information passed thru pager

       --save-opts\[=arg\]      save the option state to a config file

       --load-opts=str        load options from a config file

Options are specified by doubled hyphens and their name or by a single

hyphen and the flag character.

tcpprep is a 'pcap\(3\)' file pre-processor which creates a cache file which

provides "rules" for 'tcprewrite\(1\)' and 'tcpreplay\(1\)' on how to process

and send packets.



Please send bug reports to:  &lt;tcpreplay-users@lists.sourceforge.net&gt;

\[root@datatest ~\]\#

