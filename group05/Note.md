# 死锁 Deadlock
> 本笔记中 🎬[x] 标记表示此处内容对应 Canvas 系统中本课程课件 "Chapter06-Deadlocks.ppt" 第 x 页内容

 ## 基础

### Semaphore信号量函数API

引用：https://linux.die.net/man/7/sem_overview

- 信号量可被用于同步线程。以Posix semaphore的规范为例，信号量相关API被在C中包括在<semaphore.h>或<sys/sem.h>中。

- 而Posix规定的函数接口包括：

```
int sem_init(sem_t *sem, int pshared, unsigned int value);
int sem_close(sem_t *sem);
sem_t *sem_open(const char *name, int oflag);
sem_t *sem_open(const char *name, int oflag, mode_t mode, unsigned int value);
int sem_post(sem_t *sem);
int sem_wait(sem_t *sem);
int sem_trywait(sem_t *sem);
int sem_timedwait(sem_t *sem, const struct timespec *abs_timeout);
int sem_getvalue(sem_t *sem, int *sval);
int sem_destroy(sem_t *sem);
int sem_unlink(const char *name);
```

- 对它的操作基本类似先前的pthread_cond相关的操作，调用sem_wait其会将对应sem_t结构里的sem value减一，若减至零则进程阻塞。而sem_post会将对应sem_t结构里的sem value加一，并唤醒所有等待sem_t的进程。
- 而区别在于sem_open操作可以创建具名信号量，在name参数输入其外部名字，在oflag参数为调用输入O_CREAT即可创建一个新的具名信号量，而后其它进程可以通过sem_open操作在name参数输入其外部名字，在oflag参数输入O_EXCL即可共享该信号量。而sem_unlink可以解除当前进程与对应信号量的联系。而为了实现这一点，这些具名信号量是被存在内核空间的，它只有在被调用sem_destoy后才会被销毁，否则将一直占用内核空间。而创建匿名信号量就等同与pthread_cond的操作。

### Deadlock死锁

- 定义：在操作系统中，一个**进程或线程**正在**等待被其他进程占用的资源**，而正在占用该资源的进程同样正在等待另一资源。若这种**等待状态永远无法改变**，则称该系统进入了**死锁**
  
- https://en.wikipedia.org/wiki/Deadlock
  
- 死锁的四个条件（需要共同成立，破解死锁仅需解决任意条件）

  - 共享互斥（Mutual exclusion）
   - 会阻塞进程并持续等待资源（Hold and wait）
   - 资源仅能由占有的进程释放，不允许其他进程抢占（No preemption）
   - 需求关系成环（Circular wait condition）

- 死锁例子： 🎬[7..10]

  ![](.\DeadLockSample(1).png)

  ![](.\DeadLockSample(2).png)

  ![](.\DeadLockSample(3).png)

  其中(a)(b)(c)为ABC进程各自状态，(d)(k)为进程切换的不同顺序

## 解决死锁的方法
共四点方案，其中第一点实际使用较少 🎬[11]，后续详细说明第二至第四点
1. 无视死锁
2. 检测到死锁并尝试恢复
3. 通过更稳定的资源分配算法确保运行过程中不出现死锁
4. 通过系统结构性设计预防死锁出现
### 检测死锁与恢复
🎬[12..19]
- 将进程和资源视为节点，需求 / 占有关系视为有向边，若该图上存在环则出现死锁
	- 进程 P 占有资源 S，对应一条 S 指向 P 的边
	- 进程 P 需要资源 S，对应一条 P 指向 S 的边
	- 不存在进程指向进程或资源指向资源的边
	- 例：🎬[13]
- 即判断有向图中是否存在环

🎬[20]

* 三种恢复的方法。
  * **Preemption**: 重新分配可抢占的资源（存档-重新分配-新进程释放-读档-旧进程启动）
  * **Rollback**: 在进程中插入若干*checkpoints*，当发生*deadlock*时，使其中一个进程回退到占有冲突资源之前的*checkpoint*，将冲突资源分配给另一个进程以解锁。但回退不能引发不期望的影响，如重复输出。
  * **Killing Processes**: *deadlock*发生时终止若干进程以解除死锁，例如终止一个环外的进程来获得额外的资源，在能带来额外所需资源的进程中，优先选择终止后无不良影响的进程。

### 银行家算法
🎬[21..23]
- 场	景：复数个进程，每个进程已经占有了部分资源，且运行过程中所需资源存在一个上限。系统此时还剩余若干空闲资源
- 安全 / 不安全状态：若系统此时能通过安排这些进程以某一顺序运行使得所有进程都完成，则为安全状态；若无法完成则为不安全状态（Safe and unsafe States）
	- 系统将空闲资源全部提供给 (运行所需资源上限 - 已占有资源) 最小的进程，若该进程能顺利运行，则将其运行完成，然后回收所有该进程占有的资源补充入系统空闲资源
	- 重复上述过程，若能所有进程都能完成则为安全状态，否则为不安全状态
> 1. 例如申请内存等场景中，死锁出现条件中的需求关系成环如何理解？
> - 将所有内存视为一个节点。一个进程申请内存，视为存在一条该进程指向内存的边。该进程又已占有部分内存，即存在一条内存指向该进程的边。故成环，该环过大则导致不安全状态。更具体地说，或许可以根据内存分配粒度将内存分为很多个节点，申请和占有都是进程与若干个内存结点之间的边
>
> 2. 由于银行家算法涉及进程运行所需内存上限，该值在现实中是难以确定的，所以在 Linux 等系统中，基本不使用原始的银行家算法。更常见的策略是使用类似银行准备金的机制，始终保留小部分内存，以便内存不足时供高优先级程序运行。内存不足时，系统首先会回收资源（僵尸进程、内存泄漏等），而后会考虑将部分程序转移至虚拟内存
### 结构性预防
🎬[28]
- 共享互斥：SPOOL 技术（https://en.wikipedia.org/wiki/Spooling）
- 需要等待资源：事前确定所有资源请求（银行家算法）
- 无抢占：使用虚拟资源
- 需求成环：将资源有序化，限制需求（例如进程申请资源必须升序，实现方法可使用屏障）

## 活锁 Livelock
- 定义：活锁与死锁相似，而区别是在活锁中的进程在不断根据其它进程的结果改变自己的状态而不是往下走。
  - https://en.wikipedia.org/wiki/Deadlock#Livelock
- 与死锁区别在于，活锁现象中进程需求资源时不阻塞进程
- 例如：两进程以 `while(1)` 形式不断询问所需资源是否空闲，若空闲则退出循环并释放自身资源所占有的资源，而两进程一开始互相占有对方所需资源，则导致活锁。
- 活锁浪费 CPU 计算资源且同死锁同样导致进程无法进一步工作。解决活锁的方法与死锁类似。

## Starvation饥饿

- 在多个进程竞争锁资源的时候，如果锁的分配并不公平，那么就会导致某些进程一直拿不到锁，从而该进程一直在等待。则该进程就进入饥饿状态。而解决饥饿即改变锁的分配方式。因此诞生了公平锁与不公平锁，不公平锁能够提高锁的吞吐量，但也不可避免地会导致一些进程陷入饥饿。公平锁一种很直接的分配方式即按照进程申请锁的先后顺序分配锁。而在锁释放时，我们可以让进程进行一次抢锁，而这就是不公平锁的一种实现方案。

## 课后
- 自学多资源银行家算法
	- 研读课件幻灯片 🎬[24..25]