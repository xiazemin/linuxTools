1.Docker桥接snat网络原理图

![](/assets/importveth.png)

2.使用pipework第三方开源网络配置工具配置docker容器网络和host主机网络在同一个网段

\#1）.逻辑示意图

![](/assets/importvethpair.png)

\#2）.CentOS7需安装网络和桥接工具

yum install -y net-tools

yum install -y bridge-utils

yum -y install docker-io

禁用selinux

vim /etc/sysconfig/selinux

SELINUX=disabled

setenforce 0

\#3\) .安装pipework开源网络配置工具

\#git clone [https://github.com/jpetazzo/pipework](https://github.com/jpetazzo/pipework)

wget [https://github.com/jpetazzo/pipework/archive/master.zip](https://github.com/jpetazzo/pipework/archive/master.zip)

unzip master.zip

cp ./pipework-master/pipework  /usr/local/bin/

chmod +x /usr/local/bin/pipework

\#第2步和第3步和第4步放入脚本一起执行，否则会出现断网连不上的现象

\#4\) .绑定虚拟网桥和物理网卡，实现物理网卡和虚拟网卡公用ip

service docker stop

ifconfig docker0 down

brctl delbr docker0

brctl addbr br0

brctl addif br0 eth0

\#5）.把物理网卡地址配置为虚拟网桥的管理地址，因为容器不会直接和物理网卡通信

ip addr del 10.0.0.10/24 dev eth0

ifconfig br0 10.0.0.10/24 up

\#6）.删除以前的路由，添加新的路由，出口指向br0虚拟网卡，网关指向物理交换机的网关

route del default

route add default gw 10.0.0.254

\#7）.启动一个不带网络的容器

\#\#echo 'DOCKER\_OPTS="-b=br0"' &gt;&gt; /etc/default/docker

docker run -itd --name test --net=none centos /bin/bash

\#8）.给已经存在的名称为test的容器配置地址和网关（如果是centos6升级内核安装的docker这步可能会报错，需要升级iproute）

pipework br0 test 10.0.0.100/24@10.0.0.254

\#9）多个容器

运行一个容器：

\[root@localhost ~\]\# pipework br0 $\(docker run -d -it -p 80:80 --name testduliip centos\) 172.16.146.113/24@172.16.146.1

再运行一个：

\[root@localhost ~\]\# pipework br0 $\(docker run -d -it --net=none --name testduliip01 centos\) 172.16.146.112/24@172.16.146.1

重启容器后需要再次指定：

pipework br0 testduliip  172.16.146.113/24@172.16.146.1

pipework br0 testduliip01  172.16.146.112/24@172.16.146.1

