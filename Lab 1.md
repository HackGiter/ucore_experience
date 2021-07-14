# Lab 1

## Exercise 1：理解通过make生成执行文件的过程

### 1.1.Description

1. 操作系统镜像文件ucore.img是如何一步一步生成的？(需要比较详细地解释Makefile中每一条相关命令和命令参数的含义，以及说明命令导致的结果)
2. 一个被系统认为是符合规范的硬盘主引导扇区的特征是什么？

### 1.2.Note

1.Linux的make工具能实现自动化编译，，在makefile文件中编写所需要使用的命令：

make [-f makeFileName]

2.makefile的基本规范

### 1.3.makefile

makefile的基本语法包括三部分，三部分加在一起称为一组规则，递归式推导目标，有自定义变量与系统变量

（1）目标文件：此规则中想要生成的文件

（2）依赖文件：要生成此规则中的目标文件所需要的文件

（3）指此规则中要执行的动作

在linux中，文件分为属性和数据，每个文件有三种时间，分别用于记录与文件属性和文件数据乡姑纳的时间：atime（access time）、mtime（modify time）、ctime（change time）

![](C:\Users\lee19\AppData\Roaming\Typora\typora-user-images\image-20210711220449531.png)

![image-20210711220524908](C:\Users\lee19\AppData\Roaming\Typora\typora-user-images\image-20210711220524908.png)



#### 1.3.1.符号：

$ ：扩展打开makefile中定义的变量（注：make定义了很多默认变量，${MAKE} 就是预设的 make 这个命令的名称（或者路径））

@ ：通常makefile会将其执行的命令行在执行前输出到屏幕上。将‘@’添加到命令行前，命令将不被make回显出

\# ：makefile中的注释

%：用来匹配任意多个非空字符，如%.o代表所有以.o为结尾的文件

#### 1.3.2.赋值 & 语法：

= ：是最基本的赋值（全局的变量值替换）

:= ：是覆盖之前的值（局部的变量值替换，根据赋值位置）

?= ：是如果没有被赋值过就赋予等号后面的值

+= ：是添加等号后面的值

ifndef ：基本语法，if not define

include ：类似C语言中的include

AWK ：正则查找Makefile文件的匹配内容

SED ：正则替换Makefile文件的匹配内容

#### 1.3.3.变量

shell ：默认Linux命令行调用，可以在前面添加@

![image-20210711220737005](C:\Users\lee19\AppData\Roaming\Typora\typora-user-images\image-20210711220737005.png)

#### 1.3.4.Linux Shell命令

sudo ：以root级别执行命令，需要root password

su ：su root转到root用户，需要root password

exit ：退出命令行

reboot ：重启Linux

if ：基本语法，例if [ command ];then command; elif [ command ];then command; else command; fi

