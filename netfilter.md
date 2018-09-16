Netfilter是Linux 2.4.x引入的一个子系统，它作为一个通用的、抽象的框架，提供一整套的hook函数的管理机制，使得诸如数据包过滤、网络地址转换\(NAT\)和基于协议类型的连接跟踪成为了可能。

netfilter的架构就是在整个网络流程的若干位置放置了一些检测点（HOOK），而在每个检测点上登记了一些处理函数进行处理。

IP层的五个HOOK点的位置如下图所示

\[1\]:NF\_IP\_PRE\_ROUTING：刚刚进入网络层的数据包通过此点（刚刚进行完版本号，校验

和等检测）， 目的地址转换在此点进行；

\[2\]:NF\_IP\_LOCAL\_IN：经路由查找后，送往本机的通过此检查点，INPUT包过滤在此点进行；

\[3\]:NF\_IP\_FORWARD：要转发的包通过此检测点，FORWARD包过滤在此点进行；

\[4\]:NF\_IP\_POST\_ROUTING：所有马上便要通过网络设备出去的包通过此检测点，内置的源地址转换功能（包括地址伪装）在此点进行；

\[5\]:NF\_IP\_LOCAL\_OUT：本机进程发出的包通过此检测点，OUTPUT包过滤在此点进行。

在IP层代码中，有一些带有NF\_HOOK宏的语句，如IP的转发函数中有：

如果在编译内核时没有配置netfilter时，就相当于调用最后一个参数，此例中即执行

ip\_forward\_finish函数；否则进入HOOK点，执行通过nf\_register\_hook（）登记的功能

（这句话表达的可能比较含糊，实际是进入nf\_hook\_slow（）函数，再由它执行登记的

函数）。

NF\_HOOK宏的参数分别为：

⒈ pf：协议族名，netfilter架构同样可以用于IP层之外，因此这个变量还可以有诸如

PF\_INET6，PF\_DECnet等名字。

⒉hook：HOOK点的名字，对于IP层，就是取上面的五个值；

⒊skb：不用多解释了吧；

⒋indev：进来的设备，以struct net\_device结构表示；

⒌outdev：出去的设备，以struct net\_device结构表示；

（后面可以看到，以上五个参数将传到用nf\_register\_hook登记的处理函数中。）

⒍okfn：是个函数指针，当所有的该HOOK点的所有登记函数调用完后，转而走此流程。

这些点是已经在内核中定义好的，除非你是这部分内核代码的维护者，否则无权增加

或修改，而在此检测点进行的处理，则可由用户指定。像packet filter,NAT,connection

track这些功能，也是以这种方式提供的。正如netfilter的当初的设计目标－－提供一

个完善灵活的框架，为扩展功能提供方便。

如果我们想加入自己的代码，便要用nf\_register\_hook函数，其函数原型为：

int nf\_register\_hook\(struct nf\_hook\_ops \*reg\)

我们考察一下struct nf\_hook\_ops结构：

我们的工作便是生成一个struct nf\_hook\_ops结构的实例，并用nf\_register\_hook将

其HOOK上。其中list项我们总要初始化为{NULL,NULL}；由于一般在IP层工作，pf总是

PF\_INET；hooknum就是我们选择的HOOK点；一个HOOK点可能挂多个处理函数，谁先谁后

，便要看优先级，即priority的指定了。netfilter\_ipv4.h中用一个枚举类型指定了内

置的处理函数的优先级：

hook是提供的处理函数，也就是我们的主要工作，其原型为：

它的五个参数将由NFHOOK宏传进去。

了解了这些，基本上便可以可以写一个lkm出来了。

