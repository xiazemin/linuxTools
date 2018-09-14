ss

ss命令也是一个查看网络连接的工具\(another utility to investigate sockets\),用来显示处于活动状态的套接字信息。关于ss的描述，引用Linux命令大全-ss命令

ss命令可以用来获取socket统计信息，它可以显示和netstat类似的内容。但ss的优势在于它能够显示更多更详细的有关TCP和连接状态的信息，而且比netstat更快速更高效。当服务器的socket连接数量变得非常大时，无论是使用netstat命令还是直接cat /proc/net/tcp，执行速度都会很慢。可能你不会有切身的感受，但请相信我，当服务器维持的连接达到上万个的时候，使用netstat等于浪费 生命，而用ss才是节省时间。 天下武功唯快不破。ss快的秘诀在于，它利用到了TCP协议栈中tcp\_diag。tcp\_diag是一个用于分析统计的模块，可以获得Linux 内核中第一手的信息，这就确保了ss的快捷高效。当然，如果你的系统中没有tcp\_diag，ss也可以正常运行，只是效率会变得稍慢。

其中比较常用的参数包括:

-l 查看处于LISTEN状态的连接

-t 查看tcp连接

-4 查看ipv4连接

-n 不进行域名解析

因此我们可以通过ss命令查看本地监听的所有端口\(和netstat命令功能类似\):

ss  -t -l -n -4

输出如图:ss

curl

curl是强大的URL传输工具，支持FILE, FTP, HTTP, HTTPS, IMAP, LDAP, POP3,RTMP, RTSP, SCP, SFTP, SMTP, SMTPS, TELNET以及TFTP等协议。我们使用这个命令最常用的功能就是通过命令行发送HTTP请求以及下载文件，它几乎能够模拟所有浏览器的行为请求，比如模拟refer\(从哪个页面跳转过来的）、cookie、agent（使用什么浏览器）等等，同时还能够模拟表单数据。

curl -X POST -d "DDDDD=2013140333&upass=1q2w3e4r&save\_me=1&R1=0" 10.3.8.211

以上方法利用curl往认证服务器发送POST请求，发送数据为用户名以及密码（模拟表单输入）。

具体用法参考buptLogin。

Openstack的命令行工具，比如nova，传入--debug参数就会显示curl往nova-api的curl REST请求。

curl命令非常强大，掌握了它能够发挥巨大的作用，其他有用参数列举如下：

-i 显示头部信息

-I 只显示头部信息，不显示正文

-X 指定请求方法，比如GET、POST等

-d 发送数据

--form模拟表单，利用这个参数可以上传文件、模拟点击按钮等

-A 指定用户代理，比如Mozilla/4.0,有些坑爹网址必须使用IE访问怎么办

-b 设置cookie

-c 指定cookie文件

-e 指定referer，有些网址必须从某个页面跳转过去

--header 设置请求的头部信息

--user 有些页面需要HTTP认证， 传递name:password认证

wget

wget是一个强大的非交互网络下载工具（The non-interactive network downloader），虽然curl也支持文件下载，不过wget更强大，比如支持断点下载等。

最简单的用法直接加上文件URL即可：

wget [http://xxx/xxx/video.mp4](http://xxx/xxx/video.mp4)

使用-r参数为递归的下载网页，默认递归深度为5，相当于爬虫，用户可以通过-l指定递归深度。

注意wget默认没有开启断点下载功能，需要手动传入-c参数。

如果需要批量下载，可以把所有的URL写入文件download.txt,然后通过-i指定下载文件列表：

wget -i download.txt

如果用户不指定保存文件名，wget默认会以最后一个符合/的后面的字符作为保存文件名，有时不是我们所期望的，此时需要-O指定保存的文件名。

通过--limit-rate可以限制下载的最大速度。

使用-b可以实现后台下载。

另外wget甚至可以镜像整个网站:

wget --mirror -p --convert-links -P int32bit [http://int32bit.me](http://int32bit.me)

wget还支持指定下载文件的格式，比如只下载jpg图片:

wget -A.jpg -r -l 2 [http://int32bit.me/](http://int32bit.me/)

axel

axel是一个多线程下载工具（A light download accelerator for Linux），通过建立多连接，能够大幅度提高下载速度，所以我经常使用这个命令开挂下载大文件，比wget快多了，并且默认就支持断点下载:

开启20个线程下载文件:

axel -n 20 URL

这个强大的下载工具极力推荐，非常好用！

nethogs

我们前面介绍的iftop工具能够根据主机查看流量\(by host\)，而nethogs则可以根据进程查看流量信息\(Net top tool grouping bandwidth per process\)。ubuntu14.04中使用apt-get安装的有bug，需要手动安装：

sudo apt-get install build-essential libncurses5-dev libpcap-dev

git clone [https://github.com/raboof/nethogs](https://github.com/raboof/nethogs)

cd nethogs

make -j 4

编译完后执行

./nethogs eth1

我们指定了监控的网卡为eth1，结果如图：

![](/assets/importss.png)由于eth1是私有ip，只有ssh进程，从图中我们可以看到它的进程号为17264，程序为sshd，共发送了1.593MB数据，接收了607.477MB数据（scp了一个镜像文件）。按m键还能切换视角查看当前流量。

