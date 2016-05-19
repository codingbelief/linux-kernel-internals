# eMMC 简介

eMMC 是 embedded MultiMediaCard 的简称。MultiMediaCard，即 MMC， 是一种闪存卡（Flash Memory Card）标准，它定义了 MMC 的架构以及访问　Flash Memory 的接口和协议。而 eMMC 则是对 MMC 的一个拓展，以满足更高标准的性能、成本、体积、稳定、易用等的需求。

eMMC 的整体架构如图片 1 所示：

![](emmc_architecture.png)  
**图片 1： eMMC 整体架构**

eMMC 内部主要可以分为 Flash Memory、Flash Controller 以及 Host Interface 三大部分，后续的章节将会对这三个部分进行详细介绍。

## Flash Memory

Flash Memory 是一种非易失性的存储器，通常在嵌入式系统中用于存放系统、应用和数据等，类似与 PC 系统中的硬盘。  
目前，绝大部分手机和平板等移动设备中所使用的 eMMC 内部的 Flash Memory 都属于 NAND Flash。

### 1.3 eMMC 内部 Flash Memory 的分区 

eMMC 在内部对 Flash Memory 划分了几个主要区域，如下图所示：

![](emmc_partitions.png)  
**图片 4：eMMC 内部分区**

1. BOOT Area Partition 1 & 2  
  此分区主要是为了支持从 eMMC 启动系统而设计的。  
  该分区的数据，在 eMMC 上电后，可以通过很简单的协议就可以读取出来。同时，大部分的 SOC 都可以通过 GPIO 或者 FUSE 的配置，让 ROM 代码在上电后，将 eMMC BOOT 分区的内容加载到 SOC 内部的 SRAM 中执行。
  
2. RPMB Partition  
  RPMB 是 Replay Protected Memory Block 的简称，它通过 RSA 和 Write Counter 来保证保存在 RPMB 内部的数据不被非法篡改。  
  在实际应用中，RPMB 分区通常用来保存安全相关的数据，例如指纹数据、安全支付相关的密钥等。

3. User Data Area Partition  
  此分区则主要用于存储系统和用户数据。
  
注：除了上述的分区，eMMC 还会有一些特殊的分区，具体的细节可以参考 eMMC 5.1 协议标准[12]。

## 2. Flash Controller

NAND Flash 直接接入 Host 时，Host 端通常需要有 NAND Flash Translation Layer，即 NFTL 或者 NAND Flash 文件系统来做坏块管理、ECC等的功能。  
eMMC 则在其内部集成了 Flash Controller，用于完成擦写均衡、坏块管理、ECC校验等功能。相比于直接将 NAND Flash 接入到 Host 端，eMMC 屏蔽了 NAND Flash 的物理特性，可以减少 Host 端软件的复杂度，让 Host 端专注于上层业务，省去对 NAND Flash 进行特殊的处理。

![](emmc_vs_nand_flash.png)  
**图片 5：NAND Flash 与 eMMC**

在某些 eMMC 中，Flash Controller 还会完成一些特殊的工作。例如，一个 16GB 的 TLC eMMC，在出厂后， 先让内部的 NAND Flash 处于 SLC 状态，这样可以在初期获得较好的性能。当达到一定条件后（eg. 使用了总存储容量的 1/3 时），Flash Controller 将 NAND Flash 切换到 TLC 状态，以确保能够达到标称的数据存储容量。这种 NAND Flash 状态的切换，由 Flash Controller 进行，对于 Host 来说，是完全透明的。

## 3. Host Interface

eMMC 与 Host 之间的连接如下图所示：

![](emmc_host_interfaces.png)  
**图片 6：eMMC Interface**

各个信号的用途如下所示：

**CLK**  
用于同步的时钟信号

**Data Strobe**  
此信号是从 Device 端输出的时钟信号，频率和 CLK 信号相同，用于同步从 Device 端输出的数据。该信号在 eMMC 5.0 中引入。

**CMD**  
此信号用于发送 Host 的 command 和 Device 的 response。

**DAT0-7**  
用于传输数据的 8 bit 总线。

Host 与 eMMC 之间的通信都是 Host 以一个 Command 开始发起的。针对不同的 Command，Device 会做出不同的响应。详细的通信协议相关内容，请参考 [eMMC 总线协议](./emmc_bus_protocol.html) 章节。


## 4. 参考资料

1. [NOR NAND Flash Guide: Selecting a Flash Storage Solution](https://www.micron.com/~/media/documents/products/product-flyer/flyer_nor_nand_flash_guide.pdf) [PDF]  
2. [Wiki: Common Flash Memory Interface](https://en.wikipedia.org/wiki/Common_Flash_Memory_Interface) [Web]  
3. [Quick Guide to Common Flash Interface](https://www.spansion.com/Support/Application%20Notes/Quick_Guide_to_CFI_AN.pdf) [PDF]  
4. [MICRON NOR Flash Technology](https://www.micron.com/products/nor-flash) [Web]  
5. [MICRON NAND Flash Technology](https://www.micron.com/products/nand-flash) [Web]  
6. [Wiki：闪存](https://zh.wikipedia.org/wiki/%E9%97%AA%E5%AD%98) [Web]  
7. [Wiki：Flash File System](https://en.wikipedia.org/wiki/Flash_file_system) [Web]  
8. [Wear Leveling in Micron® NAND Flash Memory](https://www.micron.com/~/media/documents/products/technical-note/nand-flash/tn2961_wear_leveling_in_nand.pdf) [PDF]  
9. [Understanding Flash: The Flash Translation Layer](https://flashdba.com/2014/09/17/understanding-flash-the-flash-translation-layer/) [Web]
10. [谈NAND Flash的底层结构和解析](http://blog.sina.com.cn/s/blog_4b4b54da01016rx3.html) [Web]  
11. [闪存基础](http://www.ssdfans.com/?p=45) [Web]  
12. [Embedded Multi-Media Card (e•MMC) Electrical Standard (5.1)](http://www.jedec.org/sites/default/files/docs/JESD84-B51.pdf)  [PDF]  
