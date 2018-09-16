#### 1.1.1.1如何实现 {#1111-如何实现}

先来看看包传递过来的流程，如下图。包从网卡到内存，到内核态，最后给用户程序使用。我们知道tcpdump程序运行在用户态，那如何实现从内核态的抓包呢？

![](https://img-blog.csdn.net/2018032822340794)  


　　这个就是通过libpcap库来实现的，tcpdump调用libpcap的api函数，由libpcap进入到内核态到链路层来抓包,如下图。图中的BPF是过滤器，可以根据用户设置用于数据包过滤减少应用程序的数据包的包数和字节数从而提高性能。BufferQ是缓存供应用程序读取的数据包。我们可以说tcpdump底层原理其实就是libpcap的实现原理。

![](https://img-blog.csdn.net/20180328223424291)  


而libpcap在linux系统链路层中抓包是通过PF\_PACKET套接字来实现的\(不同的系统其实现机制是由差异的\)，该方法在创建的时候，可以指定第二参数为SOCK\_DGRAM或者SOCK\_RAW，影响是否扣除链路层的首部。

            libpcap在内核收发包的接口处将skb\_clone\(\)拿走的包.

关于内核中如何注册网络协议和钩子函数的过程，此处先不展开，后续专门讲解。我们接下去是看下libpcap的一些实现及其api.

#### 1.1.1.2libpcap {#1112-libpcap}

当在系统中输入tcpdump –version的时候，输出的其实还有libpcap，足见其在tcpdump中的地位。

            其实最早的编译系统和过滤引擎是在tcpdump项目中的，后来为了编译其他抓包的应用，将其独立出来。现在libpcap提供独立于平台的库和API，来满足执行网络嗅探。

tcpdump.c正式使用libpcap里的函数完成两个最关键的动作：获取捕获报文的接口，和捕获报文并将报文交给callback。

libpcap支持“伯克利包过滤（BPF）”语法。BPF能够通过比较第2、3、4层协议中各个数据字段值的方法对流量进行过滤。Libpcap的使用逻辑如下图：

![](https://img-blog.csdn.net/20180328223442376)  


如果愿意，大家也可以基于libpcap开发一个类似tcpdump的抓包工具。需要注意的是如果使用分组捕获设备，只能在单个接口上接收到达的分组。



#### 1.1.1.3核心函数 {#1113-核心函数}

我们先来看下libpcap中的一些核心函数，根据函数的功能，可以分为如下几类：

lÂ Â  为读包打开句柄

lÂ Â  为抓包选择链路层

lÂ Â  抓包函数

lÂ Â  过滤器

lÂ Â  选定抓包方向（进还是出）

lÂ Â  抓统计信息

lÂ Â  将包写入文件打开句柄

lÂ Â  写包

lÂ Â  注入包

lÂ Â  报告错误

lÂ Â  获取库版本信息

官方的介绍查看[http://www.tcpdump.org/manpages/pcap.3pcap.html](http://www.tcpdump.org/manpages/pcap.3pcap.html)

常用的一些函数如下：

pcap\_lookupdev，如果分组捕获设备未曾指定\(-i命令行选项\)，该函数选择一个设备。

pcap\_open\_offine打开一个保存的文件。

pcap\_setfilter设置过滤器

pcap\_open\_live打开选择的设备。

pcap\_next接收一个包

pcap\_dump将包写入到pcap\_dump\_t结构体

pcap\_loopupnet返回分组捕获设备的网络地址和子网掩码，然后在调用pcap\_compile时必须指定这个子网掩码。

pcap\_compile把cmd字符数组中构造的过滤器字符串编译成一个过滤器程序，存放在fcode中。

pcap\_setfilter把编译出来的过滤器程序装载到分组捕获设备，同时引发用该过滤器选取的分组的捕获。

pcap\_datalink返回分组捕获设备的数据链路类型。

等等，那么如何去使用libpcap库呢，一起来看下。

#### 1.1.1.4使用准备 {#1114-使用准备}

先在系统中安装pcap-dev包（apt-get install pcap-dev），然后创建一个test.c文件如下：

\#include &lt;stdio.h&gt;

\#include &lt;pcap.h&gt;



int

main \(int argc, char \*argv\[\]\)

{

  char \*dev, errbuf\[PCAP\_ERRBUF\_SIZE\];



  dev = pcap\_lookupdev \(errbuf\);

  if \(dev == NULL\)

    {

      fprintf \(stderr, “Couldn’t find default device: %s\n”, errbuf\);

      return \(2\);

    }

  printf \(“Device: %s\n”, dev\);

  return \(0\);

}

然后编译如下：

gcc test.c -lpcap -lpthread

            就可以执行了，在系统中寻找一个可以抓包的接口。

            有了接口设备，可以继续创建嗅探会话了，使用函数

pcap\_t \*pcap\_open\_live\(char \*device, int snaplen, int promisc, int to\_ms,        char \*ebuf\)

其中snaplen是pcap抓包的字节数, promisc 是否启用混杂模式（不是混杂模式的话就只抓给本机的包。），to\_ms是否超时，ebuf存放错误信息。

            创建了嗅探会话之后，就要一个过滤器。可以只提取我们想要的数据。过滤器在应用之前必须要先编译，调用函数如下：

int pcap\_compile\(pcap\_t \*p, struct bpf\_program \*fp, char \*str, int optimize,            bpf\_u\_int32 netmask\)

　　第一个参数就是pcap\_open\_live返回的值，fp 存储的过滤器的版本，optimize是表示是否需要优化，最后netmask是过滤器使用的所在子网掩码。

            有了过滤器之后就是要使用编译器，调用函数：

int pcap\_setfilter\(pcap\_t \*p, struct bpf\_program \*fp\)

            到此整个代码流程参考如下代码段：

        \#include &lt;pcap.h&gt;

        …

        pcap\_t \*handle;             /\* Session handle \*/

        char dev\[\] = “rl0”;         /\* Device to sniff on \*/

        char errbuf\[PCAP\_ERRBUF\_SIZE\];    /\* Error string \*/

        struct bpf\_program fp;            /\* The compiled filter expression \*/

        char filter\_exp\[\] = “port 23”;    /\* The filter expression \*/

        bpf\_u\_int32 mask;          /\* The netmask of our sniffing device \*/

        bpf\_u\_int32 net;            /\* The IP of our sniffing device \*/



        if \(pcap\_lookupnet\(dev, &net, &mask, errbuf\) == -1\) {

               fprintf\(stderr, “Can’t get netmask for device %s\n”, dev\);

               net = 0;

               mask = 0;

        }

        handle = pcap\_open\_live\(dev, BUFSIZ, 1, 1000, errbuf\);

        if \(handle == NULL\) {

               fprintf\(stderr, “Couldn’t open device %s: %s\n”, dev, errbuf\);

               return\(2\);

        }

        if \(pcap\_compile\(handle, &fp, filter\_exp, 0, net\) == -1\) {

               fprintf\(stderr, “Couldn’t parse filter %s: %s\n”, filter\_exp, pcap\_geterr\(handle\)\);

               return\(2\);

        }

        if \(pcap\_setfilter\(handle, &fp\) == -1\) {

               fprintf\(stderr, “Couldn’t install filter %s: %s\n”, filter\_exp, pcap\_geterr\(handle\)\);

               return\(2\);

        }

#### 1.1.1.5开始抓包 {#1115-开始抓包}

已经准备好监听抓包，并设置了过滤器，下面就是启动抓包了。

抓包技术有两种，一种是一次抓一个包；另一种是等待有n个包的时候在一起抓。

            先看抓一次抓一个包，使用函数如下：

u\_char \*pcap\_next\(pcap\_t \*p, struct pcap\_pkthdr \*h\)

            第一个参数就是创建的会话句柄，第二个参数是存放包信息的。

            这个函数是比较少用的，现在大多数抓包工具都是使用第二种技术抓包的，其用到的函数就是：

int pcap\_loop\(pcap\_t \*p, int cnt, pcap\_handler callback, u\_char \*user\)

            第一个参数是创建的会话句柄，第二个参数是数量（抓几个包），就是这个参数制定抓多少包，抓完就结束了，第三个函数是抓到足够数量后的回调函数，每次抓到都会调用回调函数，第四个参数经常设置为NULL,在一些应用中会有用。

            和pcap\_loop函数类似的是pcap\_dispatch,两者用法基本一致，主要差异是pcap\_dispatch只会执行一次回调函数，而pcap\_loop会一直调用回调函数处理包。

            其回调函数的定义如下：

        void got\_packet\(u\_char \*args, const struct pcap\_pkthdr \*header,          const u\_char \*packet\);

            是void型的，第一个参数args是pcap\_loop函数的最后一个参数，第二个参数是pcap的头其包含了抓住的包的信息，第三个就是包本身了。

       struct pcap\_pkthdr {

              struct timeval ts; /\* time stamp \*/

              bpf\_u\_int32 caplen; /\* length of portion present \*/

              bpf\_u\_int32 len; /\* length this packet \(off wire\) \*/

       };

            关于包本身其实是一个字符串指针，怎么去寻找我的ip头，tcp头，以及头中的内容呢？这就需要是使用C语言中异常强大的指针了，定义一个宏如下：

/\* Ethernet addresses are 6 bytes \*/

\#define ETHER\_ADDR\_LEN      6



       /\* Ethernet header \*/

       struct sniff\_ethernet {

              u\_char ether\_dhost\[ETHER\_ADDR\_LEN\]; /\* Destination host address \*/

              u\_char ether\_shost\[ETHER\_ADDR\_LEN\]; /\* Source host address \*/

              u\_short ether\_type; /\* IP? ARP? RARP? etc \*/

       };



       /\* IP header \*/

       struct sniff\_ip {

              u\_char ip\_vhl;              /\* version &lt;&lt; 4 \| header length &gt;&gt; 2 \*/

              u\_char ip\_tos;              /\* type of service \*/

              u\_short ip\_len;             /\* total length \*/

              u\_short ip\_id;              /\* identification \*/

              u\_short ip\_off;             /\* fragment offset field \*/

       \#define IP\_RF 0x8000        /\* reserved fragment flag \*/

       \#define IP\_DF 0x4000        /\* dont fragment flag \*/

       \#define IP\_MF 0x2000        /\* more fragments flag \*/

       \#define IP\_OFFMASK 0x1fff   /\* mask for fragmenting bits \*/

              u\_char ip\_ttl;               /\* time to live \*/

              u\_char ip\_p;         /\* protocol \*/

              u\_short ip\_sum;             /\* checksum \*/

              struct in\_addr ip\_src,ip\_dst; /\* source and dest address \*/

       };

       \#define IP\_HL\(ip\)           \(\(\(ip\)-&gt;ip\_vhl\) & 0x0f\)

       \#define IP\_V\(ip\)            \(\(\(ip\)-&gt;ip\_vhl\) &gt;&gt; 4\)



       /\* TCP header \*/

       typedef u\_int tcp\_seq;



       struct sniff\_tcp {

              u\_short th\_sport;    /\* source port \*/

              u\_short th\_dport;    /\* destination port \*/

              tcp\_seq th\_seq;              /\* sequence number \*/

              tcp\_seq th\_ack;              /\* acknowledgement number \*/

              u\_char th\_offx2;     /\* data offset, rsvd \*/

       \#define TH\_OFF\(th\)   \(\(\(th\)-&gt;th\_offx2 & 0xf0\) &gt;&gt; 4\)

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

              u\_short th\_win;              /\* window \*/

              u\_short th\_sum;             /\* checksum \*/

              u\_short th\_urp;             /\* urgent pointer \*/

};

/\* ethernet headers are always exactly 14 bytes \*/

\#define SIZE\_ETHERNET 14



       const struct sniff\_ethernet \*ethernet; /\* The ethernet header \*/

       const struct sniff\_ip \*ip; /\* The IP header \*/

       const struct sniff\_tcp \*tcp; /\* The TCP header \*/

       const char \*payload; /\* Packet payload \*/



       u\_int size\_ip;

       u\_int size\_tcp;

            通过以上结构体定义，可以从回调函数的包指针地址出发，逐个找到链路帧头、IP帧头、TCP帧头、数据负载了1.1.1.1如何实现

先来看看包传递过来的流程，如下图。包从网卡到内存，到内核态，最后给用户程序使用。我们知道tcpdump程序运行在用户态，那如何实现从内核态的抓包呢？

![](https://img-blog.csdn.net/2018032822340794)  


　　这个就是通过libpcap库来实现的，tcpdump调用libpcap的api函数，由libpcap进入到内核态到链路层来抓包,如下图。图中的BPF是过滤器，可以根据用户设置用于数据包过滤减少应用程序的数据包的包数和字节数从而提高性能。BufferQ是缓存供应用程序读取的数据包。我们可以说tcpdump底层原理其实就是libpcap的实现原理。

![](https://img-blog.csdn.net/20180328223424291)  


而libpcap在linux系统链路层中抓包是通过PF\_PACKET套接字来实现的\(不同的系统其实现机制是由差异的\)，该方法在创建的时候，可以指定第二参数为SOCK\_DGRAM或者SOCK\_RAW，影响是否扣除链路层的首部。

            libpcap在内核收发包的接口处将skb\_clone\(\)拿走的包.

关于内核中如何注册网络协议和钩子函数的过程，此处先不展开，后续专门讲解。我们接下去是看下libpcap的一些实现及其api.

#### 1.1.1.2libpcap {#1112-libpcap}

当在系统中输入tcpdump –version的时候，输出的其实还有libpcap，足见其在tcpdump中的地位。

            其实最早的编译系统和过滤引擎是在tcpdump项目中的，后来为了编译其他抓包的应用，将其独立出来。现在libpcap提供独立于平台的库和API，来满足执行网络嗅探。

tcpdump.c正式使用libpcap里的函数完成两个最关键的动作：获取捕获报文的接口，和捕获报文并将报文交给callback。

libpcap支持“伯克利包过滤（BPF）”语法。BPF能够通过比较第2、3、4层协议中各个数据字段值的方法对流量进行过滤。Libpcap的使用逻辑如下图：

![](https://img-blog.csdn.net/20180328223442376)  


如果愿意，大家也可以基于libpcap开发一个类似tcpdump的抓包工具。需要注意的是如果使用分组捕获设备，只能在单个接口上接收到达的分组。



#### 1.1.1.3核心函数 {#1113-核心函数}

我们先来看下libpcap中的一些核心函数，根据函数的功能，可以分为如下几类：

lÂ Â  为读包打开句柄

lÂ Â  为抓包选择链路层

lÂ Â  抓包函数

lÂ Â  过滤器

lÂ Â  选定抓包方向（进还是出）

lÂ Â  抓统计信息

lÂ Â  将包写入文件打开句柄

lÂ Â  写包

lÂ Â  注入包

lÂ Â  报告错误

lÂ Â  获取库版本信息

官方的介绍查看[http://www.tcpdump.org/manpages/pcap.3pcap.html](http://www.tcpdump.org/manpages/pcap.3pcap.html)

常用的一些函数如下：

pcap\_lookupdev，如果分组捕获设备未曾指定\(-i命令行选项\)，该函数选择一个设备。

pcap\_open\_offine打开一个保存的文件。

pcap\_setfilter设置过滤器

pcap\_open\_live打开选择的设备。

pcap\_next接收一个包

pcap\_dump将包写入到pcap\_dump\_t结构体

pcap\_loopupnet返回分组捕获设备的网络地址和子网掩码，然后在调用pcap\_compile时必须指定这个子网掩码。

pcap\_compile把cmd字符数组中构造的过滤器字符串编译成一个过滤器程序，存放在fcode中。

pcap\_setfilter把编译出来的过滤器程序装载到分组捕获设备，同时引发用该过滤器选取的分组的捕获。

pcap\_datalink返回分组捕获设备的数据链路类型。

等等，那么如何去使用libpcap库呢，一起来看下。

#### 1.1.1.4使用准备 {#1114-使用准备}

先在系统中安装pcap-dev包（apt-get install pcap-dev），然后创建一个test.c文件如下：

\#include &lt;stdio.h&gt;

\#include &lt;pcap.h&gt;



int

main \(int argc, char \*argv\[\]\)

{

  char \*dev, errbuf\[PCAP\_ERRBUF\_SIZE\];



  dev = pcap\_lookupdev \(errbuf\);

  if \(dev == NULL\)

    {

      fprintf \(stderr, “Couldn’t find default device: %s\n”, errbuf\);

      return \(2\);

    }

  printf \(“Device: %s\n”, dev\);

  return \(0\);

}

然后编译如下：

gcc test.c -lpcap -lpthread

            就可以执行了，在系统中寻找一个可以抓包的接口。

            有了接口设备，可以继续创建嗅探会话了，使用函数

pcap\_t \*pcap\_open\_live\(char \*device, int snaplen, int promisc, int to\_ms,        char \*ebuf\)

其中snaplen是pcap抓包的字节数, promisc 是否启用混杂模式（不是混杂模式的话就只抓给本机的包。），to\_ms是否超时，ebuf存放错误信息。

            创建了嗅探会话之后，就要一个过滤器。可以只提取我们想要的数据。过滤器在应用之前必须要先编译，调用函数如下：

int pcap\_compile\(pcap\_t \*p, struct bpf\_program \*fp, char \*str, int optimize,            bpf\_u\_int32 netmask\)

　　第一个参数就是pcap\_open\_live返回的值，fp 存储的过滤器的版本，optimize是表示是否需要优化，最后netmask是过滤器使用的所在子网掩码。

            有了过滤器之后就是要使用编译器，调用函数：

int pcap\_setfilter\(pcap\_t \*p, struct bpf\_program \*fp\)

            到此整个代码流程参考如下代码段：

        \#include &lt;pcap.h&gt;

        …

        pcap\_t \*handle;             /\* Session handle \*/

        char dev\[\] = “rl0”;         /\* Device to sniff on \*/

        char errbuf\[PCAP\_ERRBUF\_SIZE\];    /\* Error string \*/

        struct bpf\_program fp;            /\* The compiled filter expression \*/

        char filter\_exp\[\] = “port 23”;    /\* The filter expression \*/

        bpf\_u\_int32 mask;          /\* The netmask of our sniffing device \*/

        bpf\_u\_int32 net;            /\* The IP of our sniffing device \*/



        if \(pcap\_lookupnet\(dev, &net, &mask, errbuf\) == -1\) {

               fprintf\(stderr, “Can’t get netmask for device %s\n”, dev\);

               net = 0;

               mask = 0;

        }

        handle = pcap\_open\_live\(dev, BUFSIZ, 1, 1000, errbuf\);

        if \(handle == NULL\) {

               fprintf\(stderr, “Couldn’t open device %s: %s\n”, dev, errbuf\);

               return\(2\);

        }

        if \(pcap\_compile\(handle, &fp, filter\_exp, 0, net\) == -1\) {

               fprintf\(stderr, “Couldn’t parse filter %s: %s\n”, filter\_exp, pcap\_geterr\(handle\)\);

               return\(2\);

        }

        if \(pcap\_setfilter\(handle, &fp\) == -1\) {

               fprintf\(stderr, “Couldn’t install filter %s: %s\n”, filter\_exp, pcap\_geterr\(handle\)\);

               return\(2\);

        }

#### 1.1.1.5开始抓包 {#1115-开始抓包}

已经准备好监听抓包，并设置了过滤器，下面就是启动抓包了。

抓包技术有两种，一种是一次抓一个包；另一种是等待有n个包的时候在一起抓。

            先看抓一次抓一个包，使用函数如下：

u\_char \*pcap\_next\(pcap\_t \*p, struct pcap\_pkthdr \*h\)

            第一个参数就是创建的会话句柄，第二个参数是存放包信息的。

            这个函数是比较少用的，现在大多数抓包工具都是使用第二种技术抓包的，其用到的函数就是：

int pcap\_loop\(pcap\_t \*p, int cnt, pcap\_handler callback, u\_char \*user\)

            第一个参数是创建的会话句柄，第二个参数是数量（抓几个包），就是这个参数制定抓多少包，抓完就结束了，第三个函数是抓到足够数量后的回调函数，每次抓到都会调用回调函数，第四个参数经常设置为NULL,在一些应用中会有用。

            和pcap\_loop函数类似的是pcap\_dispatch,两者用法基本一致，主要差异是pcap\_dispatch只会执行一次回调函数，而pcap\_loop会一直调用回调函数处理包。

            其回调函数的定义如下：

        void got\_packet\(u\_char \*args, const struct pcap\_pkthdr \*header,          const u\_char \*packet\);

            是void型的，第一个参数args是pcap\_loop函数的最后一个参数，第二个参数是pcap的头其包含了抓住的包的信息，第三个就是包本身了。

       struct pcap\_pkthdr {

              struct timeval ts; /\* time stamp \*/

              bpf\_u\_int32 caplen; /\* length of portion present \*/

              bpf\_u\_int32 len; /\* length this packet \(off wire\) \*/

       };

            关于包本身其实是一个字符串指针，怎么去寻找我的ip头，tcp头，以及头中的内容呢？这就需要是使用C语言中异常强大的指针了，定义一个宏如下：

/\* Ethernet addresses are 6 bytes \*/

\#define ETHER\_ADDR\_LEN      6



       /\* Ethernet header \*/

       struct sniff\_ethernet {

              u\_char ether\_dhost\[ETHER\_ADDR\_LEN\]; /\* Destination host address \*/

              u\_char ether\_shost\[ETHER\_ADDR\_LEN\]; /\* Source host address \*/

              u\_short ether\_type; /\* IP? ARP? RARP? etc \*/

       };



       /\* IP header \*/

       struct sniff\_ip {

              u\_char ip\_vhl;              /\* version &lt;&lt; 4 \| header length &gt;&gt; 2 \*/

              u\_char ip\_tos;              /\* type of service \*/

              u\_short ip\_len;             /\* total length \*/

              u\_short ip\_id;              /\* identification \*/

              u\_short ip\_off;             /\* fragment offset field \*/

       \#define IP\_RF 0x8000        /\* reserved fragment flag \*/

       \#define IP\_DF 0x4000        /\* dont fragment flag \*/

       \#define IP\_MF 0x2000        /\* more fragments flag \*/

       \#define IP\_OFFMASK 0x1fff   /\* mask for fragmenting bits \*/

              u\_char ip\_ttl;               /\* time to live \*/

              u\_char ip\_p;         /\* protocol \*/

              u\_short ip\_sum;             /\* checksum \*/

              struct in\_addr ip\_src,ip\_dst; /\* source and dest address \*/

       };

       \#define IP\_HL\(ip\)           \(\(\(ip\)-&gt;ip\_vhl\) & 0x0f\)

       \#define IP\_V\(ip\)            \(\(\(ip\)-&gt;ip\_vhl\) &gt;&gt; 4\)



       /\* TCP header \*/

       typedef u\_int tcp\_seq;



       struct sniff\_tcp {

              u\_short th\_sport;    /\* source port \*/

              u\_short th\_dport;    /\* destination port \*/

              tcp\_seq th\_seq;              /\* sequence number \*/

              tcp\_seq th\_ack;              /\* acknowledgement number \*/

              u\_char th\_offx2;     /\* data offset, rsvd \*/

       \#define TH\_OFF\(th\)   \(\(\(th\)-&gt;th\_offx2 & 0xf0\) &gt;&gt; 4\)

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

              u\_short th\_win;              /\* window \*/

              u\_short th\_sum;             /\* checksum \*/

              u\_short th\_urp;             /\* urgent pointer \*/

};

/\* ethernet headers are always exactly 14 bytes \*/

\#define SIZE\_ETHERNET 14



       const struct sniff\_ethernet \*ethernet; /\* The ethernet header \*/

       const struct sniff\_ip \*ip; /\* The IP header \*/

       const struct sniff\_tcp \*tcp; /\* The TCP header \*/

       const char \*payload; /\* Packet payload \*/



       u\_int size\_ip;

       u\_int size\_tcp;

            通过以上结构体定义，可以从回调函数的包指针地址出发，逐个找到链路帧头、IP帧头、TCP帧头、数据负载了

