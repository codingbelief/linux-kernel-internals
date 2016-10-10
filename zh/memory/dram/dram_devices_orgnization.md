# DRAM Devices Orgnization

随着系统对内存容量、带宽等方面的需求提高，系统会接入多个 DRAM Devices。而多个 DRAM Devices 不同的组织方式，会带来不同的效果。本小节将对不同的组织方式及其效果进行简单介绍。

## Single Channel DRAM Controller 组织方式

Single Channel 指 DRAM Controller 只有一组控制和数据总线。
在这种场景下，DRAM Controller 与单个或者多个 DRAM Devices 的连接方式如下所示：

### 连接单个 DRAM Device

## 共用数据总线

## 共用控制和地址总线

为何要由多个 Devices 组成一个 Rank？

为何要创造 Channel 的概念？

Device 内部的 Channel 概念？

Host 端的 DRAM Controller 可以有多组数据总线、地址总线和控制总线

1 个 CS 对应一个 Rank！  
Channel 如何区别？ 
每个 Channel 上的控制信号是一样的？ 
数据总线拼接到一起？   

如果是共用数据总线的，那么必然需要在控制信号上加以区分，以保证不会同时访问总线。  

理论上，channel 是拓展总线的  
如果是数据总线是拼接的，那么可以共用控制信号  

DIMM(Dual In-line Memory Module)


SIMM (Single In-line Memory Module)

rank指的是连接到同一个cs(Chip Select，片选)的所有内存颗粒chips，内存控制器能够对同一个rank的所有chips同时进行读写操作，而在同一个rank的chip也分享同样的控制信号。以目前的电脑来说，因为一组channel的位宽是64bit，所以能够同时读写8byte的资料，如果是具有ECC功能的内存控制器和ECC内存模组，那么一组channel的位宽就是72bit

Modern CPU microarchitectures support triple or quadruple memory channels. These multiple independent channels increases data transfer rates due to concurrent access of multiple DIMMs.

Dual Channel memory controllers are memory controllers where the DRAM devices are separated on to two different buses to allow two memory controllers to access them in parallel. This doubles the theoretical amount of bandwidth of the bus. In theory, more channels can be built (a channel for every DRAM cell would be the ideal solution), but due to wire count, line capacitance, and the need for parallel access lines to have identical lengths, more channels are very difficult to add.


1. host 只存在一个 channel 的场景  
1.1 单设备  
1.2 多设备  
1.2.1 是否共享数据总线  
2. host 存在多个 channel 的场景  
2.1 多个单 channel 设备  
2.2 单个多 channel 设备  