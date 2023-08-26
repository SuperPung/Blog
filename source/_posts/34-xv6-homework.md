---
url: xv6-homework
title: xv6 实验
date: 2020-12-29 19:01:25
categories: [技术]
tags: [操作系统实验]
---

Reports of MIT 6.828 xv6 homeworks

<!--more-->

# xv6 system calls

## Part One: System call tracing

修改xv6内核，在实现系统调用时打印系统调用的名称和返回值。

只需在`syscall.c`中系统调用时增加一行输出即可。

为了输出系统调用名称，增加对应的数组：

```c
static char *syscalls_name[] = {
[SYS_fork]    "fork",
[SYS_exit]    "exit",
[SYS_wait]    "wait",
[SYS_pipe]    "pipe",
[SYS_read]    "read",
[SYS_kill]    "kill",
[SYS_exec]    "exec",
[SYS_fstat]   "fstat",
[SYS_chdir]   "chdir",
[SYS_dup]     "dup",
[SYS_getpid]  "getpid",
[SYS_sbrk]    "sbrk",
[SYS_sleep]   "sleep",
[SYS_uptime]  "uptime",
[SYS_open]    "open",
[SYS_write]   "write",
[SYS_mknod]   "mknod",
[SYS_unlink]  "unlink",
[SYS_link]    "link",
[SYS_mkdir]   "mkdir",
[SYS_close]   "close",
};
```

利用定义的数组，在`syscall`函数中增加输出（11行）：

```c
void
syscall(void)
{
  int num;
  struct proc *curproc = myproc();

  num = curproc->tf->eax;
  if(num > 0 && num < NELEM(syscalls) && syscalls[num]) {
    curproc->tf->eax = syscalls[num]();
    cprintf("%s -> %d\n", syscalls_name[num], curproc->tf->eax);
  } else {
    cprintf("%d %s: unknown sys call %d\n",
            curproc->pid, curproc->name, num);
    curproc->tf->eax = -1;
  }
}
```

重新运行`make qemu`，得到输出如下：

```shell
$ make qemu
qemu-system-i386 -serial mon:stdio -drive file=fs.img,index=1,media=disk,format=raw -drive file=xv6.img,index=0,media=disk,format=raw -smp 2 -m 512
xv6...
cpu1: starting 1
cpu0: starting 0
sb: size 1000 nblocks 941 ninodes 200 nlog 30 logstart 2 inodestart 32 bmap start 58
exec -> 0
open -> 0
dup -> 1
dup -> 2
iwrite -> 1
nwrite -> 1
iwrite -> 1
twrite -> 1
:write -> 1
 write -> 1
swrite -> 1
twrite -> 1
awrite -> 1
rwrite -> 1
twrite -> 1
iwrite -> 1
nwrite -> 1
gwrite -> 1
 write -> 1
swrite -> 1
hwrite -> 1

write -> 1
fork -> 2
exec -> 0
open -> 3
close -> 0
$write -> 1
 write -> 1


```

## Part Two: Date system call

增加一个新的系统调用，输出当前UTC（Coordinated Universal Time，协调世界时）时间。

根据提示，通过分析已经实现的系统调用（如`uptime`）来创建新的系统调用`date`。

`grep -n uptime *.[chS]`查看所有含`uptime`的`.c`、`.h`和`.S`文件：

```shell
syscall.c:105:extern int sys_uptime(void);
syscall.c:124:[SYS_uptime]  sys_uptime,
syscall.c:149:[SYS_uptime]  "uptime",
syscall.h:15:#define SYS_uptime 14
sysproc.c:83:sys_uptime(void)
user.h:25:int uptime(void);
usys.S:31:SYSCALL(uptime)
```

依次在上述文件的对应位置添加系统调用`date`。

`syscall.h`（添加系统调用编号）：

```c
 #define SYS_date 22
```

`syscall.c`（添加系统调用函数的外部声明）:

```c
[SYS_date]  "date"
};
···
extern int sys_date(void);
···
[SYS_date] sys_date
```

`user.h`（添加用户态函数的定义）：

```c
int date(struct rtcdate*);
```

