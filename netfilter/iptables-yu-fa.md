iptables \(-t nat/filter\) -nvL 默认规则--保存于 /etc/sysconfig/iptables







iptables -F 清空规则（但却无法清空配置文件的规则，重启后会加载service iptables restart）







iptables -Z 清零



下例命令为jp阻断 封掉， DRPO可以用REJECT命令代替







如何删除新加入规则 （虽然 iptables -D INPUT -s 192.168.188.1 -p tcp --sport 1234 -d 192.168.188.128 --dport 80 -j DROP亦能实现删除 但是input过于繁琐，推荐使用以下方式）



iptalbes -nvL --llne-number



iptables -D INPUT 8（the line that you want to delete）



iptables -P INPUT DROP 清除全部规则 \(轻易不要用\)///// iptables -P OUTPUT ACCEPT







\#\#iptables nat 表应用







\#\#准备环境pre-work



1.准备个workstations 编辑虚拟机设置







2. 添加网卡







3.将网络连接模式改成区段（为了确保实验环境，网卡2不连接window而是连接某区段）







4.配置另一个workstations（fred linux 克隆）的网卡，选择Lan区段







5.添加成功，并尝试给新网卡ens37设置ip







可以通过下命令手动设置ip、端口（重启之后就会没有了，如果想要一直存在就要更改配置文件）







6.同理，配置fred 克隆workstation ens37网卡







7.测试是否ping的通







\#\#调配



需求1



1. A机器上打开路由转发 \(此默认值是0，意为关闭\)



cat /proc/sys/net/ipv4/ip\_forward







打开端口转发 echo "1" &gt; /proc/sys/net/ipv4/ip\_forward







2. 增加一条实现上网的规则



iptables -t nat -A POSTROUTING -s 192.168.100.0/24 -o ens33 -j MASQUERADE



3.克隆机机器上设置网关为192.168.100.1







实现克隆机联网







4.设置dns



vi /etc/resolv.confvi







需求2（端口映射）



1. A机器上打开端口转发



echo "1" &gt; /proc/sys/net/ipv4/ip\_forward



2. 增加iptables 规则（先删除之前的规则 以免影响接下来的实验）---此为进去的操作



iptables -t nat -A PREROUTING-d 192.168.88.128 -p tcp --dport 1122 -j DNAT --to 192.168.100.100:22



从88.128进来通过端口1122 转发到100.100的22端口







之后设置出去的端口



iptables -t nat -A POSTROUTING -s 192.168.100.100 -j SNAT --to 192.168.88.128



3.克隆机器添加网关



克隆机机器上设置网关为192.168.100.1



实验成功







\#\#iptables规则备份和恢复

