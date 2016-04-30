# eMMC 简介

eMMC 是 embedded MultiMediaCard 的简称。MultiMediaCard，即 MMC， 是一种闪存卡（Flash Memory Card）标准，它定义了 MMC 的架构以及访问　Flash Memory 的接口和协议。而 eMMC 则是对 MMC 的一个拓展，以满足更高标准的性能、成本、体积、稳定、易用等的需求。

eMMC 的整体架构如图片 1 所示：


eMMC 内部主要可以分为 Flash Memory、Flash Controller 以及 CPU Interface 三大部分，后续的章节将会对这三个部分进行详细介绍。

## Flash Memory

Flash Memory 是一种非易失性的存储器，通常在嵌入式系统中用于存放系统、应用和数据等，类似与 PC 系统中的硬盘。
Flash Memory 在读取时，可以随机按字节进行读取，但是在写入时有一定的限制。Flash Memory 在进行写入操作时，只能将当前为 1 的比特改写为 0，而无法将已经为 0 的比特改写为 1，只有在按块擦除的操作中，才能把整块的比特改写为 1。

根据硬件物理实现上的不同，Flash Memory 主要可以分为 NOR Flash 和 NAND Flash 两类，主要的差异如下所示：

* NOR Flash 的读取速度比 NAND Flash 稍块；
* NAND Flash 的读取速度比 NOR Flash 快很多；
* NAND Flash 的擦出速度比 NOR Flash 快很多；
* NAND Flash 的擦除单元更小，相应的擦除电路更加简单；
* NOR可以直接使用，并在上面直接运行代码，而NAND需要I/O接口，因此使用时需要驱动
* 
（注：擦除 NOR Flash 时是以 64～128KB 的块进行的，执行一个写入 / 擦除操作的时间为 5s，擦除 NAND Flash 是以 8～32KB 的块进行的，执行相同的操作最多只需要 4ms。）

### NOR Flash


NOR Flash 根据与 CPU 端接口的不同，可以分为 Parallel Nor Flash 和 Serial Nor Flash 两类。

内部架构
cpu 接口
特性

### NAND Flash
内部架构
cpu 接口
Nand Flash 特性
  读写方式
  读写速度
  ECC
  Bad Block Management
  Wear-Leveling

### Flash Controller

### CPU Interfaces



## eMMC 读写操作