`usys.S`（添加用户态函数的实现）：

```C
SYSCALL(date)
```

`sysproc.c`（添加系统调用函数的实现）：

```c
int
sys_date(struct rtcdate *r)
{
    if (argptr(0, (void *)&r, sizeof(*r)) < 0)
       return -1;
  	cmostime(r);
		return 0;
 }
```

根据提示，新建文件 `date.c` ，添加使用此系统调用函数的方法：

```c
#include "types.h"
#include "user.h"
#include "date.h"

int main(int argc, char *argv[])
{
    struct rtcdate r;
    if (date(&r))
    {
        printf(2, "date failed\n");
        exit();
    }

    // your code to print the time in any format you like...
    printf(1, "%d/%d/%d %d:%d:%d\n", r.month, r.day, r.year, r.hour, r.minute, r.second);

    exit();
}
```

在`MakeFile`中添加`UPROGS`对应的命令：

```makefile
    _date\
```

注释掉`Part One`的更改，重新运行`make qemu`，得到输出如下：

```c
$ date
12/28 2020 7:15:27
```

# xv6 lazy page allocation

## Part One: Eliminate allocation from sbrk()

实现页面延迟分配的第一步，消除系统调用`sbrk`中的分配。

修改系统调用`sbrk`的实际实现`sys_sbrk`，使它只将进程的内存空间大小增加`n`，而不进行实际的分配。

在`sysproc.c`的`sys_sbrk`函数中，根据提示，增加进程大小`n`，并返回旧的大小。不分配内存，注释掉`growproc`函数的调用：

```c
int
sys_sbrk(void)
{
  int addr;
  int n;

  if(argint(0, &n) < 0)
    return -1;
  addr = myproc()->sz;
  myproc()->sz += n;
  // if(growproc(n) < 0)
  //   return -1;
  return addr;
}
```

重新运行`make qemu`并键入`echo hi`，得到输出如下：

```shell
$ echo hi
pid 3 sh: trap 14 err 6 on cpu 0 eip 0x12e9 addr 0x4004--kill proc
```

## Part Two: Lazy allocation

实现页面延迟分配的第二步，响应第一步造成的错误，使进程继续执行。

根据提示，修改`trap.c`中的代码，以通过在故障地址处映射新分配的物理内存页来响应用户空间中的页面错误，然后返回到用户空间以使进程继续执行。

根据提示，在`trap.c`的`trap`函数`switch`语句中增加`case`：

```c
  case T_PGFLT: {
    char *mem;
    mem = kalloc();
    if(mem != 0){
			uint va = PGROUNDDOWN(rcr2());
			memset(mem, 0, PGSIZE);
			extern int mappages(pde_t *pgdir, void *va, uint size, uint pa, int perm);
			if(mappages(myproc()->pgdir,(void *)va, PGSIZE, V2P(mem), PTE_W|PTE_U) >= 0)
				break;
    }
  }
```

若在`case T_PGFLT:`后未加`{}`，运行`make qemu`则会报以下错误：

```shell
$ make qemu
gcc -fno-pic -static -fno-builtin -fno-strict-aliasing -O2 -Wall -MD -ggdb -m32 -Werror -fno-omit-frame-pointer -fno-stack-protector   -c -o trap.o trap.c
trap.c: In function ‘trap’:
trap.c:86: error: a label can only be part of a statement and a declaration is not a statement
<内置>: recipe for target 'trap.o' failed
make: *** [trap.o] Error 1
```

根据提示，为了在`trap.c`中调用`mappages`函数，需要删除`vm.c`中`mappages`函数声明中的`static`。

重新运行`make qemu`并键入`echo hi`，得到输出如下：

```shell
$ echo hi
hi
```

# xv6 CPU alarm

增加系统调用`alarm`，当进程使用CPU时间时，它会定期向进程发出警报。

仿照实验`xv6 system calls`中增加系统调用`date`的方法增加系统调用`alarm`，按照提示操作。

增加系统调用`alarm`：

`syscall.h`（添加系统调用编号）：

```c
 #define SYS_alarm 23
```

