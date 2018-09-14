默认配置文件 /etc/pf.conf

crub-anchor "com.apple/\*"   \# 流量整形  \* 表示含有所有的子规则/子anchor规则

nat-anchor "com.apple/\*"    \# NAT规则

rdr-anchor "com.apple/\*"    \# 重定向规则

dummynet-anchor "com.apple/\*"   \# 不清楚~~~

anchor "com.apple/\*"    \# 过滤规则

load anchor "com.apple" from "/etc/pf.anchors/com.apple"    \# 从文件载入规则

常用命令

sudo pfctl -e   \# enable pf

sudo pfctl -E   \# Enable pf and increment the pf enable reference count.

sudo pfctl -d   \# disable pf

sudo pfctl -v -n -f /etc/pf.conf    \# 验证pf.conf文件是否有问题

sudo pfctl -sa  \# -s all 查看所有

sudo pfctl -sr  \# -s rules 查看规则

sudo pfctl -s nat   \# 查看nat规则

\# 更多命令可查看 man pfctl

pfctl添加配置



echo "block all" \| sudo pfctl -Ef - \# 拒绝所有流量 == 完全断网

echo "block return out quick on en0 proto tcp to any port {443}" \| sudo pfctl -f - \# 拒绝访问443

配置文件中添加配置



在配置文件中配置信息顺序书写： Macro -&gt; Table -&gt; Option -&gt; Scrub -&gt; Queue -&gt; Translation -&gt; Filter



\# /etc/pf.conf

rdr on bridge100 proto tcp from any to ip.chinaz.com port 80 -&gt; 127.0.0.1 port 8080 \# 重定向

block all   \# 拒绝所有

pass out all    \# 允许出流量

block out proto tcp to port 80  \# 拒绝连接80端口

通过锚\(Anchor\)动态修改规则



/etc/pf.conf 文件中描述的”com.apple”锚是从/etc/pf.anchors/com.apple中加载的，但cat /etc/pf.anchors/com.apple\(如下\)，仅仅只是anchor的定义，实际的rules会在运行时系统调用pfctl命令来加入。



\#

\# AirDrop anchor point.

\#

anchor "200.AirDrop/\*"



\#

\# Application Firewall anchor point.

\#

anchor "250.ApplicationFirewall/\*"

