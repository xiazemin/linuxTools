曾经的 ipfw 已经被 pf 所替换。

首先我们要开启系统的端口转发功能。

本次开机生效：

\# IPv4 的转发

$ sudo sysctl -w net.inet.ip.forwarding=1

net.inet.ip.forwarding: 0 -&gt; 1

\# IPv6 的转发

$ sudo sysctl -w net.inet6.ip6.forwarding=1

net.inet6.ip6.forwarding: 0 -&gt; 1

开机启动配置，需以 root 身份添加或修改 /etc/sysctl.conf 文件，加入以下两行：

net.inet.ip.forwarding=1

net.inet6.ip6.forwarding=1

查看当前端口转发功能状态：

$ sudo sysctl -a \| grep forward

net.inet.ip.forwarding: 0

net.inet6.ip6.forwarding: 0

开启端口转发之后，即可配置端口转发规则。你可以跟着手册来：

$ man pfctl

$ man pf.conf

或者跟着下文手动新建文件。如 /etc/pf.anchors/http 文件内容如下：

rdr pass on lo0 inet proto tcp from any to any port 80 -&gt; 127.0.0.1 port 8080

rdr pass on lo0 inet proto tcp from any to any port 443 -&gt; 127.0.0.1 port 4443

rdr pass on en0 inet proto tcp from any to any port 80 -&gt; 127.0.0.1 port 8080

rdr pass on en0 inet proto tcp from any to any port 443 -&gt; 127.0.0.1 port 4443

检查其正确性：

$ sudo pfctl -vnf /etc/pf.anchors/http

修改 pf 的主配置文件 /etc/pf.conf 开启我们添加的锚点 http。

pf.conf 对指令的顺序有严格要求，相同的指令需要放在一起，否则会报错 Rules must be in order: options, normalization, queueing, translation, filtering.

\# 在 rdr-anchor "com.apple/\*" 下添加

rdr-anchor "http-forwarding"

\# 在 load anchor "com.apple" from "/etc/pf.anchors/com.apple" 下添加

load anchor "http-forwarding" from "/etc/pf.anchors/http"

最后导入并允许运行：

$ sudo pfctl -ef /etc/pf.conf

使用 -e 命令启用 pf 服务。使用 -E 命令强制重启 pf 服务：

$ sudo pfctl -E

使用 -d 命令关闭 pf：

$ sudo pfctl -d

从 Mavericks 起 pf 服务不再默认开机自启。如需开机启动 pf 服务，请往下看。

新版 Mac OS 10.11 EI Captian 加入了系统完整性保护机制，需重启到安全模式执行下述命令关闭文件系统保护。

$ csrutil enable --without fs

然后才能修改 /System/Library/LaunchDaemons/com.apple.pfctl.plist 文件实现开机自启用配置。

向 plist 文件中添加 -e 行，如下所示：

&lt;string&gt;pfctl&lt;/string&gt;

&lt;string&gt;-e&lt;/string&gt;

&lt;string&gt;-f&lt;/string&gt;

&lt;string&gt;/etc/pf.conf&lt;/string&gt;

