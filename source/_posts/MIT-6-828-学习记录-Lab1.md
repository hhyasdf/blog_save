---
title: MIT 6.828 学习记录 Lab1
date: 2016-11-21
categories: MIT 6.828 学习记录
tag: OS
---

&nbsp;&nbsp;&nbsp;&nbsp;Lab 1分为三个部分，第一个部分主要讲x86的汇编语言、QEMU（x86虚拟机）和PC的开机启动步骤。第二个部分测试6.828内核的boot loader（在lib的boot目录下）。第三个部分探究6.828内核最初的模板，JOS，在lib的kernel目录下。（默认Linux操作系统）

&nbsp;&nbsp;&nbsp;&nbsp;本课程用到的文件用Git将**https://pdos.csail.mit.edu/6.828/2016/jos.git** clone至本地即可。当然，你也需要安装QEMU（推荐编译安装）。

*************************************
### Part 1 : PC Bootstrap###

&nbsp;&nbsp;&nbsp;&nbsp;第一个练习的目的是向你介绍x86汇编语言和PC引导过程，然后让你开始学会QEMU和QEMU/GDB调试。在这个lab里你不需要写任何代码，但是无论如何你应该浏览并理解它，准备好回答下面列出的问题。

* Getting Started with x86 assembly

&nbsp;&nbsp;&nbsp;&nbsp;如果你并不熟悉x86汇编语言，你在这个课程中将会很快地熟悉它！[PC Assembly Language Book](https://pdos.csail.mit.edu/6.828/2016/readings/pcasm-book.pdf) 是一个不错的开始。

&nbsp;&nbsp;&nbsp;&nbsp;Warning: Unfortunately the examples in the book are written for the NASM assembler, whereas we will be using the GNU assembler. NASM uses the so-called Intel syntax while GNU uses the AT&T syntax. While semantically equivalent, an assembly file will differ quite a lot, at least superficially, depending on which syntax is used. Luckily the conversion between the two is pretty simple, and is covered in Brennan’s Guide to Inline Assembly.

>Exercise 1. Familiarize yourself with the assembly language materials available on [the 6.828 reference page](https://pdos.csail.mit.edu/6.828/2016/reference.html). You don’t have to read them now, but you’ll almost certainly want to refer to some of this material when reading and writing x86 assembly.

>We do recommend reading the section “The Syntax” in [Brennan’s Guide to Inline Assembly](http://www.delorie.com/djgpp/doc/brennan/brennan_att_inline_djgpp.html). It gives a good (and quite brief) description of the AT&T assembly syntax we’ll be using with the GNU assembler in JOS.

&nbsp;&nbsp;&nbsp;&nbsp;Certainly the definitive reference for x86 assembly language programming is Intel’s instruction set architecture reference, which you can find on [the 6.828 reference page](https://pdos.csail.mit.edu/6.828/2016/reference.html) in two flavors: an HTML edition of the old 80386 Programmer’s Reference Manual, which is much shorter and easier to navigate than more recent manuals but describes all of the x86 processor features that we will make use of in 6.828; and the full, latest and greatest [IA-32 Intel Architecture Software Developer’s Manuals](http://www.intel.com/content/www/us/en/processors/architectures-software-developer-manuals.html) from Intel, covering all the features of the most recent processors that we won’t need in class but you may be interested in learning about. An equivalent (and often friendlier) set of manuals is [available from AMD](http://developer.amd.com/documentation/guides/Pages/default.aspx#manuals). Save the Intel/AMD architecture manuals for later or use them for reference when you want to look up the definitive explanation of a particular processor feature or instruction.

* 模拟运行x86

&nbsp;&nbsp;&nbsp;&nbsp;本课程用**QEMU**模拟x86系统的运行，用虚拟机的好处在于debug：你可以结合QEMU和**GDB**在虚拟的x86系统中设置断点。我们也会用这种方法逐步地研究系统的早期启动过程。

&nbsp;&nbsp;&nbsp;&nbsp;首先需要进行”Sofrware Setup”：编译build一个6.828 boot loader 和 kernel。

	athena% cd lab
    athena% make
    + as kern/entry.S
    + cc kern/entrypgdir.c
    + cc kern/init.c
    + cc kern/console.c
    + cc kern/monitor.c
    + cc kern/printf.c
    + cc kern/kdebug.c
    + cc lib/printfmt.c
    + cc lib/readline.c
    + cc lib/string.c
    + ld obj/kern/kernel
    + as boot/boot.S
    + cc -Os boot/main.c
    + ld boot/boot
    boot block is 380 bytes (max 510)
    + mk obj/kern/kernel.img

&nbsp;&nbsp;&nbsp;&nbsp;如果你得到像“undefined reference to ‘_\_udivdi3’”的报错，你的系统中可能没有32-bit gcc multilib，你可以尝试安装gcc-multilib包。

&nbsp;&nbsp;&nbsp;&nbsp;现在你已经可以运行QEMU了，通过将**obj/kern/kernel.img**作为虚拟机中的“virtual hard disk”，我们的虚拟机中的硬盘镜像将会包含我们的boot loader(obj/boot/boot)和我们的kernel(obj/kernel)。

	athena% make qemu

&nbsp;&nbsp;&nbsp;&nbsp;这条用选项执行QEMU的操作会设置虚拟硬盘以及直接串口输出到终端。QEMU窗口中将会出现一些打印文本。

    Booting from Hard Disk...
    6828 decimal is XXX octal!
    entering test_backtrace 5
    entering test_backtrace 4
    entering test_backtrace 3
    entering test_backtrace 2
    entering test_backtrace 1
    entering test_backtrace 0
    leaving test_backtrace 0
    leaving test_backtrace 1
    leaving test_backtrace 2
    leaving test_backtrace 3
    leaving test_backtrace 4
    leaving test_backtrace 5
    Welcome to the JOS kernel monitor!
    Type 'help' for a list of commands.
    K>

&nbsp;&nbsp;&nbsp;&nbsp;**‘Booting from Hard Disk…’**之后的内容都是由我们的 JOS 简单内核打印的；K> 是JOS内核中包含的一个小交互控制程序打印的提示符，这些文本也会出现在你运行QEMU的shell窗口中。这是因为为了测试和实验室分级，我们设置了JOS内核将它的控制台（console）输出不仅写至虚拟VGA display（像在QEMU窗口中看到的一样），还写入虚拟机的串口（QEMU定期输出它到自己的标准输出）。同样JOS内核会接受键盘和串口的输入，所以我们既可以在虚拟VGA display窗口中也可以在terminal中输入命令运行QEMU。你也可以选择**make qemu-nox**不启动虚拟VGA而只用串口控制，如果你是SSH远程登录linux的话这样会方便一些。

&nbsp;&nbsp;&nbsp;&nbsp;JOS 的kernel monitor中只用两条命令可以使用，**help** 和 **kerninfo**。

    K> help
    help - display this list of commands
    kerninfo - display information about the kernel
    K> kerninfo
    Special kernel symbols:
      entry  f010000c (virt)  0010000c (phys)
      etext  f0101a75 (virt)  00101a75 (phys)
      edata  f0112300 (virt)  00112300 (phys)
      end    f0112960 (virt)  00112960 (phys)
    Kernel executable memory footprint: 75KB
    K>

&nbsp;&nbsp;&nbsp;&nbsp;help命令的功能很显然，我们主要讨论kerninfo命令的输出的意义。需要了解的一点是，kernel monitor是“直接”存在虚拟PC的磁盘上的，这也就意味着你可以直接将obj/kern/kernel.img的内容拷贝到一个真实磁盘的前几个扇区里，将它插入一台真正的电脑并打开，你就可以在显示屏上看到和在QEMU窗口上一样的输出。（虽然这样做会覆盖磁盘上的MBR和第一个分区的开始部分，导致磁盘上原来的信息全部丢失）

* The PC’s Physical Address Space

&nbsp;&nbsp;&nbsp;&nbsp;我们现在将会深入了解计算机启动的更多的细节。一个PC的物理地址空间固定地有以下大概布局：

    +------------------+  <- 0xFFFFFFFF (4GB)
    |      32-bit      |
    |  memory mapped   |
    |     devices      |
    |                  |
    /\/\/\/\/\/\/\/\/\/\

    /\/\/\/\/\/\/\/\/\/\
    |                  |
    |      Unused      |
    |                  |
    +------------------+  <- depends on amount of RAM
    |                  |
    |                  |
    | Extended Memory  |
    |                  |
    |                  |
    +------------------+  <- 0x00100000 (1MB)
    |     BIOS ROM     |
    +------------------+  <- 0x000F0000 (960KB)
    |  16-bit devices, |
    |  expansion ROMs  |
    +------------------+  <- 0x000C0000 (768KB)
    |   VGA Display    |
    +------------------+  <- 0x000A0000 (640KB)
    |                  |
    |    Low Memory    |
    |                  |
    +------------------+  <- 0x00000000

&nbsp;&nbsp;&nbsp;&nbsp;首先的电脑，基于16-bit的Intel 8088处理器，只能寻址1MB物理内存。因此早期电脑的物理地址空间从0x00000000开始，在0x000FFFFF而不是0xFFFFFFFF结束。这640KB被叫做“Low Memory”的区域是早期电脑能用的RAM;事实上非常早的电脑只能使用RAM的16KB、32KB或64KB。

&nbsp;&nbsp;&nbsp;&nbsp;从0x000A0000到0xFFFFFFFF的384KB区域被硬件保留作为特殊用途比如说视频播放缓冲区和固定存储器里的固件。保留区域中最重要的部分是the Basic Input/Output System (BIOS)，它占据了从0x000F0000到0x000FFFFF的64KB空间。早期的电脑中BIOS被储存在ROM中，但是现在电脑将BIOS储存在可更新闪存里。BIOS负责执行基本系统的安装比如说激活显卡和检查已安装的内存大小。在执行完基本系统的安装后，BIOS会从合适的位置（软盘、硬盘、CD-ROM或者网络）导入操作系统，然后将机器的控制转移给操作系统。

&nbsp;&nbsp;&nbsp;&nbsp;在Intel的80286和80386处理器（分别支持16MB和4GB物理内存空间）最终“打破兆字节障碍”后，PC设计者们仍然保留了原始的1MB低字节物理地址空间的原始结构来保证现存软件的向后兼容性。因此，现代PC在物理内存中有一个“洞”（从0x000A0000到0x00100000），将RAM分为”low” or “conventional memory”（the first 640KB）和”extended memory”（everything else）。另外，在PC的32-bit物理地址空间最高处，所有物理RAM之上，现在通常被BIOS保留提供给PCI设备(32位内存映射设备)。

&nbsp;&nbsp;&nbsp;&nbsp;近来的x86处理器可以访问物理RAM中4GB甚至更大的空间，因此RAM可以延伸至0xFFFFFFFF之上更远的地址。这样的话，BIOS必须准备在系统RAM的顶部32-bit可寻址空间留下第二个“洞”，来为32-bit的设备留下空间。因为设计的限制，JOS只能使用PC物理内存的首256MB，因此现在我们会假设PC只有32-bit物理寻址空间。但是多年来，处理复杂的物理地址空间和硬件组织的其他方面的演变是操作系统发展的重要实际挑战之一。

* The ROM BIOS

&nbsp;&nbsp;&nbsp;&nbsp;在这个部分，你会用QEMU的调试功能研究一个IA-32兼容的电脑怎样启动。

&nbsp;&nbsp;&nbsp;&nbsp;打开两个命令行窗口，在其中一个输入**make qemu-gdb**，将启动QEMU，但QEMU会停止在处理器执行第一条指令之前，然后等待GDB的调试连接。在第二个命令行窗口（lab目录下），运行**make gdb**。你可以看到下面的输出：

    athena% make gdb
    GNU gdb (GDB) 6.8-debian
    Copyright (C) 2008 Free Software Foundation, Inc.
    License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
    This is free software: you are free to change and redistribute it.
    There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
    and "show warranty" for details.
    This GDB was configured as "i486-linux-gnu".
    + target remote localhost:26000
    The target architecture is assumed to be i8086
    [f000:fff0] 0xffff0:    ljmp   $0xf000,$0xe05b
    0x0000fff0 in ?? ()
    + symbol-file obj/kern/kernel
    (gdb)

&nbsp;&nbsp;&nbsp;&nbsp;我们提供了一个.gdbinit文件设置GDB来debug启动早期使用的16-bit代码，并且使它连接到监听QEMU。

&nbsp;&nbsp;&nbsp;&nbsp;你会见到有一行输出：

    [f000:fff0] 0xffff0:    ljmp   $0xf000,$0xe05b

&nbsp;&nbsp;&nbsp;&nbsp;这是GDB分解的第一条被执行的指令。从这个输出你可以总结到一些东西：

* 这台IBM PC从物理地址0x000FFFF0开始执行，这是ROM BIOS保留的64KB区域的顶部。
* 这台PC开始执行的虚拟地址的（段寄存器）CS=0xF000，（指令寄存器）IP=0xFFF0。
* 第一条被执行的是jmp指令，它会jmp到段地址CS=0xF000，IP=0xe05b。

&nbsp;&nbsp;&nbsp;&nbsp;为什么QEMU会这样启动？这就是IBM用在他们原始PC上的Intel 8088处理器的设计。因为一台PC上的BIOS是固定在物理地址0x000F0000-0x000FFFFF的范围，这个设计保证了BIOS在通电或系统重启后首先得到控制权，这是关键性的，因为刚刚通电时机器的RAM中没有任何别的处理器能执行的软件。QEMU虚拟机有它自己的BIOS，它将其放置在处理器的模拟物理空间的相同地址。一旦处理器重置，（模拟）处理器进入实模式并将CS设置为0xF000，将IP设为0xFFF0，因此这次执行开始于这个（CS:IP）段地址。段地址0xF000：FFF0是怎样转换成物理地址的呢？

&nbsp;&nbsp;&nbsp;&nbsp;为了回答这个 问题我们需要了解一些**实地址模式**的寻址。在实地址模式中（电脑开机时的状态），地址是根据式子： physical address = 16 * segment + offset，转化的。因此，当PC将CS寄存器设为0xF000、IP设为0xFFF0时，实际物理地址为：

    16 * 0xf000 + 0xfff0   # in hex multiplication by 16 is
    = 0xf0000 + 0xfff0     # easy--just append a 0.
    = 0xffff0

&nbsp;&nbsp;&nbsp;&nbsp;0xFFFF0是在BIOS所在位置末尾（0x100000）之前16字节的地方。因此，我们自然也不会奇怪BIOS做的第一件事就是jmp到BIOS内容中更靠前面一些的地方，毕竟只有16位做不了什么。

>Use GDB’s si (Step Instruction) command to trace into the ROM BIOS for a few more instructions, and try to guess what it might be doing. You might want to look at [Phil Storrs I/O Ports Description](http://web.archive.org/web/20040404164813/members.iweb.net.au/~pstorr/pcbook/book2/book2.htm), as well as other materials on the [6.828 reference materials page](https://pdos.csail.mit.edu/6.828/2016/reference.html). No need to figure out all the details - just the general idea of what the BIOS is doing first.

&nbsp;&nbsp;&nbsp;&nbsp;当BIOS运行时，它设置一个中断描述符表并初始化各种各样的设备比如说VGA display。这就是从QEMU窗口中看到的“Starting SeaBIOS”的来源。

&nbsp;&nbsp;&nbsp;&nbsp;在初始化PCI总线和所有BIOS知道的重要设备之后，它会搜索一个可以启动的设备比如说软盘、硬盘或CD-ROM。最后，当它找到一个可启动的盘时，BIOS会从盘中读取引导加载程序（boot loader）然后将控制权转移给它。

********************************************
### Part 2 : The Boot Loader###

&nbsp;&nbsp;&nbsp;&nbsp;PC的软盘和硬盘被划分为一些大小为512字节的叫做**扇区**的区域。一个扇区是盘的**最小传输粒度**：每个读或写操作的作用对象必须为一个或多个扇区大小并且要和扇区边界对齐。如果盘是可以启动的，第一个扇区就叫做**引导扇区**（boot sector），因为这是引导加载程序代码所在的位置。当BIOS找到一个可启动的软盘或者硬盘，它会将这512字节的引导扇区加载进内存中物理地址0x7C00到0x7DFF的位置，然后用一个jmp指令将CS:IP设置为0000:7C00，再将控制传递给引导加载程序。像BIOS加载地址，这些地址是相当任意的，但它们对PC是固定和标准化的。

&nbsp;&nbsp;&nbsp;&nbsp;在PC的演化过程中，从CD-ROM中启动的能力出现得很晚，因此，PC架构师利用机会重新思考了引导过程。因而，现代BIOS从CD-ROM启动的方式要更加复杂一些（也更加强大）。CD-ROM的单个扇区大小为2048字节而不是512字节，并且在转移控制给引导加载程序之前，BIOS可以从CD-ROM中加载一个大得多的启动镜像到内存中（不止一个扇区）。For more information, see the [El Torito” Bootable CD-ROM Format Specification](https://pdos.csail.mit.edu/6.828/2016/readings/boot-cdrom.pdf).

&nbsp;&nbsp;&nbsp;&nbsp;然而，对于6.828的课程，我们会用传统的硬盘启动原理。这意味着我们的引导加载程序必须适合512字节。引导加载程序由一个汇编语言源文件（boot/boot.S）和一个C语言源文件（boot/main.c）组成。仔细地阅读这些源文件并确定你理解发生了什么。引导加载程序一定要具有两个主要功能：

1. 第一，引导加载程序要将处理器从从实模式切换到32-bit保护模式，因为只有在这个模式下软件可以访问处理器内物理地址空间1MB以上的所有内存。保护模式在[《PC Assembly Language》](https://pdos.csail.mit.edu/6.828/2016/readings/pcasm-book.pdf)中的1.2.7和1.2.8章中有简单的描述，在Intel architecture manuals中有详细描述。现在你只需要知道在保护模式中分段地址（段：位移 对）与物理地址的转化和实模式是不一样的，并且在转换之后偏移是32位而不是16位。
2. 第二，引导加载程序通过x86的特殊I/O指令直接访问IDE磁盘设备寄存器从硬盘中读取内核。如果你想要了解更多的关于这x86的特殊I/O指令，你可以查看[the 6.828 reference page](https://pdos.csail.mit.edu/6.828/2016/reference.html)的“IDE hard drive controller”部分。这堂课里你不需要了解太多有关特定设备的编程：编写设备驱动程序实际上是OS开发的一个非常重要的部分，但是从概念和架构的角度它也是最无趣的部分之一。

&nbsp;&nbsp;&nbsp;&nbsp;在你理解了引导加载程序代码之后，阅读obj/boot/boot.asm文件。这文件是GNUmakefile在编译引导加载程序之后创建的引导加载程序的反汇编。这个反汇编文件可以让我们很简单地知道所有引导加载程序的代码在物理内存中的准确位置，并且我们也可以更容易在GDB中通过单步调试boot loader理解引导加载的过程。同样的，obj/kern/kernel.asm包含了JOS的反汇编，这在调试中是很有用的。

&nbsp;&nbsp;&nbsp;&nbsp;在GDB中你可以用**b**命令设置地址断点。例如，b *0x7c00 在地址位0x7C00处设置了一个断点，然后你可以用c和si命令继续执行：**c**命令会使QEMU执行到下一个断点处，**si N**命令会接着一次执行N条命令。

&nbsp;&nbsp;&nbsp;&nbsp;要检查内存中的指令（除了立即执行的下一条命令，GDB自动打印），你可以用**x/i**命令，这条命令的格式为**x/Ni ADDR**，其中的N是需要反汇编的连续指令的数量，ADDR是开始反汇编的内存地址。

>Exercise 3. Take a look at the [lab tools guide](https://pdos.csail.mit.edu/6.828/2016/labguide.html), especially the section on GDB commands. Even if you’re familiar with GDB, this includes some esoteric GDB commands that are useful for OS work.

>Set a breakpoint at address 0x7c00, which is where the boot sector will be loaded. Continue execution until that breakpoint. Trace through the code in boot/boot.S, using the source code and the disassembly file obj/boot/boot.asm to keep track of where you are. Also use the x/i command in GDB to disassemble sequences of instructions in the boot loader, and compare the original boot loader source code with both the disassembly in obj/boot/boot.asm and GDB.

>Trace into bootmain() in boot/main.c, and then into readsect(). Identify the exact assembly instructions that correspond to each of the statements in readsect(). Trace through the rest of readsect() and back out into bootmain(), and identify the begin and end of the for loop that reads the remaining sectors of the kernel from the disk. Find out what code will run when the loop is finished, set a breakpoint there, and continue to that breakpoint. Then step through the remainder of the boot loader.

&nbsp;&nbsp;&nbsp;&nbsp;你需要能够回答以下问题：

1. 处理器何时开始执行32位代码？是什么原因导致从16位模式切换到32位模式？
2. 引导加载程序执行的最后一条指令是什么，它刚刚加载的内核的第一条指令是什么？
3. 内核的第一条指令在哪？
4. 引导加载程序是怎样决定它需要读取多少个扇区才能从磁盘中取得整个内核的？它从哪获得的信息？ 


  
* Loading the Kernel

&nbsp;&nbsp;&nbsp;&nbsp;我们现在将会了解引导加载程序里C语言部分的更多细节（boot/main.c），但在这之前我们可以预习一下C语言基础，

>Exercise 4. Read about programming with pointers in C. The best reference for the C language is The C Programming Language by Brian Kernighan and Dennis Ritchie (known as 'K&R'). We recommend that students purchase this book (here is an [Amazon Link](http://www.amazon.com/C-Programming-Language-2nd/dp/0131103628/sr=8-1/qid=1157812738/ref=pd_bbs_1/104-1502762-1803102?ie=UTF8&s=books)) or find one of [MIT's 7 copies](http://library.mit.edu/F/AI9Y4SJ2L5ELEE2TAQUAAR44XV5RTTQHE47P9MKP5GQDLR9A8X-10422?func=item-global&doc_library=MIT01&doc_number=000355242&year=&volume=&sub_library=).

>Read 5.1 (Pointers and Addresses) through 5.5 (Character Pointers and Functions) in K&R. Then download the code for [pointers.c](https://pdos.csail.mit.edu/6.828/2016/labs/lab1/pointers.c), run it, and make sure you understand where all of the printed values come from. In particular, make sure you understand where the pointer addresses in lines 1 and 6 come from, how all the values in lines 2 through 4 get there, and why the values printed in line 5 are seemingly corrupted.

>There are other references on pointers in C (e.g., [A tutorial by Ted Jensen](https://pdos.csail.mit.edu/6.828/2016/readings/pointers.pdf) that cites K&R heavily), though not as strongly recommended.

>Warning: Unless you are already thoroughly versed in C, do not skip or even skim this reading exercise. If you do not really understand pointers in C, you will suffer untold pain and misery in subsequent labs, and then eventually come to understand them the hard way. Trust us; you don't want to find out what "the hard way" is.

&nbsp;&nbsp;&nbsp;&nbsp;为了更好地理解 boot/main.c 你需要知道**ELF二进制**是什么。当你编译链接一个像JOS内核一样的C语言程序时，编译器会将C语言源文件（.c）转换成以硬件所期望的二进制格式编码的汇编语言指令的目标文件（.o）。然后链接器将所有的编译生成的目标文件链接为一个单独的二进制映像（像 kern/kernel）。而这个二进制映像就是 ELF 格式，即“Executable and Linkable Format”。

&nbsp;&nbsp;&nbsp;&nbsp;Full information about this format is available in [the ELF specification](https://pdos.csail.mit.edu/6.828/2016/readings/elf.pdf) on [our reference page](https://pdos.csail.mit.edu/6.828/2016/reference.html), but you will not need to delve very deeply into the details of this format in this class. Although as a whole the format is quite powerful and complex, most of the complex parts are for supporting dynamic loading of shared libraries, which we will not do in this class. The [Wikipedia page](http://en.wikipedia.org/wiki/Executable_and_Linkable_Format) has a short description.

&nbsp;&nbsp;&nbsp;&nbsp;基于本课程的目的，你可以把ELF可执行文件看成带有加载信息的头部后面跟着几个程序段的整体（每个程序段是要在指定地址加载到内存中的连续的代码块或数据）。引导加载程序不会调整它的代码或数据，它直接将ELF文件加载进内存然后执行。

&nbsp;&nbsp;&nbsp;&nbsp;一个ELF二进制文件以固定长度的ELF头部开始，紧跟着一个变长的程序头部，这个头部会列出每个要加载的程序段。ELF头部的C语言定义在inc/elf.h文件里。我们所感兴趣的程序段有：

* .text: The program's executable instructions.
* .rodata: Read-only data, such as ASCII string constants produced by the C compiler. (We will not bother setting up the hardware to prohibit writing, however.)
* .data: The data section holds the program's initialized data, such as global variables declared with initializers like int x = 5;

&nbsp;&nbsp;&nbsp;&nbsp;当链接器计算程序在内存中的布局时，它会为未初始化的全局变量在内存中保存一个叫 .bss 的程序段（跟在 .data后面），像 int x;。C语言中要求未初始2化的全局变量的初始值为0。因此，在ELF二进制文件中不需要保存 .bss 的内容; 相反，链接器只记录.bss程序段的地址和大小。加载程序或程序本身必须将.bss程序段排列为0。

&nbsp;&nbsp;&nbsp;&nbsp;要测试JOS内核中所有程序段的名字、大小和链接地址，你可以输入：

	athena% i386-jos-elf-objdump -h obj/kern/kernel

&nbsp;&nbsp;&nbsp;&nbsp;如果你的计算机像大多数现代Linuxen和BSD默认使用ELF工具链，你也可以用 objdump 代替 i386-jos-elf-objdump。

&nbsp;&nbsp;&nbsp;&nbsp;从命令输出中你会看到比我们列出的更多的程序段，但那些对于我们的目的来说并不重要。其他的程序段大多数保存的是调试信息，它们通常包括在程序的可执行文件中，但是不会由程序加载器加载到内存中。

&nbsp;&nbsp;&nbsp;&nbsp;在命令的输出中我们尤其要注意的是 .text 程序段的“VMA”（or link address）和“LMA”（or load address）。一个程序段的 load address（加载地址） 是程序段被加载到内存中的内存地址。

&nbsp;&nbsp;&nbsp;&nbsp;一个程序段的 link address（链接地址） 是程序段预期要执行的内存地址。链接器会将 link address 用不同的方式编码成二进制。例如当代码需要一个全局变量的地址时，如果它从不被链接的地址执行，二进制将不会工作。（可以生成不包含任何这种绝对地址的位置无关代码。 这被现代共享库广泛使用，但它具有性能和复杂性成本，因此我们不会在6.828中使用它。）

&nbsp;&nbsp;&nbsp;&nbsp;通常，链接地址和加载地址是相同的。例如，我们来看看引导加载程序的 .text 程序段：

	athena% i386-jos-elf-objdump -h obj/boot/boot.out

&nbsp;&nbsp;&nbsp;&nbsp;引导加载程序用 ELF 的程序头部来决定如何加载程序段。程序头部明确了ELF对象中的哪些部分会被加载进内存以及它们所在的内存地址。你可以观察到程序头部，输入：

	athena% i386-jos-elf-objdump -x obj/kern/kernel

&nbsp;&nbsp;&nbsp;&nbsp;在 objdump 的输出中程序头部在“Program Headers”下方被列出。被标记为“LOAD”的部分就是需要被加载到内存里的ELF对象区域。每个程序头部还给出了其他信息像虚拟地址（“vaddr”）、物理地址（“paddr”）和加载区域大小（“memsz”和“filesz”）。

&nbsp;&nbsp;&nbsp;&nbsp;回到boot / main.c中，每个程序头部的ph-> p_pa字段包含段的目的地物理地址（在这种情况下，它实际上是一个物理地址，尽管ELF规范对该字段的实际含义含糊不清） 。

&nbsp;&nbsp;&nbsp;&nbsp;BIOS将引导扇区加载到从地址0x7c00开始的内存中，因此0x7c00就是引导扇区的加载地址，又因为这也是引导扇区开始执行的地址，所以这也是它的链接地址。我们通过在 boot/Makefrag 中将 -Ttext 0x7C00 传递给链接器来设置链接地址，因此链接器将在生成的代码中产生正确的内存地址。

>Exercise 5. Trace through the first few instructions of the boot loader again and identify the first instruction that would "break" or otherwise do the wrong thing if you were to get the boot loader's link address wrong. Then change the link address in boot/Makefrag to something wrong, run make clean, recompile the lab with make, and trace into the boot loader again to see what happens. Don't forget to change the link address back and make clean again afterward!

&nbsp;&nbsp;&nbsp;&nbsp;回到我们之前看到的内核的加载地址和链接地址。与引导加载程序不同的是，内核的这两个地址不一样：内核告诉引导程序将其加载到低内存地址中（1MB），但它希望从高内存地址开始执行。我们将在下一个部分研究这个工作怎样完成。

&nbsp;&nbsp;&nbsp;&nbsp;除开段信息，在 ELF 头部中还有个重要的域叫做 e_entry。这个字段保存了程序入口的链接地址：也就是程序正文段（.text）中程序开始执行的内存地址。你可以观察到内核的入口点，通过输入：

	athena% i386-jos-elf-objdump -f obj/kern/kernel
	
&nbsp;&nbsp;&nbsp;&nbsp;现在你应该可以理解 boot/main.c 中的微型ELF头部了。它将内核的每一个段从磁盘读入内存中段的加载地址然后跳转到内核的入口点执行内核。

>Exercise 6. We can examine memory using GDB's x command. The GDB manual has full details, but for now, it is enough to know that the command x/Nx ADDR prints N words of memory at ADDR. (Note that both 'x's in the command are lowercase.) Warning: The size of a word is not a universal standard. In GNU assembly, a word is two bytes (the 'w' in xorw, which stands for word, means 2 bytes).

>Reset the machine (exit QEMU/GDB and start them again). Examine the 8 words of memory at 0x00100000 at the point the BIOS enters the boot loader, and then again at the point the boot loader enters the kernel. Why are they different? What is there at the second breakpoint? (You do not really need to use QEMU to answer this question. Just think.)


*************************************
### Part 3 : The Kernel ###

&nbsp;&nbsp;&nbsp;&nbsp;在这一部分，我们将会观察微型JOS内核中的更多的细节（你也会在这一部分中写一些代码）。和引导加载程序一样，内核的开始也是一些汇编代码，用来搭建C语言代码的运行环境。

* Using virtual memory to work around position dependence

&nbsp;&nbsp;&nbsp;&nbsp;我们之前发现，相比于引导加载程序的加载地址和链接地址完全一样，内核的两个地址则不一致。

&nbsp;&nbsp;&nbsp;&nbsp;为了将处理器虚拟内存中的低内存地址空间留给用户程序运行，操作系统内核通常会被链接和运行在（非常）高虚拟内存地址空间，比如说 0xf0100000。这种安排的原因我们会在 Lab2 了解德更加清楚。

&nbsp;&nbsp;&nbsp;&nbsp;许多机器并没有能达到 0xf0100000 的大内存，因此我们不能依赖于将内核保存到物理内存的 0xf0100000 处。相反，我们会用处理器的内存管理硬件将虚拟地址的 0xf0100000 （内核希望开始运行的内存地址）映射到物理内存的 0x00100000 （内核加载到物理内存中的地方）。这样，尽管内核处在能为用户进程留出足够内存空间的高虚拟内存地址，但它实际将被加载到PC物理内存中的 1MB 处（就在BIOS ROM上面）。这种方式要求 PC 至少有几MB的物理内存（这样才能达到 0x00100000），而似乎自1990年后生产的 PC 都可以达到这一条件。

&nbsp;&nbsp;&nbsp;&nbsp;在 Lab2 中，我们将映射整个底部256MB的PC的物理地址空间（ from physical addresses 0x00000000 through 0x0fffffff, to virtual addresses 0xf0000000 through 0xffffffff respectively）。你现在应该明白为什么 JOS 只能使用物理内存的低256MB空间了。

&nbsp;&nbsp;&nbsp;&nbsp;现在，我们将映射前4MB的物理内存，这将足以让我们开始运行。我们通过在 kern/entrypgdir.c 中使用手写的、静态初始化的页目录（page directory）和页表（page table）来实现。但现在，你并不需要理解它工作的细节，只需要知道它的作用。

&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;