echo ：字符串输出，可以是标准输出（屏幕）或者重定向文件（注：-e 开启转义）；还可以显示命令执行结果，用反引号（`` ` ``），例：echo `` ` ``date`` ` ``

chmod ：change mode，控制用户对文件的权限的命令，例 chmod +rwx filename

<img src="https://www.runoob.com/wp-content/uploads/2014/08/file-permissions-rwx.jpg" alt="img" style="zoom:50%;" />

which ：在环境变量$PATH设置的目录里查找符合条件的文件

mkdir ：创建文件夹， make directory

cp ：copy，复制文件或文件夹

mv ：move，移动文件或文件夹

rm ：remove，删除文件或文件夹

tar ：压缩、解压缩文件

touch ：一是用于把已存在文件的时间标签更新为系统当前的时间（默认方式），它们的数据将原封不动地保留下来；二是用来创建新的空文件

call ：call命令用来从一个批处理脚本中调用另一个批处理脚本

dd ：用指定大小的块拷贝一个文件，并在拷贝的同时进行指定的转换，例 
dd if=boot/loader.bin of=/~/Desktop/Bochs/hd60M.img bs=512 count=3 seek=2 conv=notrunc（bs：单次写入的块大小，字节为单位；count：写入的块数量；seek：跳过的块数量；conv：是否截断）

nasm ：编译汇编文件文件工具（.S文件），-f format 类似文件格式elf，-I include（例：头文件），-o output

ld ：属于二进制工具集（GNU Binutils），是GNU链接器，用于将目标文件与库链接为可执行文件或库文件，例 ld -Ttext 0xc0001500 -e main -m elf_i386 -o kernel.bin kernel/main.o lib/kernel/print.o

gcc ：C语言编译器，-I include（例：头文件），-O output，-m32 32位程序

objdump ：gcc工具，用来查看编译后目标文件的组成输出，-S 尽可能反汇编出源代码，尤其当编译的时候指定了-g这种调试参数时，效果比较明显。隐含了-d参数

\> ：重定向符号

\>& ：表示重定向后面的不是一个文件，而是文件描述符，1：stdout，2：stderr，0：stdin；也可以写作&\>

### 1.4.简要回答

1. makefile中的CC指代gcc编译器，首先是编译C文件生成.O文件，即二进制文件；然后通过ld命令链接需要的.o文件生成kernel可执行文件，作为操作系统内核文件。bootblock是主引导记录（MBR，Master Boot Record），BIOS（Base Input & Output System，即基本输入输出系统）完成其开机所需要的任务后调用该文件以启动操作系统。
2. 硬件主引导扇区的特征是512个字节，且位于0x7c00处，这样BIOS就可以直接找到了；其次，扇区的最后2个字节也必须是0x55、0xaa。

## Exercise 2：使用qemu执行并调试lab1中的软件

### 2.1.Description

为了熟悉使用qemu和gdb进行的调试工作，我们进行如下的小练习：

1. 从CPU加电后执行的第一条指令开始，单步跟踪BIOS的执行。
2. 在初始化位置0x7c00设置实地址断点,测试断点正常。
3. 从0x7c00开始跟踪代码运行,将单步跟踪反汇编得到的代码与bootasm.S和 bootblock.asm进行比较。
4. 自己找一个bootloader或内核中的代码位置，设置断点并进行测试。

### 2.2.Note

1. 修改lab1/tools/gdbinit：

   ```makefile
   set architecture i8086
   target remote :1234
   ```

   

2. 在lab1目录下，执行以下命令就可以使gdb停在BIOS的第一条指令处：

   ```shell
   make debug
   ```

   ```assembly
   0xffff0: ljmp $0xf000,$0xe05b
   ```

3. 此时会弹出gdb的调试界面，可以使用gdb命令进行操作了

4. 此时的CS=0xf,IP=fff0

### 2.3.gdb命令

gdb软件可以调试可执行文件，是个相当厉害的软件，但是个人对其的掌握实在是皮毛而已，不过在微机原理课程中也算有所了解，挺好用的把。下面是常用指令及一般简单用法介绍：

x：examine命令，查看内存地址中的值，格式：x/ [nfu] ] [addr]。n就是number的意思，表示需要限时的内存单元个数；f就是format的意思，表示addr指向的内存内容的输出格式，如x为十六进制、d为十进制、t表示二进制、f就是浮点数格式、s为字符串、u无符号十进制、i表示指令地址格式；u即unit，指以多少个字节作为一个内存单元，默认为4。

I：list命令，列出程序源代码，格式：list number

b：breakpoint命令，设置断点，b offset表示在偏移地址断点，十进制；使用b *offset，可以用十六进制的偏移地址

info：information命令，格式：info name，name是函数或变量的名称，或者使用info address

r：run命令，运行程序

c：continue命令，继续运行程序，直到运行到断点或者结束

n：next命令，格式：n [count] 表示执行count条指令，如果遇到函数调用不会进入函数

s：step命令，格式：step [count]和next命令类似，但是遇到函数调用会进入函数内

ni & si：nexti命令 & stepi命令，用于单步跟踪机器指令，一条程序代码有可能由数条机器指令完成

p：print命令，可以通过该命令查看参数或程序运行数据

w：watch命令，一般用于监察某变量或表达式的值是否变化，如果发生变化就会立即停止运行

q：quit命令，退出运行指令

$(register)：指代寄存器

### 2.4.bootloader

bootloader实现了从模拟16位的实模式到32位的保护模式的切换，建立了全局描述表（Global Descriptor Table，GDT）用于内存段的记录，启用了分段机制；读取操作系统到内存中，过渡到了内核。

### 2.5.简要回答

1. 根据题目tips然后使用gdb的命令就可以完成1和2问，这里就不写了，贴个图。

   ![image-20210713183351300](C:\Users\lee19\AppData\Roaming\Typora\typora-user-images\image-20210713183351300.png)

2. 使用meld工具就可以对比bootasm.S和bootlock.asm的代码了

   ![image-20210713183559524](C:\Users\lee19\AppData\Roaming\Typora\typora-user-images\image-20210713183559524.png)

3. 第4问自己设置断点，b offset后c就可以了

## Exercise 3：分析bootloader进入保护模式的过程

### 3.1.Description

BIOS将通过读取硬盘主引导扇区到内存，并转跳到对应内存中的位置执行bootloader。请分析bootloader是如何完成从实模式进入保护模式的。

阅读bootasm.S源码，了解实模式到保护模式的切换过程

1. 为何开启A20，以及如何开启A20
2. 如何初始化GDT表
3. 如何使能和进入保护模式

### 3.2.Note & Answer

1. 16位CPU的时代时，有20位地址线，采用段基址乘以16再加上段内偏移地址进行寻址，故最大寻址空间时1MB；到了32位CPU就有了16位实模式，对于20位地址线的CPU如果寻址超过20位就直接溢出，类似于取模，自动实现了地址回绕；到了32位CPU，通过再键盘控制器上的输出线来控制第21根地址线（A20）的有效性，称为A20Gate。A20Gate被打开，就可以访问超1MB的地址；关闭则地址回绕。兼容贯穿了操作系统和CPU的历史。其中常用的是通过键盘控制器0x64进行控制；由于键盘控制器的速度慢，就有了A20快速门（Fast Gate 20），I/O端口0x92

   ```assembly
   seta20.1:
       inb $0x64, %al		# Wait for not busy(8042 input buffer empty).
       testb $0x2, %al
       jnz seta20.1
   
       movb $0xd1, %al		# 0xd1 -> port 0x64
       outb %al, $0x64		# 0xd1 means: write data to 8042's P2 port
   
   seta20.2:
       inb $0x64, %al		# Wait for not busy(8042 input buffer empty).
       testb $0x2, %al
       jnz seta20.2
   
       movb $0xdf, %al		# 0xdf -> port 0x60
       outb %al, $0x60		# 0xdf = 11011111, means set P2's A20 bit(the 1 bit) to 1
   ```

   ```assembly
   in al, 0x92
   or al, 0000_0010B
   out 0x92, al
   ```

   

2. 初始化GDT

   ```assembly
   ...
       # Switch from real to protected mode, using a bootstrap GDT
       # and segment translation that makes virtual addresses
       # identical to physical addresses, so that the
       # effective memory map does not change during the switch.
       lgdt gdtdesc
   ...
   # Bootstrap GDT
   .p2align 2                                   	# force 4 byte alignment
   gdt:
       SEG_NULLASM                                     # null seg
       SEG_ASM(STA_X|STA_R, 0x0, 0xffffffff)			# code seg for bootloader and kernel
       SEG_ASM(STA_W, 0x0, 0xffffffff)	# data seg for bootloader and kernel
   
   gdtdesc:
       .word 0x17                                      # sizeof(gdt) - 1
       .long gdt                                       # address gdt
   ```

3. 进入保护模式，设置CR0寄存器的PE位。控制寄存器CRx，CR0寄存器的第0位PE，Protection Enable，用于启用保护模式。![image-20210713193128827](C:\Users\lee19\AppData\Roaming\Typora\typora-user-images\image-20210713193128827.png)

```assembly
    lgdt gdtdesc
    movl %cr0, %eax
    orl $CR0_PE_ON, %eax
    movl %eax, %cr0

    # Jump to next instruction, but in 32-bit code segment.
    # Switches processor into 32-bit mode.
    ljmp $PROT_MODE_CSEG, $protcseg

