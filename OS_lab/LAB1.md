# 练习1：理解通过make生成执行文件的过程。1.操作系统镜像文件ucore.img是如何一步一步生成的？(需要比较详细地解释Makefile中每一条相关命令和命令参数的含义，以及说明命令导致的结果)

## 手册

### gcc 选项

-fno-builtin
不接受不是两个下划线开头的内建函数(built-in function).目前受影响的函数有_exit, abort, abs, alloca, cos, exit, fabs, labs, memcmp, memcpy,
sin, sqrt, strcmp, strcpy,和 strlen

-f-PIC
如果支持这种目标机,编译器就输出位置无关目标码.适用于动态连接(dynamic
linking),即使分支需要大范围 转移.

-ggdb
以本地格式(如果支持)输出调试信息,尽可能包括 GDB 扩展

-gstabs
以 stabs 格式(如果支持)输出调试信息,不包括 GDB 扩展.这是大多数 BSD 系统上 DBX
使用的格式.

-nostdinc
不要在标准系统目录中寻找头文件.只搜索-I选项指定的目录(以及当前目录,如果
合适).
结合使用-nostdinc'和-I选项,你可以把包含文件搜索限制在显式指定的目录.

-c
编译或汇编源文件,但是不作连接.编译器输出对应于源文件的目标文件.
缺省情况下, GCC 通过用.o替换源文件名后缀.c, .i, .s,等等,产生目
标文件名.可以使用-o 选项选择其他名字.
GCC 忽略-c 选项后面任何无法识别的输入文件(他们不需要编译或汇编).
-o file

指定输出文件为 file.该选项不在乎 GCC 产生什么输出,无论是可执行文件,目标文件,
汇编文件还是 预处理后的 C 代码.
由于只能指定一个输出文件,因此编译多个输入文件时,使用-o选项没有意义,除非
输出一个可执行文件.
如果没有使用-o选项,默认的输出结果是:可执行文件为a.out,
source.suffix 的目标文件是source.o,汇编文件是 source.s,而预处
理后的 C 源代码送往标准输出.



### ld选项
-E:				对于ELF格式文件，把所有符号添加到动态符号表
-m:				模拟指定的连接器
-T:				指定命令文件 (和 -c 相同)
-Ttext:				使用指定的地址作为文本段的起始点
-e:				使用指定的符号作为程序的初始执行点
-N:				指定读取/写入文本和数据段



### dd命令
dd [选项]
if =输入文件（或设备名称）
of =输出文件（或设备名称）
count=blocks 只拷贝输入的blocks块。
conv = notrunc 不截短输出文件。
seek=BLOCKS 跳过一段以后才输出

## 猜测部分

-fno-PIC
如果不支持这种目标机,编译器就输出位置无关目标码.适用于动态连接(dynamic
linking),即使分支需要大范围 转移.

-fno-stack-protector
不采用栈保护机制

-march=i686
指定为32位系统

-m32
输出32的目标码


## ans

create kernel

编译并生成：

obj/kern/init/init.o

obj/kern/libs/stdio.o

obj/kern/libs/readline.o

obj/kern/debug/panic.o

obj/kern/debug/kdebug.o

obj/kern/debug/kmonitor.o

obj/kern/driver/clock.o

obj/kern/driver/console.o

obj/kern/driver/picirq.o

obj/kern/driver/intr.o

obj/kern/trap/trap.o

obj/kern/trap/vectors.o

obj/kern/trap/trapentry.o

obj/kern/mm/pmm.o

obj/libs/string.o

obj/libs/printfmt.o

链接

tools/kernel.ld -o bin/kernel  obj/kern/init/init.o obj/kern/libs/stdio.o obj/kern/libs/readline.o obj/kern/debug/panic.o obj/kern/debug/kdebug.o obj/kern/debug/kmonitor.o obj/kern/driver/clock.o obj/kern/driver/console.o obj/kern/driver/picirq.o obj/kern/driver/intr.o obj/kern/trap/trap.o obj/kern/trap/vectors.o obj/kern/trap/trapentry.o obj/kern/mm/pmm.o  obj/libs/string.o obj/libs/printfmt.o



create bootblock

编译并生成：
obj/boot/bootasm.o

 obj/boot/bootmain.o

obj/sign/tools/sign.o

链接
obj/boot/bootasm.o obj/boot/bootmain.o -o obj/bootblock.o



create ucore.img 

复制

dd if=/dev/zero of=bin/ucore.img count=10000

dd if=bin/bootblock of=bin/ucore.img conv=notrunc

dd if=bin/kernel of=bin/ucore.img seek=1 conv=notrunc

# 2.一个被系统认为是符合规范的硬盘主引导扇区的特征是什么？

struct stat {
    uint32_t st_mode;                   // protection mode and file type
    size_t st_nlinks;                   // number of hard links
    size_t st_blocks;                   // number of blocks file is using
    size_t st_size;                     // file size (bytes)
};

<input filename> <output filename>
输入
可以打开
大小<=510字节
数据大小和stat中大小一致
最后2字节为0x55AA
最终输出文件为512字节

# 练习2：使用qemu执行并调试lab1中的软件。

## bug

p2如果使用新版本的qume会 产生错误这是因为gdb抱怨不支持32位 可以下载旧版本qume 如2.8.1于 https://download.qemu.org/ 同时在make时会报错 /builddir/build/BUILD/qemu-2.11.0-rc1/util/memfd.c:40:12: error: static declaration of memfd_create follows non-static declaration 只需要将qume中memfd.c中的memfd_create函数定义删掉 同时在创建软连接时注意使用- 网页复制会出问题

## 向文件输出调试信息

set logging file set logging on ... set logging off

