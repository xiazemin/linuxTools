rule存放在特定表的特定chain上，每条rule包含下面两部分信息：



Matching

Matching就是如何匹配一个数据包，匹配条件很多，比如协议类型、源/目的IP、源/目的端口、in/out接口、包头里面的数据以及连接状态等，这些条件可以任意组合从而实现复杂情况下的匹配。详情请参考Iptables matches



Targets

Targets就是找到匹配的数据包之后怎么办，常见的有下面几种：



DROP：直接将数据包丢弃，不再进行后续的处理



RETURN： 跳出当前chain，该chain里后续的rule不再执行



QUEUE： 将数据包放入用户空间的队列，供用户空间的程序处理



ACCEPT： 同意数据包通过，继续执行后续的rule



跳转到其它用户自定义的chain继续执行



当然iptables包含的targets很多很多，但并不是每个表都支持所有的targets，

rule所支持的target由它所在的表和chain以及所开启的扩展功能来决定，具体每个表支持的targets请参考Iptables targets and jumps。



用户自定义Chains

除了iptables预定义的5个chain之外，用户还可以在表中定义自己的chain，用户自定义的chain中的rule和预定义chain里的rule没有区别，不过由于自定义的chain没有和netfilter里面的钩子进行绑定，所以它不会自动触发，只能从其它chain的rule中跳转过来。



连接追踪（Connection Tracking）

Connection Tracking发生在NF\_IP\_PRE\_ROUTING和NF\_IP\_LOCAL\_OUT这两个地方，一旦开启该功能，Connection Tracking模块将会追踪每个数据包（被raw表中的rule标记过的除外），维护所有的连接状态，然后这些状态可以供其它表中的rule引用，用户空间的程序也可以通过/proc/net/ip\_conntrack来获取连接信息。下面是所有的连接状态：



这里的连接不仅仅是TCP的连接，两台设备的进程用UDP和ICMP（ping）通信也会被认为是一个连接



NEW: 当检测到一个不和任何现有连接关联的新包时，如果该包是一个合法的建立连接的数据包（比如TCP的sync包或者任意的UDP包），一个新的连接将会被保存，并且标记为状态NEW。



ESTABLISHED: 对于状态是NEW的连接，当检测到一个相反方向的包时，连接的状态将会由NEW变成ESTABLISHED，表示连接成功建立。对于TCP连接，意味着收到了一个SYN/ACK包， 对于UDP和ICMP，任何反方向的包都可以。



RELATED: 数据包不属于任何现有的连接，但它跟现有的状态为ESTABLISHED的连接有关系，对于这种数据包，将会创建一个新的连接，且状态被标记为RELATED。这种连接一般是辅助连接，比如FTP的数据传输连接（FTP有两个连接，另一个是控制连接），或者和某些连接有关的ICMP报文。



INVALID: 数据包不和任何现有连接关联，并且不是一个合法的建立连接的数据包，对于这种连接，将会被标记为INVALID，一般这种都是垃圾数据包，比如收到一个TCP的RST包，但实际上没有任何相关的TCP连接，或者别的地方误发过来的ICMP包。



UNTRACKED: 被raw表里面的rule标记为不需要tracking的数据包，这种连接将会标记成UNTRACKED。

