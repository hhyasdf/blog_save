---
title: MIT 6.828 /boot/boot.S /boot/main.c 解析
categories: MIT 6.828 学习记录
date: 2016-12-02
tag: OS
---

首先总结一些基本问题（掌握的还不是很扎实）：

* 处理器的位数：处理器一次能处理的（单个）数据长度（位数）,即数据总线长度。
* 16位代码和32位代码的不同：
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;指令集不同，汇编指令对应的二进制机器码可能不同（虽然as通常用来编写纯32位的80x86代码，但是1995年后它对编写运行于实模式或16位保护模式的代码也提供有限的支持。为了让as汇编时产生16位代码，需要在运行于16位模式的指令语句之前添加汇编命令”.code16”，并且使用汇编命令”.code32”让as汇编器切换回32位代码汇编方式。as不区分16位和32位汇编语句，在16位和32位模式下每条指令的功能完全一样而与模式无关。as总是为汇编语句产生32位的指令代码而不管指令将运行在16位还是32位模式下。如果使用汇编命令”.code16”让as处于16位模式下，那么as会自动为所有指令加上一个必要的操作数宽度前缀而让指令运行在16位模式。请注意，因为as为所有指令添加了额外的地址和操作数宽度前缀，所以汇编产生的代码长度和性能会受到影响。）

