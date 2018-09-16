Linux下的Bridge也是一种虚拟设备，这多少和vlan有点相似，它依赖于一个或多个从设备。与VLAN不同的是，它不是虚拟出和从设备同一层次的镜像设备，而是虚拟出一个高一层次的设备，并把从设备虚拟化为端口port，且同时处理各个从设备的数据收发及转发，再加上netfilter框架的一些东西，使得它的实现相比vlan复杂得多。

1.Bridge的功能框图

```
它是Linux下虚拟出来bridge设备，Linux下可用brctl命令创建br设备，如下
```

brctl addbr brname

然后添加port，并进行相应配置，就可以使用了

brctl addif brname eth0

brctl addif brname eth1

ifconfig brname IP up

ifconfig eth0 0.0.0.0 up

ifconfig eth1 0.0.0.0 up

可见br设备是建立在从设备之上的（这些从设备可以是实际设备，也可以是vlan设备等），并且可以为br准备一个IP（br设备的MAC地址是它所有从设备中最小的MAC地址），这样该主机就可以通过这个br设备与网络中的其它主机通信了（详见发送功能框图）。

另外它的从设备被虚拟化为端口port，它们的IP及MAC都不再可用，且它们被设置为接收任何包，最终由bridge设备来决定数据包的去向：接收到本机、转发、丢弃（详见接收功能框图）。

![](/assets/importbr.png)![](/assets/importbrin.png)

## 2.Bridge设备的创建

和Vlan一样，bridge也被当成一个module加载进内核，它的module\\_init\\(\\)函数和vlan差不多，进行一些namespace的注册，特殊的是它还注册了一个netfilter\\_ops，在内核全局的HOOK函数表中增加了7个函数，其中5个的pf=Bridge，另两个的pf分别为INET、INET6，它们主要用于bridge中的netfilter操作（后面会细讲）。

最后，也是我们这最关心的是，它注册了一个ioctl函数br\\_ioctl\\_deviceless\\_stub\\(\\)，该ioctl函数和vlan的一样，都会作为sock\\_ioctl\\(\\)的特殊情况被调用。映射到应用层，它应该是对某个socket插口进行ioctl操作，详见brctl源码。该ioctl函数中最主要的就是br\\_add\\_bridge\\(net,buf\\)，用于创建bridge设备，如下图所示：

![](/assets/importbridge.png)

该函数调用netdev\_alloc\(\)，申请net\_device\(\)，并分配私有空间net\_bridge\(\)结构，指明初始化函数为br\_dev\_setup\(\)，最后register\_netdev\(\)把该设备组册进内核，可见bridge设备和一般的设备差不多。

主要看其中br\_dev\_setup\(\)，首先初始化设备的type、flags为bridge，然后最关键的是设置其dev-&gt;netdev\_ops = br\_netdev\_ops，即内核为bridge设备准备好了一套通用的驱动函数，这个直接关系到bridge的工作方法，后面再细讲。然后初始化私有空间net\_bridge\(\)结构，设置bridge的本地设备及从设备的list（当然这是还没有从设备加进来），然后设置了桥的group\_address，即上一节所说的特殊的MAC地址，最后还初始化了timer相关的。

这时bridge还不完整，还需添加port从设备，由命令brctl addif brname portdev完成，但要注意，虽然还是brctl命令，但此时的操作对象是已经存在的bridge设备，映射到内核中就是br\\_netdev\\_ops-&gt;ioctl\\(\\)中的br\\_add\\_if\\(\\)（它是br设备的ioctl操作，和之前那个sock\\_ioctl的分支不是一个层次上的）。至于怎么从应用层直接操作底层的net\\_device设备的，可以参见brctl源码，以后再看吧，先看看这里的br\\_add\\_if\\(\\)

首先判断dev从设备必须不是loopback，不是bridge，不是其他bridge的port，且要是ethernet设备，才能继续；然后根据br、dev选择一个index号，并分配一个新的net\_bridge\_port结构，初始化之，并将它加入bridge的port\_list中；最后br的一些物理参数，其MAC地址为所有从设备中MAC最小的（由上一节知，从设备被设置成全接收模式，其IP和MAC都没有了），且其MTU也为所有从设备中最小的。上面设置br的相关参数，下面还要设置从设备，首先使dev-&gt;master=br\\_dev（实际上就是构成上一节数据结构中的索引关系）；然后设置dev-&gt;prive\\_flags加上IFF\\_BRIDGE\\_PORT，这样它就不能再作为其他br的从设备了；最后也是最关键的，设置dev-&gt;rx\\_handler为br\\_handler\\_frame\\(\\)，为数据接收作准备。

3.Bridge设备的发送流程

前面也讲过了，Linux下的bridge设备，对下层而言是一个桥设备，进行数据的转发（实际上对下也有接收能力，下一节讲）。而对上层而言，它就像普通的ethernet设备一样，有自己的IP和MAC地址，那么上层当然可以把它加入路由系统，并利用它发送数据啦，并且很容易想到，它的发射函数最终肯定是利用某个从设备的驱动去完成实际的发送的，这个和VLAN是相通的。

上层根据目的IP地址，路由选择了该br\_dev设备发送，并且由ARP缓存中得到了对应的目的MAC，填写在了skb中，然后启动了发送流程dev\_queue\_xmit\(skb\)。因为此时的skb-&gt;dev为br\_dev，无queue，直接去调用br设备的发送函数，该函数就是br\_netdev\_ops中定义的br\_dev\_xmit\(skb,br\_dev\)。该函数首先根据目的MAC地址，确定是广播还是单播，这里仅讨论单播时，根据DMAC在net\\_bridge的fdb\\_hash中找到相应的net\\_bridge\\_fdb\\_entry项，并索引到对应的端口net\\_bridge\\_port。最后利用该端口的从设备来发送数据，注意，这里是直接调用dev-&gt;ops-&gt;ndo\\_start\\_xmit\\(skb,dev\\)的，一放面这里的dev已经是从设备了，另一方面，这里没有像VLAN中那样重定位skb-&gt;dev，并重启发送流程dev\\_queue\\_xmit\\(\\)，是因为一个从设备只能作为一个bridge的port，没有其它身份，不存在竞争问题。



4.Bridge设备的接收流程

和VLAN一样，实际接收由硬件设备完成，最终通过netif\\_receive\\_skb\\(skb\\)函数提交给上层，而在该函数中会处理vlan、bridge这类特殊设备。与LVAN的仅是把skb设备重定位以实现对上层透明的要求不同，Bridge接受过程复杂得多，因此专门注册了一个函数来处理，即前面提到的rx\\_handler\\(\\)，它被注册在port设备的net\\_device结构中（这算是port设备失去自身IP、MAC的一个补偿吧J），如下图所示。只有作为bridge的从设备才会注册rx\\_handler\\(\\)，并在这里执行，处理桥接，普通的设备不会执行到这里。





