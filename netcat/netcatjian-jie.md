NetCat是一个非常简单的Unix工具，可以读、写TCP或UDP网络连接\(network connection\)。它被设计成一个可靠的后端\(back-end\) 工具，能被其它的程序程序或脚本直接地或容易地驱动。同时，它又是一个功能丰富的网络调试和开发工具，因为它可以建立你可能用到的几乎任何类型的连接，以及一些非常有意思的内建功能。NetCat，它的实际可运行的名字叫nc，应该早很就被提供，就象另一个没有公开但是标准的Unix工具。



GNU也有一个netcat项目，但此处学习的不是GNU的那个。 



最简单的使用方法，”nc host port”，能建立一个TCP连接，连向指定的主机和端口。接下来，你的从标准输入中输入的任何内容都会被发送到指定的主机，任何通过连接返回来的信息都被显示在你的标准输出上。这个连接会一直持续下去，至到连接两端的程序关闭连接。注意，这种行为不同于大多数网络程序，它们会在从标准输入读到一个文件结束符后退出。



NetCat还可以当服务器使用，监听任意指定端口的连接请求\(inbound connection \)，并可做同样的读写操作。除了较小限制外，它实际并不关心自己以“客户端”模式还是“服务器”模式运行，它都会来回运送全部数据。在任何一种模式下，都可以设置一个非活动时间来强行关闭连接。



它还可以通过UDP来完成这些功能，因此它就象一个telnet那样的UDP程序，用来测试你的UDP服务器。正如它的“U”所指的，UDP跟TCP相比是一种不可靠的数据传输，一些系统在使用UDP 传送大量数据时会遇到麻烦，但它还有一些用途。



你可能会问“为什么不用telnet来连接任意的端口”？问题提得好\(valid\)，这儿有一些理由。Telnet有“标准输入文件结束符\(standard input EOF\)”问题，所以需要在脚本中延迟计算以便等待网络输出结束。这就是netcat持续运行直到连接被关闭的主要原因。Telnet也不能传输任意的二进制数据，因为一些特定的字符会被解释为Telnet的参数而被从数据流中去除。Telnet还将它的一些诊断信息显示到标准输出上，而NetCat会将这信息与它的输出分开以不改变真实数据的传输，除非你要求它这么做。当然了，Telnet也不能监听端口，也不能使用UDP。 NetCat没有这些限制，比Telnet更小巧和快捷，而且还有一些其它的功能。



 



NetCat的一些主要功能：



\*支持连出和连入\(outbound andinbound connection\)，TCP和UDP，任意源和目的端口



\*全部DNS正向/反向检查，给出恰当的警告



\*使用任何源端口



\*使用任何本地设置的网络资源地址



\*内建端口扫描功能，带有随机数发生器



\*内建loosesource-routing功能



\*可能标准输入读取命令行参数



\*慢发送模式，每N秒发送一行



\*以16进制显示传送或接收的数据



\*允许其它程序服务建立连接，可选



\*对Telnet应答，可选



 



编译NetCat



==========



　　编译NetCat是非常简单的。检查一下Makefile，找到符合你的系统类型的SYSTYPE如何拼写,然后运行“make”，然后可执行的nc就会出现了。如果没有合适的SYSTYPE，用”generic”试试。其Makefile中有dos, ultrix, sunos, solaris-static, solaris, aix, linux, irix, osf,freebsd, bsdi, netbsd, hpux, unixware, aux, next, generic等SYSTYPE，其中generic不算系统类型，则dos其实并不支持。在本文一开始的NetCat的链接页面中，也有一个Windows 版本的NetCat，是另一个人做的移植。



　　Linux的sys/time.h并不真正支持FD\_SETSIZE的表示，编译时会有一个无害的警告。在一些系统中编译时，可能会与signal\(\)有关的指针类型警告，但不影响编译结果。



 



开发NetCat的功能



===============



NetCat小巧且功能强大，描述它的功能就是象描述瑞士军刀的功能一样。



如果没有提供命令行参数，NetCat会提示你从标准输入来输入命令参数，然后NetCat会在内部解析输入。用这种办法输入命令式参数，可以用来防止借助“ps”来查看你的命令行参数。



　　主机参数可以是一个名字或一个IP地址。如果-n出现，则它接受IP地址，而不再对计算机的名字或域名进行解析。如果没有-n，但加上-v，则NetCat可进行正/反向域名解析，并警告the all-too-common problem of mismatched name in DNS 。这会耗费稍多一点时间，但在某些情况下会有用处。如，你想知道某个IP的主机名，NetCat可省却你手工查找的时间。



　　要建立对外的连接，必须提供一个端口号，可以是个数字，也可以/etc/services列表中的端口服务名。当-n 出现时，则只有数字形式的端口可以接收。



　　-v参数，可以将一些关于连接建立信息输出到标准错误。-v参数多出现几次，则显示的信息会更多一些。如果-v参数没有出现，则NetCat将默默地工作，至到出现错误为止。



