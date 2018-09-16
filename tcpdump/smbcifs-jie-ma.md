tcpdump 已可以对SMB/CIFS/NBT相关应用的数据包内容进行解码\(nt: 分别为'Server Message Block Common', 'Internet File System'

'在TCP/IP上实现的网络协议NETBIOS的简称'. 这几个服务通常使用UDP的137/138以及TCP的139端口\). 原来的对IPX和NetBEUI SMB数据包的

解码能力依然可以被使用\(nt: NetBEUI为NETBIOS的增强版本\).





tcpdump默认只按照最简约模式对相应数据包进行解码, 如果我们想要详尽的解码信息可以使用其-v 启动选现. 要注意的是, -v 会产生非常详细的信息,

比如对单一的一个SMB数据包, 将产生一屏幕或更多的信息, 所以此选项, 确有需要才使用.



关于SMB数据包格式的信息, 以及每个域的含义可以参看www.cifs.org 或者samba.org 镜像站点的pub/samba/specs/ 目录. linux 上的SMB 补丁

\(nt \| rt: patch\)由 Andrew Tridgell \(tridge@samba.org\)提供.





NFS 请求和回应



tcpdump对Sun NFS\(网络文件系统\)请求和回应的UDP数据包有如下格式的打印输出:

src.xid &gt; dst.nfs: len op args

src.nfs &gt; dst.xid: reply stat len op results



以下是一组具体的输出数据

sushi.6709 &gt; wrl.nfs: 112 readlink fh 21,24/10.73165

wrl.nfs &gt; sushi.6709: reply ok 40 readlink "../var"

sushi.201b &gt; wrl.nfs:

144 lookup fh 9,74/4096.6878 "xcolors"

wrl.nfs &gt; sushi.201b:

reply ok 128 lookup fh 9,74/4134.3150



第一行输出表明: 主机sushi向主机wrl发送了一个'交换请求'\(nt: transaction\), 此请求的id为6709\(注意, 主机名字后是交换

请求id号, 而不是源端口号\). 此请求数据为112字节, 其中不包括UDP和IP头部的长度. 操作类型为readlink\(nt: 即此操作为读符号链接操作\),

操作参数为fh 21,24/10.73165\(nt: 可按实际运行环境, 解析如下, fd 表示描述的为文件句柄, 21,24 表示此句柄所对应设

备的主/从设备号对, 10表示此句柄所对应的i节点编号\(nt:每个文件都会在操作系统中对应一个i节点, 限于unix类系统中\),

73165是一个编号\(nt: 可理解为标识此请求的一个随机数, 具体含义需补充\)\).



第二行中, wrl 做了'ok'的回应, 并且在results 字段中返回了sushi想要读的符号连接的真实目录\(nt: 即sushi要求读的符号连接其实是一个目录\).



第三行表明: sushi 再次请求 wrl 在'fh 9,74/4096.6878'所描述的目录中查找'xcolors'文件. 需要注意的是, 每行所显示的数据含义依赖于其中op字段的

类型\(nt: 不同op 所对应args 含义不相同\), 其格式遵循NFS 协议, 追求简洁明了.



 



如果tcpdump 的-v选项\(详细打印选项\) 被设置, 附加的信息将被显示. 比如:

sushi.1372a &gt; wrl.nfs:

148 read fh 21,11/12.195 8192 bytes @ 24576

wrl.nfs &gt; sushi.1372a:

reply ok 1472 read REG 100664 ids 417/0 sz 29388



\(-v 选项一般还会打印出IP头部的TTL, ID， length, 以及fragmentation 域, 但在此例中, 都略过了\(nt: 可理解为,简洁起见, 做了删减\)\)

在第一行, sushi 请求wrl 从文件 21,11/12.195\(nt: 格式在上面有描述\)中, 自偏移24576字节处开始, 读取8192字节数据.

Wrl 回应读取成功; 由于第二行只是回应请求的开头片段, 所以只包含1472字节\(其他的数据将在接着的reply片段中到来, 但这些数据包不会再有NFS

头, 甚至UDP头信息也为空\(nt: 源和目的应该要有\), 这将导致这些片段不能满足过滤条件, 从而没有被打印\). -v 选项除了显示文件数据信息, 还会显示

