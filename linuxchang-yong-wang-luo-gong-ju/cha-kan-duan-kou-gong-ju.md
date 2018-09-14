## netstat {#netstat}

这个命令用来查看**当前建立的网络连接\(深刻理解netstat每一项代表的含义\)**。最经典的案例就是查看本地系统打开了哪些端口：

```bash
fgp@controller:~$ sudo netstat -lnpt
[sudo] password for fgp:
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:3306            0.0.0.0:*               LISTEN      2183/mysqld
tcp        0      0 0.0.0.0:11211           0.0.0.0:*               LISTEN      2506/memcached
tcp        0      0 0.0.0.0:9292            0.0.0.0:*               LISTEN      1345/python
tcp        0      0 0.0.0.0:6800            0.0.0.0:*               LISTEN      2185/ceph-osd
tcp        0      0 0.0.0.0:6801            0.0.0.0:*               LISTEN      2185/ceph-osd
tcp        0      0 0.0.0.0:28017           0.0.0.0:*               LISTEN      1339/mongod
tcp        0      0 0.0.0.0:6802            0.0.0.0:*               LISTEN      2185/ceph-osd
tcp        0      0 0.0.0.0:6803            0.0.0.0:*               LISTEN      2185/ceph-osd
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1290/sshd

```

netstat能够查看所有的网络连接，包括unix socket连接，其功能非常强大。

另外使用netstat还可以查看本地路由表：

```

```

以上`Genmask`为`0.0.0.0`的表示**默认路由，即连接外网的路由**。网络中0.0.0.0的IP地址表示整个网络，即网络中的所有主机。它的作用是帮助路由器发送路由表中无法查询的包。如果设置了全零网络的路由，路由表中无法查询的包都将送到全零网络的路由中去。

## lsof {#lsof}

`lsof`命令用来查看打开的文件\(list open files\)，由于在Linux中一切皆文件，那socket、pipe等也是文件，因此能够查看网络连接以及网络设备，其中和网络最相关的是`-i`选项，它输出符合条件的进程（4、6、协议、:端口、 @ip等），它的格式为`[46][protocol][@hostname|hostaddr][:service|port]`,比如查看22端口有没有打开，哪个进程打开的:

```

```

可见22端口是sshd这个命令，其进程号pid为1290打开的。

可以指定多个条件，但默认是OR关系的，如果需要AND关系，必须传入`-a`参数，比如查看22端口并且使用Ipv6连接的进程：

```

```

列出所有与`192.168.56.1`（我的宿主机IP地址）的ipv4连接：

```

```

## iftop {#iftop}

用过`top`以及`iotop`的，自然能够大致猜到`iftop`的功能，它是用于**查看网络流量的工具**（display bandwidth usage on an interface by host）：

```
sudo iftop
```

![](http://int32bit.me/img/posts/Linux常用网络工具总结/iftop.png "iftop")