* boot/boot.S

        #include <inc/mmu.h>  

        # Start the CPU: switch to 32-bit protected mode, jump into C.  
        # The BIOS loads this code from the first sector of the hard disk into  
        # memory at physical address 0x7c00 and starts executing in real mode  
        # with %cs=0 %ip=7c00.  
        # 启动CPU，切换到32位保护模式，跳转到C代码  
        # BIOS从硬盘的第一个扇区加载这个代码到  
        # 物理内存地址为0x7c00的地方，cs=0,ip=7c00  

        # 下面的3条.set指令类似于宏定义  
                                         # 内核代码段选择子  
        .set PROT_MODE_CSEG, 0x8         # kernel code segment selector  
                                         # 内核数据段选择子  
        .set PROT_MODE_DSEG, 0x10        # kernel data segment selector  
                                         # 保护模式使能标志  
        .set CR0_PE_ON,      0x1         # protected mode enable flag  # 

        # 定义一个全局名字start(不是_start的话应该在编译时有设置)  
        .globl start  
        start:  
                                      # CPU刚启动为16位模式
          .code16                     # Assemble for 16-bit mode  
                                      # 关中断  
          cli                         # Disable interrupts  
                                      # 清方向标志  
          cld                         # String operations increment  

          # Set up the important data segment registers (DS, ES, SS).  
          # 设置重要的数据段寄存器  
                                      # ax清零  
          xorw    %ax,%ax             # Segment number zero  
                                      # ds清零  
          movw    %ax,%ds             # -> Data Segment  
                                      # es清零  
          movw    %ax,%es             # -> Extra Segment  
                                      # ss清零  
          movw    %ax,%ss             # -> Stack Segment  

          # Enable A20:  
          #   For backwards compatibility with the earliest PCs, physical  
          #   address line 20 is tied low, so that addresses higher than  
          #   1MB wrap around to zero by default.  This code undoes this.  
          #   打开A20地址线  
          #   为了兼容早期的PC机，第20根地址线在实模式下不能使用  
          #   所以超过1MB的地址，默认就会返回到地址0，重新从0循环计数，  
          #   下面的代码打开A20地址线  

        seta20.1:  
          # 从0x64端口读入一个字节的数据到al中  
          inb     $0x64,%al               # Wait for not busy  
          # test指令可以当作and指令，只不过它不会影响操作数  
          testb   $0x2,%al  
          # 如果上面的测试中发现al的第2位为0，就不执行该指令  
          # 否则就循环检查  
          jnz     seta20.1  

          # 将0xd1写入到al中  
          movb    $0xd1,%al               # 0xd1 -> port 0x64  
          # 将al中的数据写入到端口0x64中  
          outb    %al,$0x64  

        seta20.2:  
          # 从0x64端口读取一个字节的数据到al中  
          inb     $0x64,%al               # Wait for not busy  
          # 测试al的第2位是否为0  
          testb   $0x2,%al  
          # 如果上面的测试中发现al的第2位为0，就不执行该指令  
          # 否则就循环检查  
          jnz     seta20.2  

          # 将0xdf写入到al中  
          movb    $0xdf,%al               # 0xdf -> port 0x60  
          # 将al中的数据写入到0x60端口中  
          outb    %al,$0x60  

          # Switch from real to protected mode, using a bootstrap GDT  
          # and segment translation that makes virtual addresses   
          # identical to their physical addresses, so that the   
          # effective memory map does not change during the switch.  

          # 将全局描述符表描述符加载到全局描述符表寄存器  
          lgdt    gdtdesc  

          # cr0中的第0位为1表示处于保护模式  
          # cr0中的第0位为0，表示处于实模式  
          # 把控制寄存器cr0加载到eax中  
          movl    %cr0, %eax  
          # 将eax中的第0位设置为1  
          orl     $CR0_PE_ON, %eax  
          # 将eax中的值装入cr0中  
          movl    %eax, %cr0  

          # Jump to next instruction, but in 32-bit code segment.  
          # Switches processor into 32-bit mode.  
          # 跳转到32位模式中的下一条指令  
          # 将处理器切换为32位工作模式  
          # 下面这条指令执行的结果会将$PROT_MODE_CSEG加载到cs中，cs对应的高速缓冲存储器会加载代码段描述符  
          # 同样将$protcseg加载到ip中  
          ljmp    $PROT_MODE_CSEG, $protcseg  

          .code32                     # Assemble for 32-bit mode  
        protcseg:  
          # Set up the protected-mode data segment registers  
          # 设置保护模式下的数据寄存器  
          # 将数据段选择子装入到ax中  
          movw    $PROT_MODE_DSEG, %ax    # Our data segment selector  
          # 将ax装入到其他数据段寄存器中，在装入的同时，  
          # 数据段描述符会自动的加入到这些段寄存器对应的高速缓冲寄存器中  
          movw    %ax, %ds                # -> DS: Data Segment  
          movw    %ax, %es                # -> ES: Extra Segment  
          movw    %ax, %fs                # -> FS  
          movw    %ax, %gs                # -> GS  
          movw    %ax, %ss                # -> SS: Stack Segment  

          # Set up the stack pointer and call into C.  
          # 设置栈指针，并且调用c函数  
          # 覆盖（删除）之前的代码段
          movl    $start, %esp  
          # 调用main.c中的bootmain函数  
          call bootmain  

          # If bootmain returns (it shouldn't), loop.  
          # 如果bootmain返回的话，就一直循环  
        spin:  
          jmp spin  

          # Bootstrap GDT  
          # 强制4字节对齐  
        .p2align 2                                # force 4 byte alignment  

          # 全局描述符表  
        gdt:  
          SEG_NULL              # null seg  
          # 代码段描述符  
          SEG(STA_X|STA_R, 0x0, 0xffffffff) # code seg  
          # 数据段描述符  
          SEG(STA_W, 0x0, 0xffffffff)           # data seg  

          # 全局描述符表对应的描述符  
        gdtdesc:  
          .word   0x17                            # sizeof(gdt) - 1  
          .long   gdt                             # address gdt  

********************************** 

