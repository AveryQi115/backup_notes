# 虚拟内存

- 虚拟内存的概念就是任意一个字节有一个虚拟地址和一个物理地址；虚拟内存分割为虚拟页，物理内存分割为物理页。虚拟页有以下三个不相交子集组成
  - 未分配的页
  - 没有缓存在物理地址中的已分配页
  - 缓存在物理地址中的已分配页
- 虚拟地址->MMU->物理地址->DRAM缓存->磁盘（DRAM缓存就是操作系统写的那个buffer）
  - DRAM缓存不命中的开销更大，需要读磁盘，写也是用写回而不是直写
- 页命中
  - 程序维护一个页表(PTE数组)，PTE由有效位(是否被缓存)和物理页号或磁盘地址组成
  - 按虚拟地址查询页表，若当前页已缓存，则查到物理页号(物理内存地址)
- 缺页(**page fault**)(DRAM缓存不命中)
  - 按虚拟地址查询页表，当前页未缓存，触发缺页异常
  - 内核响应缺页异常，根据DRAM缓存规则选择牺牲页
  - 若牺牲页有修改，牺牲页复制回磁盘
  - 修改页表中涉及牺牲页的条目
  - 异常处理程序返回， 重新启动引发缺页的指令，命中缓存返回



# 内存映射

`void *mmap(void *start,size_t length,int prot,int flags,int fd,off_t offsize); `

Prot:

- `PROT_EXEC` 映射区域可被执行
- `PROT_READ` 映射区域可被读取
- `PROT_WRITE` 映射区域可被写入
- `PROT_NONE` 映射区域不能存取

Flags:

- `MAP_FIXED` 如果参数start所指的地址无法成功建立映射时，则放弃映射，不对地址做修正。通常不鼓励用此旗标。
- `MAP_SHARED` 对映射区域的写入数据会复制回文件内，而且允许其他映射该文件的进程共享。
- `MAP_PRIVATE` 对映射区域的写入操作会产生一个映射文件的复制，即私人的“写入时复制”（copy on write）对此区域作的任何修改都不会写回原来的文件内容。
- `MAP_ANONYMOUS` 建立匿名映射。此时会忽略参数fd，不涉及文件，而且映射区域无法和其他进程共享。
- `MAP_DENYWRITE` 只允许对映射区域的写入操作，其他对文件直接写入的操作将会被拒绝。
- `MAP_LOCKED` 将映射区域锁定住，这表示该区域不会被置换（swap）