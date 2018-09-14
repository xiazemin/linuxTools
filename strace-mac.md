在linux下的strace跟踪命令在os x下找寻不见鸟，取而代之的是 dtruss命令，在os x下看一个程序的动态库依赖可以使用 otools -L xxx命令



Linux下经常会用到ldd查看程序使用了哪些共享链接库，很方便的一个工具，在Mac OS X上没有这个命令，如果想在Mac OS X查看程序使用了哪些链接库可以用otool 来代替。



例：



$ otool -L /usr/bin/vim

/usr/bin/vim:

/usr/lib/libncurses.5.4.dylib \(compatibility version 5.4.0, current version 5.4.0\)

/usr/lib/libiconv.2.dylib \(compatibility version 7.0.0, current version 7.0.0\)

/usr/lib/libSystem.B.dylib \(compatibility version 1.0.0, current version 1213.0.0\)





otool还有很多其它参数：



-f print the fat headers

-a print the archive header

-h print the mach header

-l print the load commands

-L print shared libraries used

-D print shared library id name

-t print the text section \(disassemble with -v\)

-p &lt;routine name&gt; start dissassemble from routine name

-s &lt;segname&gt; &lt;sectname&gt; print contents of section

-d print the data section

-o print the Objective-C segment

-r print the relocation entries

-S print the table of contents of a library

-T print the table of contents of a dynamic shared library

-M print the module table of a dynamic shared library

-R print the reference table of a dynamic shared library

-I print the indirect symbol table

-H print the two-level hints table

-G print the data in code table

-v print verbosely \(symbolically\) when possible

-V print disassembled operands symbolically

-c print argument strings of a core file

-X print no leading addresses or headers

-m don’t use archive\(member\) syntax

-B force Thumb disassembly \(ARM objects only\)

-q use llvm’s disassembler \(the default\)

-Q use otool\(1\)’s disassembler

-mcpu=arg use \`arg’ as the cpu for disassembly

-j print opcode bytes

-C print linker optimization hints

–version print the version of otool



详细使用请参看手册。



Linux中的strace可以查看程序运行时的系统调用，有时对于调试程序很有帮助，Mac OS X中可用dtruss \(需要root\)替代



例：



sudo dtruss df -h



其它参数：



-p PID \# examine this PID

-n name \# examine this process name

-t syscall \# examine this syscall only

-a \# print all details

-c \# print syscall counts

-d \# print relative times \(us\)

-e \# print elapsed times \(us\)

-f \# follow children

-l \# force printing pid/lwpid

-o \# print on cpu times

-s \# print stack backtraces

-L \# don’t print pid/lwpid

-b bufsize \# dynamic variable buf size

eg,

dtruss df -h \# run and examine “df -h”

dtruss -p 1871 \# examine PID 1871

dtruss -n tar \# examine all processes called “tar”

dtruss -f test.sh \# run test.sh and follow children

