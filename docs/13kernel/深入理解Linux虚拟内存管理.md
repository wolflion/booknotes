## 《深入理解Linux虚拟内存管理》

+ 200页是理论的，后面的附录，是代码方面
+ 目录 在（10/674）
### chap2、描述物理内存   （28/674）
#### 0、
+ 每个簇都被认为是一个节点，在linux中的`struct pg_data_t`体现了这一概念
+ 在内存中，每个节点被分成很多的称为**管理区（zone）** 的块，用于表示内存中的某个范围。一个管理区由一个`struct zone_struct`描述，并被**定义为`zone_t`**
+ 管理区分为几段:
	+ ZONE_DMA，内存的首部16MB
	+ ZONE_NORAML，16~896MB
	+ ZONE_HIGHMEM，896~末尾
+ 系统的内存划分成大小确定的许多块，这些块也称为**页面帧**。每个物理页面帧由一个`struct page`描述，所有的结构都存储在一个**全局mem_map数组中**，该数组通常存放在`ZONE_NORMAL`的首部。
+ 图2.1，**节点、管理区和页面的关系**

#### 2.1、节点
+ linux/mmzone.h中`typedef struct pglist_data{}pg_data_t;`
+ `pgdat_list`链表维护，由函数`init_bootmem_core()`初始化节点
#### 2.2、管理区
+ linux/mmzone.h中的`struct zone_t{};`描述，`zone_structs`用于**跟踪诸如页面使用情况统计数，空闲区域信息和锁信息**。
##### 2.2.1、管理区极值
+ **当系统中的可用内存很少时，守护程序kswapd被唤醒开始释放页面**。
+ 每个管理区有3个极值:
	+ pages_low
	+ pages_min
	+ pages_high
##### 2.2.2、计算管理区大小
+ 用`setup_memory()`
+ mm/bootmem.c
##### 2.2.3、管理区等待队列表
#### 2.3、管理区的初始化
+ `paging_init()`
#### 2.4、初始化mem_map
+ 在NUMA系统中，全局mem_map被处理为一个起始于`PAGE_OFFSET`的虚拟数组。`free_area_init_node()`
+ 在UMA系统中，`free_are_init()`使用config_page_data作为结点，并将全局mem_map作为该节点的局部mem_map
#### 2.5、页面
+ linux/mm.h中的`struct page{};`，类型`mem_map_t`
+ **表2.1 描述页面状态的标志位**
#### 2.6、页面映射到管理区
+ linux/page_alloc.c中的`zone_table`
#### 2.7、高端内存
+ 内核通过`kmap()`将高端内存的页面临时映射成`ZONE_NORMAL`
#### 2.8、2.6中有哪些新特性
+ 节点
+ 管理区
+ 页面
+ Per-CPU上的页面链表

### chap3、页表管理  （45/674）
#### 0、
+ **三层页表机制来完成内存管理**
#### 3.1、描述页目录
+ **每个进程都有一个指向其自己PGD的指针（`mm_struct->pgd`），它其实就是一个物理页面帧**。
+ asm/page.h中的`pgd_t`类型
+ `PMD_SHIFT`表示在线性地址中第二级页表所映射的位。
#### 3.2、描述页表项
+ **三层页表中的每一个项PTE，PMD，PGD分别由pte_t，pmd_t，pgd_t描述**
#### 3.3、页表项的使用
+ asm/pgtable.h中
	+ `pgd_offset()`
+ mm/memory.c中`follow_page()`
#### 3.4、页表项的转换和设置
+ `mk_pte()`
+ `pte_page()`
+ `set_pte()`
#### 3.5、页表的分配和释放
#### 3.6、内核页表
##### 3.6.1、引导初始化
+ arch/i386/kernel/head.S中的`startup_32()`
##### 3.6.2、收尾工作
+ `paging_init()`
#### 3.7、地址和struct page之间的映射
##### 3.7.1、物理和虚拟内核地址之间的映射
+ asm-i386/page.h中
##### 3.7.2、struct page和物理地址间的映射
+ asm-i386/pae.h中
#### 3.8、转换后援缓冲区（TLB）
+ asm/pgtable.h中
#### 3.9、一级CPU高速缓存管理
+ asm/pgtable.h中
#### 3.10、2.6中有哪些新特性
+ MMU-less体系结构的支持
+ 反向映射
+ 基于对象的反向映射
+ 高端内存中的PTE
+ 大型TLB文件系统
	+ linux/hugetlb.h中
+ 高速缓存刷新管理
	+ `flush_page_to_ram()`

### chap4、进程地址空间  （65/674）

#### 4.1、线性地址空间
+ 地址空间分为两个部分：
	+ 一个是随上下文切换而改变的用户空间部分
	+ 一个是保持不变的内核空间部分  **两者的分界点由PAGE_OFFSET决定，在x86中它的值是`0xC0000000`**
#### 4.2、地址空间的管理
+ 进程可使用的地址空间由`mm_struct`管理
#### 4.3、进程地址空间描述符
##### 0、
+ linux/sched.h中的`mm_struct`
##### 4.3.1、分配一个描述符
+ `Allocate_mm()`只是一个预处理宏，它从slab allocator中分配一个mm_struct
+ `mm_alloc()`从slab中分配，然后调用`mm_init()`对其初始化
##### 4.3.2、初始化一个描述符
+ `mm_struct`通过`init_mm()`初始化
##### 4.3.3、销毁一个描述符
+ **如果mm_users变成0，所有的映射区域通过`exit_mmap()`释放，同时释放页表**。
#### 4.4、内存区域
+ linux/mm.h中的`struct vm_area_struct{};`
##### 4.4.1、内存区域的操作
#### 4.5、异常处理
#### 4.6、缺页中断
#### 4.7、复制到用户空间/从用户空间复制
#### 4.8、2.6中有哪些新特性