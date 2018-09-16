概述

libpcap是一个网络数据包捕获函数库，tcpdump就是以libpcap为基础的。

主要作用：

捕获各种数据包，例如：网络流量统计

过滤网络数据包，例如：过滤掉本地上的一些数据，类似防火墙

分析网络数据包，例如：分析网络协议，数据的采集

存储网络数据包，例如：保存捕获的数据以为将来进行分析

 



libpcap的抓包框架

pcap\_lookupdev\(\):函数用来查找网络设备，返回可被pcap\_open\_live\(\)函数调用的网络设备名指针。

pcap\_lookupnet\(\):函数获得指定网络设备的网络号和掩码。

pcap\_open\_live\(\):函数用于打开设备，并且返回用于捕获网络数据包的数据包捕获描述字。对于此网络设备的操作都要基于此网络设备描述字。

pcap\_compile\(\):函数用于将用户制定的过滤策略编译到过滤程序中

pcap\_setfilter\(\):函数用于设置过滤器

pcap\_loop\(\):与pcap\_next\(\)和pcap\_next\_ex\(\)两个函数一样用来捕获数据包

pcap\_close\(\):函数用于关闭网络设备，释放资源

 



利用libpcap函数库开发应用程序的步骤：

打开网络设备

设置过滤规则

捕获数据

关闭网络设备

 



详细步骤：

首先要使用libpcap，需要包含pcap.h头文件。

获取网络设备接口：

char \*pcap\_lookupdev\(char \* errbuf\);

功能：自动获取可用的网络设备名指针

参数：errbuf，存放出错信息字符串，有宏定义缓冲区大小，PCAP\_ERRBUF\_SIZE

返回值:成功返回设备名指针（第一个合适的网络接口的字符串指针），失败则返回NULL,同时，errbuf存放出错信息字符串

1 //自动获取网络接口形式

2 char errBuf\[PCAP\_ERRBUF\_SIZE\], \*devStr;

3 devStr = pcap\_lookupdev\(errBuf\);

4 

5 //手动获取网络接口形式只需要被devStr赋值即可

6 char errBuf\[PCAP\_ERRBUF\_SIZE\], \*devStr = “eth0”;

 



获取网络号\(ip地址\)和掩码

int pcap\_lookupnet\(char\* device,bpf\_u\_int32 \*netp,bpf\_u\_int32 \*maskp,char \*errbuf\);

功能：获取指定网卡的ip地址，子网掩码

参数：device：网络设备名，为第一步获取的网络接口字符串，也可以人为指定，如“eth0”；

         netp：存放ip地址的指针，buf\_u\_int32为32位无符号整型

         maskp：存放子网掩码的指针

         errbuf：存放出错信息

返回值：成功返回0，失败返回1

复制代码

 1 char error\_content\[PCAP\_ERRBUF\_SIZE\] = {0}; // 出错信息  

 2 char \*dev = pcap\_lookupdev\(error\_content\);  

 3 if\(NULL == dev\)  

 4 {  

 5     printf\(error\_content\);  

 6     exit\(-1\);  

 7 }  

 8   

 9   

10 bpf\_u\_int32 netp = 0, maskp = 0;  

11 pcap\_t \* pcap\_handle = NULL;  

12 int ret = 0;  

13   

14 //获得网络号和掩码  

15 ret = pcap\_lookupnet\(dev, &netp, &maskp, error\_content\);  

16 if\(ret == -1\)  

17 {  

18     printf\(error\_content\);  

19     exit\(-1\);  

20 }  

复制代码

 



打开网络接口：

pcap\_t \*pcap\_open\_live\(const char \* device,int snaplen,int promisc,int to\_ms,char \*errbuf\);

功能：打开一个用于捕获数据的网络端口

参数：device：网络接口的名字，为第一步获取的网络接口字符串，也可以人为指定，如：”eth0“

         snaplen：捕获数据包的长度，不能大于65535个字节

         promise：”1“代表混杂模式，其他值代表非混杂模式

         to\_ms：指定需要等地啊的毫秒数，超过这个时间后，获得数据包的函数会立即返回，0表示一直等待直到有数据包到来

         errbuf：存储错误信息

返回值：返回pcap\_t类型指针，后面的所有操作都要使用这个指针。

