对于每种类型的协议，数据包都会依次按照hook点的方向进行传输，每个hook点上Netfilter又按照优先级挂了很多hook函数。这些hook函数就是用来处理数据包用的。

Netfilter使用NF\_HOOK\(include/linux/netfilter.h\)宏在协议栈内部切入到Netfilter框架中。相比于2.4版本，2.6版内核在该宏的定义上显得更加灵活一些，定义如下：

\#define NF\_HOOK\(pf, hook, skb, indev, outdev, okfn\) \

```
     NF\_HOOK\_THRESH\(pf, hook, skb, indev, outdev, okfn, INT\_MIN\)
```

关于宏NF\_HOOK各个参数的解释说明：

1\)         pf：协议族名，Netfilter架构同样可以用于IP层之外，因此这个变量还可以有诸如PF\_INET6，PF\_DECnet等名字。

2\)         hook：HOOK点的名字，对于IP层，就是取上面的五个值；

3\)         skb：不解释；

4\)         indev：数据包进来的设备，以struct net\_device结构表示；

5\)         outdev：数据包出去的设备，以struct net\_device结构表示；

\(后面可以看到，以上五个参数将传递给nf\_register\_hook中注册的处理函数。\)

6\)         okfn:是个函数指针，当所有的该HOOK点的所有登记函数调用完后，转而走此流程。

而NF\_HOOK\_THRESH又是一个宏：

\#define NF\_HOOK\_THRESH\(pf, hook, skb, indev, outdev, okfn, thresh\)                \

\({int \_\_ret;                                                                                \

if \(\(\_\_ret=nf\_hook\_thresh\(pf, hook, &\(skb\), indev, outdev, okfn, thresh, 1\)\) == 1\)\

```
     \_\_ret = \(okfn\)\(skb\);                                                       \
```

\_\_ret;}\)

我们发现NF\_HOOK\_THRESH宏只增加了一个thresh参数，这个参数就是用来指定通过该宏去遍历钩子函数时的优先级，同时，该宏内部又调用了nf\_hook\_thresh函数：

