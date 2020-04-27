##  练习0：填写已有实验

##  练习1: 使用 Round Robin 调度算法（不需要编码）

### 请理解并分析sched_class中各个函数指针的用法，并结合Round Robin 调度算法描ucore的调度执行过程

在kern_init()中调用`sched_init()` 进行调度器的初始化。

初始化timer列表，调度类，run_queue，同时将rq的最大时间片设置为默认值 5，再调用调度器的初始化函数，进行调度器的初始化，最后打印出调度器的名字。（本练习为RR的调度器）

```c
void
sched_init(void) {
    list_init(&timer_list);

    sched_class = &default_sched_class;

    rq = &__rq;
    rq->max_time_slice = MAX_TIME_SLICE;
    sched_class->init(rq);

    cprintf("sched class: %s\n", sched_class->name);
}
```

在RR的初始化函数中，将队列初始化，同时将rq队列，记录进程数目的变量清零。

```c
RR_init(struct run_queue *rq) {
    list_init(&(rq->run_list));
    rq->proc_num = 0;
}
```

当 处于用户态的进程发生中断时进行调度。


```c
if (!in_kernel) {
            if (current->flags & PF_EXITING) {
                do_exit(-E_KILLED);
            }
            if (current->need_resched) {
                schedule();
            }
        }
```

设置时间片的更改

```c
RR_proc_tick(struct run_queue *rq, struct proc_struct *proc) {
    if (proc->time_slice > 0) {
        proc->time_slice --;
    }
    if (proc->time_slice == 0) {
        proc->need_resched = 1;
    }
}
```

当do_exit，do_wait，init_main，cpu_idle，lock，trap时就会执行调度函数，更改当前进程的调度状态，将`current->need_resched = 0`,如果当前进程的状态是`PROC_RUNNABLE`就将进行再次入队，否则不进行操作，并依照调度算法取出下一个要执行的进程，如果没有要执行的线程，next为空，就执行idleproc,当取出的下一个进程不为当前进程时就进行进程切换，要注意当前运行的进程并不在运行队列中。

```c
void
schedule(void) {
    bool intr_flag;
    struct proc_struct *next;
    local_intr_save(intr_flag);
    {
        current->need_resched = 0;
        if (current->state == PROC_RUNNABLE) {
            sched_class_enqueue(current);
        }
        if ((next = sched_class_pick_next()) != NULL) {
            sched_class_dequeue(next);
        }
        if (next == NULL) {
            next = idleproc;
        }
        next->runs ++;
        if (next != current) {
            proc_run(next);
        }
    }
    local_intr_restore(intr_flag);
}
```

### 请在实验报告中简要说明如何设计实现”多级反馈队列调度算法“，给出概要设计，鼓励给出详细设计

多级反馈队列调度算法，依照进程不同的优先级进行不同的调度，当高优先级队列为空时才会执行低优先级的进程，所以当时间片用完时，需要执行sched_class_enqueue操作，不能再简单的将时间片置位最大值，把进程放在队列的尾部，而需要将进程加入低一级队列中，这就需要维护一个proc_pri变量表示优先级。

## 练习2: 实现 Stride Scheduling 调度算法（需要编码）

参考答案中给出的算法似乎语义有错误，pass与stride反了

目前stride是pass ，BIG_SKEW_STRIDE/lab6_priority 是真正的stride

```c
#include <defs.h>
#include <list.h>
#include <proc.h>
#include <assert.h>
#include <default_sched.h>

#define BIG_SKEW_STRIDE 0x7FFFFFFF
static void
sched_class_init(struct run_queue *rq) {
    list_init(&(rq->run_list));
    rq->lab6_run_pool = NULL;
    rq->proc_num = 0;
}

static int
proc_stride_comp_f(void *a, void *b)
{
     struct proc_struct *p = le2proc(a, lab6_run_pool);
     struct proc_struct *q = le2proc(b, lab6_run_pool);
     int32_t c = p->lab6_stride - q->lab6_stride;
     if (c > 0) return 1;
     else if (c == 0) return 0;
     else return -1;
}



static void
sched_class_enqueue(struct run_queue *rq, struct proc_struct *proc) {
    rq->lab6_run_pool = skew_heap_insert(rq->lab6_run_pool,&(proc->lab6_run_pool));
    if (proc->time_slice == 0 || proc->time_slice > rq->max_time_slice) {
        proc->time_slice = rq->max_time_slice;
    }
    proc->rq = rq;
    rq->proc_num ++;
}

static void
skew_heap_remove(struct run_queue *rq, struct proc_struct *proc) {
    rq->lab6_run_pool = skew_heap_insert(rq->lab6_run_pool,&(proc->lab6_run_pool));
    rq->proc_num --;
}

static struct proc_struct *
sched_class_pick_next(struct run_queue *rq) {
    if (rq->lab6_run_pool == NULL) return NULL;
    struct proc_struct *p = le2proc(rq->lab6_run_pool, lab6_run_pool);
    if (p->lab6_priority == 0)
          p->lab6_stride += BIG_STRIDE;
    else p->lab6_stride += BIG_STRIDE / p->lab6_priority;
    return p;
}

static void
RR_proc_tick(struct run_queue *rq, struct proc_struct *proc) {
    if (proc->time_slice > 0) {
        proc->time_slice --;
    }
    if (proc->time_slice == 0) {
        proc->need_resched = 1;
    }
}

struct sched_class default_sched_class = {
    .name = "stride_scheduler",
    .init = stride_init,
    .enqueue = stride_enqueue,
    .dequeue = stride_dequeue,
    .pick_next = stride_pick_next,
    .proc_tick = stride_proc_tick,
};


```



