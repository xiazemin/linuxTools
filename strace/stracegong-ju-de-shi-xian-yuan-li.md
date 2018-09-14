strace工具是一个用户态的应用程序，用来追踪进程的系统调用。它的基础就是ptrace系统调用。安装strace之后，就可以使用strace命令了。

最简单的strace命令的用法就是：strace PROG，PROG是要执行的程序。strace命令执行的结果就是按照调用顺序打印出所有的系统调用，包括函数名、参数列表以及返回值。

使用strace跟踪一个进程的系统调用的基本流程如图1所示。

![](/assets/strace.png)

从图中可以看出strace做了以下几件事情：

1. 设置SIGCHLD 信号的处理函数，这个处理函数只要不是SIG\_IGN即可。由于子进程停止后是通过SIGCHLD信号通知父进程的，所以这里要防止SIGCHLD信号被忽略。

2. 创建子进程，在子进程中调用ptrace\(PTRACE\_TRACEME,0L, 0L, 0L\)使其被父进程跟踪，并通过execv函数执行被跟踪的程序。

3. 通过wait\(\)等待子进程停止，并获得子进程停止时的状态status。

4. 通过子进程的状态查看子进程是否已正常退出，如果是，则不再跟踪，随后调用ptrace发送PTRACE\_DETACH请求解除跟踪关系。

5. 子进程停止后，打印系统调用的函数名、参数和返回值。具体流程见图2。

6. 通过PTRACE\_SYSCALL让子进程继续运行，由于这个请求会让子进程在系统调用的入口处和系统调用完成时都会停止并通知父进程，这样，父进程就可以在系统调用开始之前获得参数，结束之后获得返回值。

在系统调用的入口和结束时子进程停止运行时，这时父进程认为子进程是因为收到SIGTRAP信号而停止的。所以父进程在wait\(\)后可以通过SIGTRAP来与其他信号区分开。

Strace中为每个要跟踪的进程维护了一个TCB（Trace Control Block）结构，定义如下。它保存了当前发生的系统调用的信息。

/\* Trace Control Block \*/

struct tcb {

```
int flags;    /\* See below for TCB\_ values \*/

int pid;      /\* Process Id of this entry \*/

int qual\_flg; /\* qual\_flags\[scno\] or DEFAULT\_QUAL\_FLAGS + RAW\*/

int u\_error;  /\* Error code \*/

long scno;    /\* System call number \*/

long u\_arg\[MAX\_ARGS\];    /\* System call arguments \*/

long u\_rval;      /\* Return value \*/

int curcol;       /\* Output column for this process \*/

FILE \*outf;       /\* Output file for this process \*/

const char \*auxstr;/\*Auxiliary info from syscall \(see RVAL\_STR\) \*/

const struct\_sysent \*s\_ent;/\* sysent\[scno\] or dummy struct for bad scno \*/

struct timeval stime;/\*System time usage as of last process wait \*/

struct timeval dtime;    /\* Delta for system time usage \*/

struct timeval etime;    /\* Syscall entry time \*/

          /\* Support fortracing forked processes: \*/

long inst\[2\];     /\* Saved clone args \(badly named\) \*/
```

};

上面已经提到，子进程会在系统调用前后各停止一次，所以打印系统调用信息时分为两个阶段：在系统调用开始时可以获取系统调用号和参数，在系统调用结束时可以获取系统调用的返回结果。通过给tcb结构的flags字段清除和添加TCB\_INSYSCALL标志位来区分系统调用的开始和结束。

![](/assets/strace1.png)

图2 strace中打印系统调用的实现流程

