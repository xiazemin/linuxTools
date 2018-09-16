NF\_IP\_PRE\_ROUTING: 接收的数据包刚进来，还没有经过路由选择，即还不知道数据包是要发给本机还是其它机器。



NF\_IP\_LOCAL\_IN: 已经经过路由选择，并且该数据包的目的IP是本机，进入本地数据包处理流程。



NF\_IP\_FORWARD: 已经经过路由选择，但该数据包的目的IP不是本机，而是其它机器，进入forward流程。



NF\_IP\_LOCAL\_OUT: 本地程序要发出去的数据包刚到IP层，还没进行路由选择。



NF\_IP\_POST\_ROUTING: 本地程序发出去的数据包，或者转发（forward）的数据包已经经过了路由选择，即将交由下层发送出去。



关于这些钩子更具体的位置，请参考Linux网络数据包的接收过程和数据包的发送过程



从上面的流程中，我们还可以看出，不考虑特殊情况的话，一个数据包只会经过下面三个路径中的一个：



本机收到目的IP是本机的数据包: NF\_IP\_PRE\_ROUTING -&gt; NF\_IP\_LOCAL\_IN



本机收到目的IP不是本机的数据包: NF\_IP\_PRE\_ROUTING -&gt; NF\_IP\_FORWARD -&gt; NF\_IP\_POST\_ROUTING



本机发出去的数据包: NF\_IP\_LOCAL\_OUT -&gt; NF\_IP\_POST\_ROUTING



注意： netfilter所有的钩子（hooks）都是在内核协议栈的IP层，由于IPv4和IPv6用的是不同的IP层代码，所以iptables配置的rules只会影响IPv4的数据包，而IPv6相关的配置需要使用ip6tables。



iptables中的表（tables）

iptables用表（table）来分类管理它的规则（rule），根据rule的作用分成了好几个表，比如用来过滤数据包的rule就会放到filter表中，用于处理地址转换的rule就会放到nat表中，其中rule就是应用在netfilter钩子上的函数，用来修改数据包的内容或过滤数据包。目前iptables支持的表有下面这些：



Filter

从名字就可以看出，这个表里面的rule主要用来过滤数据，用来控制让哪些数据可以通过，哪些数据不能通过，它是最常用的表。



NAT

里面的rule都是用来处理网络地址转换的，控制要不要进行地址转换，以及怎样修改源地址或目的地址，从而影响数据包的路由，达到连通的目的，这是家用路由器必备的功能。



Mangle

里面的rule主要用来修改IP数据包头，比如修改TTL值，同时也用于给数据包添加一些标记，从而便于后续其它模块对数据包进行处理（这里的添加标记是指往内核skb结构中添加标记，而不是往真正的IP数据包上加东西）。



Raw

在netfilter里面有一个叫做connection tracking的功能（后面会介绍到），主要用来追踪所有的连接，而raw表里的rule的功能是给数据包打标记，从而控制哪些数据包不被connection tracking所追踪。



Security

里面的rule跟SELinux有关，主要是在数据包上设置一些SELinux的标记，便于跟SELinux相关的模块来处理该数据包。



chains

上面我们根据不同功能将rule放到了不同的表里面之后，这些rule会注册到哪些钩子上呢？于是iptables将表中的rule继续分类，让rule属于不同的链（chain），由chain来决定什么时候触发chain上的这些rule。



iptables里面有5个内置的chains，分别对应5个钩子：



PREROUTING: 数据包经过NF\_IP\_PRE\_ROUTING时会触发该chain上的rule.



INPUT: 数据包经过NF\_IP\_LOCAL\_IN时会触发该chain上的rule.



FORWARD: 数据包经过NF\_IP\_FORWARD时会触发该chain上的rule.



OUTPUT: 数据包经过NF\_IP\_LOCAL\_OUT时会触发该chain上的rule.



POSTROUTING: 数据包经过NF\_IP\_POST\_ROUTING时会触发该chain上的rule.



每个表里面都可以包含多个chains，但并不是每个表都能包含所有的chains，因为某些表在某些chain上没有意义或者有些多余，比如说raw表，它只有在connection tracking之前才有意义，所以它里面包含connection tracking之后的chain就没有意义。（connection tracking的位置会在后面介绍到）