复制代码

 1 char error\_content\[PCAP\_ERRBUF\_SIZE\] = {0}; // 出错信息  

 2 char \*dev = pcap\_lookupdev\(error\_content\);  // 获取网络接口  

 3 if\(NULL == dev\)  

 4 {  

 5     printf\(error\_content\);  

 6     exit\(-1\);  

 7 }  

 8   

 9 // 打开网络接口  

10 pcap\_t \* pcap\_handle = pcap\_open\_live\(dev, 65535, 1, 0, error\_content\);  

11 if\(NULL == pcap\_handle\)  

12 {  

13     printf\(error\_content\);  

14     exit\(-1\);  

15 }  

复制代码

 



获取数据包：

方法一：const u\_char \*pcap\_next\(pcap\_t \*p,struct pcap\_pkthdr \*h\);

功能：捕获一个网络数据包，收到一个数据包立即返回

参数：p：pcap\_open\_live\(\)返回的pcap\_t类型的指针

         h：数据包头

pcap\_pkthdr类型的定义如下：

1 struct pcap\_pkthdr  

2 {  

3     struct timeval ts; // 抓到包的时间  

4     bpf\_u\_int32 caplen; // 表示抓到的数据长度  

5     bpf\_u\_int32 len; // 表示数据包的实际长度  

6 }  

 



len和caplen的区别：因为在某些情况下不能保证捕获的包是完整的，例如一个包长1480，但是你捕获到1000的时候，可能因为某些原因就终止捕获了，所以caplen是记录实际捕获的包长，也就是1000，而len就是1480

返回值：成功则返回捕获数据包的地址，失败返回NULL

复制代码

 1 const unsigned char \*p\_packet\_content = NULL; // 保存接收到的数据包的起始地址  

 2 pcap\_t \*pcap\_handle = NULL;  

 3 struct pcap\_pkthdr protocol\_header;  

 4   

 5 pcap\_handle = pcap\_open\_live\("eth0", 1024, 1, 0,NULL\);  

 6   

 7 p\_packet\_content = pcap\_next\(pcap\_handle, &protocol\_header\);   

 8 //p\_packet\_content  所捕获数据包的地址  

 9           

10 printf\("Capture Time is :%s",ctime\(\(const time\_t \*\)&protocol\_header.ts.tv\_sec\)\); // 时间  

11 printf\("Packet Lenght is :%d\n",protocol\_header.len\);   // 数据包的实际长度

复制代码

 



方法二：int pcap\_loop\(pcap\_t \*p,int cnt,pcap\_handler callback,u\_char \*user\);

功能：循环捕获网络数据包，直到遇到错误或者满足退出条件，每次捕获一个数据包就会调用callback指定的回调函数，所以，可以在回调函数中进行数据包的处理操作。

参数：p：pcap\_open\_live\(\)返回的pcap\_t类型的指针

         cnt：指定捕获数据包的个数，一旦抓到cnt个数据包，pcap\_loop立即返回，如果是-1，就会一直捕获直到出错

         callback:回调函数，名字任意，根据需要自行取名

         user：向回调函数中传递的参数

callback回调函数的定义：

void callback\(u\_char \*userarg,const struct pcap\_pkthdr \*pkthdr,const u\_char \*packet\)

userarg:pcap\_loop\(\)的最后一个参数，当收到足够数量的包后pcap\_loop会调用callback回调函数，同时将pcap\_loop\(\)的user参数传递给它

pkthdr：是收到数据包的pcap\_pkthdr类型的指针，和pcap\_next\(\)第二个参数是一样的

packet：收到的数据包数据

返回值：成功返回0，失败返回负数

方法三：int pcap\_dispatch\(pcap\_t \*p,int cnt,pcap\_handler callback,u\_char \*user\);

这个函数和pcap\_loop\(\)非常类似，只是在超过to\_ms毫秒后就会返回（to\_ms是pcap\_open\_live\(\)的第四个参数）

释放网络接口：

void pcap\_close\(pcap\_t \*p\);

功能：关闭pcap\_open\_live\(\)打开的网络接口，并释放相关资源

参数：p：需要关闭的网络接口，pcap\_open\_live\(\)的返回值

