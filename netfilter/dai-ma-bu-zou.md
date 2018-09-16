首先，定义一个nf\_hook\_ops结构体，选定要挂取的钩子点



static struct nf\_hook\_ops nfho = {  



    .hook = my\_hookfn,  



    .pf = PF\_INET,  



    .hooknum = NF\_INET\_PRE\_ROUTING,  



    .priority = NF\_IP\_PRI\_FIRST,  



    .owner = THIS\_MODULE,  



}; 



第二步，写好my\_hookfn这个hook函数，即要实现的功能（此处是将其接收包的源IP地址改成100.100.100.100.



unsigned int my\_hookfn\(unsigned int hooknum,  



    struct sk\_buff \*skb,  



    const struct net\_device \*in,  



    const struct net\_device \*out,  



    int \(\*okfn\)\(struct sk\_buff \*\)\)  



{  



    struct iphdr \*iph;  



    iph = ip\_hdr\(skb\);  



  



    /\* log the original src IP \*/  



    printk\(KERN\_INFO"src IP %pI4\n", &iph-&gt;saddr\);  



  



    /\* modify the packet's src IP \*/  



    iph-&gt;saddr = in\_aton\("100.100.100.100"\);  



  



    return NF\_ACCEPT;  



}  



注册函数会把nf\_hook\_ops放入nf\_hooks相应的位置中。



int nf\_register\_hook\(struct nf\_hook\_ops \*reg\)



{



    struct nf\_hook\_ops \*elem;



    int err;



    err = mutex\_lock\_interruptible\(&nf\_hook\_mutex\);



    if \(err &lt; 0\)



        return err;



    list\_for\_each\_entry\(elem, &nf\_hooks\[reg-&gt;pf\]\[reg-&gt;hooknum\], list\) {



        if \(reg-&gt;priority &lt; elem-&gt;priority\)



            break;



    }



    list\_add\_rcu\(&reg-&gt;list, elem-&gt;list.prev\); /\* 把netfilter实例添加到队列中 \*/



    mutex\_unlock\(&nf\_hook\_mutex\);



    return 0;



}



注销函数



void nf\_unregister\_hook\(struct nf\_hook\_ops \*reg\)



{



    mutex\_lock\(&nf\_hook\_mutex\);



    list\_del\_rcu\(&reg-&gt;list\); /\* 把netfilter实例从队列中删除 \*/



    mutex\_unlock\(&nf\_hook\_mutex\);



    synchronize\_net\(\);



}



然后进行模块的注册与注销



static int \_\_init http\_init\(void\)  



{  



    if \(nf\_register\_hook\(&nfho\)\) {  



        printk\(KERN\_ERR"nf\_register\_hook\(\) failed\n"\);  



        return -1;  



    }  



    return 0;  



}  



  



static void \_\_exit http\_exit\(void\)  



{  



    nf\_unregister\_hook\(&nfho\);  



}  



这是完整的源代码，并取名http.c



\#include &lt;linux/netfilter.h&gt;  



\#include &lt;linux/init.h&gt;  



\#include &lt;linux/module.h&gt;  



\#include &lt;linux/netfilter\_ipv4.h&gt;  



\#include &lt;linux/ip.h&gt;  



\#include &lt;linux/inet.h&gt;  



  



/\*\* 



 \* Hook function to be called. 



 \* We modify the packet's src IP. 



 \*/  



unsigned int my\_hookfn\(unsigned int hooknum,  



    struct sk\_buff \*skb,  



    const struct net\_device \*in,  



    const struct net\_device \*out,  



    int \(\*okfn\)\(struct sk\_buff \*\)\)  



{  



    struct iphdr \*iph;  



    iph = ip\_hdr\(skb\);  



  



    /\* log the original src IP \*/  



    printk\(KERN\_INFO"src IP %pI4\n", &iph-&gt;saddr\);  



  



    /\* modify the packet's src IP \*/  



    iph-&gt;saddr = in\_aton\("100.100.100.100"\);  



  



    return NF\_ACCEPT;  



}  



  



/\* A netfilter instance to use \*/  



static struct nf\_hook\_ops nfho = {  



    .hook = my\_hookfn,  



    .pf = PF\_INET,  



    .hooknum = NF\_INET\_PRE\_ROUTING,  



    .priority = NF\_IP\_PRI\_FIRST,  



    .owner = THIS\_MODULE,  



};  



  



static int \_\_init http\_init\(void\)  



{  



    if \(nf\_register\_hook\(&nfho\)\) {  



        printk\(KERN\_ERR"nf\_register\_hook\(\) failed\n"\);  



        return -1;  



    }  



    return 0;  



}  



  



static void \_\_exit http\_exit\(void\)  



{  



    nf\_unregister\_hook\(&nfho\);  



}  



  



module\_init\(http\_init\);  



module\_exit\(http\_exit\);  



MODULE\_AUTHOR\("flyking"\);  



MODULE\_LICENSE\("GPL"\); 



然后编写Makefile，此处是Makefile源码



ifneq \($\(KERNELRELEASE\),\)



    obj-m += http.o



else



    PWD := $\(shell pwd\)



    KVER := $\(shell uname -r\)



    KDIR := /lib/modules/$\(KVER\)/build



default:    



$\(MAKE\) -C $\(KDIR\)  M=$\(PWD\) modules



all:



make -C $\(KDIR\) M=$\(PWD\) modules 



clean:



rm -rf \*.o \*.mod.c \*.ko \*.symvers \*.order \*.makers



 endif

