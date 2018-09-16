网桥是一种在链路层实现中继，对帧进行转发的技术，根据MAC分区块，可隔离碰撞，将网络的多个网段在数据链路层连接起来的网络设备。





\[root@xenserver ~\]\# brctl --help

Usage: brctl \[commands\]

commands:

        addbr           &lt;bridge&gt;                add bridge

        delbr           &lt;bridge&gt;                delete bridge

        addif           &lt;bridge&gt; &lt;device&gt;       add interface to bridge

        delif           &lt;bridge&gt; &lt;device&gt;       delete interface from bridge

        setageing       &lt;bridge&gt; &lt;time&gt;         set ageing time

        setbridgeprio   &lt;bridge&gt; &lt;prio&gt;         set bridge priority

        setfd           &lt;bridge&gt; &lt;time&gt;         set bridge forward delay

        sethello        &lt;bridge&gt; &lt;time&gt;         set hello time

        setmaxage       &lt;bridge&gt; &lt;time&gt;         set max message age

        setpathcost     &lt;bridge&gt; &lt;port&gt; &lt;cost&gt;  set path cost

        setportprio     &lt;bridge&gt; &lt;port&gt; &lt;prio&gt;  set port priority

        show   