.code32                                             # Assemble for 32-bit mode
protcseg:
    # Set up the protected-mode data segment registers
    movw $PROT_MODE_DSEG, %ax                       # Our data segment
```

## Exercise 4：分析bootloader加载ELF格式的OS的过程

### 4.1.Description

通过阅读bootmain.c，了解bootloader如何加载ELF文件。通过分析源代码和通过qemu来运行并调试bootloader&OS，

1. bootloader如何读取硬盘扇区的？
2. bootloader是如何加载ELF格式的OS？

### 4.2.Note & Answer

#### 4.2.1.读取硬盘扇区

bootloader让CPU进入保护模式后，下一步的工作就是从硬盘上加载并运行OS。考虑到实现的简单性，bootloader的访问硬盘都是LBA模式的PIO（Program IO）方式，即所有的IO操作是通过CPU访问硬盘的IO地址寄存器完成。

一般主板有2个IDE通道，每个通道可以接2个IDE硬盘。访问第一个硬盘的扇区可设置IO地址寄存器0x1f0-0x1f7实现的，具体参数见下表。一般第一个IDE通道通过访问IO地址0x1f0-0x1f7来实现，第二个IDE通道通过访问0x170-0x17f实现。每个通道的主从盘的选择通过第6个IO偏移地址寄存器来设置。

表一 磁盘IO地址和对应功能

第6位：为1=LBA模式；0 = CHS模式 第7位和第5位必须为1

| IO地址 | 功能                                                         |
| ------ | ------------------------------------------------------------ |
| 0x1f0  | 读数据，当0x1f7不为忙状态时，可以读。                        |
| 0x1f2  | 要读写的扇区数，每次读写前，你需要表明你要读写几个扇区。最小是1个扇区 |
| 0x1f3  | 如果是LBA模式，就是LBA参数的0-7位                            |
| 0x1f4  | 如果是LBA模式，就是LBA参数的8-15位                           |
| 0x1f5  | 如果是LBA模式，就是LBA参数的16-23位                          |
| 0x1f6  | 第0~3位：如果是LBA模式就是24-27位 第4位：为0主盘；为1从盘    |
| 0x1f7  | 状态和命令寄存器。操作时先给命令，再读取，如果不是忙状态就从0x1f0端口读数据 |

当前 硬盘数据是储存到硬盘扇区中，一个扇区大小为512字节。读一个扇区的流程（可参看boot/bootmain.c中的readsect函数实现）大致如下：

1. 等待磁盘准备好
2. 发出读取扇区的命令
3. 等待磁盘准备好
4. 把磁盘扇区数据读到指定内存

其实，题目给的资料挺完善的，与源码可以完美对映。

#### 4.2.2.ELF文件格式

ELF(Executable and linking format)文件格式是Linux系统下的一种常用目标文件(object file)格式，有三种主要类型:

- 用于执行的可执行文件(executable file)，用于提供程序的进程映像，加载的内存执行。 这也是本实验的OS文件类型。
- 用于连接的可重定位文件(relocatable file)，可与其它目标文件一起创建可执行文件和共享目标文件。
- 共享目标文件(shared object file),连接器可将它与其它可重定位文件和共享目标文件连接成其它的目标文件，动态连接器又可将它与可执行文件和其它共享目标文件结合起来创建一个进程映像。

```c++
// elf.h
struct elfhdr {
  uint magic;  // must equal ELF_MAGIC
  uchar elf[12];
  ushort type;
  ushort machine;
  uint version;
  uint entry;  // 程序入口的虚拟地址
  uint phoff;  // program header 表的位置偏移
  uint shoff;
  uint flags;
  ushort ehsize;
  ushort phentsize;
  ushort phnum; //program header表中的入口数目
  ushort shentsize;
  ushort shnum;
  ushort shstrndx;
};
```

再迪马中可以看到读入8个扇区数据后，进行了强制转换

```c++
...
#define SECTSIZE        512
#define ELFHDR          ((struct elfhdr *)0x10000)      // scratch space
...

