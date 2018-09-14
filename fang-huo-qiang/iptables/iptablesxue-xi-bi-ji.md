1、防火墙分类

      ①包过滤防火墙\(pack filtering\)在网络层对数据包进行选择过滤，采用访问控制列表\(Access control table－ACL\)检查数据流的源地址，目的地址，源和目的端口，IP等信息。

      ②代理服务器型防火墙

2、iptables基础

      ①规则\(rules\)：网络管理员预定义的条件

      ②链\(chains\)： 是数据包传播的路径

      ③表\(tables\)：内置3个表filter表，nat表，mangle表分别用于实现包过滤网络地址转换和包重构的功能

      ④filter表是系统默认的，INPUT表\(进入的包\)，FORWORD\(转发的包\)，OUTPUT\(处理本地生成的包\)，filter表只能对包进行授受和丢弃的操作。

      ⑤nat表\(网络地址转换\)，PREROUTING\(修改即将到来的数据包\)，OUTPUT\(修改在路由之前本地生成的数据包\)，POSTROUTING\(修改即将出去的数据包\)

      ⑥mangle表，PREROUTING，OUTPUT，FORWORD，POSTROUTING，INPUT

3、其它

   iptables是按照顺序读取规则

   防火墙规则的配置建议

    Ⅰ 规则力求简单

    Ⅱ 规则的顺序很重要

    Ⅲ 尽量优化规则

    Ⅳ 做好笔记

二、配置

1、iptables命令格式

     iptables \[-t 表\] －命令 匹配 操作 （大小写敏感）

   动作选项

     ACCEPT          接收数据包

     DROP             丢弃数据包

     REDIRECT      将数据包重新转向到本机或另一台主机的某一个端口，通常功能实现透明代理或对外开放内网的某些服务

     SNAT             源地址转换

     DNAT             目的地址转换

     MASQUERADE       IP伪装

     LOG               日志功能

2、定义规则

   ①先拒绝所有的数据包，然后再允许需要的数据包

      iptalbes -P INPUT DROP

      iptables -P FORWARD DROP

      iptables -P OUTPUT ACCEPT

   ②查看nat表所有链的规则列表

      iptables -t nat -L

   ③增加，插入，删除和替换规则

     iptables \[-t 表名\] &lt;-A\|I\|D\|R&gt; 链名 \[规则编号\] \[-i\|o 网卡名称\] \[-p 协议类型\] \[-s 源ip\|源子网\] \[--sport 源端口号\] \[-d 目的IP\|目标子网\] \[--dport 目标端口号\] \[-j 动作\]

    参数：-A 增加

               -I 插入

               -D 删除

               -R 替换



三、例子

①iptables -t filter -A INPUT -s 192.168.1.5 -i eth0 -j DROP

禁止IP为192.168.1.5的主机从eth0访问本机②iptables -t filter -I INPUT 2 -s 192.168.5.0/24 -p tcp --dport 80 -j DROP

禁止子网192.168.5.0访问web服务③iptables -t filter -I INPUT 2 -s 192.168.7.9 -p tcp --dport ftp -j DROP

禁止IP为192.168.7.9访问FTP服务

④iptables -t filter -L INPUT

查看filter表中INPUT链的规则

⑤iptables -t nat -F

删除nat表中的所有规则

⑥iptables -I FORWARD -d wwww.playboy.com -j DROP

禁止访问www.playboy.com网站

⑦iptables -I FORWARD -s 192.168.5.23 -j DROP

禁止192.168.5.23上网

