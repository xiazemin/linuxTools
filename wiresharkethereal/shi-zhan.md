抓包命令tcpdump tcp -A port 22 -w a.cap 将报文内容存在a.cap里面。a.cap文件是2进制的，需要用ethreal工具专门读取。

yum安装

yum install tcpdump

1

源码安装

\# flex

yum -y install flex

\# bison

yum -y install bison

wget [http://www.tcpdump.org/release/libpcap-1.5.3.tar.gz](http://www.tcpdump.org/release/libpcap-1.5.3.tar.gz)

wget [http://www.tcpdump.org/release/tcpdump-4.5.1.tar.gz](http://www.tcpdump.org/release/tcpdump-4.5.1.tar.gz)

tar -zxvf libpcap-1.5.3.tar.gz

cd libpcap-1.5.3

./configure

sudo make install

cd ..

tar -zxvf tcpdump-4.5.1.tar.gz

cd tcpdump-4.5.1

./configure

sudo make install

yum -y install bison

使用

抓http包

tcpdump -XvvennSs 0 -i eth0 tcp\[20:2\]=0x4745 or tcp\[20:2\]=0x4854 -w /tmp/capture.pcap

下载Wireshark\_2.2.6\_Intel\_64.dmg

打开：

![](/assets/wireshark.png)