void bootmain(void) {
    // read the 1st page off disk
    readseg((uintptr_t)ELFHDR, SECTSIZE * 8, 0);

    // is this a valid ELF?
    if (ELFHDR->e_magic != ELF_MAGIC) {
        goto bad;
    }

    struct proghdr *ph, *eph;

    // load each program segment (ignores ph flags)
    ph = (struct proghdr *)((uintptr_t)ELFHDR + ELFHDR->e_phoff);
    eph = ph + ELFHDR->e_phnum;
    for (; ph < eph; ph ++) {
        readseg(ph->p_va & 0xFFFFFF, ph->p_memsz, ph->p_offset);
    }

    // call the entry point from the ELF header
    // note: does not return
    ((void (*)(void))(ELFHDR->e_entry & 0xFFFFFF))();

bad:
    outw(0x8A00, 0x8A00);
    outw(0x8A00, 0x8E00);

    /* do nothing */
    while (1);
}

```

## Exercise 5：实现函数调用堆栈跟踪函数

### 5.1.Description

们需要在lab1中完成kdebug.c中函数print_stackframe的实现，可以通过函数print_stackframe来跟踪函数调用堆栈中记录的返回地址。

提示：可阅读小节“函数堆栈”，了解编译器如何建立函数调用关系的。在完成lab1编译后，查看lab1/obj/bootblock.asm，了解bootloader源码与机器码的语句和地址等的对应关系；查看lab1/obj/kernel.asm，了解 ucore OS源码与机器码的语句和地址等的对应关系。

补充材料：

由于显示完整的栈结构需要解析内核文件中的调试符号，较为复杂和繁琐。代码中有一些辅助函数可以使用。例如可以通过调用print_debuginfo函数完成查找对应函数名并打印至屏幕的功能。具体可以参见kdebug.c代码中的注释。

### 5.2.Note & Answer

栈帧，函数调用时包括参数、返回地址等信息的记录单元，一种数据结构，栈帧结构：

|  栈底方向   | 高位地址 |
| :---------: | :------: |
|     ...     |          |
|    参数4    |          |
|    参数3    |          |
|    参数2    |          |
|    参数1    |          |
|  返回地址   |          |
| 上一层[ebp] |   ebp    |
|  局部变量   | 低位地址 |
| 保护寄存器  |   esp    |

结果如下：

```c
void print_stackframe(void) {
     /* LAB1 YOUR CODE : STEP 1 */
     /* (1) call read_ebp() to get the value of ebp. the type is (uint32_t);
      * (2) call read_eip() to get the value of eip. the type is (uint32_t);
      * (3) from 0 .. STACKFRAME_DEPTH
      *    (3.1) printf value of ebp, eip
      *    (3.2) (uint32_t)calling arguments [0..4] = the contents in address (unit32_t)ebp +2 [0..4]
      *    (3.3) cprintf("\n");
      *    (3.4) call print_debuginfo(eip-1) to print the C calling function name and line number, etc.
      *    (3.5) popup a calling stackframe
      *           NOTICE: the calling funciton's return addr eip  = ss:[ebp+4]
      *                   the calling funciton's ebp = ss:[ebp]
      */
	uint32_t ebp = read_ebp();
	uint32_t eip = read_eip();

	int i;
	for (i = 0; i < STACKFRAME_DEPTH; i++) {
		cprintf("ebp:0x%08x ", ebp);
		cprintf("eip:0x%08x ", eip);
		
		uint32_t* args = (uint32_t*)ebp + 2;
		cprintf("args:");
		cprintf("0x%08x ", *(args+0));
		cprintf("0x%08x ", *(args+1));
		cprintf("0x%08x ", *(args+2));
		cprintf("0x%08x ", *(args+3));
		cprintf("\n");
		print_debuginfo(eip-1);
		eip = *((uint32_t*)ebp+1);
		ebp = *((uint32_t*)ebp+0);
	}
}

