## 练习0：填写已有实验
本实验依赖实验1/2/3/4。请把你做的实验1/2/3/4的代码填入本实验中代码中有“LAB1”/“LAB2”/“LAB3”/“LAB4”的注释相应部分。
## 练习1: 加载应用程序并执行

init_main---->user_main---->kernel_execve------>sys_exec------>do_execve
inti_main中调用user_main(汇编)`kernel_thread(user_main, NULL, 0);`
user_main中调用kernel_execve在通过一系列宏定义调用do_execve调用load_icode
```c
.text
.globl kernel_thread_entry
kernel_thread_entry:        # void kernel_thread(void)

    pushl %edx              # push arg
    call *%ebx              # call fn//调用user_main(汇编)`

    pushl %eax              # save the return value of fn(arg)
    call do_exit            # call do_exit to terminate current thread


```
```c
// user_main - kernel thread used to exec a user program
static int
user_main(void *arg) {
#ifdef TEST
    KERNEL_EXECVE2(TEST, TESTSTART, TESTSIZE);
#else
    KERNEL_EXECVE(exit);//user_main中调用kernel_execve
#endif
    panic("user_main execve failed.\n");
}
```
KERNEL_EXECVE是一个宏定义
```c
#define KERNEL_EXECVE(x) ({                                        \	//调用do_execve
            extern unsigned char _binary_obj___user_##x##_out_start[],  \
                _binary_obj___user_##x##_out_size[];                    \
            __KERNEL_EXECVE(#x, _binary_obj___user_##x##_out_start,     \
                            _binary_obj___user_##x##_out_size);//         \
        })
```
###  请在实验报告中描述当创建一个用户态进程并加载了应用程序后，CPU是如何让这个应用程序最终在用户态执行起来的。即这个用户态进程被ucore选择占用CPU执行（RUNNING态）到具体执行应用程序第一条指令的整个经过。
为内存管理数据结构申请空间，并初始化
```c
//(1) create a new mm for current process
    if ((mm = mm_create()) == NULL) {
        goto bad_mm;
    }
```
分配一个页，并`memcpy(pgdir, boot_pgdir, PGSIZE);`把内核页表中的内容拷贝过来，
最后`mm->pgdir = pgdir;`设定好内存 管理程序的pgdir

```c
//(2) create a new PDT, and mm->pgdir= kernel virtual addr of PDT
    if (setup_pgdir(mm) != 0) {
        goto bad_pgdir_cleanup_mm;
    }
```
读取文件，找到program header 表，e_phoff记录了program header 表的位置偏移量，
检查文件是否为ELF格式(e_magic)
```c
//(3.1) get the file header of the bianry program (ELF format)
    struct elfhdr *elf = (struct elfhdr *)binary;
    //(3.2) get the entry of the program section headers of the bianry program (ELF format)
    struct proghdr *ph = (struct proghdr *)(binary + elf->e_phoff);
    //(3.3) This program is valid?
    if (elf->e_magic != ELF_MAGIC) {
        ret = -E_INVAL_ELF;
        goto bad_elf_cleanup_pgdir;
    }