复制代码

 1 // 打开网络接口  

 2 pcap\_t \* pcap\_handle = pcap\_open\_live\("eth0", 1024, 1, 0, error\_content\);  

 3 if\(NULL == pcap\_handle\)  

 4 {  

 5     printf\(error\_content\);  

 6     exit\(-1\);  

 7 }  

 8   

 9 //// ……  

10 //// ……  

11   

12 pcap\_close\(pcap\_handle\); //释放网络接口  

复制代码

 



复制代码

//接收一个数据包

\#include &lt;stdio.h&gt;  

\#include &lt;pcap.h&gt;  

\#include &lt;arpa/inet.h&gt;  

\#include &lt;time.h&gt;  

\#include &lt;stdlib.h&gt;  

struct ether\_header  

{  

    unsigned char ether\_dhost\[6\];   //目的mac  

    unsigned char ether\_shost\[6\];   //源mac  

    unsigned short ether\_type;      //以太网类型  

};  

\#define BUFSIZE 1514  

  

int main\(int argc,char \*argv\[\]\)  

{  

    pcap\_t \* pcap\_handle = NULL;  

    char error\_content\[100\] = "";   // 出错信息  

    const unsigned char \*p\_packet\_content = NULL;       // 保存接收到的数据包的起始地址  

    unsigned char \*p\_mac\_string = NULL;         // 保存mac的地址，临时变量  

    unsigned short ethernet\_type = 0;           // 以太网类型  

    char \*p\_net\_interface\_name = NULL;      // 接口名字  

    struct pcap\_pkthdr protocol\_header;  

    struct ether\_header \*ethernet\_protocol;  

  

    //获得接口名  

    p\_net\_interface\_name = pcap\_lookupdev\(error\_content\);  

    if\(NULL == p\_net\_interface\_name\)  

    {  

        perror\("pcap\_lookupdev"\);  

        exit\(-1\);  

    }  

      

    //打开网络接口  

    pcap\_handle = pcap\_open\_live\(p\_net\_interface\_name,BUFSIZE,1,0,error\_content\);  

    p\_packet\_content = pcap\_next\(pcap\_handle,&protocol\_header\);  

      

    printf\("------------------------------------------------------------------------\n"\);  

    printf\("capture a Packet from p\_net\_interface\_name :%s\n",p\_net\_interface\_name\);  

    printf\("Capture Time is :%s",ctime\(\(const time\_t \*\)&protocol\_header.ts.tv\_sec\)\);  

    printf\("Packet Lenght is :%d\n",protocol\_header.len\);  

      

    /\* 

    \*分析以太网中的 源mac、目的mac 

    \*/  

    ethernet\_protocol = \(struct ether\_header \*\)p\_packet\_content;  

    p\_mac\_string = \(unsigned char \*\)ethernet\_protocol-&gt;ether\_shost;//获取源mac  

    printf\("Mac Source Address is %02x:%02x:%02x:%02x:%02x:%02x\n",\*\(p\_mac\_string+0\),\*\(p\_mac\_string+1\),\*\(p\_mac\_string+2\),\*\(p\_mac\_string+3\),\*\(p\_mac\_string+4\),\*\(p\_mac\_string+5\)\);  

    p\_mac\_string = \(unsigned char \*\)ethernet\_protocol-&gt;ether\_dhost;//获取目的mac  

    printf\("Mac Destination Address is %02x:%02x:%02x:%02x:%02x:%02x\n",\*\(p\_mac\_string+0\),\*\(p\_mac\_string+1\),\*\(p\_mac\_string+2\),\*\(p\_mac\_string+3\),\*\(p\_mac\_string+4\),\*\(p\_mac\_string+5\)\);  

  

    /\* 

    \*获得以太网的数据包的地址，然后分析出上层网络协议的类型 

    \*/  

    ethernet\_type = ntohs\(ethernet\_protocol-&gt;ether\_type\);  

    printf\("Ethernet type is :%04x\t",ethernet\_type\);  

    switch\(ethernet\_type\)  

    {  

        case 0x0800:printf\("The network layer is IP protocol\n"\);break;//ip  

        case 0x0806:printf\("The network layer is ARP protocol\n"\);break;//arp  

        case 0x0835:printf\("The network layer is RARP protocol\n"\);break;//rarp  

        default:printf\("The network layer unknow!\n"\);break;  

    }  

      

    pcap\_close\(pcap\_handle\);  

    return 0;  

}  

