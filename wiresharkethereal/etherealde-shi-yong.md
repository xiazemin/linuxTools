双击启动桌面上ethereal图标，按ctrl+K进行“capture option”的选择,听说还支持Wlan的连接，我没有条件测试

 

Ethereal的使用解释\[转\]

每一个参数的解释：

Interface是选择捕获接口

Capture packets in promiscuous mode表示是否打开混杂模式，打开即捕获所有的报文，一般我们只捕获到本机收发的数据报文，所以关掉

Limit each packet 表示 限制每个报文的大小

Capture filters 过滤器（灵活使用，事半功倍）

Capture files 即捕获数据包的保存的文件名以及保存位置，右边update list of pack...表示实时更新数据报文。

点Start 开始捕获

 Ethereal的使用解释\[转\]



capture option确认选择后，点击ok就开始进行抓包

同时就会弹出“Ethereal:capture form \(nic\) driver”，其中\(nic\)代表本机的网卡型号。

同时该界面会以协议的不同统计捕获到报文的百分比

点击stop即可以停止抓包

在抓包的同时，可以通过最小化 or 使用alt+tab的快捷键直接切换到 报文浏览的主界面

报文界面：

Ethereal的使用解释\[转\]

 

以上是Ethereal最最简单的操作！

 

4各个菜单的解释

  4.1 File的下拉菜单

             open，打开抓包文件，快捷键ctrl+O 

             open recent 地球人都知道

             Merge 字面是合并的意思，其实是追加的意思，即当前捕获的报文追加到

                   先前已保存的抓包文件中

             Save和save as即保存 、选择保存格式Save as 有个地方需要注意，在文件格式的地方，需要选择windows base1.1 或者2.0 这才能和sniffer兼容

Ethereal的使用解释\[转\]

             file set 偶也不知道啥意思。

             Export是输出的意思 Print 打印 Quit退出

 

  4.2 Edit 下拉菜单

        Find Packet 就是查询报文，快捷键是ctrl+F，可以支持不同格式的查找 输入正确的语句，那么背景为 绿色，语句错误或缺少背景就为 红色 

              

Ethereal的使用解释\[转\]

       Find Next/Find Preyious 查找

       Time Reference 是报文的“书签”，方便大量报文的查询，使用后，选中的那条报文，时间列就会显示 \*REF\*

       Ethereal的使用解释\[转\]

      Mark Packet（toggle）是标记报文

      Mark all packets 和 Unamrk all packet即标记所有报文 、取消标记所有报文

      preference   选项 ,一般默认的

4.3VIew菜单（查看）

     Ethereal的使用解释\[转\]

4.4Go菜单（跳转到）

     Back  同样双方的上个报文

     Forward 同样双方的下一个报文

     Go to packet  查找到指定号码的报文

     First packet 第一个报文

     Last packet 最后一个报文

4.5CAPTURE 菜单（抓包）

     Ethereal的使用解释\[转\]

4.6Analyze 菜单（分析）

                              

Display filters

     显示过滤可以直接在主界面的filter上选择

Enable protocols

     是否启用该协议的解析，点选该协议后，相关的上层协议才能显示出来

Decode As 

     用户定义报文协议说明

User Specified Decodes

     用户修改的报文编译

其余的在后面有详细说明



 



Ethereal的使用解释\[转\]

4.7Statistics 菜单（统计）

                                               

Summmary 报文的详细信息

Protocol hierarchy  协议层即各协议层报文的统计

Conversations 显示该会话报文的信息（双方通信的报文信息）    

End points 分别显示单方的报文信息

IO Graphs  报文通信的心跳图

Conversation List  会话列表，后面是协议

Endpoints List 终端列表  ，后面是协议

Service Response Time 服务相应时间  显示某个请求和相应回复之间的时间间隔



下面的是各个协议了



 



 



 



 



 



 



 



 



 



 



 Ethereal的使用解释\[转\]

 



下面是重点介绍的捕获过滤和显示过滤

此菜单分别在抓取capture和analyze分析中的带filters的菜单项里面

首先讲捕获过滤（capture filters）Ethereal的使用解释\[转\]

过滤字符串，例如

a.捕获 MAC地址为 00:d0:f8:00:00:03 网络设备通信的所有报文

  ether host 00:d0:f8:00:00:03

b.捕获 IP地址为 192.168.10.1 网络设备通信的所有报文

  host 192.168.10.1

c.捕获网络web浏览的所有报文

  tcp port 80

d.捕获192.168.10.1除了http外的所有通信数据报文

  host 192.168.10.1 and not tcp port 80

e.不捕获ARP

  not arp

f.只捕获IP协议

  ip

g.捕获所有80端口的包

  port 80

h.没有IPX和DNS

  not IPX and port not 53

提示：如果以 默认主机和端口的设置捕获 tcp/ip报文，你将看不到自身的arp报文。

相关语法：

\[src\|dst\] host &lt;host&gt;  

ether \[src\|dst\] host &lt;ehost&gt;

gateway host &lt;host&gt;

\[src\|dst\] net &lt;net&gt; \[{mask &lt;mask&gt;}\|{len &lt;len&gt;}

\[tcp\|udp\] \[src\|dst\] port &lt;port&gt;

less\|greater &lt;length&gt;

ip\|ether proto &lt;protocol&gt;

ether\|ip broadcast\|multicast

&lt;expr&gt; relop &lt;expr&gt;

PS:1:\[src\|dst\]源\|目的

    2:\[tcp\|udp\]包类别

    3:less\|greater 小于\|大于

    4:Equal: eq, == \(等于）

      Not equal: ne, != （不等于）

      Greater than: gt, &gt; （大于）

      Less Than: lt, &lt;    （小于）

      Greater than or Equal to: ge, &gt;= （大等于）

      Less than or Equal to: le, &lt;=    （小等于）

Display filters（显示过滤）

和capture filters有所不同

a显示 以太网地址为 00:d0:f8:00:00:03 设备通信的所有报文

　　　eth.addr==00.d0.f8.00.00.03

b显示 IP地址为 192.168.10.1 网络设备通信的所有报文

　　　ip.addr==192.168.10.1

c显示所有设备web浏览的所有报文

　　　tcp.port==80

d显示192.168.10.1除了http外的所有通信数据报文

　　　ip.addr==192.168.10.1 && tcp.port!=80

