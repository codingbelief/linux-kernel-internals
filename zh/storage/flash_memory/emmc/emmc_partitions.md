# eMMC 分区管理


## eMMC 硬件分区

eMMC 标准中，将内部的 Flash Memory 划分为 4 类区域，如下图所示：

![](emmc_partitions.png)

eMMC 的各个分区有其自身的功能特性，多分区的设计，为不同的应用场景提供了便利。

### Boot Area Partitions



Boot 介绍
  为 SOC 的 ROM 代码降低复杂度，提供统一的 eMMC Boot 标准。
RPMB 介绍

GPP 介绍
  省去软件分区的解析，降低 Bootloader 的软件复杂度
  default
  enhance
  attributes
UDA 介绍
  default
  enhance
  软件分区表、MBR、GPT

eMMC 如何访问不同的分区，选定分区、分区内寻址

eMMC 应用场景中的分区过程
  在 产线生产过程中，从裸片状态下进行分区划分
  在 Bootloader 中，分区的处理
  在 Linux 中，各个分区的呈现方式


分区的概念
  硬件分区、软件分区、分区表

eMMC 上的硬件分区
  boot x2
  rpmb
  general purpose partition (enhanced and default)
  user data area (enhanced and default)
  分区寻址

分区应用示例
  
  

boot 和 rpmb 不会上文件系统，UDA 会再分区

加入分区示例图

[](http://tjworld.net/attachment/wiki/Android/HTC/Vision/EmmcPartitioning/partitions.png)

为什么使用 General Purpose Partition，有什么好处

enhanced area 是属于 user data area 内部的一个区块，

物理分区、软件分区


[](https://en.wikipedia.org/wiki/Disk_partitioning)
