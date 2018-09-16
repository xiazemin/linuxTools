因为最近接手的项目是基于嵌入式Linux openwrt的，一开始以为会跟之前的服务器开发没什么大的区别，但是遇到问题去分析的时候才发现，工具链还是有些差别的，openwrt的netstat是属于一个叫做busybox的工具集的，这个工具集是专门提供给嵌入式Linux，它的参数很简单，居然没有Linux下netstat的-p选项，因此当我想查看是哪些进程在监听哪些端口时，发现只能查看有哪些监听端口，无法得知是属于哪个进程的，lsof也没有-i选项。



但是有时候排查问题又必须知道哪个进程监听了某个端口，因此就想搞清楚Linux下的netstat是怎么实现可以查看监听端口属于哪个进程呢。



首先想法就是去下载busybox的源代码，但是感觉代码太多了，费时费力，于是灵机一动想到Linux下的另一个工具strace（追踪程序调用的系统调用），通过strace来查看netstat执行时都做了什么操作。



截取了strace输出的某一段，可以看到，调用open以及readlink遍历了/proc/3055/fd/目录下的所有文件，大家都知道这个目录是进程打开文件的目录。



技术分享图片



在strace输出的最后，可以看到调用了open打开/proc/net/udp文件，并读取里面的内容将其解析输出，这里面就记录了所有udp连接的信息，同时/proc/net/tcp对应tcp连接、/proc/net/unix对应Unix socket连接。



技术分享图片



根据这个文件的标头可以知道，第二列是local address，但是由于是16进制编码，所以需要我们手动转换成10进制。



这里其实可以发现，/proc/net/udp这个文件中的信息是不包含进程信息的，所以这也是为什么netstat在开始的时候会先遍历所有/proc/xx/fd目录，因为netstat可以通过inode将/proc/net/udp中的行和/proc/xx/fd中的文件关联起来，这样就可以得到某一行udp连接的进程信息（因为inode是唯一的）。



所以，分析到这里，我猜测busybox中的netstat应该是没有遍历所有/proc/xx/fd这一步，仅仅是读取了/proc/net/udp文件并解析输出。



明白了netstat的原理，那么即使遇到不提供netstat -p选项的嵌入式Linux，我们也能手动分析出自己想要的信息，进而解决问题。