static inline int nf\_hook\_thresh\(int pf, unsigned int hook,

```
                        struct sk\_buff \*\*pskb,



                        struct net\_device \*indev,



                        struct net\_device \*outdev,



                        int \(\*okfn\)\(struct sk\_buff \*\), int thresh,



                        int cond\)
```

{

if \(!cond\)

return 1;

\#ifndef CONFIG\_NETFILTER\_DEBUG

if \(list\_empty\(&nf\_hooks\[pf\]\[hook\]\)\)

```
     return 1;
```

\#endif

return nf\_hook\_slow\(pf, hook, pskb, indev, outdev, okfn, thresh\);

}

这个函数又只增加了一个参数cond，该参数为0则放弃遍历，并且也不执行okfn函数；为1则执行nf\_hook\_slow去完成钩子函数okfn的顺序遍历\(优先级从小到大依次执行\)。

在net/netfilter/core.h文件中定义了一个二维的结构体数组，用来存储不同协议栈钩子点的回调处理函数。

struct list\_head nf\_hooks\[NPROTO\]\[NF\_MAX\_HOOKS\];

其中，行数NPROTO为32，即目前内核所支持的最大协议簇；列数NF\_MAX\_HOOKS为挂载点的个数

在include/linux/socket.h中IP协议AF\_INET\(PF\_INET\)的序号为2，因此我们就可以得到TCP/IP协议族的钩子函数挂载点为：



PRE\_ROUTING：     nf\_hooks\[2\]\[0\]



LOCAL\_IN：        nf\_hooks\[2\]\[1\]



FORWARD：      nf\_hooks\[2\]\[2\]



LOCAL\_OUT：      nf\_hooks\[2\]\[3\]



POST\_ROUTING：          nf\_hooks\[2\]\[4\]



同时我们看到，在2.6内核的IP协议栈里，从协议栈正常的流程切入到Netfilter框架中，然后顺序、依次去调用每个HOOK点所有的钩子函数的相关操作有如下几处：



       1）、net/ipv4/ip\_input.c里的ip\_rcv函数。该函数主要用来处理网络层的IP报文的入口函数，它到Netfilter框架的切入点为：



NF\_HOOK\(PF\_INET, NF\_IP\_PRE\_ROUTING, skb, dev, NULL,ip\_rcv\_finish\)



根据前面的理解，这句代码意义已经很直观明确了。那就是：如果协议栈当前收到了一个IP报文\(PF\_INET\)，那么就把这个报文传到Netfilter的NF\_IP\_PRE\_ROUTING过滤点，去检查\[R\]在那个过滤点\(nf\_hooks\[2\]\[0\]\)是否已经有人注册了相关的用于处理数据包的钩子函数。如果有，则挨个去遍历链表nf\_hooks\[2\]\[0\]去寻找匹配的match和相应的target，根据返回到Netfilter框架中的值来进一步决定该如何处理该数据包\(由钩子模块处理还是交由ip\_rcv\_finish函数继续处理\)。



\[R\]：刚才说到所谓的“检查”。其核心就是nf\_hook\_slow\(\)函数。该函数本质上做的事情很简单，根据优先级查找双向链表nf\_hooks\[\]\[\]，找到对应的回调函数来处理数据包：



struct list\_head \*\*i;



list\_for\_each\_continue\_rcu\(\*i, head\) {



struct nf\_hook\_ops \*elem = \(struct nf\_hook\_ops \*\)\*i;



if \(hook\_thresh &gt; elem-&gt;priority\)



                  continue;



         verdict = elem-&gt;hook\(hook, skb, indev, outdev, okfn\);



         if \(verdict != NF\_ACCEPT\) { … … }



    return NF\_ACCEPT;



}



上面的代码是net/netfilter/core.c中的nf\_iterate\(\)函数的部分核心代码，该函数被nf\_hook\_slow函数所调用，然后根据其返回值做进一步处理。



2）、net/ipv4/ip\_forward.c中的ip\_forward函数，它的切入点为：



NF\_HOOK\(PF\_INET, NF\_IP\_FORWARD, skb, skb-&gt;dev, rt-&gt;u.dst.dev,ip\_forward\_finish\);



在经过路由抉择后，所有需要本机转发的报文都会交由ip\_forward函数进行处理。这里，该函数由NF\_IP\_FOWARD过滤点切入到Netfilter框架，在nf\_hooks\[2\]\[2\]过滤点执行匹配查找。最后根据返回值来确定ip\_forward\_finish函数的执行情况。



3）、net/ipv4/ip\_output.c中的ip\_output函数，它切入Netfilter框架的形式为：



NF\_HOOK\_COND\(PF\_INET, NF\_IP\_POST\_ROUTING, skb, NULL, dev,ip\_finish\_output,



                                !\(IPCB\(skb\)-&gt;flags & IPSKB\_REROUTED\)\);



这里我们看到切入点从无条件宏NF\_HOOK改成了有条件宏NF\_HOOK\_COND，调用该宏的条件是：如果协议栈当前所处理的数据包skb中没有重新路由的标记，数据包才会进入Netfilter框架。否则直接调用ip\_finish\_output函数走协议栈去处理。除此之外，有条件宏和无条件宏再无其他任何差异。



如果需要陷入Netfilter框架则数据包会在nf\_hooks\[2\]\[4\]过滤点去进行匹配查找。



4）、还是在net/ipv4/ip\_input.c中的ip\_local\_deliver函数。该函数处理所有目的地址是本机的数据包，其切入函数为：



NF\_HOOK\(PF\_INET, NF\_IP\_LOCAL\_IN, skb, skb-&gt;dev, NULL,ip\_local\_deliver\_finish\);



发给本机的数据包，首先全部会去nf\_hooks\[2\]\[1\]过滤点上检测是否有相关数据包的回调处理函数，如果有则执行匹配和动作，最后根据返回值执行ip\_local\_deliver\_finish函数。



5）、net/ipv4/ip\_output.c中的ip\_push\_pending\_frames函数。该函数是将IP分片重组成完整的IP报文，然后发送出去。进入Netfilter框架的切入点为：



NF\_HOOK\(PF\_INET, NF\_IP\_LOCAL\_OUT, skb, NULL, skb-&gt;dst-&gt;dev, dst\_output\);



对于所有从本机发出去的报文都会首先去Netfilter的nf\_hooks\[2\]\[3\]过滤点去过滤。一般情况下来来说，不管是路由器还是PC中端，很少有人限制自己机器发出去的报文。因为这样做的潜在风险也是显而易见的，往往会因为一些不恰当的设置导致某些服务失效，所以在这个过滤点上拦截数据包的情况非常少。当然也不排除真的有特殊需求的情况。



 



小节：整个Linux内核中Netfilter框架的HOOK机制可以概括如下：



在数据包流经内核协议栈的整个过程中，在一些已预定义的关键点上PRE\_ROUTING、LOCAL\_IN、FORWARD、LOCAL\_OUT和POST\_ROUTING会根据数据包的协议簇PF\_INET到这些关键点去查找是否注册有钩子函数。如果没有，则直接返回okfn函数指针所指向的函数继续走协议栈；如果有，则调用nf\_hook\_slow函数，从而进入到Netfilter框架中去进一步调用已注册在该过滤点下的钩子函数，再根据其返回值来确定是否继续执行由函数指针okfn所指向的函数

