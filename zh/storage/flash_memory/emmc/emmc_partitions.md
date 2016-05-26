# eMMC 分区管理


## Partitions Overview

eMMC 标准中，将内部的 Flash Memory 划分为 4 类区域，最多可以支持 8 个硬件分区，如下图所示：

![](emmc_partitions.png)

一般情况下，Boot Area Partitions 和 RPMB Partition 的容量大小通常都为 4MB，部分芯片厂家也会提供配置的机会。General Purpose Partitions (GPP) 则在出厂时默认不被支持，即不存在这些分区，需要用户主动使能，并配置其所要使用的 GPP 的容量大小，GPP 的数量可以为 1 - 4 个，各个 GPP 的容量大小可以不一样。User Data Area (UDA) 的容量大小则为总容量大小减去其他分区所占用的容量。更多各个分区的细节将在后续章节中描述。

eMMC 的每一个硬件分区的存储都是独立编址的，即访问地址为 0 - partition size。具体的数据读写操作实际访问哪一个硬件分区，是由 eMMC 的 Extended CSD register 的 PARTITION_CONFIG Field 中 的 Bit[2:0]: PARTITION_ACCESS 决定的，用户可以通过配置 PARTITION_ACCESS 来切换硬件分区的访问。更多数据读写相关的细节，请参考 [eMMC 总线协议](./emmc_bus_protocol.html) 章节。

eMMC 的各个硬件分区有其自身的功能特性，多分区的设计，为不同的应用场景提供了便利。

## Boot Area Partitions

Boot Area 包含两个 Boot Area Partitions，主要用于存储 Bootloader，支持 SOC 从 eMMC 启动系统。

两个 Boot Area Partitions 的大小是完全一致的，由 Extended CSD register 的 PARTITION_CONFIG Field 中 的

boot area size 的确定
从哪一个 partition boot 的配置
触发 boot 的方法
写保护



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
