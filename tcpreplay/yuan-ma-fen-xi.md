tcpreplay介绍

tcpreplay主要用于重放pcap数据包，还可以对pcap文件进行修改，比如修改ip地址和端口号等，其中主要包含了一下几个模块：

tcpreplay：pcap重放模块，其中提供了包重放速度控制，循环控制，重放模式等功能。

tcpreweite： 修改网络中mac，IP地址，端口信息。

tcpbrige：利用tcprewrite的功能实现两个网络部分的桥

tcpreplay的作者在写sendpacket（）函数时说：希望写一个通用的数据包发送api接口支持BPF, libpcap, libdnet, and Linux's PF\_PACKET，因为libnet缺乏活动性，libpcap支持模块比较新，并且缺乏非linux支持，所以作者决定同时支持这四个，他们的匹配顺序如下，如果平台支持其中最先匹配的函数，就使用它发包。由于libpcap不提供可靠的方法获取MAC地址，所以使用PF\_PACKET or BPF代替。



 \* 1. PF\_PACKET  send\(\)                    \(int\)send\(sp-&gt;handle.fd, \(void \*\)data, len, 0\); linux上面使用

 \* 2. BPF send\(\)                                            write\(sp-&gt;handle.fd, \(void \*\)data, len\);    freebsd上使用

 \* 3. libdnet eth\_send\(\)                      eth\_send\(sp-&gt;handle.ldnet, \(void\*\)data, \(size\_t\)len\);

  \* 4. pcap\_inject\(\)                            pcap\_inject\(sp-&gt;handle.pcap, \(void\*\)data, len\);{



                                                                     return \(p-&gt;inject\_op\(p, buf, size\)\);



                                                                      }



                                                                 handle-&gt;inject\_op = pcap\_inject\_linux;



                                                                  pcap\_inject\_linux\(pcap\_t \*handle, const void \*buf, size\_t size\){



                                                                ret = send\(handle-&gt;fd, buf, size, 0\);



                                                                   }

  \* 5. pcap\_sendpacket\(\)     pcap\_sendpacket\(sp-&gt;handle.pcap, data, \(int\)len\); /\* out of buffers, or hit max PHY speed, silently retry \*/从缓存发送或者使用最大速度发送



   {

if \(p-&gt;inject\_op\(p, buf, size\) == -1\)

return \(-1\);

return \(0\);

}



（1）PF\_PACKET：linux上使用，不经过协议栈，直接到用户层收发数据数据



（2）BPF：是类Unix系统上数据链路层的一种原始接口，提供原始链路层封包的收发，bsd系统上面使用，不经过协议栈，直接到用户层收发数据







（3）libnet，一个小型的接口函数库，建立一个简单统一的网络编程接口以屏蔽不同操作系统低层网络编程的差别，libnet目前可以在Linux、FreeBSD、Solaris、WindowsNT等操作系统上运行，并且提供了统一的接口



（4） libpcap：一个网络捕获数据包的开放源码，被tcpdump、snort等著名软件包使用。可以在绝大多数类unix平台下工作，如果希望libpcap能在linux上正常工作，则必须使内核支持"packet"协议，也即在编译内核时打开配置选项 CONFIG\_PACKET\(选项缺省为打开\)。windows版本为winpcap。





pcap\_inject\(\)来源于OpenBSD,调用p-&gt;inject\_op，pcap\_inject\_linux，send发送数据；



pcap\_send-packet\(\) 则来源于WinPcap, 与pcap\_inject实现一样，只是更改了接口，,返回0表示成功，-1表示失败。两个函数都提供是为了兼容。