```

![image-20210713214507599](C:\Users\lee19\AppData\Roaming\Typora\typora-user-images\image-20210713214507599.png)

## Exercise 6：完善中断初始化和处理

### 6.1.Description

1. 中断描述符表（也可简称为保护模式下的中断向量表）中一个表项占多少字节？其中哪几位代表中断处理代码的入口？
2. 请编程完善kern/trap/trap.c中对中断向量表进行初始化的函数idt_init。在idt_init函数中，依次对所有中断入口进行初始化。使用mmu.h中的SETGATE宏，填充idt数组内容。每个中断的入口由tools/vectors.c生成，使用trap.c中声明的vectors数组即可。
3. 请编程完善trap.c中的中断处理函数trap，在对时钟中断进行处理的部分填写trap函数中处理时钟中断的部分，使操作系统每遇到100次时钟中断后，调用print_ticks子程序，向屏幕上打印一行文字”100 ticks”。

> 【注意】除了系统调用中断(T_SYSCALL)使用陷阱门描述符且权限为用户态权限以外，其它中断均使用特权级(DPL)为０的中断门描述符，权限为内核态权限；而ucore的应用程序处于特权级３，需要采用｀int 0x80`指令操作（这种方式称为软中断，软件中断，Tra中断，在lab5会碰到）来发出系统调用请求，并要能实现从特权级３到特权级０的转换，所以系统调用中断(T_SYSCALL)所对应的中断门描述符中的特权级（DPL）需要设置为３。
>
> 提示：可阅读小节“中断与异常”。

