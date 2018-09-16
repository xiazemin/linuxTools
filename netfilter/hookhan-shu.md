对于每种类型的协议，数据包都会依次按照hook点的方向进行传输，每个hook点上Netfilter又按照优先级挂了很多hook函数。这些hook函数就是用来处理数据包用的。



Netfilter使用NF\_HOOK\(include/linux/netfilter.h\)宏在协议栈内部切入到Netfilter框架中。相比于2.4版本，2.6版内核在该宏的定义上显得更加灵活一些，定义如下：



\#define NF\_HOOK\(pf, hook, skb, indev, outdev, okfn\) \



         NF\_HOOK\_THRESH\(pf, hook, skb, indev, outdev, okfn, INT\_MIN\)



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



         \_\_ret = \(okfn\)\(skb\);                                                       \



\_\_ret;}\)



我们发现NF\_HOOK\_THRESH宏只增加了一个thresh参数，这个参数就是用来指定通过该宏去遍历钩子函数时的优先级，同时，该宏内部又调用了nf\_hook\_thresh函数：



static inline int nf\_hook\_thresh\(int pf, unsigned int hook,



                            struct sk\_buff \*\*pskb,



                            struct net\_device \*indev,



                            struct net\_device \*outdev,



                            int \(\*okfn\)\(struct sk\_buff \*\), int thresh,



                            int cond\)



{



if \(!cond\) 



return 1;



\#ifndef CONFIG\_NETFILTER\_DEBUG



if \(list\_empty\(&nf\_hooks\[pf\]\[hook\]\)\)



         return 1;



\#endif



return nf\_hook\_slow\(pf, hook, pskb, indev, outdev, okfn, thresh\);



}



这个函数又只增加了一个参数cond，该参数为0则放弃遍历，并且也不执行okfn函数；为1则执行nf\_hook\_slow去完成钩子函数okfn的顺序遍历\(优先级从小到大依次执行\)。



在net/netfilter/core.h文件中定义了一个二维的结构体数组，用来存储不同协议栈钩子点的回调处理函数。



struct list\_head nf\_hooks\[NPROTO\]\[NF\_MAX\_HOOKS\];



其中，行数NPROTO为32，即目前内核所支持的最大协议簇；列数NF\_MAX\_HOOKS为挂载点的个数

