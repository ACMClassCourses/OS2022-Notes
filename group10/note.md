## 5.12 note

Author：翟明舒、王照梓

### 虚拟机

#### hypervisor

hypervisor是一个软件，管理各种虚拟机和control domain。

#### Host OS

另一种虚拟机的架构，可以在Host OS 上运行hypervisor。在Host OS中同时运行了Host OS process。

与之相对的是Guest OS和Guest OS process。

#### Requirements for Virtualization

Safety Fidelity Efficiency

#### Hardware Support For Nested Page Tables

Page Table : VT -> PT，不同虚拟机的进程的页表可能有冲突。

两台虚拟机的物理地址是真实物理地址空间吗？如果是我们需要两台虚拟机物理地址交集为空。

如何保证物理地址不冲突，一台虚拟机无法意识到另一台虚拟机的存在。

一种解决方案是Guest OS放弃内存管理，交给hypervisor做内存管理，使用hyper call。

另一种方案是使用硬件支持（NPT）。



每个虚拟机都拥有一段物理内存（无论真假）。

hypervisor建立另外一张虚拟机物理内存到真实物理内存的映射表（Shadow Page Table）与Guest OS Page Table相对应。

如何保证同步修改Guest和Shadow？修改虚拟机的页表是特权指令，需要中断，hypervisor 截取指令，先修改Shadow Page Table。需要保证页表所在内存空间的只读特性。



如果物理内存满了，可以Guest OS使用自己的虚存。

Vmware的设计 每个虚拟机中放一个balloon，hypervisor让每个Guest OS申请一个页，如果Guest OS申请了虚存，那么放弃。hypervisor将收集到的物理内存分给了新的虚拟机，当物理内存重新充足时，balloon机制产生的页会回收。

### Multiple Processor System

一个芯片上 multiple core

一个版上 multiple cpu

对称多处理器 symmetric multiple processor

集群 cluster

#### 三种分类

共享内存多处理器 shared-memory multiprocessor

消息传递式多处理器 message-passing multicomputer

广域分布式系统 wide area distributed system

#### 基于bus的多处理器系统(一种shared memory multiprocessor)

(a) Without caching

(b) With caching

(c) With caching and private memory

###### 

