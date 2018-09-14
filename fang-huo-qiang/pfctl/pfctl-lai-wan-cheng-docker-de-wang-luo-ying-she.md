当使用 docker run -p port:port 来进行容器与主机的端口映射时，实际上是把容器的端口映射至虚拟机中，而不是映射至真实的 Mac 环境

\* 所以，还需要增加一步，就是虚拟机与 Mac 的端口映射，1024以上端口都没有问题，但是例如 Nginx 这样的软件使用80端口，这时就会发生映射不成功的问题



\* 因为 OSX 对于1024端口需要 root 权限，但并不要使用 root 或者 sudo 的方式来进行操作，不但不能解决端口映射的问题，还会造成VBOX 生成的文件都是 root 用户，从而造成不可预计的错误



Pfctl 工具

什么是 Pfctl，它是 Mac 上自带的一款强大的网络工具，其中端口转发只是其中的一部分



首先



设置将 Nginx 容器的端口转发至8080端口，并且将 VBOX 的8080端口映射至 Mac 本机 \(详细步骤\)

创建转发规则



规则：rdr pass on lo0 inet proto tcp from any to any port 80 -&gt; 127.0.0.1 port 8080



sudo \#规则 &gt; /etc/pf.anchors/http



使配置生效



vi /etc/pf.conf \# 最后增加两行

rdr-anchor “http”

load anchor “http” from “/etc/pf.anchors/http”

pfctl -ef /etc/pf.conf \# 配置生效

注：开机启动需要编辑文件 /System/Library/LaunchDaemons/com.apple.pfctl.plist



&lt;string&gt;pfctl&lt;/string&gt;

&lt;string&gt;-e&lt;/string&gt;

&lt;string&gt;-f&lt;/string&gt;

&lt;string&gt;/etc/pf.conf&lt;/string&gt;

10.11以上系统因为增强了安全模式，导致/System/Library/LaunchDaemons/com.apple.pfctl.plist修改失败，请重启至安全模式在进行操作