### 6.2.Note & Answer

1. 直接看mmu.h和pmm.h（宏定义注意看memlayou.ht）就可以看到中断描述符表、选择子和全局描述符表的数据结构了，8字节。通过选择子在GDT或LDT中寻找基址地址，加上IDT中的偏移量（低16位和高16位的offset）得到中断服务程序地址。

   

   ```c
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
   
   /* *
    * Set up a normal interrupt/trap gate descriptor
    *   - istrap: 1 for a trap (= exception) gate, 0 for an interrupt gate
    *   - sel: Code segment selector for interrupt/trap handler
    *   - off: Offset in code segment for interrupt/trap handler
    *   - dpl: Descriptor Privilege Level - the privilege level required
    *          for software to invoke this interrupt/trap gate explicitly
    *          using an int instruction.
    * */
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
   
   static struct segdesc gdt[] = {
       SEG_NULL,
       [SEG_KTEXT] = SEG(STA_X | STA_R, 0x0, 0xFFFFFFFF, DPL_KERNEL),
       [SEG_KDATA] = SEG(STA_W, 0x0, 0xFFFFFFFF, DPL_KERNEL),
       [SEG_UTEXT] = SEG(STA_X | STA_R, 0x0, 0xFFFFFFFF, DPL_USER),
       [SEG_UDATA] = SEG(STA_W, 0x0, 0xFFFFFFFF, DPL_USER),
       [SEG_TSS]    = SEG_NULL,
   };
   ```

   根据题目中的注意的内容，可以知道选择DPL_KERNEL，并根据memlayout.h中的宏定义

   ```c
   /* global descrptor numbers */
   #define GD_KTEXT    ((SEG_KTEXT) << 3)        // kernel text
   #define GD_KDATA    ((SEG_KDATA) << 3)        // kernel data
   #define GD_UTEXT    ((SEG_UTEXT) << 3)        // user text
   #define GD_UDATA    ((SEG_UDATA) << 3)        // user data
   #define GD_TSS        ((SEG_TSS) << 3)        // task segment selector
   
   #define DPL_KERNEL    (0)
   #define DPL_USER    (3)
   
   #define KERNEL_CS    ((GD_KTEXT) | DPL_KERNEL)
   #define KERNEL_DS    ((GD_KDATA) | DPL_KERNEL)
   #define USER_CS        ((GD_UTEXT) | DPL_USER)
   #define USER_DS        ((GD_UDATA) | DPL_USER)
   ```

   trap.h中的宏定义，知道系统调用时从用户态陷入内核态的索引

   ```c
   /* *
    * These are arbitrarily chosen, but with care not to overlap
    * processor defined exceptions or interrupt vectors.
    * */
   #define T_SWITCH_TOU                120    // user/kernel switch
   #define T_SWITCH_TOK                121    // user/kernel switch
   ```

   

   ```c
   /* idt_init - initialize IDT to each of the entry points in kern/trap/vectors.S */
   void idt_init(void) {
        /* LAB1 YOUR CODE : STEP 2 */
        /* (1) Where are the entry addrs of each Interrupt Service Routine (ISR)?
         *     All ISR's entry addrs are stored in __vectors. where is uintptr_t __vectors[] ?
         *     __vectors[] is in kern/trap/vector.S which is produced by tools/vector.c
         *     (try "make" command in lab1, then you will find vector.S in kern/trap DIR)
         *     You can use  "extern uintptr_t __vectors[];" to define this extern variable which will be used later.
         * (2) Now you should setup the entries of ISR in Interrupt Description Table (IDT).
         *     Can you see idt[256] in this file? Yes, it's IDT! you can use SETGATE macro to setup each item of IDT
         * (3) After setup the contents of IDT, you will let CPU know where is the IDT by using 'lidt' instruction.
         *     You don't know the meaning of this instruction? just google it! and check the libs/x86.h to know more.
         *     Notice: the argument of lidt is idt_pd. try to find it!
         */
   	extern uintptr_t __vectors[];
   	int i;
   	for (i = 0; i < 256; i++) {
   		SETGATE(*(idt+i), 0, GD_KTEXT, *(__vectors+i), DPL_KERNEL);
   	}
   	SETGATE(*(idt+T_SWITCH_TOK), 0, GD_KTEXT, *(__vectors+T_SWITCH_TOK), DPL_USER);
   	lidt(&idt_pd);
   }
   ```

   