附加显示文件属性信息: file type\(文件类型, ''REG'' 表示普通文件\), file mode\(文件存取模式, 8进制表示的\), uid 和gid\(nt: 文件属主和

组属主\), file size \(文件大小\).



如果-v 标志被多次重复给出\(nt: 如-vv\)， tcpdump会显示更加详细的信息.



必须要注意的是, NFS 请求包中数据比较多, 如果tcpdump 的snaplen\(nt: 抓取长度\) 取太短将不能显示其详细信息. 可使用

'-s 192'来增加snaplen, 这可用以监测NFS应用的网络负载\(nt: traffic\).



NFS 的回应包并不严格的紧随之前相应的请求包\(nt: RPC operation\). 从而, tcpdump 会跟踪最近收到的一系列请求包, 再通过其

交换序号\(nt: transaction ID\)与相应请求包相匹配. 这可能产生一个问题， 如果回应包来得太迟, 超出tcpdump 对相应请求包的跟踪范围,

该回应包将不能被分析.





AFS 请求和回应

AFS\(nt: Andrew 文件系统, Transarc , 未知, 需补充\)请求和回应有如下的答应



src.sport &gt; dst.dport: rx packet-type

src.sport &gt; dst.dport: rx packet-type service call call-name args

src.sport &gt; dst.dport: rx packet-type service reply call-name args



elvis.7001 &gt; pike.afsfs:

rx data fs call rename old fid 536876964/1/1 ".newsrc.new"

new fid 536876964/1/1 ".newsrc"

pike.afsfs &gt; elvis.7001: rx data fs reply rename



在第一行, 主机elvis 向pike 发送了一个RX数据包.

这是一个对于文件服务的请求数据包\(nt: RX data packet, 发送数据包 , 可理解为发送包过去, 从而请求对方的服务\), 这也是一个RPC

调用的开始\(nt: RPC, remote procedure call\). 此RPC 请求pike 执行rename\(nt: 重命名\) 操作, 并指定了相关的参数:

原目录描述符为536876964/1/1, 原文件名为 '.newsrc.new', 新目录描述符为536876964/1/1, 新文件名为 '.newsrc'.

主机pike 对此rename操作的RPC请求作了回应\(回应表示rename操作成功, 因为回应的是包含数据内容的包而不是异常包\).



一般来说, 所有的'AFS RPC'请求被显示时, 会被冠以一个名字\(nt: 即decode, 解码\), 这个名字往往就是RPC请求的操作名.

并且, 这些RPC请求的部分参数在显示时, 也会被冠以一个名字\(nt \| rt: 即decode, 解码, 一般来说也是取名也很直接, 比如,

一个interesting 参数, 显示的时候就会直接是'interesting', 含义拗口, 需再翻\).



这种显示格式的设计初衷为'一看就懂', 但对于不熟悉AFS 和 RX 工作原理的人可能不是很

有用\(nt: 还是不用管, 书面吓吓你的, 往下看就行\).



如果 -v\(详细\)标志被重复给出\(nt: 如-vv\), tcpdump 会打印出确认包\(nt: 可理解为, 与应答包有区别的包\)以及附加头部信息

\(nt: 可理解为, 所有包, 而不仅仅是确认包的附加头部信息\), 比如, RX call ID\(请求包中'请求调用'的ID\),

call number\('请求调用'的编号\), sequence number\(nt: 包顺序号\),

serial number\(nt \| rt: 可理解为与包中数据相关的另一个顺信号, 具体含义需补充\), 请求包的标识. \(nt: 接下来一段为重复描述,

所以略去了\), 此外确认包中的MTU协商信息也会被打印出来\(nt: 确认包为相对于请求包的确认包, Maximum Transmission Unit, 最大传输单元\).



如果 -v 选项被重复了三次\(nt: 如-vvv\), 那么AFS应用类型数据包的'安全索引'\('security index'\)以及'服务索引'\('service id'\)将会

被打印.



