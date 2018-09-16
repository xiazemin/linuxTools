UDP 数据包的显示格式，可通过rwho这个具体应用所产生的数据包来说明:

actinide.who &gt; broadcast.who: udp 84



其含义为:actinide主机上的端口who向broadcast主机上的端口who发送了一个udp数据包\(nt: actinide和broadcast都是指Internet地址\).

这个数据包承载的用户数据为84个字节.



一些UDP服务可从数据包的源或目的端口来识别，也可从所显示的更高层协议信息来识别. 比如, Domain Name service requests\(DNS 请求,

在RFC-1034/1035中\), 和Sun RPC calls to NFS\(对NFS服务器所发起的远程调用\(nt: 即Sun RPC\)，在RFC-1050中有对远程调用的描述\).



UDP 名称服务请求



\(注意:以下的描述假设你对Domain Service protoco\(nt:在RFC-103中有所描述\), 否则你会发现以下描述就是天书\(nt:希腊文天书,

不必理会, 吓吓你的, 接着看就行\)\)



名称服务请求有如下的格式:

src &gt; dst: id op? flags qtype qclass name \(len\)

\(nt: 从下文来看, 格式应该是src &gt; dst: id op flags qtype qclass? name \(len\)\)

比如有一个实际显示为:

h2opolo.1538 &gt; helios.domain: 3+ A? ucbvax.berkeley.edu. \(37\)



主机h2opolo 向helios 上运行的名称服务器查询ucbvax.berkeley.edu 的地址记录\(nt: qtype等于A\). 此查询本身的id号为'3'. 符号

'+'意味着递归查询标志被设置\(nt: dns服务器可向更高层dns服务器查询本服务器不包含的地址记录\). 这个最终通过IP包发送的查询请求

数据长度为37字节, 其中不包括UDP和IP协议的头数据. 因为此查询操作为默认值\(nt \| rt: normal one的理解\), op字段被省略.

如果op字段没被省略, 会被显示在'3' 和'+'之间. 同样, qclass也是默认值, C\_IN, 从而也没被显示, 如果没被忽略, 她会被显示在'A'之后.



异常检查会在方括中显示出附加的域:　如果一个查询同时包含一个回应\(nt: 可理解为, 对之前其他一个请求的回应\), 并且此回应包含权威或附加记录段,　

ancount, nscout, arcount\(nt: 具体字段含义需补充\) 将被显示为'\[na\]', '\[nn\]', '\[nau\]', 其中n代表合适的计数. 如果包中以下

回应位\(比如AA位, RA位, rcode位\), 或者字节2或3中任何一个'必须为0'的位被置位\(nt: 设置为1\), '\[b2&3\]=x' 将被显示, 其中x表示

头部字节2与字节3进行与操作后的值.



UDP 名称服务应答



对名称服务应答的数据包，tcpdump会有如下的显示格式

src &gt; dst: id op rcode flags a/n/au type class data \(len\)

比如具体显示如下:

helios.domain &gt; h2opolo.1538: 3 3/3/7 A 128.32.137.3 \(273\)

helios.domain &gt; h2opolo.1537: 2 NXDomain\* 0/1/0 \(97\)



第一行表示: helios 对h2opolo 所发送的3号查询请求回应了3条回答记录\(nt \| rt: answer records\), 3条名称服务器记录,

以及7条附加的记录. 第一个回答记录\(nt: 3个回答记录中的第一个\)类型为A\(nt: 表示地址\), 其数据为internet地址128.32.137.3.

此回应UDP数据包, 包含273字节的数据\(不包含UPD和IP的头部数据\). op字段和rcode字段被忽略\(nt: op的实际值为Query, rcode, 即

response code的实际值为NoError\), 同样被忽略的字段还有class 字段\(nt \| rt: 其值为C\_IN, 这也是A类型记录默认取值\)



第二行表示: helios 对h2opolo 所发送的2号查询请求做了回应. 回应中, rcode编码为NXDomain\(nt: 表示不存在的域\)\), 没有回答记录,

但包含一个名称服务器记录, 不包含权威服务器记录\(nt \| ck: 从上文来看, 此处的authority records 就是上文中对应的additional

records\). '\*'表示权威服务器回答标志被设置\(nt: 从而additional records就表示的是authority records\).

由于没有回答记录, type, class, data字段都被忽略.



flag字段还有可能出现其他一些字符, 比如'-'\(nt: 表示可递归地查询, 即RA 标志没有被设置\), '\|'\(nt: 表示被截断的消息, 即TC 标志

被置位\). 如果应答\(nt \| ct: 可理解为, 包含名称服务应答的UDP数据包, tcpdump知道这类数据包该怎样解析其数据\)的'question'段一个条

目\(entry\)都不包含\(nt: 每个条目的含义, 需补充\),'\[nq\]' 会被打印出来.



要注意的是:名称服务器的请求和应答数据量比较大, 而默认的68字节的抓取长度\(nt: snaplen, 可理解为tcpdump的一个设置选项\)可能不足以抓取

数据包的全部内容. 如果你真的需要仔细查看名称服务器的负载, 可以通过tcpdump 的-s 选项来扩大snaplen值.

