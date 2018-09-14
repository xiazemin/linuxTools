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