对于表示异常的数据包\(nt: abort packet, 可理解为, 此包就是用来通知接受者某种异常已发生\), tcpdump 会打印出错误号\(error codes\).

但对于Ubik beacon packets\(nt: Ubik 灯塔指示包, Ubik可理解为特殊的通信协议, beacon packets, 灯塔数据包, 可理解为指明通信中

关键信息的一些数据包\), 错误号不会被打印, 因为对于Ubik 协议, 异常数据包不是表示错误, 相反却是表示一种肯定应答\(nt: 即, yes vote\).



AFS 请求数据量大, 参数也多, 所以要求tcpdump的 snaplen 比较大, 一般可通过启动tcpdump时设置选项'-s 256' 来增大snaplen, 以

监测AFS 应用通信负载.



AFS 回应包并不显示标识RPC 属于何种远程调用. 从而, tcpdump 会跟踪最近一段时间内的请求包, 并通过call number\(调用编号\), service ID

\(服务索引\) 来匹配收到的回应包. 如果回应包不是针对最近一段时间内的请求包, tcpdump将无法解析该包.





KIP AppleTalk协议

\(nt \| rt: DDP in UDP可理解为, DDP, The AppleTalk Data Delivery Protocol,

相当于支持KIP AppleTalk协议栈的网络层协议, 而DDP 本身又是通过UDP来传输的,

即在UDP 上实现的用于其他网络的网络层，KIP AppleTalk是苹果公司开发的整套网络协议栈\).



AppleTalk DDP 数据包被封装在UDP数据包中, 其解封装\(nt: 相当于解码\)和相应信息的转储也遵循DDP 包规则.

\(nt:encapsulate, 封装, 相当于编码, de-encapsulate, 解封装, 相当于解码, dump, 转储, 通常就是指对其信息进行打印\).



/etc/atalk.names 文件中包含了AppleTalk 网络和节点的数字标识到名称的对应关系. 其文件格式通常如下所示:

number name



1.254 ether

16.1 icsd-net

1.254.110 ace



头两行表示有两个AppleTalk 网络. 第三行给出了特定网络上的主机\(一个主机会用3个字节来标识,

而一个网络的标识通常只有两个字节, 这也是两者标识的主要区别\)\(nt: 1.254.110 可理解为ether网络上的ace主机\).

标识与其对应的名字之间必须要用空白分开. 除了以上内容, /etc/atalk.names中还包含空行以及注释行\(以'\#'开始的行\).





AppleTalk 完整网络地址将以如下格式显示:

net.host.port



以下为一段具体显示:

144.1.209.2 &gt; icsd-net.112.220

office.2 &gt; icsd-net.112.220

jssmag.149.235 &gt; icsd-net.2



\(如果/etc/atalk.names 文件不存在, 或者没有相应AppleTalk 主机/网络的条目, 数据包的网络地址将以数字形式显示\).



在第一行中, 网络144.1上的节点209通过2端口,向网络icsd-net上监听在220端口的112节点发送了一个NBP应用数据包

\(nt \| rt: NBP, name binding protocol, 名称绑定协议, 从数据来看, NBP服务器会在端口2提供此服务.

'DDP port 2' 可理解为'DDP 对应传输层的端口2', DDP本身没有端口的概念, 这点未确定, 需补充\).



第二行与第一行类似, 只是源的全部地址可用'office'进行标识.

第三行表示: jssmag网络上的149节点通过235向icsd-net网络上的所有节点的2端口\(NBP端口\)发送了数据包.\(需要注意的是,

在AppleTalk 网络中如果地址中没有节点, 则表示广播地址, 从而节点标识和网络标识最好在/etc/atalk.names有所区别.

nt: 否则一个标识x.port 无法确定x是指一个网络上所有主机的port口还是指定主机x的port口\).



tcpdump 可解析NBP \(名称绑定协议\) and ATP \(AppleTalk传输协议\)数据包, 对于其他应用层的协议, 只会打印出相应协议名字\(

如果此协议没有注册一个通用名字, 只会打印其协议号\)以及数据包的大小.





NBP 数据包会按照如下格式显示:

icsd-net.112.220 &gt; jssmag.2: nbp-lkup 190: "=:LaserWriter@\*"