```
e_phnum记录了program header的数目，查找这个数组，检查内存是否充足。
```c
struct proghdr *ph_end = ph + elf->e_phnum;
    for (; ph < ph_end; ph ++) {
    //(3.4) find every program section headers
        if (ph->p_type != ELF_PT_LOAD) {
            continue ;
        }
        if (ph->p_filesz > ph->p_memsz) {
            ret = -E_INVAL_ELF;
            goto bad_cleanup_mmap;
        }
        if (ph->p_filesz == 0) {
            continue ;
        }
```
(还在循环中)设定虚存管理数据结构的各段，并把它加入mm中。
```c
//(3.5) call mm_map fun to setup the new vma ( ph->p_va, ph->p_memsz)
        vm_flags = 0, perm = PTE_U;
        if (ph->p_flags & ELF_PF_X) vm_flags |= VM_EXEC;
        if (ph->p_flags & ELF_PF_W) vm_flags |= VM_WRITE;
        if (ph->p_flags & ELF_PF_R) vm_flags |= VM_READ;
        if (vm_flags & VM_WRITE) perm |= PTE_W;
        if ((ret = mm_map(mm, ph->p_va, ph->p_memsz, vm_flags, NULL)) != 0) {
            goto bad_cleanup_mmap;
        }
        unsigned char *from = binary + ph->p_offset;
        size_t off, size;
        uintptr_t start = ph->p_va, end, la = ROUNDDOWN(start, PGSIZE);

        ret = -E_NO_MEM;
```
(还在循环中)分配页，将执行内容拷贝到相应的内核虚拟地址中。建立好TEXT/DATA，BSS。

```c
//(3.6) alloc memory, and  copy the contents of every program section (from, from+end) to process's memory (la, la+end)
        end = ph->p_va + ph->p_filesz;
     //(3.6.1) copy TEXT/DATA section of bianry program
        while (start < end) {
            if ((page = pgdir_alloc_page(mm->pgdir, la, perm)) == NULL) {
                goto bad_cleanup_mmap;
            }
            off = start - la, size = PGSIZE - off, la += PGSIZE;
            if (end < la) {
                size -= la - end;
            }
            memcpy(page2kva(page) + off, from, size);
            start += size, from += size;
        }
       ......
```
(还在循环中)建立好用户栈。
```c
//(4) build user stack memory
    vm_flags = VM_READ | VM_WRITE | VM_STACK;
    if ((ret = mm_map(mm, USTACKTOP - USTACKSIZE, USTACKSIZE, vm_flags, NULL)) != 0) {
        goto bad_cleanup_mmap;
    }
    assert(pgdir_alloc_page(mm->pgdir, USTACKTOP-PGSIZE , PTE_USER) != NULL);
    assert(pgdir_alloc_page(mm->pgdir, USTACKTOP-2*PGSIZE , PTE_USER) != NULL);
    assert(pgdir_alloc_page(mm->pgdir, USTACKTOP-3*PGSIZE , PTE_USER) != NULL);
    assert(pgdir_alloc_page(mm->pgdir, USTACKTOP-4*PGSIZE , PTE_USER) != NULL);
```
进入用户空间
```c
//(5) set current process's mm, sr3, and set CR3 reg = physical addr of Page Directory
    mm_count_inc(mm);
    current->mm = mm;
    current->cr3 = PADDR(mm->pgdir);
    lcr3(PADDR(mm->pgdir));
```

设立好中断帧，当iret后就返回到用户进程

```c


    //(6) setup trapframe for user environment
    struct trapframe *tf = current->tf;
    memset(tf, 0, sizeof(struct trapframe));
    /* LAB5:EXERCISE1 YOUR CODE
     * should set tf_cs,tf_ds,tf_es,tf_ss,tf_esp,tf_eip,tf_eflags
     * NOTICE: If we set trapframe correctly, then the user level process can return to USER MODE from kernel. So
     *          tf_cs should be USER_CS segment (see memlayout.h)
     *          tf_ds=tf_es=tf_ss should be USER_DS segment
     *          tf_esp should be the top addr of user stack (USTACKTOP)
     *          tf_eip should be the entry point of this binary program (elf->e_entry)
     *          tf_eflags should be set to enable computer to produce Interrupt
     */
    tf->tf_cs = USER_CS;//设置cs、ds、es、ss、esp这些均为用户的虚拟空间
    tf->tf_ds = tf->tf_es = tf->tf_ss = USER_DS;
    tf->tf_esp = USTACKTOP;
    tf->tf_eip = elf->e_entry;//与文件有关
    tf->tf_eflags = FL_IF;//使能中断
    ret = 0;
out:
    return ret;
bad_cleanup_mmap:
    exit_mmap(mm);
bad_elf_cleanup_pgdir:
    put_pgdir(mm);
bad_pgdir_cleanup_mm:
    mm_destroy(mm);
bad_mm:
    goto out;
}
```
执行
```c
.text
.globl _start
_start:
    # set ebp for backtrace
    movl $0x0, %ebp

    # move down the esp register
    # since it may cause page fault in backtrace
    subl $0x20, %esp

    # call user-program function
    call umain
1:  jmp 1b

