在Linux里面使用网桥非常简单，仅需要做两件事情就可以配置了。其一是在编译内核里把CONFIG\_BRIDGE或CONDIG\_BRIDGE\_MODULE编译选项打开；其二是安装brctl工具。第一步是使内核协议栈支持网桥，第二步是安装用户空间工具，通过一系列的ioctl调用来配置网桥。下面以一个相对简单的实例来贯穿全文，以便分析代码。

```
    Linux机器有4个网卡，分别是eth0~eth4，其中eth0用于连接外网，而eth1, eth2, eth3都连接到一台PC机，用于配置网桥。只需要用下面的命令就可以完成网桥的配置：
```

Brctl addbr br0 \(建立一个网桥br0, 同时在Linux内核里面创建虚拟网卡br0\)

Brctl addif br0 eth1

Brctl addif br0 eth2

Brctl addif br0 eth3 \(分别为网桥br0添加接口eth1, eth2和eth3\)

其中br0作为一个网桥，同时也是虚拟的网络设备，它即可以用作网桥的管理端口，也可作为网桥所连接局域网的网关，具体情况视你的需求而定。要使用br0接口时，必需为它分配IP地址。为正常工作，PC1, PC2，PC3和br0的IP地址分配在同一个网段。

  


  


  


  


  


  


  


在内核，网桥是以模块的方式存在，注册源码路径：\net\brige\br.c：

  


  


  


4.1 初始化

static int \_\_init br\_init\(void\)

{

```
br\_fdb\_init\(\); //网桥数据库初始化，分配slab缓冲区
```

\#ifdef CONFIG\_BRIDGE\_NETFILTER

```
if \(br\_netfilter\_init\(\)\) //netfilter钩子初始化

    return 1;
```

\#endif

```
brioctl\_set\(br\_ioctl\_deviceless\_stub\); //设置ioctl钩子函数:br\_ioctl\_hook

br\_handle\_frame\_hook = br\_handle\_frame;//设置报文处理钩子:br\_ioctl\_hook



//网桥数据库处理钩子

br\_fdb\_get\_hook = br\_fdb\_get;

br\_fdb\_put\_hook = br\_fdb\_put;



//在netdev\_chain通知链表上注册

register\_netdevice\_notifier\(&br\_device\_notifier\);



return 0;
```

}

```
      4.2 新建网桥



     前面说到通过brctl addbr br0命令建立网桥，此处用户控件调用的brctl命令最终对应到内核中的br\_ioctl\_deviceless\_stub处理函数：



     /net/bridge/br\_ioctl.c
```

int br\_ioctl\_deviceless\_stub\(unsigned int cmd, void \_\_user \*uarg\)

{

```
switch \(cmd\) {

case SIOCGIFBR:

case SIOCSIFBR:

    return old\_deviceless\(uarg\);



case SIOCBRADDBR: //新建网桥

case SIOCBRDELBR: //删除网桥

{

    char buf\[IFNAMSIZ\];



    if \(!capable\(CAP\_NET\_ADMIN\)\)

        return -EPERM;



    //copy\_from\_user:把用户空间的数据拷入内核空间

    if \(copy\_from\_user\(buf, uarg, IFNAMSIZ\)\)

        return -EFAULT;



    buf\[IFNAMSIZ-1\] = 0;

    if \(cmd == SIOCBRADDBR\)

        return br\_add\_bridge\(buf\);



    return br\_del\_bridge\(buf\);

}

}

return -EOPNOTSUPP;
```

}

```
      在这里，我们传入的cmd为SIOCBRADDBR.转入br\_add\_bridge\(buf\)中进行\(./net/bridge/br\_if.c\)： 
```

int br\_add\_bridge\(const char \*name\)

{

```
struct net\_device \*dev;

int ret;



//为虚拟桥新建一个net\_device

dev = new\_bridge\_dev\(name\);

if \(!dev\) 

    return -ENOMEM;



rtnl\_lock\(\);

//由内核确定接口名字，例如eth0 eth1等

if \(strchr\(dev-&gt;name, '%'\)\) {

    ret = dev\_alloc\_name\(dev, dev-&gt;name\);

    if \(ret &lt; 0\)

        goto err1;

}





//向内核注册此网络设备

ret = register\_netdevice\(dev\);

if \(ret\)

    goto err2;



/\* network device kobject is not setup until

 \* after rtnl\_unlock does it's hotplug magic.

 \* so hold reference to avoid race.

 \*/

dev\_hold\(dev\);

rtnl\_unlock\(\);



//在sysfs中建立相关信息

ret = br\_sysfs\_addbr\(dev\);

dev\_put\(dev\);

if \(ret\) 

    unregister\_netdev\(dev\);
```

out:

```
return ret;
```

err2:

```
free\_netdev\(dev\);
```

err1:

```
rtnl\_unlock\(\);

goto out;
```

}