* boot/main.c

        #include <inc/x86.h>  
        #include <inc/elf.h>  

        /*这是一个简单粗略的boot loader，它唯一的工作就是 
        从硬盘的第一个扇区启动格式为ELF的内核镜像 

        硬盘布局 
        这个程序（包括boot.S和main.c）组成了bootloader， 
        它应该存储在硬盘的第一个扇区 

        第二个扇区存储着内核映像 

        内核映像必须为ELF格式的 

        启动步骤 
        当CPU启动时，它加载BIOS到内存中并且执行BIOS 

        BIOS程序初始化设备，设置中断例程，并且将启动装置（例如硬盘） 
        中的第一个扇区的内容加载到内存，并且跳转到那里 

        假设这个bootloader存储在硬盘的第一个扇区，这个代码从BIOS接收了CPU控制权 

        控制从boot.S文件开始--这个文件设置了保护模式和一个栈，这样 
        C代码就可以运行了，然后再调用bootmain() 

        这个文件中的bootmain函数接过控制权之后，读取内核文件并且跳转到内核*/  

        //扇区的大小为512  
        #define SECTSIZE    512  
        //将内核加载到内存的起始地址  
        #define ELFHDR      ((struct Elf *) 0x10000) // scratch space  

        //该函数的作用是读取一个节的内容，也就是读取一个扇区的内容  
        void readsect(void*, uint32_t);  
        //函数的作用是读取一个程序段  
        void readseg(uint32_t, uint32_t, uint32_t);  

        void  
        bootmain(void)  
        {  
            //定义了两个程序头表项指针  
            struct Proghdr *ph, *eph;  

            //将硬盘上从第一个扇区开始的4096个字节读到内存中地址为0x10000处  
            readseg((uint32_t) ELFHDR, SECTSIZE*8, 0);  

            //检查这是否是一个合法的ELF文件  
            if (ELFHDR->e_magic != ELF_MAGIC)  
                goto bad;  

            //找到第一程序头表项的起始地址  
            ph = (struct Proghdr *) ((uint8_t *) ELFHDR + ELFHDR->e_phoff);  
            //程序头表的结束位置  
            eph = ph + ELFHDR->e_phnum;  

            //将内核加载进入内存  
            for (; ph < eph; ph++)  
                //p_pa就是该程序段应该加载到内存中的位置  
                //读取一个程序段的数据到内存中  
                readseg(ph->p_pa, ph->p_memsz, ph->p_offset);  

            //开始执行内核  
            ((void (*)(void)) (ELFHDR->e_entry))();  

        bad:  
            outw(0x8A00, 0x8A00);  
            outw(0x8A00, 0x8E00);  
            while (1)  
                /* do nothing */;  
        }  

        //这个函数的作用是从ELF文件偏移为offset处，读取count个字节到内存地址为pa处  
        void  
        readseg(uint32_t pa, uint32_t count, uint32_t offset)  
        {  
            //段的结束地址  
            uint32_t end_pa;  

            //计算段的结束地址  
            end_pa = pa + count;  

            //将pa设置为512字节对齐的地方  
            pa &= ~(SECTSIZE - 1);  

            //将相对于ELF文件头的偏移量转换为扇区，ELF格式的内核文件存放在第一个扇区中  
            offset = (offset / SECTSIZE) + 1;  

            //开始读取该程序段的内容  
            while (pa < end_pa) {  
                //每次读取程序的一个节，即一个扇区  
                //也就是将offset扇区中的内容，读到物理地址为pa的地方  
                readsect((uint8_t*) pa, offset);  
                //将pa的值增加512字节  
                pa += SECTSIZE;  
                //读取下一个扇区  
                offset++;  
            }  
        }  

        void  
        waitdisk(void)  
        {  
            // wait for disk reaady  
            while ((inb(0x1F7) & 0xC0) != 0x40)  
                /* do nothing */;  
        }  

        void  
        readsect(void *dst, uint32_t offset)  
        {  
            // wait for disk to be ready  
            waitdisk();  

            outb(0x1F2, 1);     // count = 1  
            outb(0x1F3, offset);  
            outb(0x1F4, offset >> 8);  
            outb(0x1F5, offset >> 16);  
            outb(0x1F6, (offset >> 24) | 0xE0);  
            outb(0x1F7, 0x20);  // cmd 0x20 - read sectors  

            // wait for disk to be ready  
            waitdisk();  

            // read a sector  
            insl(0x1F0, dst, SECTSIZE/4);  
        }  