`syscall.c`（添加系统调用函数的外部声明）:

```c
[SYS_alarm]  "alarm"
};
···
extern int sys_alarm(void);
···
[SYS_alarm] sys_alarm
```

`user.h`（添加用户态函数的定义）：

```c
int alarm(int ticks, void(*handler)());
```

`usys.S`（添加用户态函数的实现）：

```C
SYSCALL(alarm)
```

`sysproc.c`（添加系统调用函数的实现）：

```c
// cpu alarm
int
sys_alarm(void)
{
  int ticks;
  void (*handler)();

  if(argint(0, &ticks) < 0)
    return -1;
  if(argptr(1, (char**)&handler, 1) < 0)
    return -1;
  myproc()->alarmticks = ticks;
  myproc()->alarmhandler = handler;
  return 0;
}
```

根据提示，在 `proc.h` 的结构体 `proc` 中添加：

```c
  int alarmticks;
  int curticks;
  void (*alarmhandler)();
```

根据提示，新建文件`alarmtest.c`，添加使用此系统调用函数的方法：

```c
#include "types.h"
#include "stat.h"
#include "user.h"

void periodic();

int
main(int argc, char *argv[])
{
  int i;
  printf(1, "alarmtest starting\n");
  alarm(10, periodic);
  for(i = 0; i < 25*500000; i++){
    if((i % 250000) == 0)
      write(2, ".", 1);
  }
  exit();
}

void
periodic()
{
  printf(1, "alarm!\n");
}
```

在`MakeFile`中添加`UPROGS`对应的命令：

```makefile
    _alarmtest\
```

根据提示，在`trap.c`中处理时钟中断添加`handler`：

```c
  case T_IRQ0 + IRQ_TIMER:
    if(cpuid() == 0){
      acquire(&tickslock);
      ticks++;
      wakeup(&ticks);
      release(&tickslock);
    }

    if(myproc() != 0 && (tf->cs & 3) == 3) {
      myproc()->curticks++;
      if(myproc()->alarmticks == myproc()->curticks) {
        myproc()->curticks = 0;
        tf->esp -= 4;
        *((uint *)(tf->esp)) = tf->eip;
        tf->eip =(uint)myproc()->alarmhandler;
      }
    }
    lapiceoi();
    break;
```

重新运行`make qemu`并键入`alarmtest`，得到输出如下：

```shell
$ alarmtest
alarmtest starting
...........................................alarm!
```

根据提示：

> If you only see one "alarm!", try increasing the number of iterations in `alarmtest.c` by 10x.

将`alarmtest.c`中`for`循环的判断条件改为`i < 250*500000`，再次运行：

```shell
$ alarmtest
alarmtest starting
...............................................alarm!
.........................................................................................................alarm!
....................................................................................................................alarm!
............................................................................alarm!
..............................................................................alarm!
...........................................................................alarm!
```

# xv6 locking

探索中断和锁的相互作用。

## Don't do this

执行以下代码：

```c
  struct spinlock lk;
  initlock(&lk, "test lock");
  acquire(&lk);
  acquire(&lk);
```

根据提示，在 `spinlock.c`的`acquire` 函数中发现：

```c
  if(holding(lk))
    panic("acquire");
```

则执行代码后，连续两次申请同一个`spinlock`，内核将`panic`。

## Interrupts in ide.c

未修改时：

```shell
$ make qemu
qemu-system-i386 -serial mon:stdio -drive file=fs.img,index=1,media=disk,format=raw -drive file=xv6.img,index=0,media=disk,format=raw -smp 2 -m 512
xv6...
cpu1: starting 1
cpu0: starting 0
sb: size 1000 nblocks 941 ninodes 200 nlog 30 logstart 2 inodestart 32 bmap start 58
init: starting sh
```

在`ide.c`的`iderw`函数中，在调用`acquire`函数后调用`sti`函数，在调用`release`函数前调用`cli`函数：

