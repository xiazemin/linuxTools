为了在Linux上实现软路由，特地买了一块USB网卡\(兼容Linux的\)。在虚拟机中运行kali,插入USB网卡并直连到kali系统。

对于当前我们这个使用场景，流量需要进过PREROUTING –&gt; FORWARD –&gt; POSTROUTING三个步骤。



在PREROUTING中添加规则，修改数据包的目标IP和端口，实现将流量转发到Proxy。

FORWARD 直接允许即可。

POSTROUTING中添加规则，修改数据包的原始IP和端口，让目标站点返回的数据原路返回。\(就是我们最通常意义上里面的NAT转换\)

命令实现：



$ iptables -t nat -A PREROUTING -i wlan0 -p tcp -j DNAT --to-destination 10.1.0.2:8080

\# $ iptables -P FORWARD ACCEPT  \# 此命令上面已经执行过，可以忽略。

$ iptables -t nat -A POSTROUTING -o eth0 -s 192.168.9.0/24 -d 10.1.0.2 -j MASQUERADE

查看路由表：



$ iptables -t nat -L

    Chain PREROUTING \(policy ACCEPT\)

    target     prot opt source               destination         

    DNAT       tcp  --  anywhere             anywhere             to:10.1.0.2:8080



    Chain INPUT \(policy ACCEPT\)

    target     prot opt source               destination         



    Chain OUTPUT \(policy ACCEPT\)

    target     prot opt source               destination         



    Chain POSTROUTING \(policy ACCEPT\)

    target     prot opt source               destination         

    MASQUERADE  all  --  anywhere             anywhere            

    MASQUERADE  all  --  192.168.9.0/24       10.1.0.2