```

## 练习2: 父进程复制自己的内存空间给子进程
逐页复制内容，如果页表为空，分配新的页表，需要进行错误处理。
```c
int
copy_range(pde_t *to, pde_t *from, uintptr_t start, uintptr_t end, bool share) {
    assert(start % PGSIZE == 0 && end % PGSIZE == 0);
    assert(USER_ACCESS(start, end));
    // copy content by page unit.
    do {
        //call get_pte to find process A's pte according to the addr start
        pte_t *ptep = get_pte(from, start, 0), *nptep;
        if (ptep == NULL) {
            start = ROUNDDOWN(start + PTSIZE, PTSIZE);
            continue ;
        }
        //call get_pte to find process B's pte according to the addr start. If pte is NULL, just alloc a PT
        if (*ptep & PTE_P) {
            if ((nptep = get_pte(to, start, 1)) == NULL) {
                return -E_NO_MEM;
            }
        uint32_t perm = (*ptep & PTE_USER);
        //get page from ptep
        struct Page *page = pte2page(*ptep);
        // alloc a page for process B
        struct Page *npage=alloc_page();
        assert(page!=NULL);
        assert(npage!=NULL);
        int ret=0;
        /* LAB5:EXERCISE2 YOUR CODE
         * replicate content of page to npage, build the map of phy addr of nage with the linear addr start
         *
         * Some Useful MACROs and DEFINEs, you can use them in below implementation.
         * MACROs or Functions:
         *    page2kva(struct Page *page): return the kernel vritual addr of memory which page managed (SEE pmm.h)
         *    page_insert: build the map of phy addr of an Page with the linear addr la
         *    memcpy: typical memory copy function
         *
         * (1) find src_kvaddr: the kernel virtual address of page
         * (2) find dst_kvaddr: the kernel virtual address of npage
         * (3) memory copy from src_kvaddr to dst_kvaddr, size is PGSIZE
         * (4) build the map of phy addr of  nage with the linear addr start
         */
		uintptr_t src_kvaddr = page2kva(page);//找到虚拟地址
		uintptr_t dst_kvaddr = page2kva(npage);//找到虚拟地址
		memcpy(dst_kvaddr,src_kvaddr,PGSIZE);//以虚拟地址作为参数传输
		ret = page_insert(to,npage,start,perm);//建立映射（可能失败所以要有错误处理）
        assert(ret == 0);
        }
        start += PGSIZE;
    } while (start != 0 && start < end);
    return 0;
}
```
### 请在实验报告中简要说明如何设计实现”Copy on Write 机制“，给出概要设计，鼓励给出详细设计。
Copy on Write实际上是把资源的拷贝做了延迟，使用一个指针指向资源。当进行读操作的时候，并不影响数据的一致性，而进行写操作时，由于不同的进程可能对资源进行不同的操作，导致异常状态，此时就需要对资源进行拷贝
## 练习3: 阅读分析源代码，理解进程执行 fork/exec/wait/exit 的实现，以及系统调用的实现
* fork
1.检查进程数是否超过最大进程数
2.为子进程分配一个进程管理结构
3.将子进程的父进程设置为当前进程
4.设置栈
5.拷贝或分享内存管理数据结构
6.设置中断帧
7.(原子)为进程分配pid，并加入进程控制块，做好连接
8.唤醒子进程，返回子进程的pid
* exec
1.检查是否为用户进程，检查进程的内存是否合法
2.为存放进程名的数组申请空间，并赋值
3.切换到内核态的页表，清空当前进程的内存管理结构
4.加载新进程
5.为新进程设置好进程名
6.返回错误码
* exit
1.检查是否为用户进程
2.切换到内核态的页表，清空当前进程的内存管理结构
3.此时到`PROC_ZOMBIE`态
4.传递错误码
5.(原子操作)如果父进程处于`WT_CHILD`状态，唤醒父进程辅助回收
6.如果当前进程还有子进程，就将子进程设置为initproc的子进程，并将子进程设置为`PROC_ZOMBIE`同时将initproc设置为`WT_CHILD`唤醒initproc
7.执行调度，切换进程
* wait
1.检查是否为用户进程，检查内存是否合法
2.当pid不为0时，查找到pid对应的进程，否则检查房钱进程的所有子进程
3.如果发现有处于`PROC_ZOMBIE`的子进程，就将子进程的资源回收，把进程控制块从进程队列中删除，并释放子进程的内核堆栈和进程控制块。
4.如果有子进程但却没有处于`PROC_ZOMBIE`的子进程，就将当前进程设置为`PROC_SLEEPING`,并且等待子进程退出`WT_CHILD`，只有子进程退出了父进程才能退出。如果被唤醒，就重新进行整个过程。