　　-w参数后跟一个时间值，用以指定建立链接时的等待时间，-w如果多次出现，则后面的值将取代前面的设置。-w还用来设置连接非活动时间，当标准输入结束以后，如果等待指定的一段时间后仍没有数据返回，则NetCat会再试一次，然后关闭连接并退出。



　　当-u参数出现时，用UDP建立连接。



　　用-o logfile参数，可以将连接上往来传输的数据以16进制的形式记录到logfile中（每行的左半部分是16进制显示，右半部分为ascii显示）。其中，每行的第一个字符为”&lt;”或”&gt;”，分别表示接收的数据或发送的数据。



NetCat用-s ip-addr或-s name来绑定本地网络资源地址，-p portarg 来绑定本地端口。除了因权限限制或端口已经使用外，-p可以绑定任何端口。



Root用户可以绑定保留的1024以内的端口。如果不用-p指定端口，则使用系统给定的未使用的端口。\(-p功能在客户端状态也可以使用,-s功能并不是在所有的平台上都可用\)



　　-l参数可以使NetCat以服务器状态运行。”nc -l -p 1234 \[remote hostname\] \[remote port\]”可以用来指定入连的主机和端口，如果申请连接的主机或端口不符指定，则会断开连接。



　　当编译时置-DGAPING\_SECURITY\_HOLE，则-e参数被NetCat支持。-e后面跟一可执行程序的名称，当一个连接（入或出）被建立时，这个程序被运行。尤其当NetCat以服务器端运行时，-e参数使其有点象inetd了，只是只能运行一个进行而已。需要说明的是，-e后的程序不能从NetCat的命令行接收参数，如果有参数要传递，可能需要一个脚本。



　　当编译时置-DTELNET，则-t参数被支持，此时NetCat可以登录到一个telnetd服务器，并提供相关的握手应答，至到出现登录提示符。



NetCat用8k的读写，来尽可能高效将收到数据显示到标准输出上及将标准输入写到连接上。



-i参数，可以用来设置发送一行标准输入信息的间隔，以减少发送速度。



　　端口扫描是一探测主机服务的流行方法。NetCat的命令行中，先是参数，再是主机，最后是端口。端口可以是一些服务名、端口号，或者是一个端口范围（形如N-M）。



    ”nc -v -w 2-z -i 1 20-30”用来扫描target主机的20-30\(两端包含\)端口，-z表示不发送任何数据到TCP连接或非常有限的数据到UDP连接。-i用以指明两个端口建立连接的时间的间隔。-w用以指明连接不活动时间。通常情况下，扫描按从高到低的顺序依次扫描指定的端口，-r参数可以让NetCat在指定的端口范围内随机地扫描端口。（当-r被用于单个连接时，本地的端口在8192以上，除非用-p指定\)



　　-g可以用来指定网关（最多可达8个），-G可以用来指定source-routing pointer。\(这是原文，但我还是不明白。：（-g =&gt; Group hops Many people are interested in testing networkconnectivity using IP source routing, even if it's only to make sure their ownfirewalls are blocking source-routed packets. On systems that support it, the-gswitch can be used multiple times \[up to 8\] to construct a loose-source-routedpath for your connection, and the -G argument positions the \`\`hop pointer''within the list. If your network allows source-routed traffic in and out, youcan test connectivity to your own services via remote points in the internet.Note that although newer BSD-flavor telnets also have source-routing capability,it isn't clearly documented and the command syntax is somewhat clumsy. Netcat'shandling of \`\`-g'' is modeled after \`\`traceroute''.）



NetCat不是一个任意包发生器，但可以与raw socket通话，nit/bpf/dlpi有时也能行\( nit/bpf/dlpi may appear at some point\).推荐Drren Reed的ip\_filter包，里面有一个工具能创建并发送raw packets.



 



netcat可以作为类似于telent的客户端,也可以监听某个端口作为服务器,还可以作为扫描工具扫描对方主机的端口,还可以用来传输文件,不相信吗? 听我慢慢道来:\)



首先我们要弄明白netcat的工作原理,其实netcat的原理很简单,它就是从网络的一端读入数据,然后输出到网络的另一端,它可以使用tcp和udp协议. 之所以叫做netcat,因为它是网络上的cat,想象一下cat的功能,读出一个文件的内容,然后输出到屏幕上\(默认的stdout是屏幕,当然可以重定向到其他地方\).netcat也是如此,它读取一端的输入,然后传送到网络的另一端,就这么简单.但是千万不要小看了它,netcat可以完成很多任务,,尤其是和其他程序组合时.好了,废话少说,进入正题吧.:p



网上有两种版本的netcat,一个是@stake公司的netcat,http://www.atstake.com/research/tools/network\_utilities/  也就是最初的版本,还有一个是GNU的netcat.http://netcat.sourceforge.net/download.php我个人更倾向于使用GNU的netcat,因为它的功能更多,不过GNU的没有windows 平台的版本:confused:



至于编译和安装我想就不用说了,如果这关都过不了,我想也有点太……，看看readme和install文件，一般情况下./configure&&make&&make install就ok了，具体的./configure选项看看帮助。



 



netcat的命令行程序名字为nc,是netcat的缩写,安装完了是找不到netcat这个程序的.:\)



 



