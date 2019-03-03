# lec2：lab0 SPOC思考题

## **提前准备**
（请在上课前完成，option）

- 完成lec2的视频学习
- git pull ucore_os_lab, os_tutorial_lab, os_course_exercises  in github repos。这样可以在本机上完成课堂练习。
- 了解代码段，数据段，执行文件，执行文件格式，堆，栈，控制流，函数调用,函数参数传递，用户态（用户模式），内核态（内核模式）等基本概念。思考一下这些基本概念在不同操作系统（如linux, ucore,etc.)与不同硬件（如 x86, riscv, v9-cpu,etc.)中是如何相互配合来体现的。
- 安装好ucore实验环境，能够编译运行ucore labs中的源码。
- 会使用linux中的shell命令:objdump，nm，file, strace，gdb等，了解这些命令的用途。
- 会编译，运行，使用v9-cpu的dis,xc, xem命令（包括启动参数），阅读v9-cpu中的v9\-computer.md文档，了解汇编指令的类型和含义等，了解v9-cpu的细节。
- 了解基于v9-cpu的执行文件的格式和内容，以及它是如何加载到v9-cpu的内存中的。
- 在piazza上就学习中不理解问题进行提问。

---

## 思考题

- 你理解的对于类似ucore这样需要进程/虚存/文件系统的操作系统，在硬件设计上至少需要有哪些直接的支持？至少应该提供哪些功能的特权指令？

      对于类似ucore这样同时需要进程，虚存和文件系统的操作系统而言，在硬件设计上应当至少需要硬件支持时钟中断以完成进程的切换；
      MMU等硬件所支持的地址映射机制以完成虚存管理；稳定的存储介质来保持操作系统的持久性。
      因此，相应地需要提供中断使能等中断相关，设置内存寻址模式，设置页表等内存管理相关的，执行I/O操作等文件系统相关的特权指令。
      
- 你理解的x86的实模式和保护模式有什么区别？物理地址、线性地址、逻辑地址的含义分别是什么？

       进程内存是否得到了有效保护应当是实模式和保护模式的根本区别所在。实模式将整个物理内存看成分段区域，每一个指针都指向实际存在的物理地址。
       且对于系统程序和用户程序并没有区别对待。而在保护模式之中，物理内存并不能直接被程序访问，而是通过操作系统实现从虚地址到实地址的转换。
       这样可以有效避免实模式中用户程序更改系统程序内值所带来的巨大问题。
       物理地址指的是处理器提交到总线上用于访问计算机系统中的内存和外存的最终地址。
       线性地址是逻辑地址到物理地址变换之间的中间层。是处理器通过段机制控制下的形成的地址空间。
       逻辑地址是访问指令所给出的地址。
       
- 你理解的risc-v的特权模式有什么区别？不同 模式在地址访问方面有何特征？

       risc-v的四种特权模式分别为机器模式，监督模式，用户模式，Hypervisor模式。
       可分别简写为M mode , S mode , U mode , H mode;之后的辨析均以简写代替。
       其中，M模式未必选模式，U模式和S模式为可选模式。通过组合可以实现不同的系统。
       不同的特权级包含多个CSR寄存器，根据约定，CSR地址的高4位用于编码CSR根据特权级读写的可访问性。
       最高2位指示这个寄存器是否是可以读/写（00、01或者10),还是只读的（11)。
       后面2位指示了能够访问这个CSR所需要的最低特权级(用户级是 00，管理员级是 01）。
       具体情况如下https://img-blog.csdn.net/20180902144223890?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3AzNDA1ODkzNDQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70
       
       
       
      

- 理解ucore中list_entry双向链表数据结构及其4个基本操作函数和ucore中一些基于它的代码实现（此题不用填写内容）

- 对于如下的代码段，请说明":"后面的数字是什么含义
```
 /* Gate descriptors for interrupts and traps */
 struct gatedesc {
    unsigned gd_off_15_0 : 16;        // low 16 bits of offset in segment
    unsigned gd_ss : 16;            // segment selector
    unsigned gd_args : 5;            // # args, 0 for interrupt/trap gates
    unsigned gd_rsv1 : 3;            // reserved(should be zero I guess)
    unsigned gd_type : 4;            // type(STS_{TG,IG32,TG32})
    unsigned gd_s : 1;                // must be 0 (system)
    unsigned gd_dpl : 2;            // descriptor(meaning new) privilege level
    unsigned gd_p : 1;                // Present
    unsigned gd_off_31_16 : 16;        // high bits of offset in segment
 };
```

- 对于如下的代码段，

```
#define SETGATE(gate, istrap, sel, off, dpl) {            \
    (gate).gd_off_15_0 = (uint32_t)(off) & 0xffff;        \
    (gate).gd_ss = (sel);                                \
    (gate).gd_args = 0;                                    \
    (gate).gd_rsv1 = 0;                                    \
    (gate).gd_type = (istrap) ? STS_TG32 : STS_IG32;    \
    (gate).gd_s = 0;                                    \
    (gate).gd_dpl = (dpl);                                \
    (gate).gd_p = 1;                                    \
    (gate).gd_off_31_16 = (uint32_t)(off) >> 16;        \
}
```
如果在其他代码段中有如下语句，
```
unsigned intr;
intr=8;
SETGATE(intr, 1,2,3,0);
```
请问执行上述指令后， intr的值是多少？

### 课堂实践练习

#### 练习一

1. 请在ucore中找一段你认为难度适当的AT&T格式X86汇编代码，尝试解释其含义。

2. (option)请在rcore中找一段你认为难度适当的RV汇编代码，尝试解释其含义。

#### 练习二

宏定义和引用在内核代码中很常用。请枚举ucore或rcore中宏定义的用途，并举例描述其含义。

#### reference
 - [Intel格式和AT&T格式汇编区别](http://www.cnblogs.com/hdk1993/p/4820353.html)
 - [x86汇编指令集  ](http://hiyyp1234.blog.163.com/blog/static/67786373200981811422948/)
 - [PC Assembly Language, Paul A. Carter, November 2003.](https://pdos.csail.mit.edu/6.828/2016/readings/pcasm-book.pdf)
 - [*Intel 80386 Programmer's Reference Manual*, 1987](https://pdos.csail.mit.edu/6.828/2016/readings/i386/toc.htm)
 - [IA-32 Intel Architecture Software Developer's Manuals](http://www.intel.com/content/www/us/en/processors/architectures-software-developer-manuals.html)