2. 根据题意和tips就可以直接写了

```c
/* trap_dispatch - dispatch based on what type of trap occurred */
static void trap_dispatch(struct trapframe *tf) {
    char c;

    switch (tf->tf_trapno) {
    case IRQ_OFFSET + IRQ_TIMER:
        /* LAB1 YOUR CODE : STEP 3 */
        /* handle the timer interrupt */
        /* (1) After a timer interrupt, you should record this event using a global variable (increase it), such as ticks in kern/driver/clock.c
         * (2) Every TICK_NUM cycle, you can print some info using a funciton, such as print_ticks().
         * (3) Too Simple? Yes, I think so!
         */
	ticks++;
	if (ticks - TICK_NUM == 0) {
		print_ticks();
		ticks = 0;
	}
        break;
    case IRQ_OFFSET + IRQ_COM1:
        c = cons_getc();
        cprintf("serial [%03d] %c\n", c, c);
        break;
    case IRQ_OFFSET + IRQ_KBD:
        c = cons_getc();
        cprintf("kbd [%03d] %c\n", c, c);
        break;
    //LAB1 CHALLENGE 1 : YOUR CODE you should modify below codes.
    case T_SWITCH_TOU:
    case T_SWITCH_TOK:
        panic("T_SWITCH_** ??\n");
        break;
    case IRQ_OFFSET + IRQ_IDE1:
    case IRQ_OFFSET + IRQ_IDE2:
        /* do nothing */
        break;
    default:
        // in kernel, it must be a mistake
        if ((tf->tf_cs & 3) == 0) {
            print_trapframe(tf);
            panic("unexpected trap in kernel.\n");
        }
    }
}

```



## 扩展训练7：Challenge

### 7.1.Challenge 1

#### 7.1.1.Description

扩展proj4,增加syscall功能，即增加一用户态函数（可执行一特定系统调用：获得时钟计数值），当内核初始完毕后，可从内核态返回到用户态的函数，而用户态的函数又通过系统调用得到内核态的服务。

switch*to** 函数建议通过 中断处理的方式实现。

#### 7.1.2.Note & Answer



### 7.2.Challenge 2

#### 7.2.1.Description

用键盘实现用户模式内核模式切换。具体目标是：“键盘输入3时切换到用户模式，键盘输入0时切换到内核模式”。 基本思路是借鉴软中断(syscall功能)的代码，并且把trap.c中软中断处理的设置语句拿过来。

注意：

　1.关于调试工具，不建议用lab1_print_cur_status()来显示，要注意到寄存器的值要在中断完成后tranentry.S里面iret结束的时候才写回，所以再trap.c里面不好观察，建议用print_trapframe(tf)

　2.关于内联汇编，最开始调试的时候，参数容易出现错误，可能的错误代码如下

```
   asm volatile ( "sub $0x8, %%esp \n"
     "int %0 \n"
     "movl %%ebp, %%esp"
     : )
```

要去掉参数int %0 \n这一行

3.软中断是利用了临时栈来处理的，所以有压栈和出栈的汇编语句。硬件中断本身就在内核态了，直接处理就可以了。

