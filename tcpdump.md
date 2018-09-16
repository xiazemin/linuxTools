一、tcpdump





对于本机中进程的系统行为调用跟踪，strace是一个很好的工具，而在网络问题的调试中，tcpdump应该说是一个必不可少的工具，和大部分linux下优秀工具一样，它的特点就是简单而强大。

默认情况下，tcpdump不会抓取本机内部通讯的报文。根据网络协议栈的规定，对于报文，即使是目的地是本机，也需要经过本机的网络协议层，所以本机通讯肯定是通过API进入了内核，并且完成了路由选择。

二、linux下抓包原理

linux下的抓包是通过注册一种虚拟的底层网络协议来完成对网络报文\(准确的说是网络设备\)消息的处理权。当网卡接收到一个网络报文之后，它会遍历系统中所有已经注册的网络协议，例如以太网协议、x25协议处理模块来尝试进行报文的解析处理，这一点和一些文件系统的挂载相似，就是让系统中所有的已经注册的文件系统来进行尝试挂载，如果哪一个认为自己可以处理，那么就完成挂载。

当抓包模块把自己伪装成一个网络协议的时候，系统在收到报文的时候就会给这个伪协议一次机会，让它来对网卡收到的报文进行一次处理，此时该模块就会趁机对报文进行窥探，也就是把这个报文完完整整的复制一份，假装是自己接收到的报文，汇报给抓包模块。

先看一下网络层对于接收到的报文的处理方法

static int process\_backlog\(struct net\_device \*backlog\_dev, int \*budget\)---netif\_receive\_skb

    list\_for\_each\_entry\_rcu\(ptype, &ptype\_all, list\) {

        if \(!ptype-&gt;dev \|\| ptype-&gt;dev == skb-&gt;dev\) {

            if \(pt\_prev\)

                ret = deliver\_skb\(skb, pt\_prev, orig\_dev\);

            pt\_prev = ptype;

        }

    }

三、协议族的注册

对于这种协议，也只有在需要的时候才注册，因为它毕竟增加了系统报文的处理速度并且会消耗大量的系统skb。当抓包开始的时候，它会创建一个对应的网络套接口，这种套接口的类型就是af\_packet类型。相关实现为

linux-2.6.21\net\packet\af\_packet.c

static int packet\_create\(struct socket \*sock, int protocol\)

    sk-&gt;sk\_family = PF\_PACKET;

    po-&gt;num = proto;

……

    po-&gt;prot\_hook.func = packet\_rcv;

……



    if \(proto\) {

        po-&gt;prot\_hook.type = proto;

        dev\_add\_pack\(&po-&gt;prot\_hook\);这个接口会将prot\_hook注册到前面看到的ptype\_all队列中

        sock\_hold\(sk\);

        po-&gt;running = 1;

    }

当一个网卡上真正有报文到来的时候，它就会调用这里注册的packet\_rcv函数

……

    res = run\_filter\(skb, sk, snaplen\);如果说filter过滤失败，说明是抓包不关心的报文，直接放行，返回值非零表示不关心。

    if \(!res\)

        goto drop\_n\_restore;

……

    if \(skb\_shared\(skb\)\) {

        struct sk\_buff \*nskb = skb\_clone\(skb, GFP\_ATOMIC\);自己复制一份。

        if \(nskb == NULL\)

            goto drop\_n\_acct;



        if \(skb\_head != skb-&gt;data\) {

            skb-&gt;data = skb\_head;

            skb-&gt;len = skb\_len;

        }

        kfree\_skb\(skb\);

        skb = nskb;

    }

四、filter的执行

run\_filter---&gt;&gt;sk\_run\_filter

……



    for \(pc = 0; pc &lt; flen; pc++\) {

        fentry = &filter\[pc\];



        switch \(fentry-&gt;code\) {

        case BPF\_ALU\|BPF\_ADD\|BPF\_X:

            A += X;

            continue;

        case BPF\_ALU\|BPF\_ADD\|BPF\_K:

            A += fentry-&gt;k;

            continue;

……

       

        switch \(k-SKF\_AD\_OFF\) {

        case SKF\_AD\_PROTOCOL:

            A = ntohs\(skb-&gt;protocol\);

            continue;

        case SKF\_AD\_PKTTYPE:

            A = skb-&gt;pkt\_type;

            continue;

        case SKF\_AD\_IFINDEX:

            A = skb-&gt;dev-&gt;ifindex;

            continue;

        default:

            return 0;

        }

这个函数是执行了一个自己定义的指令集和。用户通过sockopt来注册这段指令，内核在内核态执行这些指令，完成匹配，其中包含了报文某些字段的加载，条件跳转、加减乘除以及返回等指令。当转包套接口接收到报文之后，对这个报文执行这段虚拟程序，直到遇到ret指令作为自己的返回值。通过tcpdump -d 可以显示出编译之后生成的指令，下面是一个测试输出

\[root@Harry bash-4.1\]\# tcpdump -d host 1.2.3.4

tcpdump: WARNING: eth0: no IPv4 address assigned

\(000\) ldh      \[12\]

\(001\) jeq      \#0x800           jt 2    jf 6

\(002\) ld       \[26\]  加载接收到报文的第26个字节开始的一个int类型，

\(003\) jeq      \#0x1020304       jt 12    jf 4 如果和0x1234相等，跳转到12跳指令，不等继续第四条指令。 

\(004\) ld       \[30\]

\(005\) jeq      \#0x1020304       jt 12    jf 13

\(006\) jeq      \#0x806           jt 8    jf 7

\(007\) jeq      \#0x8035          jt 8    jf 13

\(008\) ld       \[28\]

\(009\) jeq      \#0x1020304       jt 12    jf 10

\(010\) ld       \[38\]

\(011\) jeq      \#0x1020304       jt 12    jf 13

\(012\) ret      \#65535

\(013\) ret      \#0

\[root@Harry bash-4.1\]\# 

五、tcpdump的处理

对于tcpdump来说，命令行输入中除了选项之外的参数都将作为一个语法文件来处理。libcap中定义了自己的词法分析器和语法分析器。其词法分析器为scanner.l，语法分析文件为scanner.l，可见该语言的定义使用了通用的bison和flex的帮助。但是如果使用了这些工具，就意味着一个配置文件就可以写的很复杂，例如syslog-ng配置、ld连接配置脚本、bash脚本等各种语法。

这个语法文件如果感兴趣可以看看说明文档即可。

其中pcap库中没有对网卡的配置，而且tcpdump的命令处理对于-i是覆盖的，所以一次tcpdump只能侦听一个端口。如果命令行没有指定网络设备，pcap会枚举系统中所有的网络设备，然后取出第一个设备\(这个顺序在不通的系统有不同排列顺序，看一下tcpdump的输出即可确定是在那个端口\)

六、本地报文处理

    if \(res.type == RTN\_LOCAL\) {

        if \(!fl.fl4\_src\)

            fl.fl4\_src = fl.fl4\_dst;

        if \(dev\_out\)

            dev\_put\(dev\_out\);

        dev\_out = &loopback\_dev;

        dev\_hold\(dev\_out\);

        fl.oif = dev\_out-&gt;ifindex;

        if \(res.fi\)

            fib\_info\_put\(res.fi\);

        res.fi = NULL;

        flags \|= RTCF\_LOCAL;

        goto make\_route;

    }

struct net\_device loopback\_dev = {

    .name             = "lo",

