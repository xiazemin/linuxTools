PCAP是一个数据包抓取库， 很多软件都是用它来作为数据包抓取工具的。 WireShark也是用PCAP库来抓取数据包的。PCAP抓取出来的数据包并不是原始的网络字节流，而是对其进行从新组装，形成一种新的数据格式。

一个用PCAP抓取的数据包的文件格式如下：

Pcap文件头24B各字段说明：

Magic：4B：0x1A 2B 3C 4D:用来标示文件的开始

Major：2B，0x02 00:当前文件主要的版本号

Minor：2B，0x04 00当前文件次要的版本号

ThisZone：4B当地的标准时间；全零

SigFigs：4B时间戳的精度；全零

SnapLen：4B最大的存储长度

LinkType：4B链路类型

常用类型：

0            BSD loopback devices, except for later OpenBSD

```
   1            Ethernet, and Linux loopback devices

   6            802.5 Token Ring

   7            ARCnet

   8            SLIP

   9            PPP

   10          FDDI

   100        LLC/SNAP-encapsulated ATM

   101        "raw IP", with no link

   102        BSD/OS SLIP

   103        BSD/OS PPP

   104        Cisco HDLC

   105        802.11

   108        later OpenBSD loopback devices \(with the AF\_value in network byte order\)

   113        special Linux "cooked" capture

   114        LocalTalk
```

其中我们最为常见的类型就是1，以太网链路。

字段说明：

Timestamp：时间戳高位，精确到seconds

Timestamp：时间戳低位，精确到microseconds

Caplen：当前数据区的长度，即抓取到的数据帧长度，由此可以得到下一个数据帧的位置。

Len：离线数据长度：网络中实际数据帧的长度，一般不大于caplen，多数情况下和Caplen数值相等。

Packet 数据：即 Packet（通常就是链路层的数据帧去掉前面用于同步和标识帧开始的8字节和最后用于CRC校验的4字节）具体内容，长度就是Caplen，这个长度的 后面，就是当前PCAP文件中存放的下一个Packet数据包，也就是说：PCAP文件里面并没有规定捕获的Packet数据包之间有什么间隔字符串，我 们需要靠第一个Packet包确定下一组数据在文件中的起始位置，向后以此类推。

Packet 包头和Packet数据组成

字段说明：

Timestamp：时间戳高位，精确到seconds（值是自从January 1, 1970 00:00:00 GMT以来的秒数来记）

Timestamp：时间戳低位，精确到microseconds （数据包被捕获时候的微秒（microseconds）数，是自ts-sec的偏移量）

Caplen：当前数据区的长度，即抓取到的数据帧长度，由此可以得到下一个数据帧的位置。

Len：离线数据长度：网络中实际数据帧的长度，一般不大于caplen，多数情况下和Caplen数值相等。

（例如，实际上有一个包长度是1500 bytes（Len=1500），但是因为在Global Header的snaplen=1300有限制，所以只能抓取这个包的前1300个字节，这个时候，Caplen = 1300 ）

Packet 数据：即 Packet（通常就是链路层的数据帧）具体内容，长度就是Caplen，这个长度的后面，就是当前PCAP文件中存放的下一个Packet数据包，也就 是说：PCAP文件里面并没有规定捕获的Packet数据包之间有什么间隔字符串，下一组数据在文件中的起始位置。我们需要靠第一个Packet包确定。 最后，Packet数据部分的格式其实就是标准的网路协议格式了可以任何网络教材上找得到。

下面是一个PCAP数据包的实例，该数据包包含了两条消息。下图是用十六进制工具将该数据包打开后的截图。

![](/assets/importpcap.png)

图中最开始的绿色部分就是24 Bytes的Pcap Header，接下来红色的16 Bytes是第一个消息的Packet Header, 后面的红色的16 Bytes是第二个消息的Packet Header。两块蓝色的部分分别是两个消息从链路层开始的完整内容。在网络上实际传输的数据包在数据链路层上每一个Packet开始都会有7个用于同步 的字节\(10101010, 10101010, 10101010, 10101010, 10101010, 10101010, 10101010,\)和一个用于标识该Packet开始的字节\(10101011\)，最后还会有四个CRC校验字节；而PCAP文件中会把前8个字节和最 后4个校验自己去掉，因为这些信息对于协议分析是没有用处的。

用Wireshark打开一个Pcap数据包后， 每条消息的所有field会被解析出来并会按照协议层次折叠起来。第一层显示的是Frame XXX，这一级别没有对应某层具体的协议，而是对本条消息的一个概括性总结，描述了一些有用的概括性信息，比如从里面我们可以看到本条消息各种协议的层次 关系，展开其它协议层之后对应的是该协议的各个域；

