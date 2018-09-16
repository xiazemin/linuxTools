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

```
和Vlan一样，bridge也被当成一个module加载进内核，它的module\_init\(\)函数和vlan差不多，进行一些namespace的注册，特殊的是它还注册了一个netfilter\_ops，在内核全局的HOOK函数表中增加了7个函数，其中5个的pf=Bridge，另两个的pf分别为INET、INET6，它们主要用于bridge中的netfilter操作（后面会细讲）。
```

```

最后，也是我们这最关心的是，它注册了一个ioctl函数br\_ioctl\_deviceless\_stub\(\)，该ioctl函数和vlan的一样，都会作为sock\_ioctl\(\)的特殊情况被调用。映射到应用层，它应该是对某个socket插口进行ioctl操作，详见brctl源码。该ioctl函数中最主要的就是br\_add\_bridge\(net,buf\)，用于创建bridge设备，如下图所示：
```




