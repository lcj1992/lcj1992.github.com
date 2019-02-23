---
layout: post
title: 内存管理
date: 2015-12-31
categories: os
tags: os mempry
---

主存管理必须实现以下功能：

1.  映射逻辑地址到物理地址

2.  在多用户之间分配物理内存

3.  对个用户的信息提供保护措施

4.  扩充逻辑内存

#### 虚拟存储器
虚拟存储器将用户的逻辑主存和物理主存分开。虚拟地址空间可以比实际主存空间大，也可以比实际主存小。在多道程序运行环境下，操作系统把实际主存扩充成若干个虚拟内存，系统可以为每个应用程序建立一个虚存。

#### 地址映射的方式
1.编程或编译时确定地址映射关系

2.静态地址映射 在作业装入过程中进行地址映射

3.动态地址映射 在程序执行期间进行地址映射

#### 分区的分配
分区分配和回收算法使用的数据结构是空闲区队列，几种放置策略有首次适应算法、最佳适应算法、最坏适应算法。

首次适应算法 first fit：优先使用低地址空闲区
![first fit](/images/operating_system/firstFit.png)

最佳适应算法 best fit：优先使用与所需大小最接近的空闲区
![best fit](/images/operating_system/bestFit.png)


最坏适应算法 worst fit：优先使用最大的空闲区
![worst fit](/images/operating_system/worstFit.png)


#### 页式存储管理
`页表`：记录页与块之间对应关系的地址变换的机构。

`虚地址结构`：虚地址是用户程序中的逻辑地址，它包括页号和页内地址（页内位移）。

区分页号和页内地址的依椐是页的大小，页内地址占虚地址的低位部分，页号占虚地址的高位部分。 设虚地址长度为16位，页面大小为1KB： 页号 页内地址（位移量）

![分页映像存储](/images/operating_system/page.png)

![页式地址变换](/images/operating_system/pagetransform.jpg)

#### 两种页式系统

1.  简单页式系统 装入一个作业的全部页面才能投入运行

2.  请求页式系统  装入一个作业的部分页面就可投入运行

扩展页表功能：

`页号 + 主存块号 + 中断位 + 辅存地址`

中断位--标识该页是否在主存，可以通过中断位来发现锁访问的页面在不在内存。

i=1，表示此页不在主存

i=0，表示该页在主存

辅存地址---该页面在辅存的位置。

发生缺页中断，如果主存中存在空白块，则直接调入；如果主存中不存在空白块，则需要淘汰该作业在主存中的一页，也就有了页面置换算法

#### 页面置换算法
如何决定淘汰哪一页？根据页面在系统中的表现，如使用的频繁程度，进入系统的时间长短等。

为此还需要引入

`改变位`：0 表示最近没有进程访问，1 表示最近有进程访问

`引用位`：0 表示调入内存后没有修改，1 该页调入内存后修改过。

`页号 + 主存块号 + 中断位 + 辅存地址 + 改变位 + 引用位`

`颠簸（抖动）`：简单地说，导致系统效率急剧下降的主存和辅存之间的频繁页面置换现像称为“抖动”。

`先进先出淘汰算法（FIFO算法）`：总是选择在主存中驻留时间最长的一页淘汰。

`最久未使用淘汰算法（LRU算法）`：总是选择最长时间未被使用的一页淘汰

#### 段页式存储管理
在段式存储管理中结合页存储管理技术，即在程序地址空间内分段，在一个分段内划分页面，这就形成了段页式存储管理。这样地址结构就由段号、段内页号和页面位移。

段页式地址变换中要得到物理地址须经过三次主存访问（若段表、页表都在主存）：

1.  访问段表，得到页表起始地址

2.  访问页表，得到主存块号

3.  将主存块号与页内位移组合，得到物理地址


#### 实模式和保护模式
78年Intel推出x86架构，最早的8086架构是一款16位的处理器，数据总线是16位的，而地址总线是20位的，最多可以寻址1MB的地址空间。

从80286开始CPU演变出两种工作模式：实模式和保护模式，80386则是Intel推出的第一款32位处理器，数据总线和地址总线都是32位，可以寻址4G。

现在大部分的操作系统在启动时出于实模式（为了兼容历史），真正运行时处于保护模式。保护模式设计用来增强多功能和系统稳定度，比如内存保护、分页、系统以及硬件支持的虚拟内存。
![寄存器](/images/operating_system/register.png)

#### 实模式下段的管理
实模式采用16位寻址方式，在该模式下，最大寻址空间为1MB，最大分段为64KB。由于处理器的设计需要考虑到向下兼容的问题，实模式页式我们今天接触到的大多数计算机在启动后出于的寻址模式。

              实际物理地址  =  （段寄存器 << 4） + 偏移地址

例如下的一句汇编代码： MOV AX ,ES:[1200H]

#### 保护模式下的段管理
这也是当今大部分x86结构的操作系统运行时所处的模式。

1.  `段描述符`：包含有段物理首地址、段界限、段属性

2.  `全局描述符表`Global Descriptor Table：在内存中存在一个数据维护一组段描述符

3.  `段选择子`：存储的是段描述符在全局描述符表的下标，索引。

4.  `全局描述符表寄存器`GDTR：存储全局描述符表在内存中存放的位置

那么保护模式下，段寻址过程就是，处理器通过GDTR找到GDT，利用段选择子到GDT中找到需要的段描述符，而这个段描述符中就存放着真正的段的物理首地址，然后再加上偏移地址量得到最后的物理地址。

#### 保护模式的内存分页

1.  逻辑地址：程序在编译连接后，变量名字等的符号地址

2.  线性地址：经过x86保护模式下的段地址变换后的地址，即线性地址=逻辑地址 + 段首地址

3.  物理地址：内存存储单元的编址，如1GB的内存，它的物理编址是从0x00000000到0x40000000

x86的`段式地址变换`就是逻辑地址到线性地址的变换过程.

***如果x86系统未开始页式内存管理，得到的线性地址实际上也就是物理地址,但如果页式地址被开启后，得到的线性地址需要在此经过`页式地址转换`才能得到物理地址.***