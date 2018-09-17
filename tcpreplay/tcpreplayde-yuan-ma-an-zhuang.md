1， 首先下载tcpreplay的源码包，下载地址[http://tcpreplay.synfin.net/wiki/Download\#Releases](http://tcpreplay.synfin.net/wiki/Download#Releases。我下载的是tcpreplay-4.1.0.tar.gz。)

[。我下载的是tcpreplay-4.1.0.tar.gz。](http://tcpreplay.synfin.net/wiki/Download#Releases。我下载的是tcpreplay-4.1.0.tar.gz。)

2， 将tcpreplay-4.1.0.tar.gz文件上传到centos里面。然后执行命令

tar -zxvf tcpreplay-4.1.0.tar.gz

3， 之后执行命令cd tcpreplay-4.1.0

4， 此时执行./configure 会报一个缺少libpcap的错误。

5， 这时需要下载libpcap的源码包libpcap

6，执行命令，

&lt;/pre&gt;&lt;pre name="code" class="plain"&gt;unzip libpcap-master.zip

7， cd libpcap-master  --&gt; ./configure  --&gt; make --&gt; make install

8,  此时再进入tcpreplay-4.1.0目录执行： ./configure

9,  会得到一下报错 “Unable to link libpcap in /usr/local”

10, 这个时候搜了大半天发现要加一个选项，启动动态链接 “ --enable-dynamic-link”

11, ./configure --enable-dynamic-link