复制代码

 



复制代码

 1//接收多个数据包 2 \#include &lt;stdio.h&gt;  

 3 \#include &lt;pcap.h&gt;  

 4 \#include &lt;arpa/inet.h&gt;  

 5 \#include &lt;time.h&gt;  

 6 \#include &lt;stdlib.h&gt;  

 7   

 8 \#define BUFSIZE 1514  

 9   

10 struct ether\_header  

11 {  

12     unsigned char ether\_dhost\[6\];   //目的mac  

13     unsigned char ether\_shost\[6\];   //源mac  

14     unsigned short ether\_type;      //以太网类型  

15 };  

16   

17 /\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*回调函数\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*/  

18 void ethernet\_protocol\_callback\(unsigned char \*argument,const struct pcap\_pkthdr \*packet\_heaher,const unsigned char \*packet\_content\)  

19 {  

20     unsigned char \*mac\_string;              //  

21     struct ether\_header \*ethernet\_protocol;  

22     unsigned short ethernet\_type;           //以太网类型  

23     printf\("----------------------------------------------------\n"\);  

24     printf\("%s\n", ctime\(\(time\_t \*\)&\(packet\_heaher-&gt;ts.tv\_sec\)\)\); //转换时间  

25     ethernet\_protocol = \(struct ether\_header \*\)packet\_content;  

26       

27     mac\_string = \(unsigned char \*\)ethernet\_protocol-&gt;ether\_shost;//获取源mac地址  

28     printf\("Mac Source Address is %02x:%02x:%02x:%02x:%02x:%02x\n",\*\(mac\_string+0\),\*\(mac\_string+1\),\*\(mac\_string+2\),\*\(mac\_string+3\),\*\(mac\_string+4\),\*\(mac\_string+5\)\);  

29     mac\_string = \(unsigned char \*\)ethernet\_protocol-&gt;ether\_dhost;//获取目的mac  

30     printf\("Mac Destination Address is %02x:%02x:%02x:%02x:%02x:%02x\n",\*\(mac\_string+0\),\*\(mac\_string+1\),\*\(mac\_string+2\),\*\(mac\_string+3\),\*\(mac\_string+4\),\*\(mac\_string+5\)\);  

31       

32     ethernet\_type = ntohs\(ethernet\_protocol-&gt;ether\_type\);//获得以太网的类型  

33     printf\("Ethernet type is :%04x\n",ethernet\_type\);  

34     switch\(ethernet\_type\)  

35     {  

36         case 0x0800:printf\("The network layer is IP protocol\n"\);break;//ip  

37         case 0x0806:printf\("The network layer is ARP protocol\n"\);break;//arp  

38         case 0x0835:printf\("The network layer is RARP protocol\n"\);break;//rarp  

39         default:break;  

40     }  

41     usleep\(800\*1000\);  

42 }  

43   

44 int main\(int argc, char \*argv\[\]\)  

45 {  

46     char error\_content\[100\];    //出错信息  

47     pcap\_t \* pcap\_handle;  

48     unsigned char \*mac\_string;                

49     unsigned short ethernet\_type;           //以太网类型  

50     char \*net\_interface = NULL;                 //接口名字  

51     struct pcap\_pkthdr protocol\_header;  

52     struct ether\_header \*ethernet\_protocol;  

53       

54     //获取网络接口  

55     net\_interface = pcap\_lookupdev\(error\_content\);  

56     if\(NULL == net\_interface\)  

57     {  

58         perror\("pcap\_lookupdev"\);  

59         exit\(-1\);  

60     }  

61   

62     pcap\_handle = pcap\_open\_live\(net\_interface,BUFSIZE,1,0,error\_content\);//打开网络接口  

63           

64     if\(pcap\_loop\(pcap\_handle,-1,ethernet\_protocol\_callback,NULL\) &lt; 0\)  

65     {  

66         perror\("pcap\_loop"\);  

67     }  

68       

69     pcap\_close\(pcap\_handle\);  

70     return 0;  

71 }  

复制代码

 



过滤数据包：

