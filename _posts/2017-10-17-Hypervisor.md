
# Linux Hypervisor
>在虚拟化中，经常提到的一个概念就是 hypervisor，即（系统）管理程序，又称虚拟机监控器（Virtual machine monitor, 缩写 VMM）

所谓虚拟化，就是通过某种方式隐藏了底层物理硬件，从而可以让多个操作系统可以透明使用和共享它，我们把这样的架构为平台虚拟化，hypervisor 提供了底层机器虚拟化的软件层，hypervisor 所处的层级叫平台虚拟化层；

***下图是硬件虚拟化的分层架构图***

![physical-virtualization-architecture](https://github.com/tibelf/image/raw/master/image/physical-virtualization-architecture.gif)

在层级架构中，我们可以发现所有的 Guest OS 都是跑在 hypervisor 之上，所以我们可以很容易把它想象成一个操作系统，上面跑的 OS 想象成一个个应用；

## hypervisor 分类
hypervisor 一共分有两类：

***下图是hypervisor类型***
![physical-virtualization-architecture](https://github.com/tibelf/image/raw/master/image/hypervisor-type.gif)


* 类型一：直接运行于本地的物理硬件之上
* 类型二：运行于操作系统之上

但其实两者的界限定义并不是非常明晰。类型一，由于直接是构建在物理硬件上，嵌入到操作系统的内核中，所以相对来说在性能上，可用性上，安全上比类型二会更好。

## hypervisor 构成
hypervisor 介于 Guest OS 和物理机之间，实现了一些重要功能，从而可以让 Guest OS 可以和宿主机系统同时正常运行；

***下图为hypervisor的几个主要要素***

![physical-virtualization-architecture](https://github.com/tibelf/image/raw/master/image/hypervisor-elements.gif)

主要包含了以下一些特性：

* ***Hypercall API:*** 允许 Guest OS 可以向宿主机系统发送请求，这很像在应用里面，需要调用内核的时候，应用需要通过调用 Library 的方法来实现；
* ***Input/Output:*** I/O虚拟化可以实现在内核中或者直接在 Guest OS 中
* ***Interrupts:*** 外部的中断信息需要 hypervisor 来处理，或者虚拟设备的中断需要通知到来宾系统知晓；hypervisor 需要经常处理一些来宾系统中的异常（这是必须的，因为来宾系统的错误仅会停止该系统，但是hypervisor和宿主机上的其他虚拟机还可以正常运行；）；
* ***Page mapper:*** 页映射，即硬件指向特定的系统中的页（来宾或者 hypervisor ）；
* ***Scheduler:*** 来宾系统和 hypervisor 之间的重要数据传输控制

## hypervisor 解决方案
hypervisor 解决方案有多种，比如：KVM，Xen，Hyper-V，QEMU 等等；KVM 是目前比较流行的一种；

# Tips

* [剖析 Linux hypervisor](https://www.ibm.com/developerworks/cn/linux/l-hypervisor/#resources)
* [hypervisor](https://en.wikipedia.org/wiki/Hypervisor)
* [Learn about hypervisors, system virtualization, and how it works in a cloud environment](https://www.ibm.com/developerworks/cloud/library/cl-hypervisorcompare/index.html)
* [Multicore Software Development Techniques: Applications, Tips, and Tricks](https://books.google.com.hk/books?id=PXaDBAAAQBAJ&pg=PA93&lpg=PA93&dq=hypervisor+hypercall+interrupt+pagemapper&source=bl&ots=cx8zAO4Co3&sig=nJ1q-x_nIpvfnIGoH9z44x2dA9Q&hl=zh-CN&sa=X&ved=0ahUKEwiT59L1pPfWAhVMppQKHYr4BpwQ6AEIJDAA#v=onepage&q=hypervisor%20hypercall%20interrupt%20pagemapper&f=false)
* [What's the difference between Type 1 and Type 2 hypervisors?](http://searchservervirtualization.techtarget.com/feature/Whats-the-difference-between-Type-1-and-Type-2-hypervisors)