```
      网桥是一个虚拟的设备，它的注册跟实际的物理网络设备注册是一样的。我们关心的是网桥对应的net\_device结构是什么样的，继续跟踪进new\_bridge\_dev\(./net/bridge/br\_if.c\)： 
```

static struct net\_device \*new\_bridge\_dev\(const char \*name\)

{

```
struct net\_bridge \*br;

struct net\_device \*dev;



//分配net\_device

dev = alloc\_netdev\(sizeof\(struct net\_bridge\), name,

         br\_dev\_setup\);  

if \(!dev\)

    return NULL;





//网桥的私区结构为net\_bridge

br = netdev\_priv\(dev\);





//私区结构中的dev字段指向设备本身

br-&gt;dev = dev;





//队列初始化。在port\_list中保存了这个桥上的端口列表

spin\_lock\_init\(&br-&gt;lock\); 

INIT\_LIST\_HEAD\(&br-&gt;port\_list\);

spin\_lock\_init\(&br-&gt;hash\_lock\);



//下面这部份代码跟stp协议相关,我们暂不关心



br-&gt;bridge\_id.prio\[0\] = 0x80;

br-&gt;bridge\_id.prio\[1\] = 0x00;

memset\(br-&gt;bridge\_id.addr, 0, ETH\_ALEN\);



br-&gt;stp\_enabled = 0;

br-&gt;designated\_root = br-&gt;bridge\_id;

br-&gt;root\_path\_cost = 0;

br-&gt;root\_port = 0;

br-&gt;bridge\_max\_age = br-&gt;max\_age = 20 \* HZ;

br-&gt;bridge\_hello\_time = br-&gt;hello\_time = 2 \* HZ;

br-&gt;bridge\_forward\_delay = br-&gt;forward\_delay = 15 \* HZ;

br-&gt;topology\_change = 0;

br-&gt;topology\_change\_detected = 0;

br-&gt;ageing\_time = 300 \* HZ;

INIT\_LIST\_HEAD\(&br-&gt;age\_list\);



br\_stp\_timer\_init\(br\);



return dev;
```

}

```
     在br\_dev\_setup中还做了一些另外在函数指针初始化\(./net/bridge/br\_device.c\)： 
```

void br\_dev\_setup\(struct net\_device \*dev\)

{

```
//将桥的MAC地址设为零

memset\(dev-&gt;dev\_addr, 0, ETH\_ALEN\);





//初始化dev的部分函数指针，因为目前网桥设备主适用于以及网,

//以太网的部分功能对它也适用

ether\_setup\(dev\);



//设置设备的ioctl函数为br\_dev\_ioctl

dev-&gt;do\_ioctl = br\_dev\_ioctl;





//网桥与一般网卡不同，网桥统一统计它的数据包和字节数等信息

dev-&gt;get\_stats = br\_dev\_get\_stats;





// 网桥接口的数据包发送函数,真实设备要向外发送数据时,是通过网卡向外发送数据 

// 而该网桥设备要向外发送数据时，它的处理逻辑与网桥其它接口的基本一致。 

dev-&gt;hard\_start\_xmit = br\_dev\_xmit;

dev-&gt;open = br\_dev\_open;

dev-&gt;set\_multicast\_list = br\_dev\_set\_multicast\_list;

dev-&gt;change\_mtu = br\_change\_mtu;

dev-&gt;destructor = free\_netdev;

SET\_MODULE\_OWNER\(dev\);

dev-&gt;stop = br\_dev\_stop;

dev-&gt;tx\_queue\_len = 0;

dev-&gt;set\_mac\_address = NULL;

dev-&gt;priv\_flags = IFF\_EBRIDGE;
```

}

4.3  添加删除端口

```
    仅仅创建网桥，还是不够的。实际应用中的网桥需要添加实际的端口（即物理接口），如例子中的eth1, eth2等。应用程序在使用ioctl来为网桥增加物理接口，对应内核函数br\_dev\_ioctl的代码和分析如下\( /net/bridge/br\_ioctl.c\)： 
```

int br\_dev\_ioctl\(struct net\_device \*dev, struct ifreq \*rq, int cmd\)

