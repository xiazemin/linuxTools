1、 pcap解析工具 Xplico



Xplico 是一个从 pcap 文件中解析出IP流量数据的工具，可解析每个邮箱 \(POP, IMAP, 和 SMTP 协议\), 所有 HTTP 内容, VoIP calls \(SIP\) 等等



 



2、 C语言实现PCAP文件分析



实例一、



实现步骤：

1）用Wireshark软件抓包得到test.pcap文件

2）程序：分析pcap文件头 -&gt; 分析pcap\_pkt头 -&gt; 分析帧头 -&gt; 分析ip头 -&gt; 分析tcp头 -&gt; 分析http信息



\#include&lt;stdio.h&gt;



\#include&lt;string.h&gt;



\#include&lt;stdlib.h&gt;



\#include&lt;netinet/in.h&gt;



\#include&lt;time.h&gt;



\#define BUFSIZE 10240



\#define STRSIZE 1024



typedef long bpf\_int32;



typedef unsigned long bpf\_u\_int32;



typedef unsigned short  u\_short;



typedef unsigned long u\_int32;



typedef unsigned short u\_int16;



typedef unsigned char u\_int8;



//pacp文件头结构体



struct pcap\_file\_header



{



bpf\_u\_int32 magic;       /\* 0xa1b2c3d4 \*/



u\_short version\_major;   /\* magjor Version 2 \*/



u\_short version\_minor;   /\* magjor Version 4 \*/



bpf\_int32 thiszone;      /\* gmt to local correction \*/



bpf\_u\_int32 sigfigs;     /\* accuracy of timestamps \*/



bpf\_u\_int32 snaplen;     /\* max length saved portion of each pkt \*/



bpf\_u\_int32 linktype;    /\* data link type \(LINKTYPE\_\*\) \*/



};



//时间戳



struct time\_val



{



long tv\_sec;         /\* seconds 含义同 time\_t 对象的值 \*/



long tv\_usec;        /\* and microseconds \*/



};



//pcap数据包头结构体



struct pcap\_pkthdr



{



struct time\_val ts;  /\* time stamp \*/



bpf\_u\_int32 caplen; /\* length of portion present \*/



bpf\_u\_int32 len;    /\* length this packet \(off wire\) \*/



};



//数据帧头



typedef struct FramHeader\_t



{ //Pcap捕获的数据帧头



u\_int8 DstMAC\[6\]; //目的MAC地址



u\_int8 SrcMAC\[6\]; //源MAC地址



u\_short FrameType;    //帧类型



} FramHeader\_t;



//IP数据报头



typedef struct IPHeader\_t



{ //IP数据报头



u\_int8 Ver\_HLen;       //版本+报头长度



u\_int8 TOS;            //服务类型



u\_int16 TotalLen;       //总长度



u\_int16 ID; //标识



u\_int16 Flag\_Segment;   //标志+片偏移



u\_int8 TTL;            //生存周期



u\_int8 Protocol;       //协议类型



u\_int16 Checksum;       //头部校验和



u\_int32 SrcIP; //源IP地址



u\_int32 DstIP; //目的IP地址



} IPHeader\_t;



//TCP数据报头



typedef struct TCPHeader\_t



{ //TCP数据报头



u\_int16 SrcPort; //源端口



u\_int16 DstPort; //目的端口



u\_int32 SeqNO; //序号



u\_int32 AckNO; //确认号



u\_int8 HeaderLen; //数据报头的长度\(4 bit\) + 保留\(4 bit\)



u\_int8 Flags; //标识TCP不同的控制消息



u\_int16 Window; //窗口大小



u\_int16 Checksum; //校验和



u\_int16 UrgentPointer;  //紧急指针



}TCPHeader\_t;



//



void match\_http\(FILE \*fp, char \*head\_str, char \*tail\_str, char \*buf, int total\_len\); //查找 http 信息函数



//



int main\(\)



