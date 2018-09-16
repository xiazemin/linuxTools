首先我们需要定义我们需要使用的网络接口。在linux下，我们一般会定义eth0或ethx。在BSD下，可能是xl1。我们可以把网络接口定义为字符串，或者可以通过pcap获得可用的网络接口的名字。

初始化pcap。现在，我们可以将我们将要监听的网络设备告诉pcap。如果有需要的话，我们可以使pcap同时监听多个网络接口。我们可以通过“文件句柄”来区分不同的网络接口，就像我们打开文件进行文件的读取、写入一样，我们必须定义区分我们的监听“回话”，否则我们没有办法区分不同的监听对象\(网络设备\)。

如果我们仅仅想监听特殊的网络数据（例如，我们想监听TCP业务，或者我们只想监听端口号为23的业务）。我们可以自己定义一个监听规则的集合，“编译”它，然后在应用它。上面三个步骤，连接的十分紧密，那一个步骤都不能丢掉。规则其实就是定义好的字符串，我们需要将其转化为pcap可以是别的格式（所以我们需要编译）。“编译器”仅仅通过内置的函数就可以实现上述的格式转换。然后我们可以告诉pcap执行规则完成数据包的过滤。

之后，我们会告诉pcap进入主要的循环执行状态。在该状态下，pcap 会一直运行截获到我们需要的网络数据包的数量为止。每次pcap抓取到我们需要的数据包后就会调用我们事先定义好的回调函数完成后续的数据包的处理工作。该回调函数为将数据包存储到一个文件中，亦或是打印到屏幕上，或者是你想要的任何事情，她都能够办到。

当我们完成数据检测后，我们需要关闭本次事务。

通过上面5个步骤（其中步骤3是可选的），我们就可以完成数据包的截获和后续的数据包的处理，是不是很简单啊？下面我们会详细介绍每一步是如何实现的。

设置设备

设备的设置十分的简单，存在两种方式可以完成网络设备的设置，即：通过传递参数完成设定；通过pcap提供的函数完成设备的检测、设定。

首先，我们可以通过程序启动时，传递参数的方式来实现网络设备的设置。代码示例如下：

\[html\] view plain copy

\#include &lt;stdio.h&gt;  

\#include &lt;pcap.h&gt;  

  

int main\(int argc, char \*argv\[\]\)  

{  

     char \*dev = argv\[1\];  

  

     printf\("Device: %s\n", dev\);  

     return\(0\);  

}  

   2. 第二种方法就是，通过pcap库中提供的内部函数，完成网络设备的检测、获得。代码示例如下：  

  

\#include &lt;stdio.h&gt;  

\#include &lt;pcap.h&gt;  

int main\(int argc, char \*argv\[\]\)  

{  

    char \*dev, errbuf\[PCAP\_ERRBUF\_SIZE\];  

    dev = pcap\_lookupdev\(errbuf\);  

    if \(dev == NULL\) {  

        fprintf\(stderr, "Couldn't find default device: %s\n", errbuf\);  

        return\(2\);  

    }  

    printf\("Device: %s\n", dev\);  

    return\(0\);  

  

}   

通过上面的函数pcap\_lookupdev\(\)就可以实现设备的自动获取。其中参数，errbuf为了记录函数调用失败时的原因。可以通过man手册参看其使用方法。

打开设备，开始数据包的监测

可以通过函数pcap\_open\_live\(\)打开需要监测的网络设备。函数原型以及使用方式如下：



\[cpp\] view plain copy

pcap\_t \*pcap\_open\_live\(char \*device, int snaplen, int promisc, int to\_ms,char \*ebuf\)  



参数：

device:前面定义的网络设备名称。

snaplen:一个整型数据，用来定义pcap抓取的数据的字节数。

promise:如果该参数为真，表示王珂会被设置为混杂模式（如果参数为假，在某些情况下，网卡还是会被设置为混杂模式）。

to\_ms:读取数据包的超时时间，单位为ms\(0表示没有超时，一般不为0\)。

ebuf:记录错误信息。

函数返回会话的句柄。下面是该函数的使用示例：

