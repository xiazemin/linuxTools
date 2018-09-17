\[root@datatest ~\]\# tcpliveplay -H

tcpliveplay \(tcpliveplay\) - Replays network traffic stored in a pcap file on live networks using new TCP connections

Usage:  tcpliveplay \[ -&lt;flag&gt; \[&lt;val&gt;\] \| --&lt;name&gt;\[{=\| }&lt;val&gt;\] \]... \

        &lt;eth0/eth1&gt; &lt;file.pcap&gt; &lt;Destinatin IP \[1.2.3.4\]&gt; &lt;Destination mac \[0a:1b:2c:3d:4e:5f\]&gt; &lt;'random' dst port OR specify dport \#&gt;



   -V, --version              Print version information

   -h, --less-help            Display less usage information and exit

   -H, --help                 display extended usage information and exit

   -!, --more-help            extended usage information passed thru pager

       --save-opts\[=arg\]      save the option state to a config file

       --load-opts=str        load options from a config file

                - disabled as '--no-load-opts'

                - may appear multiple times



Options are specified by doubled hyphens and their name or by a single

hyphen and the flag character.

This program, 'tcpliveplay' replays a captured set of packets using new TCP

connections with the captured TCP payloads against a remote host in order

to do comprehensive vulnerability testings.



The following option preset mechanisms are supported:

 - reading file /usr/bin/.tcpliveplayrc

The basic operation of tcpliveplay is it rewrites the given pcap file in a

scheduled event format and responds with the apporiate packet if the remote

host meets tcp protocal's SEQ/ACK expectation.  Once expectations are met,

then the local packets are sent with the same payload except with new tcp

SEQ & ACK numbers meeting the response from the remote hose.



The inputted pcap file are rewritten to start at the first encounter of the

SYN packet for correct operation making this packet be the first action in

the event schedule of local host doing the replay.



For more details, please see the Tcpreplay Manual at:

http://tcpreplay.appneta.com

\[root@datatest ~\]\#

