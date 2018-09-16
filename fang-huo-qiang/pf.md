PF：OpenBSD数据包过滤

包过滤 \(以下简称 PF\) 是OpenBSD 系统上进行TCP/IP流量过滤和网络地址转换的软件系统。 PF 同样也能提供TCP/IP流量的整形和控制，并且提供带宽控制和数据包优先集控制。PF自openbsd 3.0以后作为内核的默认安装配置。以前版本的openbsd发行版使用一个不同的防火墙/NAT软件包，现在已经不再被支持。

PF 最早是由 Daniel Hartmeier 开发的，现在的开发和维护由Daniel 和openbsd小组的其他成员负责。

激活

要激活pf并且使它在启动时调用配置文件，编辑/etc/rc.conf文件，修改配置pf的一行：

pf=YES

重启系统让配置生效。

你也可以通过pfctl程序启动和停止pf

\# pfctl -e

\# pfctl -d

注意这仅仅是启动和关闭PF，实际它不会载入规则集，规则集要么在系统启动时载入，要么在PF启动后通过命令单独载入。

配置

系统引导到在rc脚本文件运行PF时PF从/etc/pf.conf文件载入配置规则。注意当/etc/pf.conf文件是默认配置文件，在系统调用 rc脚本文件时，它仅仅是作为文本文件由pfctl（8）装入并解释和插入pf（4）的。对于一些应用来说，其他的规则集可以在系统引导后由其他文件载 入。对于一些设计的非常好的unix程序，PF提供了足够的灵活性。

pf.conf 文件有7个部分:

\* 宏:用户定义的变量，包括IP地址，接口名称等等

\* 表: 一种用来保存IP地址列表的结构

\* 选项: 控制PF如何工作的变量

\* 整形: 重新处理数据包，进行正常化和碎片整理

\* 排队: 提供带宽控制和数据包优先级控制.

\* 转换: 控制网络地址转换和数据包重定向.

\* 过滤规则: 在数据包通过接口时允许进行选择性的过滤和阻止

除去宏和表，其他的段在配置文件中也应该按照这个顺序出现，尽管对于一些特定的应用并不是所有的段都是必须的。

空行会被忽略，以＃开头的行被认为是注释.

控制

引导之后，PF可以通过pfctl（8）程序进行操作，以下是一些例子：

\# pfctl -f /etc/pf.conf 载入 pf.conf 文件

\# pfctl -nf /etc/pf.conf 解析文件，但不载入

\# pfctl -Nf /etc/pf.conf 只载入文件中的NAT规则

\# pfctl -Rf /etc/pf.conf 只载入文件中的过滤规则

\# pfctl -sn 显示当前的NAT规则

\# pfctl -sr 显示当前的过滤规则

\# pfctl -ss 显示当前的状态表

\# pfctl -si 显示过滤状态和计数

\# pfctl -sa 显示任何可显示的

完整的命令列表，参阅pfctl的man手册页。

列表

一个列表允许一个规则集中指定多个相似的标准。例如，多个协议，端口号，地址等等。因此，不需要为每一个需要阻止的IP地址编写一个过滤规则，一条规则可以在列表中指定多个IP地址。列表的定义是将要指定的条目放在{ }大括号中。

当pfctl（8）在载入规则集碰到列表时，它产生多个规则，每条规则对于列表中的一个条目。例如：

block out on fxp0 from { 192.168.0.1, 10.5.32.6 } to any

展开后:

block out on fxp0 from 192.168.0.1 to any

block out on fxp0 from 10.5.32.6 to any

多种列表可以在规则中使用，并不仅仅限于过滤规则：

rdr on fxp0 proto tcp from any to any port { 22 80 } -

&gt;

\

192.168.0.6

block out on fxp0 proto { tcp udp } from { 192.168.0.1, \

10.5.32.6 } to any port { ssh telnet }

注意逗号在列表条目之间是可有可无的。

宏

宏是用户定义变量用来指定IP地址，端口号，接口名称等等。宏可以降低PF规则集的复杂度并且使得维护规则集变得容易。

宏名称必须以字母开头，可以包括字母，数字和下划线。宏名称不能包括保留关键字如：

pass, out, 以及 queue.

ext\_if = "fxp0"

block in on $ext\_if from any to any

这生成了一个宏名称为 ext\_if. 当一个宏在它产生以后被引用时，它得名称前面以$字符开头。

宏也可以展开成列表，如：

