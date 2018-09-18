第一部分：PCAP包文件格式

一 基本格式：



   文件头 数据包头数据报数据包头数据报......



二、文件头：



   



   文件头结构体

 sturct pcap\_file\_header

 {

      DWORD           magic;

      DWORD           version\_major;

      DWORD           version\_minor;

      DWORD           thiszone;

      DWORD           sigfigs;

      DWORD           snaplen;

      DWORD           linktype;

 }

 

说明：

 1、标识位：32位的，这个标识位的值是16进制的 0xa1b2c3d4。

a 32-bit        magic number ,The magic number has the value hex a1b2c3d4.

2、主版本号：16位， 默认值为0x2。

a 16-bit          major version number,The major version number should have the value 2.

3、副版本号：16位，默认值为0x04。

a 16-bit          minor version number,The minor version number should have the value 4.

4、区域时间：32位，实际上该值并未使用，因此可以将该位设置为0。

a 32-bit          time zone offset field that actually not used, so you can \(and probably should\) just make it 0;

5、精确时间戳：32位，实际上该值并未使用，因此可以将该值设置为0。

a 32-bit          time stamp accuracy field tha not actually used,so you can \(and probably should\) just make it 0;

6、数据包最大长度：32位，该值设置所抓获的数据包的最大长度，如果所有数据包都要抓获，将该值设置为65535；例如：想获取数据包的前64字节，可将该值设置为64。

a 32-bit          snapshot length" field;The snapshot length field should be the maximum number of bytes perpacket that will be captured. If the entire packet is captured, make it 65535; if you only capture, for example, the first 64 bytes of the packet, make it 64.

7、链路层类型：32位， 数据包的链路层包头决定了链路层的类型。

a 32-bit link layer type field.The link-layer type depends on the type of link-layer header that the

packets in the capture file have:

 

以下是数据值与链路层类型的对应表

0            BSD       loopback devices, except for later OpenBSD

1            Ethernet, and Linux loopback devices   以太网类型，大多数的数据包为这种类型。

6            802.5 Token Ring

7            ARCnet

8            SLIP

9            PPP

10          FDDI

100        LLC/SNAP-encapsulated ATM

101        raw IP, with no link

102        BSD/OS SLIP

103        BSD/OS PPP

104        Cisco HDLC

105        802.11

108        later OpenBSD loopback devices \(with the AF\_value in network byte order\)

113               special Linux cooked capture

114               LocalTalk



三 packet数据包头：



 



struct pcap\_pkthdr

{

struct tim         ts;

      DWORD              caplen;

      DWORD              len;

}

 

struct tim

{

DWORD       GMTtime;

DWORD       microTime

}

说明：

 

1、时间戳，包括：

秒计时：32位，一个UNIX格式的精确到秒时间值，用来记录数据包抓获的时间，记录方式是记录从格林尼治时间的1970年1月1日 00:00:00 到抓包时经过的秒数；

微秒计时：32位， 抓取数据包时的微秒值。

a time stamp, consisting of:

a UNIX-format time-in-seconds when the packet was captured, i.e. the number of seconds since January 1,1970, 00:00:00 GMT \(that GMT, \*NOT\* local time!\);  

the number of microseconds since that second when the packet was captured;



Timestamp：时间戳高位，精确到seconds（值是自从January 1, 1970 00:00:00 GMT以来的秒数来记）

Timestamp：时间戳低位，精确到microseconds （数据包被捕获时候的微秒（microseconds）数，是自ts-sec的偏移量）

 

2、数据包长度：32位 ，标识所抓获的数据包保存在pcap文件中的实际长度，以字节为单位。

a 32-bit value giving the number of bytes of packet data that were captured;



Caplen：当前数据区的长度，即抓取到的数据帧长度，由此可以得到下一个数据帧的位置。



 

3、数据包实际长度： 所抓获的数据包的真实长度，如果文件中保存不是完整的数据包，那么这个值可能要比前面的数据包长度的值大。

a 32-bit value giving the actual length of the packet, in bytes \(which may be greater than the previous number, if you are not saving the entire packet\).



Len：离线数据长度：网络中实际数据帧的长度，一般不大于caplen，多数情况下和Caplen数值相等。

（例如，实际上有一个包长度是1500 bytes（Len=1500），但是因为在Global Header的snaplen=1300有限制，所以只能抓取这个包的前1300个字节，这个时候，Caplen = 1300 ）



四：packet数据：



  即Packet（通常就是链路层的数据帧）具体内容，长度就是Caplen，这个长度的后面，就是当前PCAP文件中存放的下一个Packet数据包，也就是说：PCAP文件里面并没有规定捕获的Packet数据包之间有什么间隔字符串，下一组数据在文件中的起始位置。我们需要靠第一个Packet包确定。最后，Packet数据部分的格式其实就是标准的网路协议格式了可以任何网络教材上找得到。



 



 



五：举例分析



 



图中最开始的绿色部分就是24 Bytes的Pcap Header,接下来红色的16 Bytes是第一个消息的Pcap Header。后面的红色的16 Bytes是第二个消息的Pcap Header。两块蓝色的部分分别是两个消息从链路层开始的完整内容。在网络上实际传输的数据包在数据链路层上每一个Packet开始都会有7个用于同步的字节和一个用于标识该Packet开始的字节，最后还会有四个CRC校验字节；而PCAP文件中会把前8个字节和最后4个校验自己去掉，因为这些信息对于协议分析是没有用的。



用Wireshark打开一个PCAP数据包，每条消息的所有field会被解析出来并会按照协议层次折叠起来。第一层显示的是FrameXXX，这一级别没有对应某层具体的协议，而是对本条消息的一个概括性总结，描述了一些有用的概括性信息，比如从里面我们可以看到本条消息各种协议的层次关系，展开其它协议层之后对应的是该协议的各个域