{



struct pcap\_file\_header \*file\_header;



struct pcap\_pkthdr \*ptk\_header;



IPHeader\_t \*ip\_header;



TCPHeader\_t \*tcp\_header;



FILE \*fp, \*output;



int   pkt\_offset, i=0;



int ip\_len, http\_len, ip\_proto;



int src\_port, dst\_port, tcp\_flags;



char buf\[BUFSIZE\], my\_time\[STRSIZE\];



char src\_ip\[STRSIZE\], dst\_ip\[STRSIZE\];



char  host\[STRSIZE\], uri\[BUFSIZE\];



//初始化



file\_header = \(struct pcap\_file\_header \*\)malloc\(sizeof\(struct pcap\_file\_header\)\);



ptk\_header  = \(struct pcap\_pkthdr \*\)malloc\(sizeof\(struct pcap\_pkthdr\)\);



ip\_header = \(IPHeader\_t \*\)malloc\(sizeof\(IPHeader\_t\)\);



tcp\_header = \(TCPHeader\_t \*\)malloc\(sizeof\(TCPHeader\_t\)\);



memset\(buf, 0, sizeof\(buf\)\);



//



if\(\(fp = fopen\(“test.pcap”,”r”\)\) == NULL\)



{



printf\(“error: can not open pcap file\n”\);



exit\(0\);



}



if\(\(output = fopen\(“output.txt”,”w+”\)\) == NULL\)



{



printf\(“error: can not open output file\n”\);



exit\(0\);



}



//开始读数据包



pkt\_offset = 24; //pcap文件头结构 24个字节



while\(fseek\(fp, pkt\_offset, SEEK\_SET\) == 0\) //遍历数据包



{



i++;



//pcap\_pkt\_header 16 byte



if\(fread\(ptk\_header, 16, 1, fp\) != 1\) //读pcap数据包头结构



{



printf\(“\nread end of pcap file\n”\);



break;



}



pkt\_offset += 16 + ptk\_header-&gt;caplen;   //下一个数据包的偏移值



strftime\(my\_time, sizeof\(my\_time\), “%Y-%m-%d %T”, localtime\(&\(ptk\_header-&gt;ts.tv\_sec\)\)\); //获取时间



// printf\(“%d: %s\n”, i, my\_time\);



//数据帧头 14字节



fseek\(fp, 14, SEEK\_CUR\); //忽略数据帧头



//IP数据报头 20字节



if\(fread\(ip\_header, sizeof\(IPHeader\_t\), 1, fp\) != 1\)



{



printf\(“%d: can not read ip\_header\n”, i\);



break;



}



inet\_ntop\(AF\_INET, \(void \*\)&\(ip\_header-&gt;SrcIP\), src\_ip, 16\);



inet\_ntop\(AF\_INET, \(void \*\)&\(ip\_header-&gt;DstIP\), dst\_ip, 16\);



ip\_proto = ip\_header-&gt;Protocol;



ip\_len = ip\_header-&gt;TotalLen; //IP数据报总长度



// printf\(“%d:  src=%s\n”, i, src\_ip\);



if\(ip\_proto != 0×06\) //判断是否是 TCP 协议



{



continue;



}



//TCP头 20字节



if\(fread\(tcp\_header, sizeof\(TCPHeader\_t\), 1, fp\) != 1\)



{



printf\(“%d: can not read ip\_header\n”, i\);



break;



}



src\_port = ntohs\(tcp\_header-&gt;SrcPort\);



dst\_port = ntohs\(tcp\_header-&gt;DstPort\);



tcp\_flags = tcp\_header-&gt;Flags;



// printf\(“%d:  src=%x\n”, i, tcp\_flags\);



if\(tcp\_flags == 0×18\) // \(PSH, ACK\) 3路握手成功后



{



if\(dst\_port == 80\) // HTTP GET请求



{



http\_len = ip\_len – 40; //http 报文长度



match\_http\(fp, “Host: “, “\r\n”, host, http\_len\); //查找 host 值



match\_http\(fp, “GET “, “HTTP”, uri, http\_len\); //查找 uri 值



sprintf\(buf, “%d:  %s  src=%s:%d  dst=%s:%d  %s%s\r\n”, i, my\_time, src\_ip, src\_port, dst\_ip, dst\_port, host, uri\);



//printf\(“%s”, buf\);



if\(fwrite\(buf, strlen\(buf\), 1, output\) != 1\)



{



printf\(“output file can not write”\);



break;



}



}



}



} // end while



fclose\(fp\);



fclose\(output\);



return 0;



}



//查找 HTTP 信息



void match\_http\(FILE \*fp, char \*head\_str, char \*tail\_str, char \*buf, int total\_len\)



{



int i;



int http\_offset;



int head\_len, tail\_len, val\_len;



char head\_tmp\[STRSIZE\], tail\_tmp\[STRSIZE\];



//初始化



memset\(head\_tmp, 0, sizeof\(head\_tmp\)\);



memset\(tail\_tmp, 0, sizeof\(tail\_tmp\)\);



head\_len = strlen\(head\_str\);



tail\_len = strlen\(tail\_str\);



//查找 head\_str



http\_offset = ftell\(fp\); //记录下HTTP报文初始文件偏移



while\(\(head\_tmp\[0\] = fgetc\(fp\)\) != EOF\) //逐个字节遍历



{



if\(\(ftell\(fp\) – http\_offset\) &gt; total\_len\) //遍历完成



{



sprintf\(buf, “can not find %s \r\n”, head\_str\);



exit\(0\);



}



if\(head\_tmp\[0\] == \*head\_str\) //匹配到第一个字符



{



for\(i=1; i&lt;head\_len; i++\) //匹配 head\_str 的其他字符



{



head\_tmp\[i\]=fgetc\(fp\);



if\(head\_tmp\[i\] != \*\(head\_str+i\)\)



break;



}



if\(i == head\_len\) //匹配 head\_str 成功，停止遍历



break;



}



}



// printf\(“head\_tmp=%s \n”, head\_tmp\);



//查找 tail\_str



val\_len = 0;



while\(\(tail\_tmp\[0\] = fgetc\(fp\)\) != EOF\) //遍历



{



if\(\(ftell\(fp\) – http\_offset\) &gt; total\_len\) //遍历完成



{



sprintf\(buf, “can not find %s \r\n”, tail\_str\);



exit\(0\);



}



buf\[val\_len++\] = tail\_tmp\[0\]; //用buf 存储 value 直到查找到 tail\_str



if\(tail\_tmp\[0\] == \*tail\_str\) //匹配到第一个字符



{



for\(i=1; i&lt;tail\_len; i++\) //匹配 head\_str 的其他字符



{



tail\_tmp\[i\]=fgetc\(fp\);



if\(tail\_tmp\[i\] != \*\(tail\_str+i\)\)



break;



}



if\(i == tail\_len\) //匹配 head\_str 成功，停止遍历



{



buf\[val\_len-1\] = 0; //清除多余的一个字符



break;



}



}



}



// printf\(“val=%s\n”, buf\);



fseek\(fp, http\_offset, SEEK\_SET\); //将文件指针 回到初始偏移



}