```c
void
iderw(struct buf *b)
{
  struct buf **pp;

  if(!holdingsleep(&b->lock))
    panic("iderw: buf not locked");
  if((b->flags & (B_VALID|B_DIRTY)) == B_VALID)
    panic("iderw: nothing to do");
  if(b->dev != 0 && !havedisk1)
    panic("iderw: ide disk 1 not present");

  acquire(&idelock);  //DOC:acquire-lock
  sti();

  // Append b to idequeue.
  b->qnext = 0;
  for(pp=&idequeue; *pp; pp=&(*pp)->qnext)  //DOC:insert-queue
    ;
  *pp = b;

  // Start disk if necessary.
  if(idequeue == b)
    idestart(b);

  // Wait for request to finish.
  while((b->flags & (B_VALID|B_DIRTY)) != B_VALID){
    sleep(b, &idelock);
  }

  cli();
  release(&idelock);
}
```

重新运行`make qemu`，得到输出如下，发生`panic`：

```shell
$ make qemu
qemu-system-i386 -serial mon:stdio -drive file=fs.img,index=1,media=disk,format=raw -drive file=xv6.img,index=0,media=disk,format=raw -smp 2 -m 512
xv6...
cpu1: starting 1
cpu0: starting 0
sb: size 1000 nblocks 941 ninodes 200 nlog 30 logstart 2 inodestart 32 bmap start 58
init: starting sh
lapicid 1: panic: sched locks
 80103cde 80103eb3 801058cd 801055d1 801001c6 801017af 80106d74 80100afb 801049df 801047ae
```

发生`panic`的原因可能是在`release`释放锁之前发生中断，为了响应中断进行调度，此时又有新的锁申请，所以发生`panic`。

## Interrupts in file.c

根据提示，在`file.c`的`filealloc`函数中，在调用`acquire`函数后调用`sti`函数，在每次调用`release`函数前调用`cli`函数：

```c
// Allocate a file structure.
struct file*
filealloc(void)
{
  struct file *f;

  acquire(&ftable.lock);
  sti();
  for(f = ftable.file; f < ftable.file + NFILE; f++){
    if(f->ref == 0){
      f->ref = 1;
      cli();
      release(&ftable.lock);
      return f;
    }
  }
  cli();
  release(&ftable.lock);
  return 0;
}
```

需要`#include "x86.h"`。

重新运行`make qemu`，得到输出如下，未发生`panic`：

```shell
$ make qemu
qemu-system-i386 -serial mon:stdio -drive file=fs.img,index=1,media=disk,format=raw -drive file=xv6.img,index=0,media=disk,format=raw -smp 2 -m 512
xv6...
cpu1: starting 1
cpu0: starting 0
sb: size 1000 nblocks 941 ninodes 200 nlog 30 logstart 2 inodestart 32 bmap start 58
init: starting sh
```

未发生`panic`的原因可能是`filealloc`占用时间短、发生次数较少，只有极小概率与其他读写文件发生冲突。

## xv6 lock implementation

相关代码如下：

```c
// Release the lock.
void
release(struct spinlock *lk)
{
  if(!holding(lk))
    panic("release");

  lk->pcs[0] = 0;
  lk->cpu = 0;

  // Tell the C compiler and the processor to not move loads or stores
  // past this point, to ensure that all the stores in the critical
  // section are visible to other cores before the lock is released.
  // Both the C compiler and the hardware may re-order loads and
  // stores; __sync_synchronize() tells them both not to.
  __sync_synchronize();

  // Release the lock, equivalent to lk->locked = 0.
  // This code can't use a C assignment, since it might
  // not be atomic. A real OS would use C atomics here.
  asm volatile("movl $0, %0" : "+m" (lk->locked) : );

  popcli();
}
```

若`release`函数先将`lk->locked`清零，其他正在`acquire()`等待的进程将立即执行，将会更改`lk->pcs[0]`和`lk->cpu`，但此时`release`函数也将修改`lk->pcs[0]`和`lk->cpu`值，故发生错误。

# bigger files for xv6

增加xv6文件的最大大小，从`140 sectors`到`16523 sectors`。

根据提示，修改`Makefile`：

