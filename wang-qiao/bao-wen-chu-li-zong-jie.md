进入桥的数据报文分为几个类型，桥对应的处理方法也不同：



1.  报文是本机发送给自己的，桥不处理，交给上层协议栈；



2.  接收报文的物理接口不是网桥接口，桥不处理，交给上层协议栈；



3.  进入网桥后，如果网桥的状态为Disable，则将包丢弃不处理；



4.  报文源地址无效（广播，多播，以及00:00:00:00:00:00），丢包；



5.  如果是STP的BPDU包，进入STP处理，处理后不再转发，也不再交给上层协议栈；



6.  如果是发给本机的报文，桥直接返回，交给上层协议栈，不转发；



7.  需要转发的报文分三种情况：



1） 广播或多播，则除接收端口外的所有端口都需要转发一份；



2） 单播并且在CAM表中能找到端口映射的，只需要网映射端口转发一份即可；



3） 单播但找不到端口映射的，则除了接收端口外其余端口都需要转发；