多个表里面可以包含同样的chain，比如在filter和raw表里面，都有OUTPUT chain，那应该先执行哪个表的OUTPUT chain呢？这就涉及到后面会介绍的优先级的问题。



提示：可以通过命令iptables -L -t nat\|grep policy\|grep Chain查看到nat表所支持的chain，其它的表也可以用类似的方式查看到，比如修改nat为raw即可看到raw表所支持的chain。



每个表（table）都包含哪些chain，表之间的优先级是怎样的？

下图在上面那张图的基础上，详细的标识出了各个表的rule可以注册在哪个钩子上（即各个表里面支持哪些chain），以及它们的优先级。



图中每个钩子关联的表按照优先级高低，从上到下排列；



图中将nat分成了SNAT和DNAT，便于区分；



图中标出了connection tracking（可以简单的把connection tracking理解成一个不能配置chain和rule的表，它必须放在指定位置，只能enable和disable）。



                                    \|

                                    \| Incoming             ++---------------------++

                                    ↓                      \|\| raw                 \|\|

                           +-------------------+           \|\| connection tracking \|\|

                           \| NF\_IP\_PRE\_ROUTING \|= = = = = =\|\| mangle              \|\|

                           +-------------------+           \|\| nat \(DNAT\)          \|\|

                                    \|                      ++---------------------++

                                    \|

                                    ↓                                                ++------------++

                           +------------------+                                      \|\| mangle     \|\|

                           \|                  \|         +----------------+           \|\| filter     \|\|

                           \| routing decision \|--------&gt;\| NF\_IP\_LOCAL\_IN \|= = = = = =\|\| security   \|\|

                           \|                  \|         +----------------+           \|\| nat \(SNAT\) \|\|

                           +------------------+                 \|                    ++------------++

                                    \|                           \|

                                    \|                           ↓

                                    \|                  +-----------------+

                                    \|                  \| local processes \|

                                    \|                  +-----------------+

                                    \|                           \|

                                    \|                           \|                    ++---------------------++

 ++------------++                   ↓                           ↓                    \|\| raw                 \|\|

 \|\| mangle     \|\|           +---------------+          +-----------------+           \|\| connection tracking \|\|

 \|\| filter     \|\|= = = = = =\| NF\_IP\_FORWARD \|          \| NF\_IP\_LOCAL\_OUT \|= = = = = =\|\| mangle              \|\|

 \|\| security   \|\|           +---------------+          +-----------------+           \|\| nat \(DNAT\)          \|\|

 ++------------++                   \|                           \|                    \|\| filter              \|\|

                                    \|                           \|                    \|\| security            \|\|

                                    ↓                           \|                    ++---------------------++

                           +------------------+                 \|

                           \|                  \|                 \|

                           \| routing decision \|&lt;----------------+

                           \|                  \|

                           +------------------+

                                    \|

                                    \|

                                    ↓

                           +--------------------+           ++------------++

                           \| NF\_IP\_POST\_ROUTING \|= = = = = =\|\| mangle     \|\|

                           +--------------------+           \|\| nat \(SNAT\) \|\|

                                    \|                       ++------------++

                                    \| Outgoing

                                    ↓

以NF\_IP\_PRE\_ROUTING为例，数据包到了这个点之后，会先执行raw表中PREROUTING\(chain\)里的rule，然后执行connection tracking，接着再执行mangle表中PREROUTING\(chain\)里的rule，最后执行nat \(DNAT\)表中PREROUTING\(chain\)里的rule。



以filter表为例，它只能注册在NF\_IP\_LOCAL\_IN、NF\_IP\_FORWARD和NF\_IP\_LOCAL\_OUT上，所以它只支持INPUT、FORWARD和OUTPUT这三个chain。



以收到目的IP是本机的数据包为例，它的传输路径为：NF\_IP\_PRE\_ROUTING -&gt; NF\_IP\_LOCAL\_IN，那么它首先要依次经过NF\_IP\_PRE\_ROUTING上注册的raw、connection tracking 、mangle和nat \(DNAT\)，然后经过NF\_IP\_LOCAL\_IN上注册的mangle、filter、security和nat \(SNAT\)。

