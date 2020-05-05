## glibc 对系统调用的封装

对于系统调用，Linux 提供了 glibc 这个中介。它`更熟悉系统调用的细节，并且可以封装成更加友好的接口`。你可以直接用。

我们以最常用的系统调用 open，打开一个文件为线索，看看系统调用是怎么实现的。这一节我们仅仅会解析到从 glibc 如何调用到内核的 open.

### 32位系统调用过程

首先，我们是在`用户态进程`里面调用 open 函数。

```c
int open(const char *pathname, int flags, mode_t mode)
```

这里，我们将`请求参数`放在`寄存器`里面，根据系`统调用的名称`，得到`系统调用号`，放在`寄存器 eax` 里面，然后执行 `ENTER_KERNEL`。

这里面的 ENTER_KERNEL 是什么呢？

```c
# define ENTER_KERNEL int $0x80
```

int 就是 interrupt，也就是“中断”的意思。int $0x80 就是触发一个软中断，通过它就可以`陷入（trap）内核`。

进入内核之前，保存所有的寄存器，然后调用 do_syscall_32_irqs_on。它的实现如下：

```c
static __always_inline void do_syscall_32_irqs_on(struct pt_regs *regs)
{
  struct thread_info *ti = current_thread_info();
  unsigned int nr = (unsigned int)regs->orig_ax;
......
  if (likely(nr < IA32_NR_syscalls)) {
    regs->ax = ia32_sys_call_table[nr](
      (unsigned int)regs->bx, (unsigned int)regs->cx,
      (unsigned int)regs->dx, (unsigned int)regs->si,
      (unsigned int)regs->di, (unsigned int)regs->bp);
  }
  syscall_return_slowpath(regs);
}
```

在这里，我们看到，将`系统调用号`从 `eax` 里面取出来，然后`根据系统调用号`，在`系统调用表`中找到相应的`函数`进行调用，并将寄存器中保存的参数取出来，作为函数参数。

当系统调用结束之后，在 entry_INT80_32 之后，紧接着调用的是 `INTERRUPT_RETURN`，我们能够找到它的定义，也就是 iret。

```c
#define INTERRUPT_RETURN                iret
```

iret 指令将`原来用户态保存的现场恢复回来`，包含代码段、指令指针寄存器等。这时候用户态进程恢复执行。

这里我总结一下 32 位的系统调用是如何执行的。

![566299fe7411161bae25b62e7fe20506.jpg](../../_img/566299fe7411161bae25b62e7fe20506.jpg)

### 64位系统调用过程

64位的系统调用和32位流程上差不多，只是有一些地方有点区别。

首先，这里真正的调用，不是用中断了，而是改用 syscall 指令了。而且使用的寄存器也不一样，syscall 指令使用了一种特殊的寄存器，我们叫特殊模块寄存器（Model Specific Registers，简称 MSR）

![566299fe7411161bae25b62e7fe20506.jpg](../../_img/1fc62ab8406c218de6e0b8c7e01fdbd7.jpg)

### 系统调用表

32 位的系统调用表定义在 arch/x86/entry/syscalls/syscall_32.tbl 文件里。例如 open 是这样定义的：

```c
5  i386  open      sys_open  compat_sys_open
```

64 位的系统调用定义在另一个文件 arch/x86/entry/syscalls/syscall_64.tbl 里。例如 open 是这样定义的：

```c
2  common  open      sys_open
```

第一列的数字是`系统调用号`。可以看出，32 位和 64 位的系统调用号是不一样的。第三列是`系统调用的名字`，第四列是`系统调用在内核的实现函数`。不过，它们都是以 sys_ 开头。

### 参考链接

[极客时间-09 | 系统调用：公司成立好了就要开始接项目](https://time.geekbang.org/column/article/90394)