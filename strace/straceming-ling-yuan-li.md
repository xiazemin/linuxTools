strace是Linux系统下的一个用来跟踪系统调用的工具，它的实现基础是ptrace系统调用。使用strace工具可以跟踪一个程序执行过程中发生的系统调用。



我这里讲到的内容有一点点和mips体系相关，不过不熟悉mips也不影响阅读。



ptrace系统调用

ptrace系统调用提供了一种方法来跟踪和控制进程的执行，它可以读取和修改进程地址空间中的内容，包括寄存器的值。ptrace主要用于实现断点调试和跟踪系统调用。该系统调用的原型如下：



long ptrace\(enum \_\_ptrace\_request request, pid\_t pid, void \*addr,void \*data\);



ptrace的四个参数的含义为：



1.   request：用于选择一个操作，见下文。



2.   pid：目标进程即被跟踪进程的pid。



3.   addr和data用于修改和拷贝被跟踪进程的进程地址空间的数据。



下面的内容中将用父进程指代跟踪者，用子进程指代被跟踪者。实际上，在一个进程被跟踪之后，跟踪者进程会在某种意义上充当被跟踪进程的父进程（如使用ps命令就可以看到他们的父子关系），而子进程真正的父进程被保存在其task\_struct结构的real\_parent成员中。



使用ptrace跟踪进程

父进程跟踪一个进程的方式有两种：1.调用fork\(\)，然后子进程打上PTRACE\_TRACEME标记，并执行exec。2.父进程可以给自己打上PTRACE\_ATTACH标记来跟踪一个已有进程。



一个进程被跟踪后，他只要接收到一个信号（即使这个信号被设置为忽略）就会停止运行（SIGKILL除外），然后父进程会在每次调用wait\(\)时得到子进程停止运行的通知，这时父进程就可以检测和修改子进程了，随后父进程可以让子进程继续运行。



当父进程不想跟踪了，可以通过设置PTRACE\_KILL标记来终止子进程的运行。也可以通过设置PTRACE\_DETACH标记让子进程解除被跟踪，继续正常运行。



常用的request

PTRACE\_TRACEME



进程设置这个request目的是让自己被父进程跟踪。任何发送到该子进程的信号（除了SIGKILL）都会导致他停下来，并在父进程wait\(\)的时候通知到父进程。另外，子进程后续调用exec会导致子进程自己收到一个SIGTRAP信号，这是为了让父进程有机会在exec的新程序开始执行前获得控制权。



除非一个进程知道父进程要跟踪他，一般不会去设置这个request。设置这个请求时，pid，addr和data三个参数都会被忽略。



这个request只供子进程设置，其他的request都是只供父进程使用的。相应的，下面的request中，参数pid为被跟踪的子进程的pid。



PTRACE\_ATTACH



将pid指定的进程作为自己要跟踪的进程，并开始跟踪。这和子进程自己调用PTRACE\_TRACEME的效果相同。



设置这个request时，子进程会首先收到一个SIGSTOP信号，但并不会停止，只是导致跟踪者进程第一次被中断，从而开始跟踪，否则只能等到子进程接收到第一个信号时才开始跟踪。之后当子进程有待决信号时，进程总是会暂停，这时父进程通过SIGCHLD信号得到通知。在子进程暂停前，父进程可使用wait函数等待。



参数addr和data会被忽略。



PTRACE\_CONT



让被停掉的子进程继续运行，而当子进程再次接收到信号时会暂停。



参数data如果被设置为一个非零值并且不是SIGSTOP，那data就是父进程发送给子进程的信号，否则，不给子进程发送信号。这样一来，父进程可以控制是否向子进程发送一个信号。



参数addr会被忽略。



PTRACE\_SYSCALL



让停止的子进程继续运行（同PTRACE\_CONT），而当子进程再次接收到信号时会暂停，另外，在子进程中发生系统调用时，在系统调用的入口和结束时子进程也会停止，这时父进程认为子进程是因为收到SIGTRAP信号而停止的。



由于子进程在系统调用的入口和结束时都会停止，父进程就可以在系统调用入口处停止后，获得系统调用的参数信息，而在系统调用结束时停止后，获得系统调用的返回值。



参数addr会被忽略。



PTRACE\_DETACH



让停止的子进程继续运行（同PTRACE\_CONT），但是在这之前会先与跟踪它的父进程解除PTRACE\_ATTACH或PTRACE\_TRACEME时的关系。



参数addr会被忽略。



PTRACE\_KILL



给子进程发送一个SIGKILL信号来终止子进程。addr和data参数会被忽略。



PTRACE\_PEEKTEXT,PTRACE\_PEEKDATA



读取进程地址空间中addr地址处的内容，读出的长度为一个word（4字节），作为ptrace\(\)的返回值（long型）返回。Linux中的代码和数据的地址空间并不是分离的，所以这两个request实际上意义相同。



参数data会被忽略。



PTRACE\_PEEKUSR



在进程的USER区域读取一个word的长度。参数addr是指相对USER开头的offset，结果作为返回值。参数data会被忽略。



在mips中的进程自身信息和进程地址空间中并没有所谓的USER区域，在内核中通过参数addr的值，判断应该返回什么结果，如下：



/\* Read the word at location addr in theUSER area. \*/

    case PTRACE\_PEEKUSR: {

       struct pt\_regs \*regs;

       unsigned long tmp = 0;

 

       /\* 获得进程地址空间的pt\_regs区域的内容。 \*/

       regs =task\_pt\_regs\(child\);

       ret = 0;  /\* Default return value. \*/

 

       switch \(addr\) {

       case 0 ... 31:    /\* 通用寄存器 \*/

           tmp =regs-&gt;regs\[addr\];

           break;

       case FPR\_BASE ...FPR\_BASE + 31:

           ......

           break;

       case PC:

           tmp =regs-&gt;cp0\_epc;

           break;

       case CAUSE:

           tmp =regs-&gt;cp0\_cause;

           break;

       case BADVADDR:

           tmp =regs-&gt;cp0\_badvaddr;

           break;

       case MMHI:

           tmp = regs-&gt;hi;

           break;

       case MMLO:

           tmp = regs-&gt;lo;

           break;

       case FPC\_CSR:

           tmp =child-&gt;thread.fpu.fcr31;

           break;

       case FPC\_EIR: {   /\* implementation / version register \*/

           ......

           break;

       }

       case DSP\_BASE ...DSP\_BASE + 5: {

           ......

           break;

       }

       case DSP\_CONTROL:

           ......

           break;

       default:

           tmp = 0;

           ret = -EIO;

           goto out;

       }

       ret = put\_user\(tmp,\(unsigned long \_\_user \*\) data\);

       break;

    }

PTRACE\_POKETEXT,PTRACE\_POKEDATA



将data的内容拷贝到进程地址空间中addr指向的地址。



PTRACE\_POKEUSR



将data的内容拷贝到进程USER区域中偏移为addr的地方，一般addr要求是word-aligned的。由于要修改USER区域，内核为保证完整健全，会禁止某些域被修改。