设置过滤条件：举一些例子：

src host 192.168.1.177：只接收源ip地址是192.168.1.177的数据包

dst port 80：只接收tcp、udp的目的端口是80的数据包

not tcp：只接收不使用tcp协议的数据包

tcp\[13\] == 0x02 and \(dst port 22 or dst port 23\) ：只接收 SYN 标志位置位且目标端口是 22 或 23 的数据包（ tcp 首部开始的第 13 个字节）

icmp\[icmptype\] == icmp-echoreply or icmp\[icmptype\] == icmp-echo：只接收 icmp 的 ping 请求和 ping 响应的数据包

ehter dst 00:e0:09:c1:0e:82：只接收以太网 mac 地址是 00:e0:09:c1:0e:82 的数据包

ip\[8\] == 5：只接收 ip 的 ttl=5 的数据包（ip首部开始的第8个字节）

编译BPF过滤规则：

int pcap\_compile\(pcap\_t \*p,struct bpf\_program \*fp,char \*buf,int optimize,bpf\_u\_int32 mask\);

参数：

p：pcap\_open\_live\(\)返回的pcap\_t类型的指针

fp：存放编译后的bpf，应用过来规则时需要使用这个指针

buf：过滤规则

optimize：是否需要优化过滤表达式

mask：指定本地网络的网络掩码，不需要时可写0

返回值：成功返回0，失败返回-1

应用BPF过滤规则：

int pcap\_setfilter\(pcap\_t \*p,struct bpf\_program \*fp\);

功能：应用BPF过滤规则

参数：p：pcap\_open\_live\(\)返回的pcap\_t类型的指针

        fp：pcap\_compile\(\)的第二个参数

返回值：成功返回0，失败返回-1

复制代码

 1 \#include &lt;pcap.h&gt;  

 2 \#include &lt;time.h&gt;  

 3 \#include &lt;stdlib.h&gt;  

 4 \#include &lt;stdio.h&gt;  

 5   

 6 void getPacket\(u\_char \* arg, const struct pcap\_pkthdr \* pkthdr, const u\_char \* packet\)  

 7 {  

 8   int \* id = \(int \*\)arg;  

 9     

10   printf\("id: %d\n", ++\(\*id\)\);  

11   printf\("Packet length: %d\n", pkthdr-&gt;len\);  

12   printf\("Number of bytes: %d\n", pkthdr-&gt;caplen\);  

13   printf\("Recieved time: %s", ctime\(\(const time\_t \*\)&pkthdr-&gt;ts.tv\_sec\)\);   

14     

15   int i;  

16   for\(i=0; i&lt;pkthdr-&gt;len; ++i\)  

17   {  

18     printf\(" %02x", packet\[i\]\);  

19     if\( \(i + 1\) % 16 == 0 \)  

20     {  

21       printf\("\n"\);  

22     }  

23   }  

24     

25   printf\("\n\n"\);  

26 }  

27   

28 int main\(\)  

29 {  

30   char errBuf\[PCAP\_ERRBUF\_SIZE\], \* devStr;  

31     

32   /\* get a device \*/  

33   devStr = pcap\_lookupdev\(errBuf\);  

34     

35   if\(devStr\)  

36   {  

37     printf\("success: device: %s\n", devStr\);  

38   }  

39   else  

40   {  

41     printf\("error: %s\n", errBuf\);  

42     exit\(1\);  

43   }  

44     

45   /\* open a device, wait until a packet arrives \*/  

46   pcap\_t \* device = pcap\_open\_live\(devStr, 65535, 1, 0, errBuf\);  

47     

48   if\(!device\)  

49   {  

50     printf\("error: pcap\_open\_live\(\): %s\n", errBuf\);  

51     exit\(1\);  

52   }  

53     

54   /\* construct a filter \*/  

55   struct bpf\_program filter;  

56   pcap\_compile\(device, &filter, "dst port 80", 1, 0\);  

57   pcap\_setfilter\(device, &filter\);  

58     

59   /\* wait loop forever \*/  

60   int id = 0;  

61   pcap\_loop\(device, -1, getPacket, \(u\_char\*\)&id\);  

62     

63   pcap\_close\(device\);  

64   

65   return 0;  

66 } 

复制代码