friends = "{ 192.168.1.1, 10.0.2.5, 192.168.43.53 }"

宏能够被重复定义，由于宏不能在引号内被扩展，因此必须使用下面得语法：

host1 = "192.168.1.1"

host2 = "192.168.1.2"

all\_hosts = "{" $host1 $host2 "}"

宏 $all\_hosts 现在会展开成 192.168.1.1, 192.168.1.2.

表是用来保存一组IPv4 或者 IPv6地址。在表中进行查询是非常快的，并且比列表消耗更少的内存和cpu时间。由于这个原因，表是保存大量地址的最好方法，在50，000个地址中查询仅比在50个地址中查询稍微多一点时间。表可以用于下列用途：

\* 过滤，整形，NAT和重定向中的源或者目的地址.

\* NAT规则中的转换地址.

\* 重定向规则中的重定向地址.

\* 过滤规则选项中 route-to, reply-to, 和 dup-to的目的地址.

表可以通过在pf.conf里配置和使用pfctl生成。

配置

在 pf.conf文件中, 表是使用table关键字创建出来的。下面得关键字必须在创建表时指定。

\* constant - 这类表得内容一旦创建出来就不能被改变。如果这个属性没有指定，可以使用pfctl（8）添加和删除表里得地址，即使系统是运行在2或者更高得安全级别上。

\* persist - 即使没有规则引用这类表，内核也会把它保留在内存中。如果这个属性没有指定，当最后引用它得规则被取消后内核自动把它移出内存。

实例:

table

&lt;

goodguys

&gt;

{ 192.0.2.0/24 }

table

&lt;

rfc1918

&gt;

const { 192.168.0.0/16, 172.16.0.0/12, \

10.0.0.0/8 }

table

&lt;

spammers

&gt;

persist

block in on fxp0 from {

&lt;

rfc1918

&gt;

,

&lt;

spammers

&gt;

} to any

pass in on fxp0 from

&lt;

goodguys

&gt;

to any

地址也可以用“非”来进行修改，如：

table

&lt;

goodguys

&gt;

{ 192.0.2.0/24, !192.0.2.5 }

goodguys表将匹配除192.0.2.5外192.0.2.0/24网段得所有地址。

注意表名总是在

&lt;

&gt;

符号得里面。

表也可以由包含IP地址和网络地址的文本文件中输入：

table

&lt;

spammers

&gt;

persist file "/etc/spammers"

block in on fxp0 from

&lt;

spammers

&gt;

to any

文件 /etc/spammers 应该包含被阻塞的IP地址或者CIDR网络地址，每个条目一行。以＃开头的行被认为是注释会被忽略。

用 pfctl 进行操作

表可以使用pfctl（8）进行灵活的操作。例如，在上面产生的表中增加条目可以这样写：

\# pfctl -t spammers -T add 218.70.0.0/16

如果这个表不存在，这样会创建出这个表来。列出表中的内容可以这样：

\# pfctl -t spammers -T show

-v 参数也可以使用-Tshow 来显示每个表的条目内容统计。要从表中删除条目，可以这样：

\# pfctl -t spammers -T delete 218.70.0.0/16

更多使用pfctl操作的信息可以参阅pfctl（8）。

指定地址

除了使用IP地址来指定主机外，也可以使用主机名。当主机名被解析成IP地址时，IPv4 和 IPv6地址都被插进规则中。IP地址也可以通过合法的接口名称或者self关键字输入表中，这样的表会分别包含接口或者机器上（包括loopback地 址）上配置的所有IP地址。

一个限制时指定地址0.0.0.0/0 以及 0/0在表中不能工作。替代方法是明确输入该地址或者使用宏。

地址匹配

表中的地址查询会匹配最接近的规则，比如：

table

&lt;

goodguys

&gt;

{ 172.16.0.0/16, !172.16.1.0/24, 172.16.1.100 }

block in on dc0 all

pass in on dc0 from

&lt;goodguys&gt;

to any

任何自dc0上数据包都会把它的源地址和goodguys表中的地址进行匹配：

\* 172.16.50.5 - 精确匹配172.16.0.0/16; 数据包符合可以通过

\* 172.16.1.25 - 精确匹配!172.16.1.0/24; 数据包匹配表中的一条规则，但规则是“非”（使用“！”进行了修改）；数据包不匹配表会被阻塞。

\* 172.16.1.100 - 准确匹配172.16.1.100; 数据包匹配表，运行通过

\* 10.1.4.55 - 不匹配表，阻塞。