#### 7.2.2.Note & Answer

1. 这两道题个人也不是很明白，就参考了答案，应该看懂了。

   首先是每次进入trap时，都会调用trapentry.S中的__alltraps函数，对寄存器值进行压栈创建了trapfram的结构体，并调用trap.c中的trap函数。所以在进入trap时，根据状态进行堆栈操作：

   

   ```c
   //对TO User
       //tf->tf_cs = USER_CS;
       //tf->tf_ds = USER_DS;
       //tf->tf_es = USER_DS;
       //tf->tf_ss = USER_DS;
           if (tf->tf_cs != USER_CS) {
               switchk2u = *tf;
               switchk2u.tf_cs = USER_CS;
               switchk2u.tf_ds = switchk2u.tf_es = switchk2u.tf_ss = USER_DS;
               switchk2u.tf_esp = (uint32_t)tf + sizeof(struct trapframe) - 8;
   		
               // set eflags, make sure ucore can use io under user mode.
               // if CPL > IOPL, then cpu will generate a general protection.
               switchk2u.tf_eflags |= FL_IOPL_MASK;
   		
               // set temporary stack
               // then iret will jump to the right stack
               *((uint32_t *)tf - 1) = (uint32_t)&switchk2u;
           }
   //对TO Kernel
       //tf->tf_cs = KERNEL_CS;
       //tf->tf_ds = KERNEL_DS;
       //tf->tf_es = KERNEL_DS;
           if (tf->tf_cs != KERNEL_CS) {
               tf->tf_cs = KERNEL_CS;
               tf->tf_ds = tf->tf_es = KERNEL_DS;
               tf->tf_eflags &= ~FL_IOPL_MASK;
               switchu2k = (struct trapframe *)(tf->tf_esp - (sizeof(struct trapframe) - 8));
               memmove(switchu2k, tf, sizeof(struct trapframe) - 8);
               *((uint32_t *)tf - 1) = (uint32_t)switchu2k;
           }
           break;
   
   //在lab1_switch_to_user中，调用T_SWITCH_TOU中断。
   //注意从中断返回时，会多pop两位，并用这两位的值更新ss,sp，损坏堆栈。
   //所以要先把栈压两位，并在从中断返回后修复esp。
   	asm volatile (
   	    "sub $0x8, %%esp \n"
   	    "int %0 \n"
   	    "movl %%ebp, %%esp"
   	    : 
   	    : "i"(T_SWITCH_TOU)
   	);
   //在lab1_switch_to_kernel中，调用T_SWITCH_TOK中断。
   //注意从中断返回时，esp仍在TSS指示的堆栈中。所以要在从中断返回后修复esp。
   	asm volatile (
   	    "int %0 \n"
   	    "movl %%ebp, %%esp \n"
   	    : 
   	    : "i"(T_SWITCH_TOK)
   	);
   ```

   

2. 这个问题尚未解决，有点思路，但总没有调用，虽然可以直接像上一文中直接调用，但好像不符合题意；或者可以在console.c的文件中修改内容。

``` c
case IRQ_OFFSET + IRQ_KBD:
    c = cons_getc();
    cprintf("kbd [%03d] %c\n", c, c);
    switch (c) {
        case '0':
	if (tf->tf_cs != KERNEL_CS) {
	    tf->tf_cs = KERNEL_CS;
	    tf->tf_ds = tf->tf_es = KERNEL_DS;
	    tf->tf_eflags &= ~FL_IOPL_MASK;
	}
            print_trapframe(tf);
            break;
        case '3':
	if (tf->tf_cs != USER_CS) {
		record = *tf;		//复制tf中的内容
		record.tf_cs = USER_CS;//修改为USER的内存空间
		record.tf_ds = record.tf_es = record.tf_ss = USER_DS;//同理
		record.tf_eflags |= FL_IOPL_MASK;//修改eflags
		record.tf_esp = (uint32_t)tf + sizeof(struct trapframe) - 8;//找到上一个trapframe

		//压栈
		*((uint32_t *)tf - 1) = (uint32_t)&record;

	}
            print_trapframe(tf);
            break;
    }
    break;
```