jssmag.209.2 &gt; icsd-net.112.220: nbp-reply 190: "RM1140:LaserWriter@\*" 250

techpit.2 &gt; icsd-net.112.220: nbp-reply 190: "techpit:LaserWriter@\*" 186



第一行表示: 网络icsd-net 中的节点112 通过220端口向网络jssmag 中所有节点的端口2发送了对'LaserWriter'的名称查询请求\(nt:

此处名称可理解为一个资源的名称, 比如打印机\). 此查询请求的序列号为190.



第二行表示: 网络jssmag 中的节点209 通过2端口向icsd-net.112节点的端口220进行了回应: 我有'LaserWriter'资源, 其资源名称

为'RM1140', 并且在端口250上提供改资源的服务. 此回应的序列号为190, 对应之前查询的序列号.



第三行也是对第一行请求的回应: 节点techpit 通过2端口向icsd-net.112节点的端口220进行了回应:我有'LaserWriter'资源, 其资源名称

为'techpit', 并且在端口186上提供改资源的服务. 此回应的序列号为190, 对应之前查询的序列号.





ATP 数据包的显示格式如下:

jssmag.209.165 &gt; helios.132: atp-req 12266&lt;0-7&gt; 0xae030001

helios.132 &gt; jssmag.209.165: atp-resp 12266:0 \(512\) 0xae040000

helios.132 &gt; jssmag.209.165: atp-resp 12266:1 \(512\) 0xae040000

helios.132 &gt; jssmag.209.165: atp-resp 12266:2 \(512\) 0xae040000

helios.132 &gt; jssmag.209.165: atp-resp 12266:3 \(512\) 0xae040000

helios.132 &gt; jssmag.209.165: atp-resp 12266:5 \(512\) 0xae040000

helios.132 &gt; jssmag.209.165: atp-resp 12266:6 \(512\) 0xae040000

helios.132 &gt; jssmag.209.165: atp-resp\*12266:7 \(512\) 0xae040000

jssmag.209.165 &gt; helios.132: atp-req 12266&lt;3,5&gt; 0xae030001

helios.132 &gt; jssmag.209.165: atp-resp 12266:3 \(512\) 0xae040000

helios.132 &gt; jssmag.209.165: atp-resp 12266:5 \(512\) 0xae040000

jssmag.209.165 &gt; helios.132: atp-rel 12266&lt;0-7&gt; 0xae030001

jssmag.209.133 &gt; helios.132: atp-req\* 12267&lt;0-7&gt; 0xae030002



第一行表示节点 Jssmag.209 向节点helios 发送了一个会话编号为12266的请求包, 请求helios

回应8个数据包\(这8个数据包的顺序号为0-7\(nt: 顺序号与会话编号不同, 后者为一次完整传输的编号,

前者为该传输中每个数据包的编号. transaction, 会话, 通常也被叫做传输\)\). 行尾的16进制数字表示

该请求包中'userdata'域的值\(nt: 从下文来看, 这并没有把所有用户数据都打印出来 \).



Helios 回应了8个512字节的数据包. 跟在会话编号\(nt: 12266\)后的数字表示该数据包在该会话中的顺序号.

括号中的数字表示该数据包中数据的大小, 这不包括atp 的头部. 在顺序号为7数据包\(第8行\)外带了一个'\*'号,

表示该数据包的EOM 标志被设置了.\(nt: EOM, End Of Media, 可理解为, 表示一次会话的数据回应完毕\).



接下来的第9行表示, Jssmag.209 又向helios 提出了请求: 顺序号为3以及5的数据包请重新传送. Helios 收到这个

请求后重新发送了这个两个数据包, jssmag.209 再次收到这两个数据包之后, 主动结束\(release\)了此会话.



在最后一行, jssmag.209 向helios 发送了开始下一次会话的请求包. 请求包中的'\*'表示该包的XO 标志没有被设置.

\(nt: XO, exactly once, 可理解为在该会话中, 数据包在接受方只被精确地处理一次, 就算对方重复传送了该数据包,

接收方也只会处理一次, 这需要用到特别设计的数据包接收和处理机制\).

