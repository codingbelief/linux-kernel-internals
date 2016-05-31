# eMMC 分区管理


## Partitions Overview

eMMC 标准中，将内部的 Flash Memory 划分为 4 类区域，最多可以支持 8 个硬件分区，如下图所示：

![](emmc_partitions.png)

一般情况下，Boot Area Partitions 和 RPMB Partition 的容量大小通常都为 4MB，部分芯片厂家也会提供配置的机会。General Purpose Partitions (GPP) 则在出厂时默认不被支持，即不存在这些分区，需要用户主动使能，并配置其所要使用的 GPP 的容量大小，GPP 的数量可以为 1 - 4 个，各个 GPP 的容量大小可以不一样。User Data Area (UDA) 的容量大小则为总容量大小减去其他分区所占用的容量。更多各个分区的细节将在后续章节中描述。

eMMC 的每一个硬件分区的存储空间都是独立编址的，即访问地址为 0 - partition size。具体的数据读写操作实际访问哪一个硬件分区，是由 eMMC 的 Extended CSD register 的 PARTITION_CONFIG Field 中 的 Bit[2:0]: PARTITION_ACCESS 决定的，用户可以通过配置 PARTITION_ACCESS 来切换硬件分区的访问。更多数据读写相关的细节，请参考 [eMMC 总线协议](./emmc_bus_protocol.html) 章节。

eMMC 的各个硬件分区有其自身的功能特性，多分区的设计，为不同的应用场景提供了便利。

## Boot Area Partitions

Boot Area 包含两个 Boot Area Partitions，主要用于存储 Bootloader，支持 SOC 从 eMMC 启动系统。

### 容量大小
两个 Boot Area Partitions 的大小是完全一致的，由 Extended CSD register 的 BOOT_SIZE_MULT Field 决定，大小的计算公式如下：  
  **Size = 128Kbytes x BOOT_SIZE_MULT**  
一般情况下，Boot Area Partition 的大小都为 4 MB，即 BOOT_SIZE_MULT 为 32，部分芯片厂家会提供改写 BOOT_SIZE_MULT 的功能来改变 Boot Area Partition 的容量大小。BOOT_SIZE_MULT 最大可以为 255，即 Boot Area Partition 的最大容量大小可以为 255 x 128 KB = 32640 KB = 31.875 MB。

### 从 Boot Area 启动
eMMC 中定义了 Boot State，在 Power-up、HW reset 或者 SW reset 后，如果满足一定的条件，eMMC 就会进入该 State。进入 Boot State 的条件如下：

1. CMD 信号保持低电平不少于 74 个时钟周期时，会触发 Original boot operation，进入 Boot State。
2. 在 74 个时钟周期后，在 CMD 信号首次拉低或者 Host 发送 CMD1 之前，Host 发送参数为 0xFFFFFFFA 的 COM0时，会触发 Alternative boot operation，进入 Boot State。

在 Boot State 下，eMMC 会将大小为 128Kbytes x BOOT_SIZE_MULT 的 Boot Data 发送给 Host。  
Boot Data 根据 Extended CSD register 的 PARTITION_CONFIG Field 的 Bit[5:3]:BOOT_PARTITION_ENABLE 的设定，可以从 Boot Area Partition 1、Boot Area Partition 2 或者 User Data Area 
读出。  
Boot Data 存储在 Boot Area 比在 User Data Area 中要更加的安全，可以减少意外修改导致系统无法启动，同时无法更新系统的情况出现。  
（更多 Boot State 的细节，请参考 [eMMC 总线协议](./emmc_bus_prolocol) 的 Boot State 章节）

### 写保护
通过设定 Extended CSD register 的 BOOT_WP Field，可以为两个 Boot Area Partition 独立配置写保护功能,以防止数据被意外改写或者擦出。

eMMC 中定义了两种 Boot Area 的写保护模式：

1. Power-on write protection，使能后，如果 eMMC 掉电，写保护功能失效，需要每次 Power on 后进行配置
2. Permanent write protection，使能后，即使掉电也不会失效，主动进行关闭才会失效

## RPMB Partition
RPMB（Replay Protected Memory Block）Partition 是 eMMC 中的一个具有安全特性的分区。  
eMMC 在写入数据到 RPMB 时，会校验数据的合法性，只有指定的 Host 才能够写入，同时在读数据时，也提供了签名机制，保证 Host 读取到的数据是 RPMB 内部数据，而不是攻击者伪造的数据。

RPMB 在实际应用中，通常用于存储一些有防止非法篡改需求的数据，例如手机上指纹支付相关的公钥、序列号等。RPMB 可以对写入操作进行鉴权，但是读取并不需要鉴权，任何人都可以进行读取的操作，因此存储到 RPMB 的数据通常会进行加密后再存储。

### 容量大小
两个 RPMB Partition 的大小是由 Extended CSD register 的 BOOT_SIZE_MULT Field 决定，大小的计算公式如下：  
  **Size = 128Kbytes x BOOT_SIZE_MULT**  
一般情况下，Boot Area Partition 的大小都为 4 MB，即 RPMB_SIZE_MULT 为 32，部分芯片厂家会提供改写 RPMB_SIZE_MULT 的功能来改变 Boot Area Partition 的容量大小。RPMB_SIZE_MULT 最大可以为 255，即 Boot Area Partition 的最大容量大小可以为 255 x 128 KB = 32640 KB = 31.875 MB。

### 原理

### 数据读写


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
