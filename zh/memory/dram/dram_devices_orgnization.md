# DRAM Devices Orgnization

随着系统对内存容量、带宽、性能等方面的需求提高，系统会接入多个 DRAM Devices。而多个 DRAM Devices 不同的组织方式，会带来不同的效果。本小节将对不同的组织方式及其效果进行简单介绍。

## Single Channel DRAM Controller 组织方式

Single Channel 指 DRAM Controller 只有一组控制和数据总线。
在这种场景下，DRAM Controller 与单个或者多个 DRAM Devices 的连接方式如下所示：

### 连接单个 DRAM Device

![](./single_channel_single_device.png)

Single Channel 连接单个 DRAM Device 是最常见的一种组织方式。
由于成本、工艺等方面的因素，单个 DRAM Device 在总线宽度、容量上有所限制，在需要大带宽、大容量的产品中，通常接入多个 DRAM Devices。

### 连接多个 DRAM Devices

![](./single_channel_multi_devices_1.png)

上图中，多个 DRAM Devices 共享控制和数据总线，DRAM Controller 通过 Chip Select 分时单独访问各个 DRAM Devices。此外，在其中一个 Device 进入刷新周期时，DRAM Controller 可以按照一定的调度算法，优先执行其他 Device 访问请求，提高系统整体内存访问性能。

这种组织方式只增加总体容量，不增加带宽。

![](./single_channel_multi_devices_2.png)

上图中，多个 DRAM Devices 共享控制总线和 Chip Select 信号，DRAM Controller 同时访问每个 DRAM Devices，各个 Devices 的数据拼接到一起。

这种组织方式既增加总体容量，也增加带宽。

## Multi Channel DRAM Controller 组织方式

Multi Channel 指 DRAM Controller 只有多组控制和数据总线，每一组总线可以独立访问 DRAM Devices。
在这种场景下，DRAM Controller 与 DRAM Devices 的连接方式如下所示：

### 连接 Single Channel DRAM Device

![](./multi_channel_controller_single_channel_devices.png)

这种组织方式的优势在于多个 Devices 可以同时工作，DRAM Controller 可以对不同 Channel 上的 Devices 同时发起读写请求，提高了读写请求的吞吐率。

### 连接 Multi Channel DRAM Device

![](./multi_channel_controller_Multi_channel_devices.png)

在一些 DRAM 产品中，例如 LPDDR3、LPDDR4 等，引入了 Multi Channel 的设计，即一个 DRAM Devices 中包括多个 Channel。这样就可以在单个 Device 上达成 Multi Channel 同时访问的效果，最终带来读写请求吞吐率的提升。
