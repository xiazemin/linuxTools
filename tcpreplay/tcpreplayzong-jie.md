通过 Tcpreplay 来修改、转发通信流量需要考虑的一共需要考虑以下 3 点：

　　　　1、 确定哪有数据包是从客户端到服务器端的，哪有是从服务器端到客户端的

　　　　2 、确定新的 MAC、IP、Port

　　　　3、确定回放速率、循环次数、执行方式 



　　第一步：

　　　　用 tcpreplay 分离源/目的端口的流量。



tcpprep --port --cachfile=example.cach --pcap=example.pcap

　　这种情况下，认为所有目的端口小于1024 的，将被视为客户端－&gt;服务器的包。



　　　　　　　　认为所有目的端口大于1024 的，否则视为服务器-&gt;客户端的包。



　　该信息被存储在 tcpprp 的一个名叫 example.cach 的文件夹中。

　　--port 根据端口号区分数据包的流向。

　　--cachfile 指定输出的 cach 文件的名字，即在这里要输出为example.cach

　　--pcap 指定要处理的数据包文件，即在这里要处理的是example.pcap。

　　其实。tcpprep 支持许多的其他的模式，分离端口模式是其中的一种 。也就是说，第一步，大家自行选择。



 



 



　　第二步：

　　使用 tcprewrite 更改 ip 地址到本地网络：



$ tcprewrite --endpoints=172.16.0.1:172.16.5.35   --cachfile=example.cach --infile=example.pcap --outfile=new.pcap

　　这个例子里，我们想要所有的流量来自于 172.16.0.1 和 172.16.5.35。我们想要一个 IP 是“客户端”，一个 IP 是“服务器端 。





 　　tcprewrite 改写数据包：

　　　　--endpoints 指定数据包的 client、server 端的 ip 地址

　　　　--cachfie 上一步预处理的输出文件，即example.cach

　　　　--infile 输入 pcap 文件，即example.pcap。

　　　　--outfile 改写后的 pcap 文件 ，即new.pcap。



 



 



 



 



 



　　第三步：用 tcpreplay，发送流量通过服务提供商



tcpreplay --intf1=eth0 --intf2=eth1 --cachfile=example.cach new.pcap

　　因为我们要分离 2 个接口（eth0 和 eth1）之间的通信，我们使用第一步中创建的 cach 文件，第二步中创建的 new.pacp。

　　然后使用 tcpreplay 重发数据包。

