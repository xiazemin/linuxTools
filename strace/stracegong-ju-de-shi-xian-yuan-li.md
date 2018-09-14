strace工具是一个用户态的应用程序，用来追踪进程的系统调用。它的基础就是ptrace系统调用。安装strace之后，就可以使用strace命令了。

最简单的strace命令的用法就是：strace PROG，PROG是要执行的程序。strace命令执行的结果就是按照调用顺序打印出所有的系统调用，包括函数名、参数列表以及返回值。

使用strace跟踪一个进程的系统调用的基本流程如图1所示。

![](/assets/strace.png)

