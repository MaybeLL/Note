# 磁盘管理

-[磁盘如何存储数据](ci-pan-guan-li.md#how-do-modern-hard-disk-drives-store-data)

-[访问一个扇区花费的时间](ci-pan-guan-li.md#访问一个扇区花费的时间)

-[一系列访问磁盘请求花费的时间](ci-pan-guan-li.md#一系列访问磁盘请求花费的时间)

-[SSTF](ci-pan-guan-li.md#sstf-shortest-seek-time-first)

-[Elevator](ci-pan-guan-li.md#elevator-aka-scan-or-c-scan)

-[LOOK 调度](ci-pan-guan-li.md#look-调度)

## 磁盘如何存储数据

磁盘的存储结构：扇区（512B大小）为读写单元，原子读写。 磁盘的机械构造：磁头，磁臂，盘面，磁道

> 注：扇区是物理角度的最小读写单元，区块block是文件系统中的概念，为了更高的效率，把连续的扇区合并为一个区块，从而逻辑上增大读写单元，提高了存储效率。
>
> ## 访问一个扇区花费的时间
>
> 寻道时间，旋转时间，读扇区时间

## 一系列访问磁盘请求花费的时间

### FCFS

FCFS:如果要求磁盘有序被响应的话，则访问顺序就是来到顺序，不存在调度问题。耗时。

如果不要求磁盘请求按来到的顺序进行响应，那么对于一系列的访问访问磁盘的请求有可能缩短花费的时间吗？

os负责通过合理的安排多个请求访问磁盘的顺序，可以使得整体花费的时间缩短。这相当于最优控制问题。

基本原理：By estimating the seek and possible rotational delay of a request, the disk scheduler can know how long each request will take, and thus \(greedily\) pick the one that will take the least time to service first.

### SSTF: Shortest Seek Time First

先响应当前最近磁道的请求。 饥饿问题：如果不断的有新的很近的磁道请求到来，那么远距离的磁道请求一直不会被响应。

> How can we implement SSTF-like scheduling but avoid starvation?

### Elevator \(a.k.a. SCAN or C-SCAN\)

算法特点：额外需要知道磁头移动方向 scan算法：像电梯一样。存在访问密度不均匀的问题，从两端改变磁头移动方向会访问刚访问不久的磁道，而不是比较久没访问的另一端的扇区，这是不合理的。 C-SCAN：循环扫描。解决scan访问不均匀问题。

问题：磁头不应该移动到两端才转向，应该移动到最偏远的请求后就转向。

### LOOK 调度

解决了电梯算法的多余移动问题。

## RAID策略

使用多个物理盘构成一个逻辑盘，这个逻辑盘的读写速度更快，更好的数据安全性，具有数据纠错校验能力。

通过把数据分成多个数据块（Block）并行写入/读出多个磁盘以提高访问磁盘的速度； 通过镜像或校验操作提供容错能力；

### RAID 0

![RAID 0](https://upload.wikimedia.org/wikipedia/commons/thumb/9/9b/RAID_0.svg/195px-RAID_0.svg.png) 加速原理：使用了流水线中分段的思想，提高了数据读写的并行程度（数据分成多个不同物理盘上的区块），同一时刻读写多个盘。 RAID 0的速度是最快的。但是RAID 0既没有冗余功能，也不具备容错能力，如果一个磁盘（物理）损坏，所有数据都会丢失，危险程度与\#JBOD相当。

### RAID 3

RAID 3：为了说明白RAID 5，先说RAID 3.RAID 3是若你有n块盘，其中1块盘作为校验盘，剩余n-1块盘相当于作RAID 0同时读写，当其中一块盘坏掉时，可以通过校验码还原出坏掉盘的原始数据。这个校验方式比较特别，奇偶检验，1 XOR 0 XOR 1=0，0 XOR 1 XOR 0=1，最后的数据时校验数据，当中间缺了一个数据时，可以通过其他盘的数据和校验数据推算出来。但是这有个问题，由于n-1块盘做了RAID 0，每一次读写都要牵动所有盘来为它服务，而且万一校验盘坏掉就完蛋了。最多允许坏一块盘。n最少为3.

### RAID 5

RAID 5：在RAID 3的基础上有所区别，同样是相当于是1块盘的大小作为校验盘，n-1块盘的大小作为数据盘，但校验码分布在各个磁盘中，不是单独的一块磁盘，也就是分布式校验盘，这样做好处多多。最多坏一块盘。n最少为3.

### RAID 1

![RAID 1](https://upload.wikimedia.org/wikipedia/commons/thumb/b/b7/RAID_1.svg/195px-RAID_1.svg.png) 极度追求数据安全的方案 RAID 1：正因为RAID 0太不可靠，所以衍生出了RAID 1。如果你有n块磁盘，把其中n/2块磁盘作为镜像磁盘，在往其中一块磁盘写入数据时，也同时往另一块写数据。坏了其中一块时，镜像磁盘自动顶上，可靠性最佳，但空间利用率太低。n最少为2。