#### 练习3：分析bootloader进入保护模式的过程。

Intel早期的8086 CPU提供了20根地址线,可寻址空间范围即0~2^20(00000H~FFFFFH)的 1MB内存空间。但8086的数据处理位宽位16位，无法直接寻址1MB内存空间，所以8086提供了段地址加偏移地址的地址转换机制。PC机的寻址结构是segment:offset，segment和offset都是16位的寄存器，最大值是0ffffh，换算成物理地址的计算方法是把segment左移4位，再加上offset，所以segment:offset所能表达的寻址空间最大应为0ffff0h + 0ffffh = 10ffefh（前面的0ffffh是segment=0ffffh并向左移动4位的结果，后面的0ffffh是可能的最大offset），这个计算出的10ffefh是多大呢？大约是1088KB，就是说，segment:offset的地址表示能力，超过了20位地址线的物理寻址能力。所以当寻址到超过1MB的内存时，会发生“回卷”（不会发生异常）。但下一代的基于Intel 80286 CPU的PC AT计算机系统提供了24根地址线，这样CPU的寻址范围变为 2^24=16M,同时也提供了保护模式，可以访问到1MB以上的内存了，此时如果遇到“寻址超过1MB”的情况，系统不会再“回卷”了，这就造成了向下不兼容 。为了保持完全的向下兼容性，IBM决定在PC AT计算机系统上加个硬件逻辑，来模仿以上的回绕特征，于是出现了A20 Gate





设置为16位模式
禁用中断
将数据段、附加数据段、堆栈段寄存器置零。

## A20

从0x64端口读取状态码到%al,检查1位为0则循环检测（说明此时还input buffer非空）。
当为0时将%al 的值赋为0xd1,向0x64发送。
从0x64端口读取状态码到%al,检查1位为0则循环检测（说明此时还input buffer非空）。
当为0时将%al 的值赋为0xdf,向0x60写入设置第1位为1 使能A20。

## 保护模式

加载GDT
将%cr0读入%eax与0x01亦或后赋值给%cr0使能保护模式，
使用逻辑地址。
跳转指令 PROT_MODE_CSEG=00001(index) _ 00(TI) _ 0(RPL)

索引（Index）：在描述符表中从8192个描述符中选择一个描述符。处理器自动将这个索引值乘以8（描述符的长度），再加上描述符表的基址来索引描述符表，从而选出一个合适的描述符。
表指示位（Table Indicator，TI）：选择应该访问哪一个描述符表。0代表应该访问全局描述符表（GDT），1代表应该访问局部描述符表（LDT）。
请求特权级（Requested Privilege Level，RPL）：保护机制，在后续试验中会进一步讲解。

选择第一个段描述符 （段属性，段基地址，段界限）
此处基址为0x0 所以映射前后地址一致，逻辑地址和线性地址相同又无分页机制所以和物理地址相同。

设置保护模式的寄存器初值
%ebp=0  基址指针寄存器
%esp=0x7c00  堆栈指针寄存器
调用call bootmain

#### 练习4：分析bootloader加载ELF格式的OS的过程。

### waitdisk

waitdisk-等待磁盘准备好
检查0x1f7是否是忙状态，当非忙状态时跳出

### readsect

readsect-将@secno的单个扇区读入@dst
调用waitdisk
0x1f2置1表示读写一个扇区，将剩余IO地址置位LBA的相应位。其中0x1f6的第四位：为0主盘，为1从盘。
0x1f7地址输入读命令读取扇区。
调用waitdisk
调用 insl 写入到dsa

### insl

CLD使DF复位
repne循环
insl读入4字节

### readseg

内核将@offset处的@count字节读取到虚拟地址@va中，复制的数量可能会超出要求。
计算扇区边界，把字节转换为扇区。
调用readsect读取扇区。

### bootmain

读取第一页，从0开始。
通过检查ELFHDR->e_magic判断是否为ELF格式。
ph=program header 表的起始位置
eph表的终止位置
循环读取。
将e_entry转换为函数并执行 ，将控制权给ucore

# 练习5：实现函数调用堆栈跟踪函数 

cld相对应的指令是std，二者均是用来操作方向标志位DF.Direction Flag）。cld使DF 复位，即是让DF=0，std使DF置位，即DF=1.这两个指令用于串操作指令中。通过执行cld或std指令可以控制方向标志DF，决定内存地址是增大(DF=0，向高地址增加)还是减小(DF=1，向地地址减小)。

从0x7c00开始

关闭中断，串操作内存地址增大。

将DS,ES,SS置0

```c
uint32_t eip = read_eip();
	uint32_t ebp = read_ebp();
	uint32_t i = 0;
	for( ; eip!=0 && i < STACKFRAME_DEPTH; i++)
	{
		printf("ebp:0x%08x eip:0x%08x args:",ebp,eip);
		uint32_t * args = (uint32_t *)ebp + 2;
		for(uint32_t j = 0; j < 4; j++)
		{
			cprintf("0x%08x ",*(args+4*j));			
		}
		cprintf("\n");
		print_debuginfo(eip-1);
		eip = (uint32_t *)ebp + 4;
		ebp = (uint32_t *)ebp;
	}
```

## 练习6：完善中断初始化和处理

8   base:16-31 + shift (0~15 48~63)

```c
ticks++;
		if(ticks % TICK_NUM==0)
			print_ticks();
```





```c
extern uintptr_t __vectors[];
	int i;
    for (i = 0; i < sizeof(idt)/sizeof(struct gatedesc); i ++) {
		SETGATE(idt[i],0,GD_KTEXT,__vectors[i],0);
	}
	SETGATE(idt[i],1,GD_KTEXT,__vectors[i],3);
	lidt(&idt_pd);
```
