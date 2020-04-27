## 练习1
### 请说明proc_struct中struct context context和struct trapframe *tf成员变量含义和在本实验中的作用是啥？
将state置位PROC_UNINIT，表示还没有初始化
cr3置位内核的cr3
其余变量全部清零，或赋值为NULL，表示未进行初始化。
```
		proc->state = PROC_UNINIT;//表示还没有初始化
        proc->pid = -1;
        proc->runs = 0;
        proc->kstack = 0;//表明是一个内核进程
        proc->need_resched = 0;
        proc->parent = NULL;
        proc->mm = NULL;
        memset(&(proc->context), 0, sizeof(struct context));
        proc->tf = NULL;
        proc->cr3 = boot_cr3;//内核进程
        proc->flags = 0;
        memset(proc->name, 0, PROC_NAME_LEN);
```
context存储的是进程的上下文，将上下文存储好，当进程再次被调度时，就可以从上次执行到的状态再次开始执行。
tf 保存的是发生中断时寄存器状态，便于在中断处理完成后能够恢复现场，继续执行。

## 练习2
### 请说明ucore是否做到给每个新fork的线程一个唯一的id？
可以为每个新fork的线程分配唯一的pid，注意到在get_pid中，static_assert(MAX_PID > MAX_PROCESS)，说明pid的范围是比进程数要多的。而分配pid的过程是一个无线循环，只有当检查完proc_list的全部线程，都不冲突时才会跳出循环分配出pid。同时分配时的操作原子性也保证了上述条件。
```
if (proc = alloc_proc() != NULL){
		proc->parent = current;//参考答案，将设置线程的父线程为当前线程。
		if(setup_kstack(proc) != -E_NO_MEM){
			if (copy_mm(clone_flags&CLONE_VM,proc)!=0)
				goto bad_fork_cleanup_kstack;//答案的错误处理
			copy_thread(proc,stack,tf);
			bool intr_flag;
			local_intr_save(intr_flag);//参考答案：关闭中断处理机制，保证分配操作的原子性。
			{
				int pid = get_pid(proc);
				proc->pid = pid;
				hash_proc(proc);
				list_add_after(proc_list,proc);
				nr_process++;
			}
			local_intr_restore(intr_flag);//重新使能中断
			wakeup_proc(proc);
			ret = pid;
		}
		else{
			goto bad_fork_cleanup_kstack;
		}
	}
	else{
		goto bad_fork_cleanup_proc;
	}
	
fork_out:
    return ret;

bad_fork_cleanup_kstack:
    put_kstack(proc);
bad_fork_cleanup_proc:
    kfree(proc);
    goto fork_out;
```
## 练习三
### 在本实验的执行过程中，创建且运行了几个内核线程？
两个idleproc和init_main
### 语句local_intr_save(intr_flag);....local_intr_restore(intr_flag);在这里有何作用?请说明理由
local_intr_save(intr_flag);//关闭中断处理机制，保证分配操作的原子性。
local_intr_restore(intr_flag);//重新使能中断 此时已经完成进程切换
关闭中断机制十分重要，这保证了操作的原子性，防止出现错误，导致系统崩溃。

### 阅读代码，理解 proc_run 函数和它调用的函数如何完成进程切换的。

proc_run函数：

```
proc_run(struct proc_struct *proc) {
    if (proc != current) {//当调用函数的进程不是当前进程时执行进程间的切换
        bool intr_flag;
        struct proc_struct *prev = current, *next = proc;
        local_intr_save(intr_flag);//关闭中断处理机制，保证分配操作的原子性。
        {
            current = proc;//重新设置当前线程为调用参数的线程
            load_esp0(next->kstack + KSTACKSIZE);//加载栈指针
            lcr3(next->cr3);//加载GDT
            switch_to(&(prev->context), &(next->context));//进行上下文的切换
        }
        local_intr_restore(intr_flag);//重新使能中断 此时已经完成进程切换
    }
}
```
switch_to

把from进程的寄存器保存，注意esp，eip同时取出to进程的寄存器内容，最后取出esp后push进eip，ret此时就完成了上下文的切换。

```
.text
.globl switch_to
switch_to:                      # switch_to(from, to)

    # save from's registers
    movl 4(%esp), %eax          # eax points to from
    popl 0(%eax)                # save eip !popl
    movl %esp, 4(%eax)          # save esp::context of from
    movl %ebx, 8(%eax)          # save ebx::context of from
    movl %ecx, 12(%eax)         # save ecx::context of from
    movl %edx, 16(%eax)         # save edx::context of from
    movl %esi, 20(%eax)         # save esi::context of from
    movl %edi, 24(%eax)         # save edi::context of from
    movl %ebp, 28(%eax)         # save ebp::context of from

    # restore to's registers
    movl 4(%esp), %eax          # not 8(%esp): popped return address already
                                # eax now points to to
    movl 28(%eax), %ebp         # restore ebp::context of to
    movl 24(%eax), %edi         # restore edi::context of to
    movl 20(%eax), %esi         # restore esi::context of to
    movl 16(%eax), %edx         # restore edx::context of to
    movl 12(%eax), %ecx         # restore ecx::context of to
    movl 8(%eax), %ebx          # restore ebx::context of to
    movl 4(%eax), %esp          # restore esp::context of to

    pushl 0(%eax)               # push eip

    ret


```

schedule函数调用proc_run，cpu_idle当发现序调度的时候就执行调度。