{

```
struct net\_bridge \*br = netdev\_priv\(dev\);



switch\(cmd\) {

case SIOCDEVPRIVATE:

    return old\_dev\_ioctl\(dev, rq, cmd\);



case SIOCBRADDIF: //添加

case SIOCBRDELIF: //删除



    //同一处理函数，默认为添加

    return add\_del\_if\(br, rq-&gt;ifr\_ifindex, cmd == SIOCBRADDIF\);

}



pr\_debug\("Bridge does not support ioctl 0x%x\n", cmd\);

return -EOPNOTSUPP;
```

}

```
    下面分析具体的添加删除函数add\_del\_if \( /net/bridge/br\_ioctl.c\)：
```

static int add\_del\_if\(struct net\_bridge \*br, int ifindex, int isadd\)

{

```
struct net\_device \*dev;

int ret;



if \(!capable\(CAP\_NET\_ADMIN\)\)

    return -EPERM;



dev = dev\_get\_by\_index\(ifindex\);

if \(dev == NULL\)

    return -EINVAL;



if \(isadd\)

    ret = br\_add\_if\(br, dev\);

else

    ret = br\_del\_if\(br, dev\);



dev\_put\(dev\);

return ret;
```

}

```
       对应的添加删除函数分别为：br\_add\_if, br\_del\_if\( /net/bridge/br\_ioctl.c\):
```

int br\_add\_if\(struct net\_bridge \*br, struct net\_device \*dev\)

{

```
struct net\_bridge\_port \*p;

int err = 0;



/\*--Kernel仅支持以太网网桥--\*/

if \(dev-&gt;flags & IFF\_LOOPBACK \|\| dev-&gt;type != ARPHRD\_ETHER\)

    return -EINVAL;



/\*--把网桥接口当作物理接口加入到另一个网桥中，是不行的,逻辑和代码上都会出现 loop--\*/

if \(dev-&gt;hard\_start\_xmit == br\_dev\_xmit\)

    return -ELOOP;



/\*--该物理接口已经绑定到另一个网桥了--\*/

if \(dev-&gt;br\_port != NULL\)

    return -EBUSY;



/\*--为该接口创建一个网桥端口数据，并初始化好该端口的相关数据--\*/

if \(IS\_ERR\(p = new\_nbp\(br, dev, br\_initial\_port\_cost\(dev\)\)\)\)

    return PTR\_ERR\(p\);



/\*--将该接口的物理地址写入到 MAC-端口映射表中，该MAC是属于网桥内部端口的固定MAC地址， 

    它在fdb中的记录是固定的，不会失效（agged）--\*/

 if \(\(err = br\_fdb\_insert\(br, p, dev-&gt;dev\_addr\)\)\)

    destroy\_nbp\(p\);





 /\*--添加相应的系统文件信息--\*/

else if \(\(err = br\_sysfs\_addif\(p\)\)\)

    del\_nbp\(p\);

else {

    /\*--打开该接口的混杂模式，网桥中的各个端口必须处于混杂模式，网桥才能正确工作--\*/

    dev\_set\_promiscuity\(dev, 1\);



    /\*--加到端口列表--\*/

    list\_add\_rcu\(&p-&gt;list, &br-&gt;port\_list\);



    /\*--STP相关设置-\*/

    spin\_lock\_bh\(&br-&gt;lock\);

    br\_stp\_recalculate\_bridge\_id\(br\);

    br\_features\_recompute\(br\);

    if \(\(br-&gt;dev-&gt;flags & IFF\_UP\) 

     && \(dev-&gt;flags & IFF\_UP\) && netif\_carrier\_ok\(dev\)\)

        br\_stp\_enable\_port\(p\);

    spin\_unlock\_bh\(&br-&gt;lock\);



    /\*--设置设备的mtu--\*/

    dev\_set\_mtu\(br-&gt;dev, br\_min\_mtu\(br\)\);

}



return err;
```

}

int br\_del\_if\(struct net\_bridge \*br, struct net\_device \*dev\)

{

```
struct net\_bridge\_port \*p = dev-&gt;br\_port;



if \(!p \|\| p-&gt;br != br\) 

    return -EINVAL;



br\_sysfs\_removeif\(p\);

del\_nbp\(p\);



spin\_lock\_bh\(&br-&gt;lock\);

br\_stp\_recalculate\_bridge\_id\(br\);

br\_features\_recompute\(br\);

spin\_unlock\_bh\(&br-&gt;lock\);



return 0;
```

}