\[cpp\] view plain copy

 \#include &lt;pcap.h&gt;  

     ...  

pcap\_t \*handle;  

handle = pcap\_open\_live\(somedev, BUFSIZ, 1, 1000, errbuf\);  

if \(handle == NULL  

{  

         fprintf\(stderr, "Couldn't open device %s: %s\n", somedev, errbuf\);  

         return\(2\);  

}  

其中，BUFSIZE 被定义在 pcap.h中，程序会一直运行直到错误产生，错误信息会记录在errbuf中。



Note：

简单介绍一下，混杂模式和非混杂模式。

非混杂模式：

           非混杂模式下，主机只会检测与其相关的数据。这些数据包括：源、目的为本主机，或者经过本机的路由数据包。

混杂模式：

           混杂模式下，主机会检测线路上所有的数据包。在没有交换机的网络中，检测所有的数据包。这种模式的好处是，可以检测的数据会增加。利弊主要取决于你的目的。但是，混杂模式是可以探测的。一个主机可以通过可靠地手段检测到另一个主机的网卡是否被设置为混杂模式。再一个，混杂模式只会工作在没有交换机的网络，例如通过hub连接的，或者交换机被ARP淹没。混杂模式会导致，在网络流量巨大的时候，增加系统的负担。

过滤数据包

一般情况下，我们只会检测我们感兴趣的数据包。例如，我们会监听23\(telnet\)端口来搜索密码，监听21端口截获一个文件，或者仅仅为了监听DNS数据\(端口53\)。我们一般不会盲目监听网络上的数据包。



通过，pcap\_compile\(\)和pcap\_setfilter\(\)就可以实现对特定数据包的监听工作。



当我们创建了一个监听事务后，下一步我们就可以设置过滤的规则了。之所以使用pcap内置的过滤机制，而不是传统的if/else if 模式实现数据包的过滤主要考虑到两个原因：



pcap的数据包的过滤机制十分的高效，其直接通关BPF实现，我们省去了很多步骤。

使用pcap十分的简单。

在应用过滤规则之前，我们需要”编译“它，过滤规则一般被存储在一个字符数组中，存储格式我们可以通过查看man tcpdump的规则。pcap\_compile\(\)和pcap\_setfilter\(\)函数的原型和使用方式如下：

pcap\_compile\(\):

\[cpp\] view plain copy

int pcap\_compile\(pcap\_t \*p, struct bpf\_program \*fp, char \*str, int optimize, bpf\_u\_int32 netmask\)  

参数： 

pt:会话句柄。

fp:表示编译过的过滤规则存储的位置。

str:字符串格式的过滤规则。

optimize:表示过滤规则是否需要的优化\(1:need,0:no\)

netmask:表示过滤应用的网络的子网掩码.

         如果该函数运行成功，返回一个非-1的整数。

      2.  pcap\_setfilter\(\):

\[cpp\] view plain copy

int pcap\_setfilter\(pcap\_t \*p, struct bpf\_program \*fp\)  

           参数：

p:会话句柄。

fp:表示编译过的过滤规则存储的位置。

下面是两个函数使用的示例：

\[cpp\] view plain copy

\#include &lt;pcap.h&gt;  

     ...  

     pcap\_t \*handle;        /\* Session handle \*/  

     char dev\[\] = "eth0";       /\* Device to sniff on \*/  

     char errbuf\[PCAP\_ERRBUF\_SIZE\]; /\* Error string \*/  

     struct bpf\_program fp;     /\* The compiled filter expression \*/  

     char filter\_exp\[\] = "port 23"; /\* The filter expression \*/  

     bpf\_u\_int32 mask;      /\* The netmask of our sniffing device \*/  

     bpf\_u\_int32 net;       /\* The IP of our sniffing device \*/  

  

     if \(pcap\_lookupnet\(dev, &net, &mask, errbuf\) == -1\) {  

         fprintf\(stderr, "Can't get netmask for device %s\n", dev\);  

         net = 0;  

         mask = 0;  

     }  

     handle = pcap\_open\_live\(dev, BUFSIZ, 1, 1000, errbuf\);  

     if \(handle == NULL\) {  

         fprintf\(stderr, "Couldn't open device %s: %s\n", somedev, errbuf\);  

         return\(2\);  

     }  

     if \(pcap\_compile\(handle, &fp, filter\_exp, 0, net\) == -1\) {  

         fprintf\(stderr, "Couldn't parse filter %s: %s\n", filter\_exp, pcap\_geterr\(handle\)\);  

         return\(2\);  

     }  

     if \(pcap\_setfilter\(handle, &fp\) == -1\) {  

         fprintf\(stderr, "Couldn't install filter %s: %s\n", filter\_exp, pcap\_geterr\(handle\)\);  

         return\(2\);  

     }  

上面表示，eth0工作在混杂模式，检测端口号为23的数据包。

上面存在一个没有介绍过的函数pcap\_lookupnet\(\)，该函数的作用是通过dev获得其对应的IPV4 网络号和其对应的网络的子网掩码。后面的杂项中介绍该函数。

开始监测数据包

经过了上面一系列的准备工作，现在开始正的抓包。通常可以通过两张方法实现数据的抓取：一次只抓取一个数据包；每次抓取n个数据包。下面我们分别介绍一下，这两种方法。

single

利用函数pcap\_next\(\)，我们可以实现每次只抓取一个数据包，该函数的原型和使用方法如下：



\[cpp\] view plain copy

u\_char \*pcap\_next\(pcap\_t \*p, struct pcap\_pkthdr \*h\)  



参数：

p:会话句柄。

h:指向存储关于数据包的一些主要信息的指针，这些信息包括:数据包抓取的时间、数据包的长度、数据包一些特殊部分的长度。

该函数的返回值指向抓取的数据包，下面是该函数使用的示例。

\[cpp\] view plain copy

\#include &lt;pcap.h&gt;  

\#include &lt;stdio.h&gt;  

  

 int main\(int argc, char \*argv\[\]\)  

{  

        pcap\_t \*handle;         /\* Session handle \*/  

        char \*dev;          /\* The device to sniff on \*/  

        char errbuf\[PCAP\_ERRBUF\_SIZE\];  /\* Error string \*/  

        struct bpf\_program fp;      /\* The compiled filter \*/  

        char filter\_exp\[\] = "port 23";  /\* The filter expression \*/  

        bpf\_u\_int32 mask;       /\* Our netmask \*/  

        bpf\_u\_int32 net;        /\* Our IP \*/  

        struct pcap\_pkthdr header;  /\* The header that pcap gives us \*/  

        const u\_char \*packet;       /\* The actual packet \*/  

  

        /\* Define the device \*/  

        dev = pcap\_lookupdev\(errbuf\);  

        if \(dev == NULL\) {  

            fprintf\(stderr, "Couldn't find default device: %s\n", errbuf\);  

            return\(2\);  

        }  

        /\* Find the properties for the device \*/  

        if \(pcap\_lookupnet\(dev, &net, &mask, errbuf\) == -1\) {  

            fprintf\(stderr, "Couldn't get netmask for device %s: %s\n", dev, errbuf\);  

            net = 0;  

            mask = 0;  

        }  

        /\* Open the session in promiscuous mode \*/  

        handle = pcap\_open\_live\(dev, BUFSIZ, 1, 1000, errbuf\);  

        if \(handle == NULL\) {  

            fprintf\(stderr, "Couldn't open device %s: %s\n", somedev, errbuf\);  

            return\(2\);  

        }  

        /\* Compile and apply the filter \*/  

        if \(pcap\_compile\(handle, &fp, filter\_exp, 0, net\) == -1\) {  

            fprintf\(stderr, "Couldn't parse filter %s: %s\n", filter\_exp, pcap\_geterr\(handle\)\);  

            return\(2\);  

        }  

        if \(pcap\_setfilter\(handle, &fp\) == -1\) {  

            fprintf\(stderr, "Couldn't install filter %s: %s\n", filter\_exp, pcap\_geterr\(handle\)\);  

            return\(2\);  

        }  

        /\* Grab a packet \*/  

        packet = pcap\_next\(handle, &header\);  

        /\* Print its length \*/  

        printf\("Jacked a packet with length of \[%d\]\n", header.len\);  

        /\* And close the session \*/  

        pcap\_close\(handle\);  

        return\(0\);  

}  



上面的应用利用pcap\_lookupdev\(\)得到了一个可用的设备，然后开始在端口23\(telnet\)开始监测数据包，当获得一个数据包后，就将该数据包以字节的形式告诉用户。



multiple



另一种技术相对比较复杂，其实pcap\_next\(\)其实一般很少使用，更多的我们会使用pcap\_loop\(\)和pcap\_dispatch\(\)函数，上述两个函数的实现用到了回调函数机制。我们需要实现定义好的自己的回调函数，然后注册到pcap\_next\(\)或者pcap\_dispatch\(\)中，两个函数的使用方式和工作方式基本相同，每当检测到一个符合过滤规则的数据包，回调函数就会调用\(当然，如果没有定义过滤规则，每个数据包都会调用其对应的回调函数\)。pcap\_look\(\)函数的原型和函数的参数如下:

\[cpp\] view plain copy

int pcap\_loop\(pcap\_t \*p, int cnt, pcap\_handler callback, u\_char \*user\)  



 参数：



p:检测会话句柄；

cnt:表示结束监听之前需要截获的数据包的数量\(负数表示一直监听知道出错\)。

callback:我们定义的回调函数的名字。

user:表示传递给回调函数的参数

pcap\_dispatch\(\)的使用方法几乎和pcap\_look\(\)一样，多不同是pcap\_dispatch\(\)只会读取"一些"数据包，之后就会退出循环。可以通过man手册获得更详细的关于pcap\_look\(\)和pcap\_dispatch\(\)的区别。接下来我么有必要介绍一些回调函数的格式，回调函数不能随便定义，否则pcap\_dispatch\(\)和pcap\_look\(\)会不知道该如何调用回调函数的。回调函数的原型和参数如下：



\[cpp\] view plain copy

void got\_packet\(u\_char \*args, const struct pcap\_pkthdr \*header, const u\_char \*packet\);  



参数：



args:该参数就是我们在pcap\_loop\(\)和pcap\_dispatch\(\)最后的用户传递的参数，每当回调函数被调用时该参数都将被传递一次。

header:保存数据包截获的具体时间和数据包大小等信息。

packet:指向数据包的第一个字节\(数据包已经被序列化\)

pcap\_pkthdr\(\)结构体的具体格式如下：



 



\[cpp\] view plain copy

struct pcap\_pkthdr {  

        struct timeval ts; /\* time stamp \*/  

        bpf\_u\_int32 caplen; /\* length of portion present \*/  

        bpf\_u\_int32 len; /\* length this packet \(off wire\) \*/  

    };  



在开始读取数据包中内同时我们首先需要对数据包就行反序列化，所以我们有必要了解一下TCP/IP下各个字段的含义，下面就对TCP/IP协议中各个数据包头进行简单的解释。假设TCP/IP的链路层协议为Ethernet：



 



\[cpp\] view plain copy

/\* Ethernet addresses are 6 bytes \*/  

\#define ETHER\_ADDR\_LEN  6  

  

    /\* Ethernet header \*/  

    struct sniff\_ethernet {  

        u\_char ether\_dhost\[ETHER\_ADDR\_LEN\]; /\* Destination host address \*/  

        u\_char ether\_shost\[ETHER\_ADDR\_LEN\]; /\* Source host address \*/  

        u\_short ether\_type; /\* IP? ARP? RARP? etc \*/  

    };  

  

    /\* IP header \*/  

    struct sniff\_ip {  

        u\_char ip\_vhl;      /\* version &lt;&lt; 4 \| header length &gt;&gt; 2 \*/  

        u\_char ip\_tos;      /\* type of service \*/  

        u\_short ip\_len;     /\* total length \*/  

        u\_short ip\_id;      /\* identification \*/  

        u\_short ip\_off;     /\* fragment offset field \*/  

    \#define IP\_RF 0x8000        /\* reserved fragment flag \*/  

    \#define IP\_DF 0x4000        /\* dont fragment flag \*/  

    \#define IP\_MF 0x2000        /\* more fragments flag \*/  

    \#define IP\_OFFMASK 0x1fff   /\* mask for fragmenting bits \*/  

        u\_char ip\_ttl;      /\* time to live \*/  

        u\_char ip\_p;        /\* protocol \*/  

        u\_short ip\_sum;     /\* checksum \*/  

        struct in\_addr ip\_src,ip\_dst; /\* source and dest address \*/  

    };  

    \#define IP\_HL\(ip\)       \(\(\(ip\)-&gt;ip\_vhl\) & 0x0f\)  

    \#define IP\_V\(ip\)        \(\(\(ip\)-&gt;ip\_vhl\) &gt;&gt; 4\)  

  

    /\* TCP header \*/  

    struct sniff\_tcp {  

        u\_short th\_sport;   /\* source port \*/  

        u\_short th\_dport;   /\* destination port \*/  

        tcp\_seq th\_seq;     /\* sequence number \*/  

        tcp\_seq th\_ack;     /\* acknowledgement number \*/  

  

        u\_char th\_offx2;    /\* data offset, rsvd \*/  

    \#define TH\_OFF\(th\)  \(\(\(th\)-&gt;th\_offx2 & 0xf0\) &gt;&gt; 4\)  

        u\_char th\_flags;  

    \#define TH\_FIN 0x01  

    \#define TH\_SYN 0x02  

    \#define TH\_RST 0x04  

    \#define TH\_PUSH 0x08  

    \#define TH\_ACK 0x10  

    \#define TH\_URG 0x20  

    \#define TH\_ECE 0x40  

    \#define TH\_CWR 0x80  

    \#define TH\_FLAGS \(TH\_FIN\|TH\_SYN\|TH\_RST\|TH\_ACK\|TH\_URG\|TH\_ECE\|TH\_CWR\)  

        u\_short th\_win;     /\* window \*/  

        u\_short th\_sum;     /\* checksum \*/  

        u\_short th\_urp;     /\* urgent pointer \*/  

};  



现在，开始反序列化传递给回调函数的数据包，假设TCP/IP协议运行在Ethernet上。



 



\[cpp\] view plain copy

/\* ethernet headers are always exactly 14 bytes \*/  

\#define SIZE\_ETHERNET 14  

  

    const struct sniff\_ethernet \*ethernet; /\* The ethernet header \*/  

    const struct sniff\_ip \*ip; /\* The IP header \*/  

    const struct sniff\_tcp \*tcp; /\* The TCP header \*/  

    const char \*payload; /\* Packet payload \*/  

  

    u\_int size\_ip;  

    u\_int size\_tcp;  

And now we do our magical typecasting:   

    ethernet = \(struct sniff\_ethernet\*\)\(packet\);  

    ip = \(struct sniff\_ip\*\)\(packet + SIZE\_ETHERNET\);  

    size\_ip = IP\_HL\(ip\)\*4;  

    if \(size\_ip &lt; 20\) {  

        printf\("   \* Invalid IP header length: %u bytes\n", size\_ip\);  

        return;  

    }  

    tcp = \(struct sniff\_tcp\*\)\(packet + SIZE\_ETHERNET + size\_ip\);  

    size\_tcp = TH\_OFF\(tcp\)\*4;  

    if \(size\_tcp &lt; 20\) {  

        printf\("   \* Invalid TCP header length: %u bytes\n", size\_tcp\);  

        return;  

    }  

    payload = \(u\_char \*\)\(packet + SIZE\_ETHERNET + size\_ip + size\_tcp\);  



我们解释一下，假设我们定义数据包的起始地址为X，则Ethernet头、IP、TCP/UDP头的起始地址分别为：



 



Variable



Location \(in bytes\)



sniff\_ethernet



X



sniff\_ip



X + SIZE\_ETHERNET



sniff\_tcp



X + SIZE\_ETHERNET + {IP header length}



payload



X + SIZE\_ETHERNET + {IP header length} + {TCP header  length}



 

