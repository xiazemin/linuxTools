1.                   整体安装环境

1\)        PC的操作系统是RedHat AS 3.0



2\)        Ethereal的版本是ethereal-0.10.12.tar.gz



3\)        Libpcap的版本是libpcap-0.9.1.tar.gz。



2.                   Ethereal的安装前提

2.1               GTK+和GLIB

为了安装Ethereal，确保机子上已经安装GTK+和GLIB，并且版本要高于或是1.2.0。RedHat AS 3.0是系统自带安装了的。可以执行命令'gtk-config --version'和'glib-config --version'来查看软件版本。如果系统没有安装GTK+或 GLIB，可以在http://www.gtk.org下载。



2.2               Libpcap

为了实现抓包功能，必须在安装Ethereal之前安装Libpcap。



2.2.1           下载Libpcap

1\)        本次安装的是libpcap-0.9.1.tar.gz，可以在http://www.tcpdump.org上下载。



2.2.2           安装步骤

1\)        tar xzvf libpcap-0.9.1.tar.gz



2\)        cd libpcap-0.9.1



3\)        ./configure



4\)        make



5\)        make install



2.3               libpcre

如果不安装此软件，Ethereal就实现不了 filter语法“ maches”功能。



2.3.1           下载

本次安装的是pcre-6.3.tar.gz，可以在http://www.pcre.org上下载。



2.3.2           安装步骤

1\)        tar  xzvf  pcre-6.3.tar.gz



2\)        cd pcre-6.3



3\)        ./configure



4\)        make



5\)        make install



3.                   安装Ethereal

3.1               下载Ethereal

本次安装的是Ethereal的最新版本ethereal-0.10.12.tar.gz，可以在Ethereal的官方网站http://www.ethereal.com/下载。



3.2               安装Ethereal

1\)        tar xzvf ethereal-0.10.12.tar.gz



2\)        cd ethereal-0.10.12



3\)        ./configure --without-ucd-snmp



Congfigure命令后面有很多选项，具体可参见附件《configure 的参数.txt》，我安装时如果没有加选项，编译出现“configure: error: UCD SNMP requires -lcrypto but --with-ssl not specified”的错误，因此添加了参数“--without-ucd-snmp”。大家安装时根据错误提示，可具体选择参数。



4\)        make



5\)        make install



这样Ethereal就安装成功了，用户可以在Linux下抓包。如果是远程控制的用户，请确保Linux下安装VNC Server，操作的PC上安装了VNC View。否则远程控制界面会出现“Gtk-WARNING \*\*: cannot open display:”警告的。