- 修改`CPUS := 2`为`CPUS := 1`
- 添加`QWMUEXTRA = -snapshot`
- 在`UPROGS`中添加`_big\`：

根据提示，修改`param.h`：

- 修改`#define FSSIZE 1000`为`#define FSSIZE 20000`

根据提示，添加文件`big.c`：

```c
#include "types.h"
#include "stat.h"
#include "user.h"
#include "fcntl.h"

int
main()
{
  char buf[512];
  int fd, i, sectors;

  fd = open("big.file", O_CREATE | O_WRONLY);
  if(fd < 0){
    printf(2, "big: cannot open big.file for writing\n");
    exit();
  }

  sectors = 0;
  while(1){
    *(int*)buf = sectors;
    int cc = write(fd, buf, sizeof(buf));
    if(cc <= 0)
      break;
    sectors++;
	if (sectors % 100 == 0)
		printf(2, ".");
  }

  printf(1, "\nwrote %d sectors\n", sectors);

  close(fd);
  fd = open("big.file", O_RDONLY);
  if(fd < 0){
    printf(2, "big: cannot re-open big.file for reading\n");
    exit();
  }
  for(i = 0; i < sectors; i++){
    int cc = read(fd, buf, sizeof(buf));
    if(cc <= 0){
      printf(2, "big: read error at sector %d\n", i);
      exit();
    }
    if(*(int*)buf != i){
      printf(2, "big: read the wrong data (%d) for sector %d\n",
             *(int*)buf, i);
      exit();
    }
  }

  printf(1, "done; ok\n");

  exit();
}
```

此时重新运行`make qemu`并键入`big`，显示`wrote 140 sectors`：

```shell
$ big
.
wrote 140 sectors
done; ok
```

根据提示，修改`fs.c`中的`bmap`函数：

```c
static uint
bmap(struct inode *ip, uint bn)
{
  uint addr, *a;
  struct buf *bp;
  struct buf *bp2;
  if(bn < NDIRECT){
    if((addr = ip->addrs[bn]) == 0)
      ip->addrs[bn] = addr = balloc(ip->dev);
    return addr;
  }
  bn -= NDIRECT;

  if(bn < NINDIRECT){
    if((addr = ip->addrs[NDIRECT]) == 0)
      ip->addrs[NDIRECT] = addr = balloc(ip->dev);

    bp = bread(ip->dev, addr);
    a = (uint*)bp->data;
    if((addr = a[bn]) == 0){
      a[bn] = addr = balloc(ip->dev);
      log_write(bp);
    }
    brelse(bp);
    return addr;
  }

  bn -= NINDIRECT;

  if (bn < NINDIRECT * NINDIRECT) {
    if((addr = ip->addrs[NDIRECT+1]) == 0)
		ip->addrs[NDIRECT+1] = addr = balloc(ip->dev);

	bp = bread(ip->dev, addr);

    a = (uint *)bp->data;
    if ((addr = a[bn/NINDIRECT]) == 0) {
       a[bn/NINDIRECT] = addr = balloc(ip->dev);
	   log_write(bp);
	}

	bp2 = bread(ip->dev, addr);

	a = (uint *)bp2->data;
	if ((addr = a[bn%NINDIRECT]) == 0) {
	  a[bn%NINDIRECT] = addr = balloc(ip->dev);
      log_write(bp2);
	}

	brelse(bp2);
	brelse(bp);
	return addr;
  }

  panic("bmap: out of range");
}
```

根据提示，修改`fs.h`：

- 修改`#define NDIRECT 12`为`#define NDIRECT 11`
- 修改`#define MAXFILE (NDIRECT + NINDIRECT)`为`#define MAXFILE (NDIRECT + NINDIRECT + NINDIRECT * NINDIRECT)`
- 修改`uint addrs[NDIRECT+1]`为`uint addrs[NDIRECT+2]`

根据提示，修改`file.h`：

- 修改`uint addrs[NDIRECT+1]`为`uint addrs[NDIRECT+2]`

最终重新运行`make qemu`并键入`big`，显示`wrote 16523 sectors`：

```shell
$ big
.....................................................................................................................................................................
wrote 16523 sectors
done; ok
```