root@mail etc \#nc -h



GNU netcat 0.7.0, a rewrite of the famousnetworking tool.



Basic usages:



connect to somewhere: nc \[options\] hostnameport \[port\] ...



listen for inbound: nc -l -p port \[options\]\[hostname\] \[port\] ...



tunnel to somewhere: nc -L hostname:port -pport \[options\]



 



Mandatory arguments to long options aremandatory for short options



too.



Options:



-c, --close closeconnection on EOF from stdin



-e, --exec=PROGRAMprogram to exec after connect



-g, --gateway=LISTsource-routing hop point\[s\], up to 8



-G, --pointer=NUMsource-routing pointer: 4, 8, 12, ...



-h, --help displaythis help and exit



-i, --interval=SECSdelay interval for lines sent, ports scanned



-l, --listen listenmode, for inbound connects



-L,--tunnel=ADDRESS:PORT forward local port to remote address



-n, --dont-resolvenumeric-only IP addresses, no DNS



-o, --output=FILEoutput hexdump traffic to FILE \(implies -x\)



-p, --local-port=NUMlocal port number



-r, --randomizerandomize local and remote ports



-s, --source=ADDRESSlocal source address \(ip or hostname\)



-t, --tcp TCP mode\(default\)



-T, --telnet answerusing TELNET negotiation



-u, --udp UDP mode



-v, --verbose verbose\(use twice to be more verbose\)



-V, --version outputversion information and exit



-x, --hexdump hexdumpincoming and outgoing traffic



-w, --wait=SECStimeout for connects and final net reads



-z, --zero zero-I/Omode \(used for scanning\)



 



Remote port number can also be specified asrange. Example: '1-1024'



 



我用的是GNU的netcat,比起@stake公司的netcat多了-c 选项,不过这是很有用的一个选项,后面我们会讲到.还有GNU的-L,-t ,-T选项和@stake的-L -t 用途是不一样的,自己琢磨吧.



 



一.客户端



这是最简单的使用方式,nc



nc www.apache.org 80



get / http/1.1



HTTP/1.1 400 Bad Request



Date: Mon, 08 Dec 2003 06:23:31 GMT



Server: Apache/2.0.48-dev \(Unix\)



Content-Length: 310



Connection: close



Content-Type: text/html; charset=iso-8859-1



 



400 Bad Request



 



Bad Request



Your browser sent a request that thisserver could not understand.



 



Apache/2.0.48-dev \(Unix\) Server atwww.apache.org Port 80







呵呵,看到了什么,我什么也没说哦:p



 



二.简单服务器



nc -l -p //这里-l参数表明nc处于监听模式,-p指定端口号.



nc -l -p 1234\[假设这台主机ip为192.168.0.1\]



然后从客户端输入, nc 192.168.0.1 1234 然后你从任一端输入的数据就会显



示在另一端了.其实netcat的server和client的区别并不大,区别仅仅在于谁执



行了-l来监听端口,一旦连接建立以后,就没有什么区别了. 从这里我们也可以



了解netcat的工作原理了,通过网络链接读写数据.\[It is a simple Unix



utility which reads and writes data acrossnetwork connections,



using TCP or UDP protocol\]--@stake主页是这么说的.



 



三.telnet服务器



nc有一个-e的选项,用来指定在连接后执行的程序.



在windows平台上可以指定-e cmd.exe\[winxp,win2000,\] 如果是98就指定



command.exe.linux则指定-e bash,或者任何你喜欢的shell,或者是你自己编



写的程序,通常是做为后门:p



指定-e的效果是由你指定的程序代替了nc自己来接受另一端的输入,并把输入



\(命令\)后反馈的结果显示到另一端.



server: nc -l -p 1234 -e bash



client: nc 192.168.0.1 1234 就可以远程登陆server了



其实我们不一定非要在server端指定-e,也可以在client端指定.



server: nc -l -p 1234



client: nc -e 192.168.0.1 1234 .这样,就相当于在server上远程登陆client



了.我前面说过,有关client和server的区分是没有什么意义的.谁做为telnet



server的标准只有一个,谁执行了-e\[shell\].



 



四.ftp服务器



nc可以从任何地方接受输入,不仅仅是-e指定的程序,还可以是文件; nc可以将



输入重定向到任何地方,不仅仅是默认的屏幕.指定的方法很简单,使用 &gt; 和



somefile



例2; server: nc -l -c -p 1234 &gt;somefile



client: nc 192.168.0.1 1234/check/host.disk1



然后,可以利用linux内核的loopback特性,把host.disk以只读的方式mount上,



然后就可以做取证分析了.



\[如果真的做取证分析,一定不要在原始的受害主机硬盘上find和类似的操作,



因为这会修改时间标记而破坏原始的证据\]



 



例4. 将文件压缩后再传送.



如果你的文件很大,何不先压缩它呢,利用管道, 我们甚至不用生成压缩后的中



间文件!



源主机: tar czf - work\|nc -l -c -p 1234



目的主机: nc 192.168.0.1 1234\|tar xzvf -

