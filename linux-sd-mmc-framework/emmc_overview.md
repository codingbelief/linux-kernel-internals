# eMMC 简介

eMMC 是 embedded MultiMediaCard 的简称。MultiMediaCard，即 MMC， 是一种闪存卡（Flash Memory Card）标准，它定义了 MMC 的架构以及访问　Flash Memory 的接口和协议。而 eMMC 则是对 MMC 的一个拓展，以满足更高标准的性能、成本、体积、稳定、易用等的需求。

eMMC 的整体架构如图片 1 所示：


eMMC 内部主要可以分为 Flash Memory、Flash Controller 以及 CPU Interface 三大部分，后续的章节将会对这三个部分进行详细介绍。

## Flash Memory

Flash Memory 是一种非易失性的存储器，通常在嵌入式系统中用于存放系统、应用和数据等，类似与 PC 系统中的硬盘。
Flash Memory 在读取时，可以随机按字节进行读取，但是在写入时有一定的限制。Flash Memory 在进行写入操作时，只能将当前为 1 的比特改写为 0，而无法将已经为 0 的比特改写为 1，只有在按块擦除的操作中，才能把整块的比特改写为 1。

根据硬件实现上的不同，Flash Memory 可以分为 NOR Flash 和 NAND Flash 两类。

### NOR Flash

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